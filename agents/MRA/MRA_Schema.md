# MRA_Schema.md v1.1

**NocoDB Table ID:** `mybieof2em75t6e`  
**Base ID:** `pq249fix22t3ofv`  
**Updated:** Chat #78 · April 11, 2026

---

## Table Fields (14 Total)

| Field Name | Field ID | Type | Source | Purpose |
|------------|----------|------|--------|---------|
| Id | (auto) | AutoNumber | System | Primary key (NocoDB auto-generated) |
| MRA Run ID | `czljrhljau7a8iv` | SingleLineText | MRA computation | Format: `MRA-YYYYMMDD-[Type]` e.g. `MRA-20260410-Monthly` |
| Report Type | `cyqw1ki969a0vq0` | SingleSelect | Trigger type | Daily-48hr / Weekly / Monthly |
| Client ID | `chp1m03x1v1t3mv` | SingleLineText | MRA hardcoded (Phase 1) | PAK-001 / EDO-001 / AJI-001 |
| Total Records | `cr3fr2dpxdncllz` | Number | RDA published count | Count of published RDA records in period |
| Avg Star Rating | `cohbxugfjq590n1` | Number | RDA aggregation | Average star rating (2 decimal places) from RDA records |
| SLA Rate | `cauxg0fskprxy70` | Number | MRA computation | % approved within 48hr of **RDA Timestamp** (not ALA Timestamp) |
| T Neg | `c7qu39qw9puutdq` | Number | RDA tier count | COUNT where Confirmed Response Tier = T3 (Tier naming: T1/T2/T3 not T-POS/T-AMB/T-NEG) |
| T Amb | `corkoggtozkumne` | Number | RDA tier count | COUNT where Confirmed Response Tier = T2 |
| T Pos | `cg0u8la22vt06lc` | Number | RDA tier count | COUNT where Confirmed Response Tier = T1 |
| Delivery Status | `ci6hytuki6u9i2j` | SingleSelect | Brevo API response | Delivered / Failed |
| Magic Link Token | `can5u62qbnmin1x` | SingleLineText | MRA generation | 64-char hex token (future dashboard auth) |
| Error Log | `c5b2bybjo0equeb` | LongText | Error capture | Workflow failures, query errors |
| Platform Counts JSON | `cixk8vliloo1rao` | LongText | MRA aggregation | JSON object `{"Google":13,"OpenTable":8,"Yelp":1}` |
| CreatedAt | (auto) | DateTime | NocoDB system | Report generation timestamp |

---

## Key Corrections from v1.0

**1. Tier field naming corrected:**
- v1.0 incorrectly labeled: T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE
- v1.1 correct: T Neg / T Amb / T Pos (NocoDB field names)
- **Source:** RDA `Confirmed Response Tier` field values are T1/T2/T3 not T-NEG/T-AMB/T-POS
- **Mapping:** T1=Positive → T Pos field, T2=Ambiguous → T Amb field, T3=Negative → T Neg field

**2. Platform Counts JSON field added:**
- Field ID `cixk8vliloo1rao` confirmed working Chat #77
- Stores JSON string not native JSON type (LongText field)
- Example: `{"Google":13,"OpenTable":8,"Yelp":1}`

**3. SLA Rate formula corrected:**
- v1.0: measured from **ALA date_posted** to RDA approved_at
- v1.1: measures from **RDA Timestamp** to **Published Timestamp**
- Both timestamps exist in RDA table — no ALA join needed
- Logic: hours = (Published Timestamp - RDA Timestamp) / 3600000, pass if ≤ 48

**4. Total Records source corrected:**
- v1.0: counted from ALA table
- v1.1: counts **published RDA records** in period filtered by Client ID + Published Timestamp range
- Uses `pageInfo.totalRows` not `list.length`

**5. Avg Star Rating source corrected:**
- v1.0: queried ALA table directly
- v1.1: reads Star Rating field from **RDA published records** (Star Rating inherited from ALA via pipeline)
- Simpler — single query to RDA table, no ALA join needed

---

## Data Flow (Corrected)

```
Schedule Trigger (daily 6am / Monday 7am / 1st of month 7am)
↓
Step 2: Merge Triggers
↓
Step 3: Determine Report Type + Date Range
↓
Step 4: HTTP GET RDA table
  Filter: (Client ID,eq,PAK-001)~and(Published Timestamp,gte,[start])~and(Published Timestamp,lte,[end])
  Returns: all published RDA records in period
↓
Step 5: Check Record Count
  If pageInfo.totalRows === 0 → return [] and stop
↓
Step 6: Compute Tier Counts (Code Node)
  Count T1/T2/T3 from Confirmed Response Tier field
↓
Step 7: Compute Avg Star Rating (Code Node)
  Average of Star Rating field from RDA records
↓
Step 8: Compute Platform Counts (Code Node)
  Build JSON object from Platform field (inherited from ALA)
  Output: {"Google":N,"OpenTable":M,"Yelp":K}
↓
Step 9: Compute Response Draft Status (Code Node)
  - Avg approval hours: (Published Timestamp - RDA Timestamp) / 3600000
  - SLA rate: % where hours ≤ 48
  - Count by Approval Status
↓
Step 10: Compute Trend Signals (Code Node)
  Placeholder - returns [] in Phase 1
↓
Step 11: Compute SEO Signal (Code Node)
  - seo_keyword_hits: COUNT of drafts containing ≥1 keyword (not total appearances)
  - seo_top_keyword, seo_coverage_rate, seo_response_rate, seo_avg_velocity
↓
Step 12: Write to NocoDB MRA table (HTTP POST)
  All 14 fields including Platform Counts JSON
↓
Step 13: Build Email Body (Code Node)
  Plain text format, governance footer, Platform Inputs section
↓
Step 14: Send via Brevo API (HTTP POST)
  Sender: marellano@solofella.com
  Recipient: marellano@solofella.com
↓
Steps 15-17: Error handler if any step fails
```

**Key difference from v1.0:** Single RDA query replaces separate SIA/RDA/ALA queries. All needed data exists in published RDA records.

---

## Field Computation Details (Corrected)

### SLA Rate (48-Hour Compliance)

**Definition:** % of response drafts approved within 48 hours of **draft creation** (RDA Timestamp)

**Code:**
```javascript
let within_48hr = 0;
for (let i = 0; i < rda_records.length; i++) {
  let rda_ts = new Date(rda_records[i]['RDA Timestamp']).getTime();
  let pub_ts = new Date(rda_records[i]['Published Timestamp']).getTime();
  let hours = (pub_ts - rda_ts) / (1000 * 60 * 60);
  if (hours <= 48) {
    within_48hr = within_48hr + 1;
  }
}
let sla_rate = Math.round((within_48hr / rda_records.length) * 100);
```

**Target:** 90% compliance  
**Current performance:** Verified working April 10 2026 monthly report

---

### Tier Counts (T1/T2/T3 from RDA)

**Code:**
```javascript
let t1_count = 0;
let t2_count = 0;
let t3_count = 0;

for (let i = 0; i < rda_records.length; i++) {
  let tier = rda_records[i]['Confirmed Response Tier'] || '';
  if (tier === 'T1') {
    t1_count = t1_count + 1;
  } else if (tier === 'T2') {
    t2_count = t2_count + 1;
  } else if (tier === 'T3') {
    t3_count = t3_count + 1;
  }
}

// Store as:
// T Pos = t1_count
// T Amb = t2_count  
// T Neg = t3_count
```

**Note:** RDA uses T1/T2/T3 tier labels. MRA NocoDB fields named T Pos/T Amb/T Neg for dashboard readability.

---

### Platform Counts JSON

**Code:**
```javascript
let platform_map = {};
for (let i = 0; i < rda_records.length; i++) {
  let p = rda_records[i]['Platform'] || 'Unknown';
  if (platform_map[p]) {
    platform_map[p] = platform_map[p] + 1;
  } else {
    platform_map[p] = 1;
  }
}
let platform_counts_json = JSON.stringify(platform_map);
```

**Example output:** `{"Google":13,"OpenTable":8,"Yelp":1}`

**Email display:**
```
Platform Inputs:
- Google: 13 reviews
- OpenTable: 8 reviews
- Yelp: 1 review
```

---

### Average Approval Hours (RDA Timestamp → Published Timestamp)

**Code:**
```javascript
let total_hours = 0;
let approved_count = 0;

for (let i = 0; i < rda_records.length; i++) {
  let status = rda_records[i]['Approval Status'] || '';
  if (status === 'Approved' || status === 'Edited-Approved') {
    let rda_ts = new Date(rda_records[i]['RDA Timestamp']).getTime();
    let pub_ts = new Date(rda_records[i]['Published Timestamp']).getTime();
    let hours = (pub_ts - rda_ts) / (1000 * 60 * 60);
    total_hours = total_hours + hours;
    approved_count = approved_count + 1;
  }
}

let avg_approval_hours = approved_count > 0 
  ? Math.round(total_hours / approved_count) 
  : 0;
```

**Used in:** Weekly and Monthly reports  
**Not used in:** Daily 48hr summary

---

### SEO Keyword Hits (Drafts Containing Keywords)

**Definition:** COUNT of published drafts containing ≥1 keyword from Client Config — **not** total keyword appearances

**Code:**
```javascript
let seo_keywords = client_config['SEO Keywords'] || '';
let keywords_arr = seo_keywords.split(',');
let drafts_with_keyword = 0;

for (let i = 0; i < rda_records.length; i++) {
  let draft = (rda_records[i]['Public Response Draft'] || '').toLowerCase();
  let has_keyword = false;
  for (let j = 0; j < keywords_arr.length; j++) {
    let kw = keywords_arr[j].trim().toLowerCase();
    if (kw !== '' && draft.includes(kw)) {
      has_keyword = true;
    }
  }
  if (has_keyword) {
    drafts_with_keyword = drafts_with_keyword + 1;
  }
}
let seo_keyword_hits = drafts_with_keyword;
```

**Example:** 22 published drafts, 18 contain ≥1 keyword → `seo_keyword_hits = 18` (not the total count of all keyword appearances across all drafts)

---

## Email Template Structure (Corrected)

**Governance footer (exact text):**
```
SubtextCX detects and interprets signals only.
Operational decisions remain with your team.
```

**Sender:** `marellano@solofella.com` (verified in Brevo)  
**Recipient:** `marellano@solofella.com` (Phase 1 — from Client Config in Phase 2)

**All non-ASCII characters replaced:**
- Old: `·` and `—`
- New: `-`

---

## Token Budget

**Zero tokens.**

**No AI calls anywhere:**
- No OpenAI
- No Anthropic Claude
- No LLM of any kind

**Operations:**
- HTTP GET NocoDB (read-only)
- JavaScript loops and arithmetic
- String concatenation
- HTTP POST Brevo API

**Cost:** Infrastructure only (NocoDB free, Brevo 300 emails/day free tier)

---

## n8n 2.4.6 Constraints Applied

**All MRA Code Nodes comply:**

1. Loop variables use `let` never `const`
2. No arrow functions with `const` inside
3. No unused variable declarations
4. Use `pageInfo.totalRows` for counts
5. No spread operator — key-by-key only
6. No `for...of` loops
7. Non-ASCII characters replaced with `-`

**Verified compliant:** Chat #77 April 10 2026

---

## Verified Test Results

**First Monthly Report — April 10 2026:**
- 22 published RDA records (13 Google + 8 OpenTable + 1 Yelp)
- Avg star 4.23
- Platform Counts JSON: `{"Google":13,"OpenTable":8,"Yelp":1}`
- All sections populated ✅
- Email delivered ✅

**Second Monthly Report — April 10 2026 (after all fixes):**
- Same 22 records
- All formulas corrected ✅
- Platform Inputs section added to email ✅
- All n8n 2.4.6 constraints applied ✅

---

## Related Documents

- **HOW Document:** [SCX_MRA_HOW_v1.1.md](SCX_MRA_HOW_v1.1.md)
- **Master Continuity Document:** [MCD_v7.4.md](../../docs/MCD_v7.4.md)
- **Schema Registry:** [Schema_Registry_v2.md](../../docs/Schema_Registry_v2.md)
- **Dashboard Freelancer Brief:** [Dashboard_Freelancer_Brief.pdf](../../commercial/Dashboard_Freelancer_Brief.pdf)

---

**End of MRA_Schema.md v1.1**  
**Updated:** Chat #78 · April 11, 2026
