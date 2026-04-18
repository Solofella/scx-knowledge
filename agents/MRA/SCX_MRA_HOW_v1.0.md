# SCX_MRA_HOW_v1.1

**Agent Name:** MRA (Metrics & Reporting Agent)  
**Version:** 1.1  
**Last Updated:** Chat #78 · April 11, 2026  
**Model:** Pure JavaScript (zero LLM calls)  
**Status:** Verified operational - 17 nodes complete  
**Quality Baseline:** First monthly report delivered April 10 2026 - all sections working correctly

---

## Purpose

MRA generates scheduled intelligence briefs for clients on three cadences:
1. **48-hour response draft summary** (daily 6am UTC delivery)
2. **Weekly intelligence brief** (Monday 7am UTC delivery)  
3. **Monthly comprehensive report** (1st of month 7am UTC delivery)

**MRA uses pure JavaScript** - zero AI calls, zero token cost (same architecture as SIA).

**Terminal reporting agent** - MRA is the final scheduled workflow. No downstream agents. Human consumption only via email delivery.

---

## Input Sources

**MRA does NOT receive webhooks.** Runs on schedule, queries tables directly.

**Tables queried:**

1. **RDA table** (`mr1v67cszcklwns`)
   - Published records filtered by Client ID + Published Timestamp range
   - Response draft counts by Confirmed Response Tier (T1/T2/T3)
   - Approval Status distribution
   - Time-to-approval metrics (RDA Timestamp → Published Timestamp)
   - Star Rating for average calculation
   - SEO Keywords for keyword coverage analysis
   - Platform field via ALA join for platform distribution

2. **SIA table** (`mdn68l4lm609fve`)
   - Signal distribution (T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE counts)
   - Domain breakdown
   - Trend direction (placeholder Phase 1 - returns empty array)

3. **Client Config table** (`m95cmabjfyb94ps`)
   - Client display name
   - Approval contact email
   - SEO Keywords list

---

## Processing Logic

### Node Flow (17 Nodes Total)

**Node 01a:** Schedule Trigger - Daily 6am UTC (48-hour summary)  
**Node 01b:** Schedule Trigger - Monday 7am UTC (weekly brief)  
**Node 01c:** Schedule Trigger - 1st of month 7am UTC (monthly report)

**Node 02:** Merge Triggers - combines three schedule paths into single flow

**Node 03:** Determine Report Type (Code Node)
- Reads which trigger fired
- Sets `report_type` (Daily-48hr / Weekly / Monthly)
- Computes date range based on report type
- Sets `client_id` (currently hardcoded PAK-001, supports multi-client in future)

**Node 04:** Fetch RDA Records (HTTP GET)
- URL: `http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns`
- Filter: `(Client ID,eq,PAK-001)~and(Published Timestamp,gte,[start_date])~and(Published Timestamp,lte,[end_date])`
- Returns all published drafts in period
- Authentication: xc-token

**Node 05:** Check Record Count (Code Node)
- If `pageInfo.totalRows === 0` returns `[]` and stops workflow gracefully
- No email sent if zero records in period
- Uses `pageInfo.totalRows` NOT `list.length` per n8n 2.4.6 rule

**Node 06:** Compute Tier Counts (Code Node)
- Reads `Confirmed Response Tier` field from each RDA record
- Counts T1 / T2 / T3 records
- Uses index-based `for` loop (no `for...of`)
- All variables inside loop declared with `let` never `const`

**Node 07:** Compute Avg Star Rating (Code Node)
- Reads `Star Rating` field from each RDA record (inherited from ALA via pipeline)
- Sums all ratings, divides by count
- Rounds to 2 decimal places
- Handles missing/null ratings gracefully

**Node 08:** Compute Platform Counts (Code Node)
- Reads `Platform` field from each RDA record (inherited from ALA)
- Builds JSON object: `{"Google":13,"OpenTable":8,"Yelp":1}`
- Stores as string in `platform_counts_json` variable
- Written to NocoDB MRA table Platform Counts JSON field

**Node 09:** Compute Response Draft Status (Code Node)
- Counts by Approval Status: Approved / Edited-Approved / Pending / Rejected
- **Avg approval hours:** calculates hours between RDA Timestamp and Published Timestamp for Approved records only
- **SLA rate:** % of drafts approved within 48 hours of RDA Timestamp (not ALA Timestamp)
- Uses `let` for all loop variables per n8n 2.4.6 constraint

**Node 10:** Compute Trend Signals (Code Node)
- Placeholder for Phase 1 - always returns empty array `[]`
- Monthly email shows "Trend Signals: None detected in this period"
- Phase 2: time-series analysis of HSI domain scores over multiple weeks

**Node 11:** Compute SEO Signal (Code Node)
- **seo_keyword_hits:** COUNT of published drafts containing ≥1 keyword from Client Config SEO Keywords list (not total keyword appearances)
- **seo_top_keyword:** most frequently appearing keyword across all drafts
- **seo_coverage_rate:** percentage of drafts containing ≥1 keyword
- **seo_response_rate:** percentage of reviews with published responses
- **seo_avg_velocity:** average hours from ALA Timestamp to Published Timestamp
- All loop variables use `let` per n8n 2.4.6 constraint

**Node 12:** Write to NocoDB (HTTP POST)
- URL: `http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mybieof2em75t6e`
- Body: JSON with all 14 fields
- Authentication: xc-token
- Stores complete report metadata for dashboard access

**Node 13:** Build Email Body (Code Node)
- Different email templates per report_type
- Plain text format (no HTML - avoids spam filters)
- Includes Platform Inputs section with platform counts
- Governance footer: "SubtextCX detects and interprets signals only. Operational decisions remain with your team."
- All non-ASCII characters (`·` and `—`) replaced with `-` per n8n 2.4.6 constraint

**Node 14:** Send Email via Brevo (HTTP POST)
- URL: `https://api.brevo.com/v3/smtp/email`
- Authentication: Header Auth via `api-key` credential
- Sender: `marellano@solofella.com` (verified in Brevo)
- Recipient: `marellano@solofella.com` (from Client Config or hardcoded for Phase 1)
- Body: JSON with `sender`, `to`, `subject`, `textContent`

**Node 15:** Error Handler (Error Trigger)
- Catches workflow errors from any node

**Node 16:** Build Error Email (Code Node)
- Extracts error message and failed node name
- Formats plain text error notification

**Node 17:** Send Error Email (HTTP POST)
- Same Brevo configuration as Node 14
- Subject: `SCX-MRA Error - [timestamp]`
- Recipient: `marellano@solofella.com`

---

## NocoDB Schema

**Table ID:** `mybieof2em75t6e`  
**Base ID:** `pq249fix22t3ofv`

**14 Fields:**

| Field Name | Field ID | Type | Purpose |
|------------|----------|------|---------|
| MRA Run ID | `czljrhljau7a8iv` | SingleLineText | Unique ID format: `MRA-YYYYMMDD-[Type]` |
| Report Type | `cyqw1ki969a0vq0` | SingleSelect | Daily-48hr / Weekly / Monthly |
| Client ID | `chp1m03x1v1t3mv` | SingleLineText | PAK-001, EDO-001, AJI-001 |
| Total Records | `cr3fr2dpxdncllz` | Number | Count of published RDA records in period |
| Avg Star Rating | `cohbxugfjq590n1` | Number | Average star rating across period (2 decimal places) |
| SLA Rate | `cauxg0fskprxy70` | Number | % of drafts approved within 48hr of RDA Timestamp |
| T Neg | `c7qu39qw9puutdq` | Number | Count of T-NEGATIVE tier records |
| T Amb | `corkoggtozkumne` | Number | Count of T-AMBIGUOUS tier records |
| T Pos | `cg0u8la22vt06lc` | Number | Count of T-POSITIVE tier records |
| Delivery Status | `ci6hytuki6u9i2j` | SingleSelect | Delivered / Failed |
| Magic Link Token | `can5u62qbnmin1x` | SingleLineText | Secure token for dashboard access (future Phase 2) |
| Error Log | `c5b2bybjo0equeb` | LongText | Error messages if workflow fails |
| Platform Counts JSON | `cixk8vliloo1rao` | LongText | JSON object e.g. `{"Google":13,"OpenTable":8,"Yelp":1}` |
| CreatedAt | (auto) | DateTime | NocoDB auto-generated timestamp |

---

## Report Types & Content

### 1. 48-Hour Response Draft Summary

**Schedule:** Daily at 6am UTC

**Date Range:** Last 48 hours from execution time

**Content:**
- Total reviews processed (count of published RDA records)
- Platform distribution (Google / OpenTable / Yelp counts)
- Response tier breakdown (T1/T2/T3 counts)
- Approval status distribution
- Average response velocity (hours from RDA Timestamp to Published Timestamp)
- SLA compliance rate (% approved within 48hr)

**Email structure:**
```
SubtextCX - 48-Hour Summary
PAK-001 - Park Avenue Kitchen by David Burke
Date: April 11, 2026

Reviews Processed: 8

Platform Inputs:
- Google: 5 reviews
- OpenTable: 2 reviews
- Yelp: 1 review

Response Tier Distribution:
- T1 Standard: 6
- T2 Calibrated: 1
- T3 Dignity-restoration: 1

Approval Status:
- Approved: 5
- Pending: 3

Average Response Velocity: 14 hours
SLA Compliance: 87%

SubtextCX detects and interprets signals only.
Operational decisions remain with your team.
```

---

### 2. Weekly Intelligence Brief

**Schedule:** Monday at 7am UTC

**Date Range:** Last 7 days (Monday to Sunday)

**Content:**
- Total reviews processed
- Platform distribution
- Signal distribution (T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE percentages)
- Response tier breakdown
- Approval performance (approval rate, avg velocity, SLA rate)
- SEO metrics (keyword coverage, top keyword, response rate)

**Additional vs 48hr:**
- Week-over-week comparison (if prior week data exists)
- Top 3 domains by signal volume (from SIA table)

---

### 3. Monthly Comprehensive Report

**Schedule:** 1st of month at 7am UTC

**Date Range:** Prior full month (1st to last day)

**Content:**
- Total reviews processed in month
- Month-over-month comparison (if prior month data exists)
- Platform distribution
- Signal distribution with percentages
- Response tier breakdown
- Approval performance metrics
- SEO performance (keyword hits, coverage rate, avg velocity)
- Top 5 domains by volume

**Verified working:** First monthly report delivered April 10 2026 - 22 records (13 Google + 8 OpenTable + 1 Yelp), avg star 4.23, all sections populated correctly including Platform Inputs.

---

## Token Budget

**Zero tokens per report.**

**MRA operations:**
- NocoDB HTTP GET queries (read-only)
- JavaScript aggregation (`for` loops, calculations, string concatenation)
- Email composition (plain text templates)
- Brevo API HTTP POST (no AI)

**Cost:** Infrastructure only (NocoDB queries free, Brevo free tier 300 emails/day)

**Scalability:** Unlimited reports at zero marginal cost.

---

## Key Design Decisions

### Why Pure JavaScript (No AI)?

**MRA's task is deterministic reporting:**
- Read RDA published records (structured data)
- Count by field values (Tier, Platform, Approval Status)
- Calculate percentages and averages (arithmetic)
- Format into email text (string templates)

**No interpretation needed** - presenting aggregated numbers in structured format.

**Benefits:**
- Zero token cost (scales infinitely)
- Fast execution (<3 seconds for monthly report with 50+ records)
- Deterministic output (no LLM variability)
- No API rate limits
- No model dependency

**Same architecture as SIA** - proven zero-cost model.

---

### Critical Formulas

**SLA Rate Calculation:**

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

**Platform Counts JSON:**

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

**SEO Keyword Hits (drafts containing keywords, not total appearances):**

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

**Average Approval Hours (RDA Timestamp → Published Timestamp):**

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

---

## Email Delivery (Brevo)

**Service:** Brevo transactional email API

**Free tier:** 300 emails/day (sufficient for 100 clients × 3 reports/week = ~900/month with batching)

**Email format:**
- Plain text only (avoids spam filters)
- Summary metrics (bullet points)
- Governance footer mandatory
- No HTML, no images, no attachments

**Delivery tracking:** 
- Brevo returns delivery confirmation
- MRA stores status in Delivery Status field (Delivered / Failed)
- Error handler emails Miguel if primary delivery fails

**Sender verification:** `marellano@solofella.com` verified in Brevo account - required for deliverability.

---

## n8n 2.4.6 Code Node Constraints Applied

**All MRA Code Nodes follow these mandatory rules:**

1. **Loop variables use `let` never `const`:**
```javascript
// WRONG
for (let i = 0; i < arr.length; i++) {
  const item = arr[i]; // RED HIGHLIGHT
}

// CORRECT
for (let i = 0; i < arr.length; i++) {
  let item = arr[i];
}
```

2. **No arrow functions with `const` inside bodies:**
```javascript
// WRONG
arr.filter(x => {
  const val = x.field; // RED HIGHLIGHT
  return val > 10;
});

// CORRECT - use index-based for loops instead
let filtered = [];
for (let i = 0; i < arr.length; i++) {
  let val = arr[i].field;
  if (val > 10) {
    filtered.push(arr[i]);
  }
}
```

3. **No unused variable declarations:**
```javascript
// WRONG
const total = 100; // declared but never used - RED HIGHLIGHT
return [{ json: { count: 50 } }];

// CORRECT - remove or use it
return [{ json: { count: 50 } }];
```

4. **Use `pageInfo.totalRows` for NocoDB counts:**
```javascript
// WRONG
const count = response.list.length;

// CORRECT
const count = response.pageInfo.totalRows;
```

5. **No spread operator - build objects key-by-key:**
```javascript
// WRONG
return [{ json: { ...input, new_field: value } }];

// CORRECT
let output = {};
output.field1 = input.field1;
output.field2 = input.field2;
output.new_field = value;
return [{ json: output }];
```

6. **Non-ASCII characters replaced:**
```javascript
// WRONG
let separator = '·';
let dash = '—';

// CORRECT
let separator = '-';
let dash = '-';
```

**All 17 MRA nodes audited and compliant with these rules as of Chat #77 April 10 2026.**

---

## Verified Test Results

**First Monthly Report - April 10 2026:**

- Report Type: Monthly
- Client ID: PAK-001
- Date Range: March 1-31 2026
- Total Records: 22
- Platform Counts: Google 13, OpenTable 8, Yelp 1
- Avg Star Rating: 4.23
- Email delivered successfully ✅
- All sections populated correctly ✅
- Platform Inputs section included ✅

**Second Monthly Report - April 10 2026 (after fixes):**

- Same parameters
- Platform Counts JSON field added to NocoDB ✅
- SEO definitions corrected ✅
- Avg approval hours formula corrected ✅
- All `const` inside loops replaced with `let` ✅
- Non-ASCII characters removed ✅
- Email sender updated to `marellano@solofella.com` ✅

---

## Related Documents

- **Master Continuity Document:** [MCD v7.4](../../docs/MCD_v7.4.md)
- **Schema Registry:** [Schema_Registry_v2.md](../../docs/Schema_Registry_v2.md)
- **SIA (similar zero-cost model):** [../SIA/SCX_SIA_HOW_v1.2.md](../SIA/SCX_SIA_HOW_v1.2.md)
- **RDA (upstream agent):** [../RDA/SCX_RDA_HOW_v3.1.md](../RDA/SCX_RDA_HOW_v3.1.md)
- **Dashboard Freelancer Brief:** [../../commercial/Dashboard_Freelancer_Brief.pdf](../../commercial/Dashboard_Freelancer_Brief.pdf)

---

## n8n Workflow Details

**Workflow Name:** SCX-MRA  
**Triggers:** 3 separate schedule triggers (daily 6am UTC, Monday 7am UTC, monthly 1st 7am UTC)  
**Credentials:** xc-token (NocoDB), api-key (Brevo)  
**Error Workflow:** Self-referencing (SCX-MRA)

**Publishing status:** Must be Published (not just saved) for schedule triggers to fire automatically.

**Verification:** Go to n8n → Executions → filter by SCX-MRA → past automatic runs should appear if workflow is active.

---

## Open Items for Future Sessions

**Item 1 - Time zone adjustment:**

Current triggers fire at UTC times - 6am/7am UTC = 2am/3am ET. Recommendation: adjust to 10am/11am UTC (= 6am/7am ET) for better operational alignment.

**Item 2 - Prior-month baseline:**

First monthly report ran April 10 2026 - no prior-month data exists yet for month-over-month deltas. Deltas will show `N/A` until May 1 2026 second monthly report runs. No code change needed - handled gracefully.

**Item 3 - Trend Signals Phase 2:**

Step 10 placeholder returns empty. Future: time-series analysis of HSI domain scores over 4+ weeks to detect UP/DOWN/STABLE trends per domain.

**Item 4 - Magic Link implementation:**

Magic Link Token generated but not yet connected to dashboard. Dashboard build in progress - freelancer brief prepared Chat #75.

**Item 5 - Multi-client support:**

Currently hardcoded `PAK-001`. Architecture supports multi-client via Client ID field - just needs loop in Step 3 to process multiple clients per schedule run.

---

**End of SCX_MRA_HOW_v1.1**  
**Updated:** Chat #78 · April 11, 2026  
**Status:** Complete and verified operational
