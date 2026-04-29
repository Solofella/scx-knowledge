# SCX_SIA_HOW_v4.0

**Agent Name:** SIA (Signal Intelligence Aggregator)  
**Version:** 4.0  
**Last Updated:** Chat #77 · April 28, 2026  
**Model:** None (Pure JavaScript)  
**Status:** Verified operational - 17 nodes complete

---

## Purpose

SIA is a zero-cost aggregation agent that produces time-windowed cluster intelligence for MRA (Metrics & Reporting Agent) consumption. It reads ALA records (filtered by guest review creation date), matches them to EIP records, groups by Domain + Signal Tier + Client ID, computes percentages and trend directions, and writes cluster records tagged with Report Window Type for MRA filtering.

**SIA makes ZERO AI calls.** All computation is deterministic JavaScript (`for` loops, date parsing, percentage calculations, composite key matching).

**Critical architectural shift (v4.0):** SIA now filters by **guest review creation date** (ALA `Date` field), NOT by pipeline processing timestamp. This ensures weekly/monthly reports reflect actual guest review activity for that period, not when SubtextCX processed the reviews.

---

## Three-Schedule Architecture (New in v4.0)

**SIA runs on THREE independent schedules:**

| Schedule | Trigger | Window | Purpose |
|----------|---------|--------|---------|
| **Daily** | Every day 7am UTC | Last 2 days | MRA daily reports (48-hour guest signal snapshot) |
| **Weekly** | Monday 6am UTC | Monday 00:00 → Sunday 23:59 (calendar week) | MRA weekly briefs (7-day pattern intelligence) |
| **Monthly** | 1st of month 6am UTC | Last 30 days (rolling) | MRA monthly reports (30-day trend analysis) |

**Each schedule writes cluster records tagged with `Report Window Type` (daily/weekly/monthly)**, enabling MRA to query the correct time window for each report type.

---

## Input Source Architecture (v4.0 Pattern)

**SIA does NOT receive webhooks.** It queries NocoDB tables directly on schedule.

**Data flow:**
1. **ALA table** → provides `Date` field (guest review creation date in M/D/YY format) + `Client ID`
2. **EIP table** → provides enriched signal data (Domain, Signal Type, Pain Points, Emotions)
3. **SIA clusters** → written to SIA NocoDB table with `Client ID` + `Report Window Type`

**Why this architecture (mirrors MRA pattern):**
- `Date` field exists ONLY in ALA table (never passed through EIP/ESS/HSI pipeline)
- Filtering by EIP Timestamp would report pipeline processing time, not guest review activity
- Solution: Fetch ALA → filter by Date → fetch EIP → match by ALA Record ID

**Reads from ALA table:**
- **Id** (ALA Record ID)
- **Date** (M/D/YY format: "4/23/26")
- **Client ID**
- **EIP Status** (Complete)

**Reads from EIP table:**
- **ALA Record ID** (FK to ALA)
- **Domain** (Pain Point Domain)
- **Signal Type** (maps to T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE)
- **Enriched Pain Point**
- **Enriched Emotion Tag**
- **Client ID** (passed through from ALA)
- **ESS Status** (Complete)

**Reads from SIA table (prior clusters):**
- **Client ID** + **Domain** + **Signal Tier** + **Report Window Type** → for Trend Direction comparison

---

## Node Flow (17 Nodes Total)

### TRIGGER NODES (Steps 1a-1c)

**Step 1a - Schedule Trigger Daily**
- Trigger: Every day at 7am UTC
- Fires: Step 2a

**Step 1b - Schedule Trigger Weekly**
- Trigger: Every Monday at 6am UTC
- Fires: Step 2b

**Step 1c - Schedule Trigger Monthly**
- Trigger: 1st of every month at 6am UTC
- Fires: Step 2c

---

### SET WINDOW NODES (Steps 2a-2c)

**Step 2a - Set Daily Window**
- Type: Set Node (Manual Mapping)
- Output: `window_type: "daily"`, `window_days: 2`
- Wires to: Step 3

**Step 2b - Set Weekly Window**
- Type: Set Node (Manual Mapping)
- Output: `window_type: "weekly"`, `window_days: 7`
- Wires to: Step 3

**Step 2c - Set Monthly Window**
- Type: Set Node (Manual Mapping)
- Output: `window_type: "monthly"`, `window_days: 30`
- Wires to: Step 3

**All three Set nodes converge into Step 3.**

---

### WINDOW CALCULATION (Step 3)

**Step 3 - Set Window Parameters**
- Type: Code Node
- Receives: `window_type` + `window_days` from upstream Set node
- Logic:
  - **If `window_type === 'weekly'`:** Calculate previous complete calendar week (Monday 00:00 → Sunday 23:59)
  - **Else (daily/monthly):** Rolling window from current time minus `window_days`
- Output: `window_start` (ISO 8601), `window_end` (ISO 8601), `window_type`, `window_days`, `sia_run_id`, `run_timestamp`

**Calendar week calculation (weekly only):**
```javascript
const currentDay = now.getUTCDay(); // 0=Sunday, 1=Monday, ..., 6=Saturday
let daysToLastSunday;
if (currentDay === 0) {
  daysToLastSunday = 0; // Today is Sunday
} else if (currentDay === 1) {
  daysToLastSunday = 1; // Today is Monday, go back 1 day to Sunday
} else {
  daysToLastSunday = currentDay; // Go back to most recent Sunday
}

// Last Sunday 23:59:59.999 UTC
const lastSunday = new Date(now.getTime() - daysToLastSunday * 86400000);
lastSunday.setUTCHours(23, 59, 59, 999);
windowEnd = lastSunday.toISOString();

// Previous Monday 00:00:00.000 UTC (6 days before last Sunday)
const previousMonday = new Date(lastSunday.getTime() - 6 * 86400000);
previousMonday.setUTCHours(0, 0, 0, 0);
windowStart = previousMonday.toISOString();
```

**Example outputs:**
- Daily run (April 28 7am): `window_start: 2026-04-26T07:00:00.000Z`, `window_end: 2026-04-28T07:00:00.000Z` (2 days)
- Weekly run (Monday April 28 6am): `window_start: 2026-04-21T00:00:00.000Z`, `window_end: 2026-04-27T23:59:59.999Z` (Mon-Sun)
- Monthly run (May 1 6am): `window_start: 2026-04-01T06:00:00.000Z`, `window_end: 2026-05-01T06:00:00.000Z` (30 days)

---

### PASS-THROUGH (Step 4)

**Step 4 - Set Window Bounds**
- Type: Code Node
- Function: Pass-through (all fields from Step 3 copied forward)
- Exists for architecture compatibility

---

### ALA FETCH & DATE FILTER (Steps 5-6)

**Step 5 - Fetch ALL ALA Records**
- Type: HTTP Request (GET)
- URL: `http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr?limit=1000&sort=Date`
- Auth: xc-token
- Returns: ALL ALA records (all clients, no date filter in URL)
- Why no ESS Status filter: ALA table doesn't have ESS Status field

**Step 6 - Filter ALA by Date**
- Type: Code Node
- Input: All ALA records from Step 5 + window bounds from Step 4
- Logic:
  1. Parse ALA `Date` field (M/D/YY format: "4/23/26")
  2. Convert to timestamp: `new Date(2000 + year, month - 1, day).getTime()`
  3. Filter: timestamp >= `window_start` AND < `window_end`
  4. Build `ala_client_map`: Map of ALA ID → Client ID
  5. Build `ala_id_list`: Array of filtered ALA IDs
- Output: `filtered_ala_records`, `ala_id_list`, `ala_client_map`, `ala_count`, `empty_run`

**Date parsing code:**
```javascript
for (let i = 0; i < allAla.length; i++) {
  const rec = allAla[i];
  const raw = rec['Date'] || ''; // "4/23/26"
  
  if (raw.length > 0) {
    const parts = raw.split('/'); // ["4", "23", "26"]
    if (parts.length === 3) {
      const month = parseInt(parts[0]); // 4
      const day = parseInt(parts[1]); // 23
      const year = 2000 + parseInt(parts[2]); // 2026
      const ts = new Date(year, month - 1, day).getTime();
      
      if (ts >= periodStartTs && ts < periodEndTs) {
        filteredAla.push(rec);
        const alaId = rec['Id'];
        alaIdList.push(alaId);
        alaClientMap[alaId] = rec['Client ID'] || 'UNKNOWN';
      }
    }
  }
}
```

---

### EIP FETCH & MATCH (Steps 7-8)

**Step 7 - Fetch ALL EIP Records**
- Type: HTTP Request (GET)
- URL: `http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mhicpnrahaesxmy?where=(ESS%20Status,eq,Complete)&limit=1000`
- Auth: xc-token
- Filter: ESS Status = Complete (ensures record completed pipeline)
- Limit: 1000 (explicit — NocoDB default is 25 records)
- Returns: All EIP records where ESS Status = Complete

**Step 8 - Filter EIP by ALA Match**
- Type: Code Node
- Input: All EIP records from Step 7 + filtered ALA data from Step 6
- Logic:
  1. Build `alaIdSet` from `ala_id_list` (for fast lookup)
  2. Loop through EIP records
  3. Match: EIP `ALA Record ID` in `alaIdSet`
  4. Attach: `client_id` from `ala_client_map[alaId]` to matched EIP record
  5. Collect: All matched EIP records
- Output: `eip_records` (with `client_id` attached), `total_records`, `empty_run`

**Matching code:**
```javascript
const alaIdSet = {};
for (let i = 0; i < alaIdList.length; i++) {
  alaIdSet[alaIdList[i]] = true;
}

const filteredEip = [];
for (let i = 0; i < allEip.length; i++) {
  const rec = allEip[i];
  const alaId = rec['ALA Record ID'];
  if (alaIdSet[alaId]) {
    rec.client_id = alaClientMap[alaId]; // Attach Client ID
    filteredEip.push(rec);
  }
}
```

**Natural exclusion:** ALA records without matching EIP records are dropped. This ensures only fully-processed reviews (completed ALA→EIP→ESS→HSI pipeline) are included in SIA clusters.

---

### PRIOR CLUSTER FETCH & MAP (Steps 9-10)

**Step 9 - Load Prior SIA Clusters**
- Type: HTTP Request (GET)
- URL: `http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mdn68l4lm609fve?where=(Report%20Window%20Type,eq,{{$json.window_type}})&limit=1000&sort=-Run%20Timestamp`
- Auth: xc-token
- Filter: Report Window Type matches current run (daily/weekly/monthly)
- Why this filter: Ensures weekly run compares against prior weekly run (not monthly)
- Returns: All prior SIA cluster records for this window type

**Step 10 - Build Prior N Map**
- Type: Code Node
- Input: Prior cluster records from Step 9 + EIP records from Step 8
- Logic: Build composite key map: `Client ID + Domain + Signal Tier` → Prior N count
- Output: `prior_n_map`, `has_prior_run`

**Composite key code:**
```javascript
const priorNByKey = {};
for (let i = 0; i < priorClusters.length; i++) {
  const pc = priorClusters[i];
  const key = (pc['Client ID'] || 'UNKNOWN') + '||' + pc['Domain'] + '||' + pc['Signal Tier'];
  if (!priorNByKey[key]) {
    priorNByKey[key] = parseInt(pc['N'] || 0);
  }
}
```

**Why composite key with Client ID:** PAK-001 Service Quality T-NEGATIVE must compare against prior PAK-001 Service Quality T-NEGATIVE (not EDO-001).

---

### CLUSTER BUILD (Steps 11-12)

**Step 11 - Build Clusters**
- Type: Code Node
- Input: EIP records from Step 8 + prior N map from Step 10
- Logic:
  1. Map Signal Type → Signal Tier (T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE)
  2. Group EIP records by: Domain + Signal Tier
  3. For each cluster: extract `client_id` from first record, count N, aggregate pain points + emotions
  4. Compare against prior N using composite key: `client_id + domain + signal_tier`
  5. Calculate Trend Direction: New/Stable/Growing/Declining
  6. Calculate percentage of total records
- Output: `clusters` array, cluster counts by tier, trend counts

**Signal Tier mapping:**
```javascript
function getSignalTier(signal_type) {
  if (['Dignity-Risk Signal','Negative Signal','Masked Negative Signal'].includes(signal_type))
    return 'T-NEGATIVE';
  if (['Ambiguous Negative Signal','Mixed Signal'].includes(signal_type))
    return 'T-AMBIGUOUS';
  return 'T-POSITIVE';
}
```

**Cluster grouping key:** `Domain + '||' + Signal Tier`  
**Example:** `"Service Quality & Delivery||T-NEGATIVE"`

**Trend Direction logic:**
```javascript
const clusterKey = cl.client_id + '||' + cl.domain + '||' + cl.signal_tier;
const priorN = prior_n_map[clusterKey];

if (has_prior_run && priorN !== undefined) {
  prior_n = priorN;
  n_delta = cl.n - priorN;
  const pct = priorN > 0 ? Math.abs(n_delta) / priorN : 1;
  trend_direction = pct <= trend_threshold ? 'Stable' : n_delta > 0 ? 'Growing' : 'Declining';
} else {
  trend_direction = 'New';
}
```

**Step 12 - Split Clusters**
- Type: Code Node
- Input: Clusters array from Step 11
- Logic: Convert single-item-with-array to multiple-items-with-object (n8n Loop Over Items preparation)
- Output: One item per cluster (array → items)

---

### NOCODB WRITE (Steps 13-14)

**Step 13 - Build Bulk POST Body**
- Type: Code Node
- Input: Multiple cluster items from Step 12
- Logic: Build JSON array of NocoDB records
- Output: `bulk_post_body` (JSON stringified array)

**NocoDB record structure:**
```javascript
const record = {};
record["Client ID"] = cl.client_id;
record["SIA Run ID"] = cl.sia_run_id;
record["Run Timestamp"] = cl.run_timestamp;
record["Report Window Type"] = cl.window_type; // daily/weekly/monthly
record["Window Days"] = cl.window_days;
record["Domain"] = cl.domain;
record["Signal Tier"] = cl.signal_tier;
record["N"] = cl.n;
record["Prior N"] = cl.prior_n;
record["N Delta"] = cl.n_delta;
record["Trend Direction"] = cl.trend_direction;
record["Enriched Pain Point Breakdown"] = cl.enriched_pain_point_breakdown;
record["Enriched Emotion Breakdown"] = cl.enriched_emotion_breakdown;
record["Is Singleton"] = cl.is_singleton;
record["Percentage"] = cl.pct;
```

**Step 14 - NocoDB Bulk POST Clusters**
- Type: HTTP Request (POST)
- URL: `http://nocodb:8080/api/v1/db/data/bulk/noco/pq249fix22t3ofv/mdn68l4lm609fve`
- Auth: xc-token
- Body: `{{$json.bulk_post_body}}` (RAW, Content-Type: application/json)
- Writes: All cluster records to SIA table in single bulk operation

---

## NocoDB Schema (v4.0)

**Table ID:** `mdn68l4lm609fve`  
**Table Name:** SIA Clusters

**Fields (15 total):**

| Field Name | Type | Notes |
|------------|------|-------|
| Id | AutoNumber | Primary key |
| **Client ID** | SingleLineText | **NEW v4.0** - Field ID: cqxifo3lpi5m0x6 |
| SIA Run ID | SingleLineText | Format: `SIA-YYYYMMDD-HHMMSS` |
| Run Timestamp | DateTime | ISO 8601 UTC |
| **Report Window Type** | SingleSelect | **NEW v4.0** - Values: daily/weekly/monthly - Field ID: c5wrrw2729hlwa0 |
| Window Days | Number | 2 (daily), 7 (weekly), 30 (monthly) |
| Domain | SingleLineText | Pain Point Domain from EIP |
| Signal Tier | SingleSelect | T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE |
| N | Number | Count of reviews in this cluster |
| Prior N | Number | Count from prior period (null if New) |
| N Delta | Number | N - Prior N (null if New) |
| Trend Direction | SingleSelect | New / Stable / Growing / Declining |
| Enriched Pain Point Breakdown | LongText | JSON string: `{"pain_point": count}` |
| Enriched Emotion Breakdown | LongText | JSON string: `{"emotion": count}` |
| Is Singleton | Checkbox | true if N < min_cluster_size (2) |
| Percentage | Number | Percentage of total records in this run |

---

## Multi-Client Architecture

**How it works:**
1. **Single SIA workflow** processes all clients in one run
2. **Step 5** fetches ALL ALA records (PAK-001 + EDO-001 + AJI-001 + future clients)
3. **Step 6** filters by date, builds `ala_client_map` (ALA ID → Client ID)
4. **Step 8** attaches `Client ID` to each matched EIP record
5. **Step 11** includes `client_id` in each cluster
6. **Step 13** writes `Client ID` to every SIA NocoDB record

**Result:** SIA table contains clusters for all clients, each tagged with `Client ID`.

**MRA query pattern:**
```
?where=(Client%20ID,eq,PAK-001)&(Report%20Window%20Type,eq,weekly)&sort=-Run%20Timestamp
```
Returns: Only PAK-001 weekly clusters (excludes other clients and other window types)

**Scalability:** Adding new clients requires ZERO SIA code changes. Just add Client Config NocoDB record and start processing reviews through ALA→RDA. SIA automatically picks them up.

---

## Example Output

**Weekly run Monday April 28, 2026 (analyzing April 21-27):**

**SIA writes 2 cluster records for PAK-001:**

**Record 1:**
```json
{
  "Client ID": "PAK-001",
  "SIA Run ID": "SIA-20260428-060000",
  "Run Timestamp": "2026-04-28T06:00:00.000Z",
  "Report Window Type": "weekly",
  "Window Days": 7,
  "Domain": "Physical Environment & Ambiance",
  "Signal Tier": "T-NEGATIVE",
  "N": 1,
  "Prior N": null,
  "N Delta": null,
  "Trend Direction": "New",
  "Enriched Pain Point Breakdown": "{\"Excessive noise level\":1}",
  "Enriched Emotion Breakdown": "{\"Frustration\":1}",
  "Is Singleton": true,
  "Percentage": 33.33
}
```

**Record 2:**
```json
{
  "Client ID": "PAK-001",
  "SIA Run ID": "SIA-20260428-060000",
  "Run Timestamp": "2026-04-28T06:00:00.000Z",
  "Report Window Type": "weekly",
  "Window Days": 7,
  "Domain": "Service Quality & Delivery",
  "Signal Tier": "T-POSITIVE",
  "N": 2,
  "Prior N": null,
  "N Delta": null,
  "Trend Direction": "New",
  "Enriched Pain Point Breakdown": "{\"Guest avoids detailed feedback\":2}",
  "Enriched Emotion Breakdown": "{\"Delight\":2}",
  "Is Singleton": false,
  "Percentage": 66.67
}
```

**Dashboard displays:**
- 33.33% T-NEGATIVE (1 review)
- 66.67% T-POSITIVE (2 reviews)
- Week of April 21-27, 2026
- Total 3 reviews processed

---

## Token Budget

**Zero tokens per run.**

**SIA makes:**
- Zero OpenAI API calls
- Zero Anthropic API calls
- Zero dictionary queries

**Only operations:**
- NocoDB queries (ALA, EIP, prior SIA clusters)
- JavaScript loops and calculations (date parsing, grouping, counting, percentage math)
- NocoDB bulk write (SIA clusters)

**Cost:** Infrastructure only (NocoDB queries, n8n execution time)

**Estimated execution time:**
- 100 reviews: ~3-5 seconds
- 1,000 reviews: ~10-15 seconds

---

## Key Design Decisions (v4.0)

### Why Three Schedules Instead of One?

**Daily reports need 2-day windows. Monthly reports need 30-day windows.**

**Single schedule approach would require:**
- Running daily but computing all three windows every day (wasteful)
- OR running monthly but daily reports would be 30 days old (useless)

**Three-schedule approach:**
- Daily runs at 7am (after overnight review ingestion)
- Weekly runs Monday 6am (before MRA weekly brief at 7am)
- Monthly runs 1st at 6am (before MRA monthly report at 7am)
- Each writes records tagged with Report Window Type
- MRA queries the correct window type for each report

### Why Filter by Guest Review Date (Not Pipeline Processing Time)?

**Problem:** If SIA filtered by EIP Timestamp:
- Weekly report for April 21-27 would include reviews written in March but processed in April
- Would exclude reviews written April 26 but processed April 28

**Solution:** Filter by ALA `Date` field (when guest actually wrote the review)
- Weekly report for April 21-27 includes all reviews with Date in that range
- Accurately reflects guest activity for that week

**Trade-off:** More complex architecture (fetch ALA → filter → fetch EIP → match)  
**Benefit:** Accurate time-windowed intelligence for MRA reports

### Why Calendar Week (Not Rolling 7-Day Window)?

**Business requirement:** Weekly reports should align with standard business weeks.

**Rolling window problem:**
- Monday April 28 run with rolling 7-day window: April 21 6am → April 28 6am
- Includes partial Monday April 21 and partial Monday April 28
- Trend comparison: "last week" vs "this week" have overlapping days

**Calendar week solution:**
- Monday April 28 run: April 21 00:00 → April 27 23:59 (complete week)
- Next Monday May 5 run: April 28 00:00 → May 4 23:59 (next complete week)
- No overlap, clean week-over-week comparison

### Why Client ID in Trend Direction Key?

**Problem without Client ID:**
- PAK-001 Service Quality T-NEGATIVE (N=5) vs prior N
- Step 10 lookup key: "Service Quality & Delivery||T-NEGATIVE"
- Matches against EDO-001's prior Service Quality T-NEGATIVE (N=12)
- Trend Direction: Declining (5 vs 12) — WRONG comparison

**Solution with Client ID:**
- Lookup key: "PAK-001||Service Quality & Delivery||T-NEGATIVE"
- Matches only against PAK-001's prior week
- Trend Direction: Growing (5 vs 3) — CORRECT comparison

---

## Critical Build Lessons (v4.0 Additions)

### Lesson 1: ALA Date Field Location

**`Date` field exists ONLY in ALA table.** It was never passed through the ALA→EIP→ESS→HSI pipeline chain.

**Implications:**
- EIP/ESS/HSI tables do NOT have guest review creation date
- SIA cannot filter EIP records by review date directly
- Must fetch ALA first, filter by Date, then match EIP records via ALA Record ID

**Solution:** MRA architecture pattern (fetch ALA → filter Date → fetch EIP → match by FK).

### Lesson 2: M/D/YY Date Format

**ALA stores dates as:** `"4/23/26"` (M/D/YY format, NOT ISO 8601)

**Parsing required:**
```javascript
const parts = raw.split('/'); // ["4", "23", "26"]
const month = parseInt(parts[0]); // 4
const day = parseInt(parts[1]); // 23
const year = 2000 + parseInt(parts[2]); // 2026
const ts = new Date(year, month - 1, day).getTime();
```

**Edge cases:**
- No leading zeros: `"4/3/26"` (April 3, not 04/03)
- 2-digit year: Always 20xx (2000 + YY)

### Lesson 3: NocoDB Pagination Default

**NocoDB default page size:** 25 records

**Step 7 originally had:**
```
http://nocodb:8080/api/v1/db/data/noco/.../mhicpnrahaesxmy
```

**Result:** Only 25 EIP records fetched (out of 57 total)

**Fix:** Explicit `limit=1000` in URL
```
http://nocodb:8080/api/v1/db/data/noco/.../mhicpnrahaesxmy?where=(ESS%20Status,eq,Complete)&limit=1000
```

**Lesson:** Always specify explicit limit to avoid pagination truncation.

### Lesson 4: Set Node Field Name Typos

**Problem:** Step 2a had leading space in field name: ` window_days` (space before w)

**Step 3 read:** `inp.window_days` → `undefined` → defaulted to 30

**Result:** Daily report showed Window Days = 30 instead of 2

**Fix:** Delete and re-add field with EXACT name `window_days` (no leading/trailing spaces)

**Lesson:** n8n Set node field names must match EXACTLY (case-sensitive, no extra spaces).

### Lesson 5: ESS Status Field Location

**Step 5 originally filtered by:** `?where=(ESS%20Status,eq,Complete)`

**Error:** `Column alias 'ESS Status' not found.`

**Reason:** ESS Status field exists in **EIP table**, NOT ALA table.

**Fix:** Remove ESS Status filter from Step 5 (ALA fetch). Add it to Step 7 (EIP fetch).

**Lesson:** Field location verification required before writing NocoDB filters.

### Lesson 6: Calendar Week Calculation Edge Cases

**Execution day = Sunday:**
- `currentDay = 0`
- `daysToLastSunday = 0`
- Last Sunday = today 23:59:59
- Previous Monday = 6 days before today

**Execution day = Monday:**
- `currentDay = 1`
- `daysToLastSunday = 1`
- Last Sunday = yesterday 23:59:59
- Previous Monday = 6 days before yesterday

**Execution day = Tuesday-Saturday:**
- `daysToLastSunday = currentDay`
- Last Sunday = most recent Sunday before today
- Previous Monday = 6 days before that Sunday

**Lesson:** Calendar week calculation requires special handling for Sunday and Monday execution days.

### Lesson 7: Composite Key Field Order

**Trend Direction lookup requires:**
```javascript
const key = client_id + '||' + domain + '||' + signal_tier;
```

**NOT:**
```javascript
const key = domain + '||' + signal_tier; // Missing Client ID
```

**Why it matters:** Multi-client environment requires Client ID in composite key to prevent cross-client trend comparison.

**Lesson:** Composite keys must include ALL dimensions that define uniqueness in multi-tenant architecture.

---

## Downstream Handoff

**SIA → MRA (Metrics & Reporting Agent):**

**MRA Daily Report (7am UTC):**
```
Query: ?where=(Client%20ID,eq,PAK-001)&(Report%20Window%20Type,eq,daily)&sort=-Run%20Timestamp&limit=20
```
Returns: Latest daily SIA clusters for PAK-001

**MRA Weekly Brief (Monday 7am UTC):**
```
Query: ?where=(Client%20ID,eq,PAK-001)&(Report%20Window%20Type,eq,weekly)&sort=-Run%20Timestamp&limit=20
```
Returns: Latest weekly SIA clusters for PAK-001

**MRA Monthly Report (1st 7am UTC):**
```
Query: ?where=(Client%20ID,eq,PAK-001)&(Report%20Window%20Type,eq,monthly)&sort=-Run%20Timestamp&limit=20
```
Returns: Latest monthly SIA clusters for PAK-001

**SIA → Dashboard (future):**
- Signal Pulse chart (T-NEG/T-AMB/T-POS percentages)
- Signal Distribution by domain
- Trend Signals (week-over-week changes)

**SIA does NOT pass to BRA or RDA** (those agents operate on individual records, not aggregations).

---

## Testing Verification (Chat #77, April 28, 2026)

**Weekly report (Monday April 28, 6am):**
- ✅ Window: April 21 00:00 → April 27 23:59 (7-day calendar week)
- ✅ Records found: 2 clusters for PAK-001
- ✅ Client ID: PAK-001 in both records
- ✅ Report Window Type: weekly
- ✅ Window Days: 7
- ✅ NocoDB write successful

**Daily report (April 28, 7am):**
- ✅ Window: April 26 19:32 → April 28 19:32 (2 days)
- ✅ Records found: 0 (no reviews in last 48 hours)
- ✅ Window Days: 2 (NOT 30)
- ✅ Empty result = expected behavior

**Monthly report:**
- Status: Ready to test (not executed in Chat #77)

---

## Related Documents

- **Changelog:** [SCX_SIA_CHANGELOG.md](SCX_SIA_CHANGELOG.md)
- **Schema:** [SIA_Schema.md](SIA_Schema.md)
- **Design Rationale:** [SIA_Design_Rationale.md](SIA_Design_Rationale.md)
- **Upstream Tables:** [../ALA/SCX_ALA_HOW_v4.md](../ALA/SCX_ALA_HOW_v4.md), [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md)
- **Downstream Consumer:** [../MRA/SCX_MRA_HOW_v1.md](../MRA/SCX_MRA_HOW_v1.md)
- **Dashboard Spec:** [../../commercial/Dashboard_Freelancer_Brief.md](../../commercial/Dashboard_Freelancer_Brief.md)

---

## n8n Workflow Details

**Workflow Name:** SCX-SIA  
**Triggers:** Three Schedule nodes (daily/weekly/monthly)  
**Credentials:** NocoDB xc-token only (no AI API keys)  
**Total Nodes:** 17 main nodes  
**Execution Time:** ~3-15 seconds (depends on record count)

**Critical n8n 2.4.6 Rules:**
- All aggregation logic in Code Nodes (pure JavaScript)
- No AI API calls anywhere in workflow
- No spread operator (key-by-key object assignment only)
- Use index-based `for` loops (no `for...of` in Code Nodes)
- `pageInfo.totalRows` for record counts (not `list.length`)
- Explicit `limit=1000` in all NocoDB GET requests
- Field names in Set nodes: exact match, no leading/trailing spaces
- `$workflow.staticData` unreliable — load data fresh per run

---

## Version History

**v1.0 (Chat #50, March 2026):**
- Initial concept
- Single weekly schedule
- Read from EIP table directly
- No multi-client support

**v2.0 (Chat #72, April 2026):**
- 11 nodes operational
- Schedule trigger added
- Cluster grouping by Domain + Signal Tier
- Trend Direction calculation

**v3.0 (Chat #74, April 4, 2026):**
- Verified operational
- Single weekly schedule (Monday 5am UTC)
- Filtered by EIP Timestamp
- Rolling 7-day window

**v4.0 (Chat #77, April 28, 2026):**
- **Three independent schedules** (daily 7am, weekly Mon 6am, monthly 1st 6am)
- **17 nodes** (added 6 nodes for multi-schedule support)
- **Calendar week alignment** for weekly reports (Monday 00:00 → Sunday 23:59)
- **Guest review date filtering** (ALA `Date` field, not EIP Timestamp)
- **Multi-client support** via Client ID field
- **Report Window Type** field for MRA filtering
- **Client-aware trend comparison** (composite key: Client ID + Domain + Signal Tier)
- **MRA architecture pattern** (fetch ALA → filter Date → fetch EIP → match by FK)

---

**End of SIA HOW v4.0**
