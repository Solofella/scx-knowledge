## OUTCOME: SIA SCHEMA UPDATE

**Current version fetched:** v1.0  
**Updated to:** v2.0 (Chat #77, April 28, 2026)

---

### MAJOR SCHEMA CHANGES

**1. NEW FIELDS ADDED (v2.0)**
- `Report Window Type` (SingleSelect: daily/weekly/monthly)
- `Client ID` now explicitly documented with field ID

**2. FIELD COUNT CHANGE**
- v1.0: 12 fields
- v2.0: 15 fields (added 3 new fields)

**3. COMPOSITE KEY CHANGE**
- v1.0: Domain + Signal Tier
- v2.0: **Client ID + Domain + Signal Tier** (for multi-client trend comparison)

**4. DATA SOURCE CHANGE**
- v1.0: Read from EIP table directly (filtered by EIP Timestamp)
- v2.0: Read from ALA table (filtered by guest review Date), then match to EIP table

---

### COMPLETE UPDATED DOCUMENT FOLLOWS BELOW:

---

# SIA_Schema_v2.0

**NocoDB Table ID:** `mdn68l4lm609fve`  
**Base ID:** `pq249fix22t3ofv`  
**Last Updated:** Chat #77 · April 28, 2026

---

## Table Fields (15 Total)

| Field Name | NocoDB Type | Field ID | Source | Purpose |
|------------|-------------|----------|--------|---------|
| Id | AutoNumber | Auto | System | Primary key |
| **Client ID** | SingleLineText | cqxifo3lpi5m0x6 | ALA via Step 6 | Multi-client identifier (PAK-001, EDO-001, etc.) |
| SIA Run ID | SingleLineText | - | Step 3 | Unique run identifier: `SIA-YYYYMMDD-HHMMSS` |
| Run Timestamp | DateTime | - | Step 3 | ISO 8601 UTC timestamp of SIA execution |
| **Report Window Type** | SingleSelect | c5wrrw2729hlwa0 | Step 2a/2b/2c | daily / weekly / monthly (NEW v2.0) |
| Window Days | Number | - | Step 2a/2b/2c | 2 (daily), 7 (weekly), 30 (monthly) |
| **Domain** | SingleLineText | - | EIP via Step 8 | Cluster key: Service Quality & Delivery, Food & Beverage Quality, etc. |
| **Signal Tier** | SingleSelect | - | Step 11 computation | Cluster key: T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE |
| **N** | Number | - | Step 11 computation | Count of reviews in this cluster |
| **Prior N** | Number | - | Step 11 computation | Count from prior period (null if Trend Direction = New) |
| **N Delta** | Number | - | Step 11 computation | N - Prior N (null if New) |
| **Trend Direction** | SingleSelect | - | Step 11 computation | New / Stable / Growing / Declining |
| **Enriched Pain Point Breakdown** | LongText (JSON) | - | Step 11 computation | JSON object: `{"pain_point": count}` sorted by frequency |
| **Enriched Emotion Breakdown** | LongText (JSON) | - | Step 11 computation | JSON object: `{"emotion": count}` sorted by frequency |
| Is Singleton | Checkbox | - | Step 11 computation | true if N < min_cluster_size (2) |
| Percentage | Number | - | Step 11 computation | (N / Total Records) × 100, rounded to 2 decimals |

---

## Key Changes in v2.0

### NEW: Client ID Field (Multi-Client Support)

**Field ID:** `cqxifo3lpi5m0x6`  
**Type:** SingleLineText  
**Source:** ALA table via Step 6 mapping

**Purpose:** Enables single SIA workflow to process all clients (PAK-001, EDO-001, AJI-001, future clients) in one run. Each cluster record is tagged with the client it belongs to.

**Composite key for trend comparison:**
```javascript
const key = client_id + '||' + domain + '||' + signal_tier;
```

**MRA query pattern:**
```
?where=(Client%20ID,eq,PAK-001)&(Report%20Window%20Type,eq,weekly)
```
Returns: Only PAK-001 weekly clusters

### NEW: Report Window Type Field (Time Window Filtering)

**Field ID:** `c5wrrw2729hlwa0`  
**Type:** SingleSelect  
**Values:** `daily` | `weekly` | `monthly`  
**Source:** Step 2a/2b/2c Set nodes

**Purpose:** Enables MRA to filter SIA clusters by time granularity. Weekly reports query `Report Window Type = weekly`, daily reports query `daily`, etc.

**Why this matters:**
- Prior N lookup must compare like-for-like (weekly vs prior weekly, not weekly vs monthly)
- Step 9 filter: `?where=(Report%20Window%20Type,eq,{{$json.window_type}})`
- Ensures Trend Direction accuracy across different report types

### CHANGED: Data Source Architecture

**v1.0 approach:**
- Step 3: Query EIP table directly
- Step 4: Filter by EIP Timestamp (pipeline processing time)

**v2.0 approach (mirrors MRA pattern):**
- Step 5: Query ALA table (all records)
- Step 6: Filter ALA by `Date` field (guest review creation date, M/D/YY format)
- Step 7: Query EIP table (all records)
- Step 8: Match EIP to filtered ALA via `ALA Record ID`, attach `Client ID`

**Why the change:**
- `Date` field exists ONLY in ALA table (never passed through pipeline)
- Filtering by EIP Timestamp reports pipeline activity, not guest review activity
- New architecture ensures weekly report for April 21-27 includes reviews written by guests in that week (not reviews processed by SubtextCX in that week)

---

## Relationships

**Upstream Tables:**
- **ALA** (`m57efwbtrvwohhr`) → provides `Date` field, `Client ID`, `ALA Record ID`
- **EIP** (`mhicpnrahaesxmy`) → provides `Domain`, `Signal Type`, `Enriched Pain Point`, `Enriched Emotion Tag`

**Downstream Consumers:**
- **MRA** (Metrics & Reporting Agent) → queries SIA table for daily/weekly/monthly reports
- **Dashboard** (future) → queries SIA table for Signal Pulse, Signal Distribution, Trend Signals

**Query Pattern (MRA):**
```
Base filter: Client ID = [specific client]
+ Report Window Type = [daily/weekly/monthly]
+ Sort: -Run Timestamp
+ Limit: Latest run only
```

---

## Field Computation Details

### Domain + Signal Tier (Aggregation Cluster Keys)

**Source:** EIP records (filtered by ALA Date match)

**Signal Tier mapping logic (Step 11):**
```javascript
function getSignalTier(signal_type) {
  if (['Dignity-Risk Signal','Negative Signal','Masked Negative Signal'].includes(signal_type))
    return 'T-NEGATIVE';
  if (['Ambiguous Negative Signal','Mixed Signal'].includes(signal_type))
    return 'T-AMBIGUOUS';
  return 'T-POSITIVE';
}
```

**Clustering logic:**
```javascript
const clusterMap = {};

for (let i = 0; i < eip_records.length; i++) {
  const rec = eip_records[i];
  const domain = rec['Domain'] || 'Unknown';
  const tier = getSignalTier(rec['Signal Type'] || '');
  const key = domain + '||' + tier;
  
  if (!clusterMap[key]) {
    clusterMap[key] = {
      domain: domain,
      signal_tier: tier,
      n: 0,
      pain_points: {},
      emotions: {},
      client_id: rec.client_id || 'UNKNOWN' // NEW v2.0
    };
  }
  
  clusterMap[key].n++;
  const pp = rec['Enriched Pain Point'] || 'Unknown';
  clusterMap[key].pain_points[pp] = (clusterMap[key].pain_points[pp] || 0) + 1;
  const em = rec['Enriched Emotion Tag'] || 'Unknown';
  clusterMap[key].emotions[em] = (clusterMap[key].emotions[em] || 0) + 1;
}
```

**Possible domain values (from Pain Point Master v4.1, 13 domains):**
- Service Quality & Delivery
- Food & Beverage Quality
- Value & Pricing Perception
- Physical Environment & Ambiance
- Staff Behavior & Attitude
- Hygiene & Cleanliness
- Expectation vs. Reality
- Communication & Responsiveness
- Reservation & Booking Experience
- Technology & Digital Experience
- Loyalty & Recognition
- Accessibility & Inclusion
- Safety & Wellbeing

**Signal Tier values:**
- **T-NEGATIVE:** Negative or dignity-risk guest signals
- **T-AMBIGUOUS:** Mixed or unclear intent signals
- **T-POSITIVE:** Positive guest signals

---

### N (Count of Reviews per Cluster)

**Computation:** Simple frequency count

```javascript
clusterMap[key].n++; // Increment for each EIP record in cluster
```

**Example:**
- "Service Quality & Delivery||T-NEGATIVE" → 5 reviews
- "Food & Beverage Quality||T-POSITIVE" → 12 reviews

---

### Prior N (Count from Prior Period)

**Source:** SIA NocoDB table (prior run)

**Lookup logic (Step 10):**
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

**Composite key includes Client ID (NEW v2.0):**
- Ensures PAK-001 compares against prior PAK-001 data (not EDO-001)

**Step 11 lookup:**
```javascript
const clusterKey = cl.client_id + '||' + cl.domain + '||' + cl.signal_tier;
const priorN = prior_n_map[clusterKey];
```

**Value:** Number or `null` (if no prior period data)

---

### N Delta (Change from Prior Period)

**Computation:** Current N - Prior N

```javascript
if (has_prior_run && priorN !== undefined) {
  prior_n = priorN;
  n_delta = cl.n - priorN;
} else {
  n_delta = null; // No prior data
}
```

**Examples:**
- Current N = 5, Prior N = 3 → N Delta = +2
- Current N = 2, Prior N = 8 → N Delta = -6
- Current N = 5, Prior N = null → N Delta = null

---

### Trend Direction (Week-over-Week Change)

**Computation:** Compare N Delta against threshold (20% change)

```javascript
const trend_threshold = 0.20; // 20%

if (has_prior_run && priorN !== undefined) {
  const pct = priorN > 0 ? Math.abs(n_delta) / priorN : 1;
  if (pct <= trend_threshold) {
    trend_direction = 'Stable'; // Within ±20%
  } else if (n_delta > 0) {
    trend_direction = 'Growing'; // Increased >20%
  } else {
    trend_direction = 'Declining'; // Decreased >20%
  }
} else {
  trend_direction = 'New'; // First appearance
}
```

**Possible values:**
- **New:** No prior period data (first occurrence of this cluster)
- **Stable:** Change within ±20% of prior count
- **Growing:** Increase >20% from prior count
- **Declining:** Decrease >20% from prior count

**Example:**
- Prior N = 10, Current N = 13 → Change = 30% → **Growing**
- Prior N = 10, Current N = 11 → Change = 10% → **Stable**
- Prior N = 10, Current N = 7 → Change = -30% → **Declining**

---

### Enriched Pain Point Breakdown (JSON)

**Computation:** Frequency count + sort descending

```javascript
const ppEntries = Object.keys(cl.pain_points).map(k => ({ k: k, v: cl.pain_points[k] }));
ppEntries.sort((a,b) => b.v - a.v); // Descending by count

const ppBreakdown = {};
for (let p = 0; p < ppEntries.length; p++) {
  const e = ppEntries[p];
  ppBreakdown[e.k] = e.v;
}

clusterItem.enriched_pain_point_breakdown = JSON.stringify(ppBreakdown);
```

**Example JSON output:**
```json
{
  "Inattentive staff presence": 3,
  "Slow service during peak": 2,
  "Order accuracy failure": 1
}
```

**MRA usage:** Display top 3 pain points per cluster

---

### Enriched Emotion Breakdown (JSON)

**Computation:** Same logic as pain points

```javascript
const emEntries = Object.keys(cl.emotions).map(k => ({ k: k, v: cl.emotions[k] }));
emEntries.sort((a,b) => b.v - a.v);

const emBreakdown = {};
for (let m = 0; m < emEntries.length; m++) {
  const e = emEntries[m];
  emBreakdown[e.k] = e.v;
}

clusterItem.enriched_emotion_breakdown = JSON.stringify(emBreakdown);
```

**Example JSON output:**
```json
{
  "Frustration": 4,
  "Disappointment": 2
}
```

---

### Is Singleton (Cluster Size Flag)

**Computation:** Boolean flag for clusters below minimum size

```javascript
const min_cluster_size = 2;
clusterItem.is_singleton = cl.n < min_cluster_size;
```

**Purpose:** Filter out statistically insignificant clusters from trend analysis

**Example:**
- N = 1 → Is Singleton = true (exclude from trends)
- N = 2 → Is Singleton = false (include in trends)

---

### Percentage (% of Total Reviews)

**Computation:** (Cluster N / Total Records) × 100

```javascript
const totalN = clusters.reduce((sum, c) => sum + c.n, 0);
for (let c = 0; c < clusters.length; c++) {
  clusters[c].pct = totalN > 0 ? Math.round((clusters[c].n / totalN) * 10000) / 100 : 0;
}
```

**Example:**
- Total records in window: 13
- Cluster N = 5
- Percentage = (5 / 13) × 100 = 38.46%

---

## Token Budget

**Zero tokens per run.** No AI API calls.

**Operations (all deterministic JavaScript):**
- NocoDB GET ALA records (Step 5)
- Date parsing M/D/YY → timestamp (Step 6)
- NocoDB GET EIP records (Step 7)
- Array matching by ALA Record ID (Step 8)
- NocoDB GET prior SIA clusters (Step 9)
- Composite key map building (Step 10)
- Clustering by Domain + Signal Tier (Step 11)
- Frequency counting, percentage math, trend comparison (Step 11)
- JSON serialization (Step 11-13)
- NocoDB bulk POST (Step 14)

**Cost:** Infrastructure only (NocoDB API calls, n8n execution time, ~3-15 seconds per run)

---

## Data Read from ALA Table (NEW v2.0)

**Step 5 query:**
```
GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr?limit=1000&sort=Date
```

**Fields read:**
- **Id** (ALA Record ID)
- **Date** (M/D/YY format: "4/23/26")
- **Client ID**
- **EIP Status** (Complete) — not filtered in URL, filtered in Step 8 via EIP match

**Step 6 processing:**
- Parse Date field: `"4/23/26"` → `Date(2026, 3, 23)`
- Filter by window: `timestamp >= window_start && timestamp < window_end`
- Build maps: `ala_id_list`, `ala_client_map`

---

## Data Read from EIP Table

**Step 7 query:**
```
GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mhicpnrahaesxmy?where=(ESS%20Status,eq,Complete)&limit=1000
```

**Fields read:**
- **ALA Record ID** (FK to ALA)
- **Domain**
- **Signal Type**
- **Enriched Pain Point**
- **Enriched Emotion Tag**
- **ESS Status** (filter: Complete)

**Step 8 matching:**
- Match: EIP `ALA Record ID` in `ala_id_list`
- Attach: `client_id` from `ala_client_map[ALA Record ID]`
- Result: Filtered EIP records with `client_id` field attached

---

## Example SIA Output Records (v2.0)

**Weekly run: Monday April 28, 2026 (analyzing April 21-27)**

**Client: PAK-001, 3 reviews in window**

| Client ID | Domain | Signal Tier | N | Prior N | N Delta | Trend | Report Window Type | Window Days |
|-----------|--------|-------------|---|---------|---------|-------|-------------------|-------------|
| PAK-001 | Physical Environment & Ambiance | T-NEGATIVE | 1 | null | null | New | weekly | 7 |
| PAK-001 | Service Quality & Delivery | T-POSITIVE | 2 | null | null | New | weekly | 7 |

**Enriched Pain Point Breakdown examples:**
```json
{"Excessive noise level": 1}
{"Guest avoids detailed feedback": 2}
```

**Enriched Emotion Breakdown examples:**
```json
{"Frustration": 1}
{"Delight": 2}
```

---

## Data Flow (v2.0 Architecture)

```
Schedule Trigger (3 independent schedules)
├─ Daily: 7am UTC → Step 2a (window_type=daily, window_days=2)
├─ Weekly: Monday 6am UTC → Step 2b (window_type=weekly, window_days=7)
└─ Monthly: 1st 6am UTC → Step 2c (window_type=monthly, window_days=30)
    ↓
Step 3: Set Window Parameters
  - Weekly: Calculate Monday 00:00 → Sunday 23:59 (calendar week)
  - Daily/Monthly: Rolling window (current time - window_days)
    ↓
Step 4: Set Window Bounds (pass-through)
    ↓
Step 5: Fetch ALL ALA Records
  - Query: ALA table, no filters (all clients, all dates)
  - Returns: Array of ALA records
    ↓
Step 6: Filter ALA by Date
  - Parse M/D/YY → timestamp
  - Filter: timestamp in [window_start, window_end)
  - Build: ala_id_list, ala_client_map (ALA ID → Client ID)
    ↓
Step 7: Fetch ALL EIP Records
  - Query: EIP table, filter ESS Status = Complete
  - Returns: Array of EIP records
    ↓
Step 8: Filter EIP by ALA Match
  - Match: EIP ALA Record ID in ala_id_list
  - Attach: client_id from ala_client_map
  - Result: Filtered EIP records with client_id
    ↓
Step 9: Load Prior SIA Clusters
  - Query: SIA table, filter Report Window Type = current window_type
  - Returns: Prior period cluster records
    ↓
Step 10: Build Prior N Map
  - Composite key: Client ID + Domain + Signal Tier
  - Result: prior_n_map[key] = Prior N
    ↓
Step 11: Build Clusters
  - Group: Domain + Signal Tier
  - Include: client_id in each cluster
  - Count: N per cluster
  - Lookup: Prior N via composite key
  - Compute: N Delta, Trend Direction, Percentage
  - Aggregate: Pain points, Emotions (JSON breakdowns)
    ↓
Step 12: Split Clusters (array → items)
    ↓
Step 13: Build Bulk POST Body
  - Add: Client ID, Report Window Type to each record
  - Serialize: JSON.stringify(records)
    ↓
Step 14: NocoDB Bulk POST
  - Write: All cluster records to SIA table
```

**No webhooks. No LLM calls. Pure schedule-triggered aggregation.**

---

## Query Examples

### MRA Weekly Brief Query (Monday 7am)

```
GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mdn68l4lm609fve
  ?where=(Client%20ID,eq,PAK-001)~and(Report%20Window%20Type,eq,weekly)
  &sort=-Run%20Timestamp
  &limit=20
```

**Returns:** Latest weekly SIA clusters for PAK-001

**MRA uses this to build:**
- Signal distribution (T-NEG/T-AMB/T-POS percentages)
- Domain breakdown (which domains have highest signal counts)
- Trend signals (Growing/Declining clusters)

### MRA Daily Report Query (Every day 7am)

```
GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mdn68l4lm609fve
  ?where=(Client%20ID,eq,PAK-001)~and(Report%20Window%20Type,eq,daily)
  &sort=-Run%20Timestamp
  &limit=20
```

**Returns:** Latest daily SIA clusters for PAK-001 (last 48 hours)

### MRA Monthly Report Query (1st of month 7am)

```
GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mdn68l4lm609fve
  ?where=(Client%20ID,eq,PAK-001)~and(Report%20Window%20Type,eq,monthly)
  &sort=-Run%20Timestamp
  &limit=20
```

**Returns:** Latest monthly SIA clusters for PAK-001 (last 30 days)

---

## Related Documents

- **HOW Document:** [SCX_SIA_HOW_v4.md](SCX_SIA_HOW_v4.md)
- **Changelog:** [SCX_SIA_CHANGELOG.md](SCX_SIA_CHANGELOG.md)
- **Design Rationale:** [SIA_Design_Rationale.md](SIA_Design_Rationale.md)
- **Upstream Tables:** [../ALA/ALA_Schema.md](../ALA/ALA_Schema.md), [../EIP/EIP_Schema.md](../EIP/EIP_Schema.md)
- **Downstream Consumer:** [../MRA/MRA_Schema.md](../MRA/MRA_Schema.md)

---

## Version History

**v1.0:**
- 12 fields
- Single weekly schedule
- Read from EIP table directly
- Filtered by EIP Timestamp
- Composite key: Domain + Signal Tier

**v2.0 (Chat #77, April 28, 2026):**
- **15 fields** (added Client ID, Report Window Type, explicit field IDs documented)
- **Three schedules** (daily/weekly/monthly)
- **Read from ALA table** (filtered by guest review Date field)
- **Composite key:** Client ID + Domain + Signal Tier (multi-client trend comparison)
- **MRA-compatible query structure** (Client ID + Report Window Type filters)

---

**End of SIA Schema v2.0**
