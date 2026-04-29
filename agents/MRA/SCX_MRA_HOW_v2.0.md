# SCX_MRA_HOW_v2.0

**Agent Name:** MRA (Metrics & Reporting Agent)  
**Version:** 2.0  
**Last Updated:** Chat #80 · April 28, 2026  
**Model:** Pure JavaScript (zero LLM calls)  
**Status:** Production-ready - 23 nodes complete + 3 error handler nodes

---

## Purpose

MRA generates scheduled intelligence reports for clients on three cadences:
1. **48-hour response draft summary** (daily 7am UTC delivery)
2. **Weekly intelligence brief** (Monday 7am UTC delivery)  
3. **Monthly comprehensive report** (1st of month 7am UTC delivery)

**MRA uses pure JavaScript** - zero AI calls, zero token cost. All computation is deterministic aggregation and formatting.

**CRITICAL ARCHITECTURE PRINCIPLE:** Reports filter by **guest review posting date (ALA Date field)**, NOT by pipeline processing date (RDA Timestamp). This ensures reports reflect when guests actually posted reviews, not when SubtextCX processed them.

---

## Input Sources

**MRA does NOT receive webhooks.** Runs on n8n schedule triggers, queries NocoDB tables directly.

**Tables queried:**

1. **ALA table** (`m57efwbtrvwohhr`)
   - **Date field** (M/D/YY format) — PRIMARY date filter for all reports
   - Platform (Google, OpenTable, Yelp, TripAdvisor)
   - Star Rating
   - Client ID

2. **RDA table** (`mr1v67cszcklwns`)
   - ALA Record ID (FK to ALA for date matching)
   - Confirmed Response Tier (T1/T2/T3)
   - Approval Status (Pending/Pending-Elevated/Approved/Edited-Approved/Published)
   - RDA Timestamp (draft generation time)
   - Published Timestamp (approval time — for SLA calculation)
   - Public Response Draft (for SEO keyword detection)

3. **SIA table** (`mdn68l4lm609fve`)
   - Report Window Type (daily/weekly/monthly) — NEW field added Chat #80
   - Domain (pain point category)
   - Signal Tier (T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE)
   - N (signal count)
   - Prior N (prior period count for trend direction)
   - Trend Direction (UP/DOWN/STABLE/null for new clusters)
   - Is Singleton (exclude from trend signals)

4. **Client Config table** (`m95cmabjfyb94ps`)
   - Client ID
   - Client Name (display name for email)
   - Approval Contact Email (recipient address)
   - SEO Keywords (array for SEO performance tracking)

---

## Architecture Overview — 23 Nodes

**Three Schedule Triggers (Node 1a/1b/1c):**
- **Daily:** Every day 7am UTC → report_type = 'daily'
- **Weekly:** Every Monday 7am UTC → report_type = 'weekly'
- **Monthly:** 1st of month 7am UTC → report_type = 'monthly'

**Processing Flow:**

**Steps 1-5:** Session initialization
- Merge trigger inputs
- Set report_type, period window (48hr/7day/30day)
- Calculate period_start and period_end timestamps
- Generate MRA Run ID and magic link token

**Steps 6-8:** Fetch data sources
- Step 6: Fetch ALL ALA records for client (no date filter in URL)
- Step 7a: Build RDA query URL
- Step 7b: Fetch ALL RDA records for client (no date filter in URL)

**Step 9:** **CRITICAL DATE FILTERING NODE**
- Builds ALA Date lookup map (parses M/D/YY → timestamp)
- Filters RDA by matching ALA Record ID
- Checks if ALA Date falls within period window
- Calculates platform_counts from matched ALA records
- Calculates avg_star_rating from matched ALA records
- Calculates tier_breakdown from matched RDA records
- Calculates all_time_pending from ALL RDA records (not just period)

**Steps 10a-10b:** Fetch SIA data
- Step 10a: Build dynamic SIA query URL using report_type
- Step 10b: Fetch SIA clusters where Report Window Type = report_type

**Step 11:** Process SIA + SEO
- Count signal distribution (t_negative_count, t_ambiguous_count, t_positive_count)
- Aggregate domain totals
- Build trend_signals array (exclude singletons)
- Calculate SEO keyword hits in published drafts

**Step 12:** Generate magic link token + expiration

**Step 13:** Skip NocoDB write for daily reports

**Step 14:** IF check — daily reports bypass NocoDB write

**Step 15:** NocoDB POST (weekly/monthly only)

**Step 16:** Build Brevo email payload

**Step 17:** Build HTML email body (branded design)

**Step 18:** PATCH NocoDB delivery status

**Step 19:** Send email via Brevo API

**Error Handler:** 3 nodes (trigger, log, email notification)

---

## Date Filtering Architecture (Chat #78-80 Fix)

**Problem discovered Chat #78:**
MRA was filtering by RDA Timestamp (pipeline processing date) instead of ALA Date (guest review posting date). This caused incorrect metrics when reviews were processed days after posting.

**Solution (Chat #80):**

**Step 6:** Fetch ALL ALA records (no date filter)
```javascript
const url = 'http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr?where=(Client%20ID,eq,PAK-001)&limit=1000';
```

**Step 9:** Filter RDA by ALA Date matching
```javascript
// Build ALA Date lookup map
const alaDateMap = {};
for (let i = 0; i < alaRecords.length; i++) {
  let alaId = String(alaRecords[i]['Id']);
  let alaDateRaw = alaRecords[i]['Date'] || '';  // M/D/YY format
  
  // Parse M/D/YY to timestamp
  let parts = alaDateRaw.split('/');
  let month = parseInt(parts[0]);
  let day = parseInt(parts[1]);
  let year = 2000 + parseInt(parts[2]);
  let alaTs = new Date(year, month - 1, day).getTime();
  
  alaDateMap[alaId] = { ala_record: alaRecords[i], ala_timestamp: alaTs };
}

// Filter RDA by ALA Date
for (let i = 0; i < allRda.length; i++) {
  let alaId = String(rec['ALA Record ID'] || '');
  
  if (alaDateMap[alaId]) {
    let alaTs = alaDateMap[alaId].ala_timestamp;
    if (alaTs >= periodStartTs && alaTs < periodEndTs) {
      filteredRda.push(rec);
      // Build platform_counts from matched ALA record
    }
  }
}
```

**Key insight:** Guest review date (ALA Date) is the authoritative timestamp for all MRA reports, not pipeline processing time.

---

## SIA Integration (Chat #80 Fix)

**SIA workflow updated to support 3 window types:**
- Daily: 2-day window
- Weekly: 7-day window
- Monthly: 30-day window

**New SIA field:** Report Window Type (SingleSelect: daily/weekly/monthly)

**MRA Step 10a builds dynamic query:**
```javascript
const report_type = s9.report_type || 'weekly';
const url = 'http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mdn68l4lm609fve?where=(Report%20Window%20Type,eq,' + report_type + ')&limit=1000&sort=-Run%20Timestamp';
```

**Result:** MRA weekly reports query SIA clusters with 7-day window, monthly reports query 30-day window. No more stale data mismatch.

---

## all_time_pending Field (Chat #80 Addition)

**Purpose:** Daily reports show total pending drafts across all time, not just last 48 hours.

**Calculated in Step 9:**
```javascript
let all_time_pending = 0;
for (let i = 0; i < allRda.length; i++) {
  let status = (allRda[i]['Approval Status'] || '').trim();
  if (status === 'Pending' || status === 'Pending-Elevated') {
    all_time_pending++;
  }
}
```

**Passed through Steps 9 → 10a → 11 → 12 → 13 → 16 → 17**

**Displayed in daily report:**
```
📋 Total Pending Approval (All Time):
  Awaiting your review: 57 draft(s)
```

---

## Email Design (Chat #80 HTML Update)

**Format:** HTML email with inline CSS (email client compatible)

**SubtextCX Brand Colors:**
- **#002D72** (Navy Blue) — Header gradient, key metrics
- **#475a28** (Olive Green) — Header gradient, positive signals, domain borders
- **#ae4900** (Burnt Orange) — Alert boxes, negative signals, CTA button
- **#F6F6F6** (Light Gray) — Card backgrounds, platform items
- **#A8A9AD** (Medium Gray) — Labels, muted text
- **#111111** (Near Black) — Primary text

**Email Structure:**
1. **Header:** Navy-to-Olive gradient with report title
2. **Metric Cards:** Light gray background with navy left border
3. **Alert Boxes:** Light orange background with burnt orange border (T3 warnings)
4. **Platform Grid:** 2-column layout with gray cards
5. **Domain Items:** Olive green left border
6. **Trend Badges:** Color-coded by tier (red/orange/green)
7. **CTA Button:** Burnt orange background (#ae4900), light gray text (#F6F6F6)
8. **Footer:** Gray background with governance note

**Brevo API change:**
- Old: `textContent: email_body`
- New: `htmlContent: email_body`

---

## Report Types & Content

### 1. 48-Hour Response Draft Summary (Daily)

**Schedule:** Every day 7am UTC

**Content:**
- New drafts generated in last 48 hours (by tier)
- Platform breakdown for last 48 hours
- **Total pending approval (all time)** — not just 48hr window
- T3 alert if any dignity-risk drafts present
- Link to Google Sheet approval interface

**Example HTML email:**
```
Header: 48-Hour Draft Summary
PAK-001 • 2026-04-28

📊 New Drafts Generated
  T1: 5 draft(s)
  T2: 2 draft(s)
  T3: 1 draft(s)
  Total: 8

⚠️ 1 draft(s) require elevated attention (T3 — dignity-risk signals)

📈 Platform Breakdown (Last 48 Hours)
  Google: 3
  OpenTable: 4
  Yelp: 1
  TripAdvisor: 0

📋 Pending Approval
  57 draft(s) awaiting your review (all time)

[View & Approve Drafts →]
```

---

### 2. Weekly Intelligence Brief

**Schedule:** Monday 7am UTC

**Content:**
- Signal distribution (T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE counts)
- Top 5 pain point domains
- Platform breakdown
- Performance metrics (avg star rating, SLA compliance rate)
- Trend signals (non-singleton clusters with UP/DOWN/STABLE)
- SEO performance (keyword hits, top keyword, coverage rate, response velocity)
- Dashboard link with magic token

**Example HTML email:**
```
Header: Weekly Intelligence Brief
PAK-001 • 2026-04-21 – 2026-04-28

📊 Signal Distribution (Past 7 Days)
  T-NEGATIVE: 1 signal(s)
  T-AMBIGUOUS: 0 signal(s)
  T-POSITIVE: 1 signal(s)
  Total signals: 2

🔍 Top Pain Point Domains
  Service Quality & Delivery: 2 signals
  Physical Environment & Ambiance: 1 signal

📈 Platform Breakdown
  Google: 1  |  OpenTable: 2
  Yelp: 0    |  TripAdvisor: 0

📊 Performance Metrics
  Average Star Rating: 3.67 ⭐
  SLA Compliance Rate: 0%

🔍 Trend Signals
  [T-POSITIVE] Service Quality & Delivery — 2 signals (new cluster)

📊 SEO Performance
  Drafts with keyword placed: 0
  Top keyword: none
  Coverage rate: 0%
  Response velocity: 0 hrs avg

[View Dashboard →]
```

---

### 3. Monthly Pattern Report

**Schedule:** 1st of month 7am UTC

**Content:**
- 30-day signal summary (total reviews, avg rating, tier percentages)
- Top 5 pain point domains (ranked)
- T3 dignity-risk events count
- Platform performance
- Response metrics (total drafts, SLA compliance, avg approval time)
- Dashboard link with magic token

**Example HTML email:**
```
Header: Monthly Pattern Report
PAK-001 • April 2026

📊 30-Day Signal Summary
  Total reviews processed: 15
  Average star rating: 4.2 ⭐
  T-NEGATIVE: 3 (20%)
  T-AMBIGUOUS: 2 (13%)
  T-POSITIVE: 10 (67%)

🔍 Top 5 Pain Point Domains
  1. Service Quality & Delivery: 8 signals
  2. Food & Beverage Quality: 5 signals
  3. Physical Environment: 2 signals

⚠️ 2 dignity-risk event(s) this month

📈 Platform Performance
  Google: 8   |  OpenTable: 5
  Yelp: 2     |  TripAdvisor: 0

📊 Response Metrics
  Total drafts generated: 15
  Approved within 48hr SLA: 73%
  Average approval time: 18 hours

[View Dashboard →]
```

---

## NocoDB Schema

**Table ID:** `mybieof2em75t6e`

**20 Fields:**

| Field Name | Field ID | Type | Purpose |
|------------|----------|------|---------|
| Id | (auto) | AutoNumber | Primary key |
| MRA Run ID | `czljrhljau7a8iv` | SingleLineText | MRA-YYYYMMDD-HHMMSS |
| Report Type | `cyqw1ki969a0vq0` | SingleSelect | daily / weekly / monthly |
| Client ID | `chp1m03x1v1t3mv` | SingleLineText | PAK-001, EDO-001, etc. |
| Period Start | (TBD) | DateTime | ISO 8601 UTC |
| Period End | (TBD) | DateTime | ISO 8601 UTC |
| Total Records | `cr3fr2dpxdncllz` | Number | Reviews in period |
| Avg Star Rating | `cohbxugfjq590n1` | Number | Average rating |
| SLA Compliance Rate | `cauxg0fskprxy70` | Number | % approved within 48hr |
| T-Negative Count | `c7qu39qw9puutdq` | Number | Negative signal clusters |
| T-Ambiguous Count | `corkoggtozkumne` | Number | Ambiguous signal clusters |
| T-Positive Count | `cg0u8la22vt06lc` | Number | Positive signal clusters |
| Top Domains JSON | (TBD) | JSON | Domain breakdown array |
| Tier Breakdown JSON | (TBD) | JSON | T1/T2/T3 counts + approval metrics |
| Delivery Status | `ci6hytuki6u9i2j` | SingleSelect | sent / failed / pending |
| Magic Link Token | `can5u62qbnmin1x` | SingleLineText | Secure dashboard access token |
| Error Log | `c5b2bybjo0equeb` | LongText | Delivery failures, data errors |
| Platform Counts JSON | `cixk8vliloo1rao` | JSON | Review counts by platform |
| Run Timestamp | (TBD) | DateTime | Report generation time |
| Delivery Timestamp | (TBD) | DateTime | Email sent time |

**Daily reports skip NocoDB write** (Step 13 sets skip_nocodb: true, Step 14 returns empty array).

---

## Step-by-Step Node Decomposition

### Step 1a/1b/1c — Schedule Triggers

Three separate cron triggers merge into Step 2:
- **1a:** `0 7 * * *` (daily 7am UTC) → report_type = 'daily'
- **1b:** `0 7 * * 1` (Monday 7am UTC) → report_type = 'weekly'
- **1c:** `0 7 1 * *` (1st of month 7am UTC) → report_type = 'monthly'

### Step 2 — Merge Triggers

Combines all 3 trigger outputs into single execution path.

### Step 3 — Set Report Type & Window

```javascript
const report_type = trigger_data.report_type || 'daily';
const window_days = report_type === 'daily' ? 2 : (report_type === 'weekly' ? 7 : 30);

const now = new Date();
const period_end = now.toISOString();

const start = new Date(now);
start.setDate(start.getDate() - window_days);
const period_start = start.toISOString();
```

### Step 4 — Get Client List

Fetches all active clients from Client Config table. (Phase 1: hardcoded to PAK-001 only)

### Step 5 — Generate MRA Run ID + Magic Token

```javascript
const d = now.toISOString().slice(0,10).replace(/-/g,'');
const t = now.toISOString().slice(11,19).replace(/:/g,'');
const mra_run_id = 'MRA-' + d + '-' + t;

const crypto = require('crypto');
const magic_token = crypto.randomBytes(16).toString('hex');
```

### Step 6 — Fetch ALL ALA Records

**URL:** `http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr?where=(Client%20ID,eq,PAK-001)&limit=1000`

**No date filter in URL** — filtering happens in Step 9.

### Step 7a — Build RDA Query URL

Simple pass-through for now (Phase 1).

### Step 7b — Fetch ALL RDA Records

**URL:** `http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns?where=(Client%20ID,eq,PAK-001)&limit=1000`

**No date filter in URL** — filtering happens in Step 9.

### Step 9 — **CRITICAL DATE FILTERING + AGGREGATION**

**Input:**
- ALA records (all)
- RDA records (all)
- period_start, period_end

**Process:**

1. Build ALA Date lookup map
```javascript
const alaDateMap = {};
for (let i = 0; i < alaRecords.length; i++) {
  let alaId = String(alaRecords[i]['Id']);
  let alaDateRaw = alaRecords[i]['Date'] || '';
  
  let parts = alaDateRaw.split('/');
  let month = parseInt(parts[0]);
  let day = parseInt(parts[1]);
  let year = 2000 + parseInt(parts[2]);
  let alaTs = new Date(year, month - 1, day).getTime();
  
  alaDateMap[alaId] = { ala_record: alaRecords[i], ala_timestamp: alaTs };
}
```

2. Filter RDA by ALA Date + build metrics
```javascript
const filteredRda = [];
const platform_counts = {};
let star_sum = 0;

for (let i = 0; i < allRda.length; i++) {
  let rec = allRda[i];
  let alaId = String(rec['ALA Record ID'] || '');
  
  if (alaDateMap[alaId]) {
    let alaTs = alaDateMap[alaId].ala_timestamp;
    
    if (alaTs >= periodStartTs && alaTs < periodEndTs) {
      filteredRda.push(rec);
      
      let alaRec = alaDateMap[alaId].ala_record;
      let plat = alaRec['Platform'] || 'Unknown';
      platform_counts[plat] = (platform_counts[plat] || 0) + 1;
      star_sum += parseFloat(alaRec['Star Rating'] || 0);
    }
  }
}

const total_rda = filteredRda.length;
const avg_star_rating = total_rda > 0 ? Math.round((star_sum / total_rda) * 100) / 100 : 0;
```

3. Calculate tier breakdown + SLA
```javascript
const tier_map = { T1: {}, T2: {}, T3: {} };

for (let i = 0; i < filteredRda.length; i++) {
  let rec = filteredRda[i];
  let tier = extractTier(rec['Confirmed Response Tier']);
  let status = rec['Approval Status'];
  
  tier_map[tier].total++;
  if (status === 'Approved' || status === 'Edited-Approved') tier_map[tier].approved++;
  if (status === 'Pending' || status === 'Pending-Elevated') tier_map[tier].pending++;
  
  // Velocity calculation for SLA
  if (status === 'Published') {
    let hrs = (publishedTs - rdaTs) / 3600000;
    if (hrs <= 48) sla_compliant++;
  }
}
```

4. **Calculate all_time_pending**
```javascript
let all_time_pending = 0;
for (let i = 0; i < allRda.length; i++) {
  let status = (allRda[i]['Approval Status'] || '').trim();
  if (status === 'Pending' || status === 'Pending-Elevated') {
    all_time_pending++;
  }
}
```

**Output:** total_records, avg_star_rating, platform_counts, tier_breakdown, sla_compliance_rate, all_time_pending

### Step 10a — Build SIA Query URL

```javascript
const report_type = s9.report_type || 'weekly';
const url = 'http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mdn68l4lm609fve?where=(Report%20Window%20Type,eq,' + report_type + ')&limit=1000&sort=-Run%20Timestamp';
```

**Passes all_time_pending forward.**

### Step 10b — Fetch SIA Clusters

HTTP GET using Step 10a URL.

### Step 11 — Process SIA + SEO

**Input:** SIA clusters from Step 10b

**Process:**

1. Filter to latest run
```javascript
const latestRunTs = allClusters[0]['Run Timestamp'];
const clusters = allClusters.filter(c => c['Run Timestamp'] === latestRunTs);
```

2. Count signal distribution
```javascript
let t_negative_count = 0;
let t_ambiguous_count = 0;
let t_positive_count = 0;

for (let i = 0; i < clusters.length; i++) {
  let tier = clusters[i]['Signal Tier'];
  if (tier === 'T-NEGATIVE') t_negative_count++;
  else if (tier === 'T-AMBIGUOUS') t_ambiguous_count++;
  else if (tier === 'T-POSITIVE') t_positive_count++;
}
```

3. Aggregate domain totals
```javascript
const domain_map = {};
for (let i = 0; i < clusters.length; i++) {
  let domain = clusters[i]['Domain'];
  let n = clusters[i]['N'];
  if (!domain_map[domain]) domain_map[domain] = { domain, total: 0 };
  domain_map[domain].total += n;
}
```

4. Build trend signals (exclude singletons)
```javascript
const trend_signals = [];
for (let i = 0; i < clusters.length; i++) {
  if (!clusters[i]['Is Singleton']) {
    trend_signals.push({
      domain: clusters[i]['Domain'],
      tier: clusters[i]['Signal Tier'],
      n: clusters[i]['N'],
      prior_n: clusters[i]['Prior N'],
      trend_direction: clusters[i]['Trend Direction']
    });
  }
}
```

5. SEO keyword detection
```javascript
const seo_keywords = s9.seo_keywords || [];
const published_drafts = s9.published_drafts || [];

let seo_keyword_hits = 0;
for (let i = 0; i < published_drafts.length; i++) {
  let text = published_drafts[i].draft_text.toLowerCase();
  for (let j = 0; j < seo_keywords.length; j++) {
    if (text.includes(seo_keywords[j].toLowerCase())) {
      seo_keyword_hits++;
      break;
    }
  }
}
```

**Passes all_time_pending forward.**

### Step 12 — Generate Magic Link Token + Expiration

```javascript
const crypto = require('crypto');
const magic_token = crypto.randomBytes(16).toString('hex');

const exp = new Date();
exp.setDate(exp.getDate() + 7);
const token_expires_at = exp.toISOString();
```

**Passes all_time_pending forward.**

### Step 13 — Build NocoDB POST Body + Skip Flag

```javascript
const skip_nocodb = report_type === 'daily' ? true : false;

const nocodb_body = {
  'MRA Run ID': s12.mra_run_id,
  'Report Type': s12.report_type,
  'Client ID': s12.client_id,
  'Total Records': s12.total_records,
  'Avg Star Rating': s12.avg_star_rating,
  'SLA Compliance Rate': s12.sla_compliance_rate,
  // ... all 20 fields
};
```

**Passes all_time_pending forward.**

### Step 14 — IF Node (Skip NocoDB for Daily)

```javascript
if (s13.skip_nocodb === true) {
  return [];  // Halts FALSE branch
}
return [{ json: s13 }];
```

### Step 15 — NocoDB POST (Weekly/Monthly Only)

HTTP POST to MRA table. Daily reports skip this step.

### Step 16 — Build Brevo Payload

```javascript
const result = {
  mra_run_id: s13.mra_run_id,
  report_type: s13.report_type,
  client_id: s13.client_id,
  // ... all fields INCLUDING all_time_pending
  all_time_pending: s13.all_time_pending
};
```

### Step 17 — Build HTML Email Body

**Daily report:**
```javascript
email_body = '<!DOCTYPE html><html><head><style>' + emailStyles + '</style></head><body>';
email_body += '<div class="container">';
email_body += '<div class="header"><h1>48-Hour Draft Summary</h1><p>' + client_id_str + ' • ' + period_end + '</p></div>';
email_body += '<div class="content">';
email_body += '<div class="section"><h2 class="section-title"><span class="emoji">📊</span>New Drafts Generated</h2>';
// ... tier cards HTML
email_body += '<div class="section"><h2 class="section-title"><span class="emoji">📈</span>Platform Breakdown</h2>';
// ... platform grid HTML
email_body += '<div class="section"><h2 class="section-title"><span class="emoji">📋</span>Pending Approval</h2>';
email_body += '<div class="metric-card"><div class="metric-large">' + (p.all_time_pending || 0) + '</div>';
email_body += '<div class="metric-label">Draft(s) awaiting your review (all time)</div></div>';
email_body += '<a href="..." class="cta-button" style="color: #F6F6F6; text-decoration: none; border-bottom: none;">View & Approve Drafts →</a>';
email_body += '</div></div></body></html>';
```

**Weekly/Monthly reports:** Similar HTML structure with signal distribution, domains, trends, SEO.

**Brevo object:**
```javascript
const brevo_obj = {
  sender: { name: 'SubtextCX Reports', email: 'marellano@solofella.com' },
  to: [{ email: p.approval_email, name: client_name }],
  subject: email_subject,
  htmlContent: email_body  // NOT textContent
};
```

### Step 18 — PATCH NocoDB Delivery Status

```javascript
const patch_body = {
  'Delivery Status': 'sent',
  'Delivery Timestamp': new Date().toISOString()
};
```

### Step 19 — Send Email via Brevo

HTTP POST to `https://api.brevo.com/v3/smtp/email`

---

## Token Budget

**Zero tokens per report.**

**MRA operations:**
- NocoDB queries (read-only)
- JavaScript aggregation
- String concatenation
- Brevo API call (no AI)

**Cost:** Infrastructure only (~$0 at pilot scale)

---

## Key Locked Lessons (Chat #78-80)

### Lesson 1: Date Filtering by Source Timestamp

**Problem:** Filtering by RDA Timestamp (processing time) instead of ALA Date (guest posting time) caused incorrect period metrics.

**Solution:** Always filter by the source event timestamp (when guest posted review), not when the pipeline processed it.

**Implementation:** Build ALA Date lookup map, filter RDA by matching ALA Record ID, check ALA Date against period window.

### Lesson 2: SIA Window Type Alignment

**Problem:** MRA weekly reports were querying 30-day SIA data because SIA only ran monthly.

**Solution:** SIA now has 3 schedule branches (daily/weekly/monthly) with Report Window Type field. MRA queries SIA by matching report_type.

**Implementation:** Step 10a builds dynamic URL: `?where=(Report Window Type,eq,weekly)`

### Lesson 3: all_time_pending Field Chain

**Problem:** Daily reports need all-time pending count, but tier_breakdown only shows period metrics.

**Solution:** Calculate all_time_pending separately in Step 9 from ALL RDA records (not filtered by period).

**Implementation:** Add all_time_pending to return statement in Steps 9, 10a, 11, 12, 13, 16.

### Lesson 4: HTML Email Client Compatibility

**Problem:** CSS gradient and text-decoration not working in some email clients.

**Solution:** Use inline styles on critical elements (CTA button, links).

**Implementation:**
```html
<a href="..." class="cta-button" style="color: #F6F6F6; text-decoration: none; border-bottom: none;">
```

### Lesson 5: M/D/YY Date Parsing

**Problem:** ALA Date field uses M/D/YY format (e.g., "4/24/26"), not ISO 8601.

**Solution:** Parse manually:
```javascript
let parts = alaDateRaw.split('/');
let month = parseInt(parts[0]);
let day = parseInt(parts[1]);
let year = 2000 + parseInt(parts[2]);
let alaTs = new Date(year, month - 1, day).getTime();
```

### Lesson 6: Type Safety (String vs Integer)

**Problem:** ALA Record ID is Integer in NocoDB, but RDA stores it as String.

**Solution:** Always wrap in String() for comparisons:
```javascript
let alaId = String(rec['ALA Record ID'] || '');
```

---

## Related Documents

- **Changelog:** [SCX_MRA_CHANGELOG.md](SCX_MRA_CHANGELOG.md)
- **MCD v7.4.1:** [SubtextCX_MCD_v7_4_1.docx](../../SubtextCX_MCD_v7_4_1.docx)
- **SIA HOW:** [../SIA/SCX_SIA_HOW_v2.md](../SIA/SCX_SIA_HOW_v2.md) (3-branch architecture)
- **Dashboard Spec:** [../../commercial/Dashboard_Freelancer_Brief.md](../../commercial/Dashboard_Freelancer_Brief.md)

---

## n8n Workflow Details

**Workflow Name:** SCX-MRA  
**Triggers:** 3 separate cron schedules (daily/weekly/monthly)  
**Credentials:** NocoDB xc-token, Brevo API key

**Critical n8n 2.4.6 Rules:**
- All aggregation in Code Nodes (pure JavaScript)
- No AI API calls anywhere
- Index-based `for` loops (no `for...of`)
- `pageInfo.totalRows` for counts (not `list.length`)
- Key-by-key object construction (no spread operator)
- Type-safe ID comparisons (wrap in String())
- M/D/YY date parsing (not ISO 8601)

---

**End of MRA HOW v2.0**
