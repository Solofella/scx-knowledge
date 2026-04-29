# MRA_Schema_v2.0

**NocoDB Table ID:** `mybieof2em75t6e`  
**Base ID:** `pq249fix22t3ofv`  
**Last Updated:** Chat #80 · April 28, 2026

---

## Table Fields (20 Total)

| Field Name | Field ID | Type | Source | Purpose |
|------------|----------|------|--------|---------|
| Id | (auto) | AutoNumber | System | Primary key |
| MRA Run ID | `czljrhljau7a8iv` | SingleLineText | MRA computation | MRA-YYYYMMDD-HHMMSS |
| Report Type | `cyqw1ki969a0vq0` | SingleSelect | Cron trigger | daily / weekly / monthly |
| Client ID | `chp1m03x1v1t3mv` | SingleLineText | Query parameter | PAK-001, EDO-001, etc. |
| Period Start | (TBD) | DateTime | MRA computation | ISO 8601 UTC start of period |
| Period End | (TBD) | DateTime | MRA computation | ISO 8601 UTC end of period |
| Total Records | `cr3fr2dpxdncllz` | Number | Step 9 aggregation | Count of reviews in period (filtered by ALA Date) |
| Avg Star Rating | `cohbxugfjq590n1` | Number | Step 9 aggregation | Average rating (1-5 scale, 2 decimals) |
| SLA Compliance Rate | `cauxg0fskprxy70` | Number | Step 9 computation | % drafts approved within 48hr |
| T-Negative Count | `c7qu39qw9puutdq` | Number | Step 11 SIA aggregation | Count of T-NEGATIVE signal clusters |
| T-Ambiguous Count | `corkoggtozkumne` | Number | Step 11 SIA aggregation | Count of T-AMBIGUOUS signal clusters |
| T-Positive Count | `cg0u8la22vt06lc` | Number | Step 11 SIA aggregation | Count of T-POSITIVE signal clusters |
| Top Domains JSON | (TBD) | JSON | Step 11 aggregation | Domain breakdown array |
| Tier Breakdown JSON | (TBD) | JSON | Step 9 aggregation | T1/T2/T3 counts + approval metrics |
| Delivery Status | `ci6hytuki6u9i2j` | SingleSelect | Brevo API response | sent / failed / pending |
| Magic Link Token | `can5u62qbnmin1x` | SingleLineText | Step 12 generation | Secure dashboard access token (32-char hex) |
| Error Log | `c5b2bybjo0equeb` | LongText | Error handler | Email delivery failures, query errors |
| Platform Counts JSON | `cixk8vliloo1rao` | JSON | Step 9 aggregation | Review counts by platform (Google/OpenTable/Yelp/TripAdvisor) |
| Run Timestamp | (TBD) | DateTime | Step 5 generation | Report generation time |
| Delivery Timestamp | (TBD) | DateTime | Step 18 PATCH | Email sent timestamp |

**Daily reports skip NocoDB write** (Step 14 returns empty array, never reaches Step 15 POST).

---

## Relationships

**Upstream Tables (Read-Only Queries):**

1. **ALA table** (`m57efwbtrvwohhr`)
   - **Date field** (M/D/YY format) — PRIMARY date filter
   - Platform, Star Rating, Client ID
   - Query: All records for client, filter by Date in Step 9

2. **RDA table** (`mr1v67cszcklwns`)
   - ALA Record ID (FK for date matching)
   - Confirmed Response Tier, Approval Status
   - RDA Timestamp, Published Timestamp (for SLA)
   - Public Response Draft (for SEO keyword detection)
   - Query: All records for client, filter by ALA Date in Step 9

3. **SIA table** (`mdn68l4lm609fve`)
   - **Report Window Type** (daily/weekly/monthly) — NEW Chat #80
   - Domain, Signal Tier, N, Prior N, Trend Direction
   - Query: Filter by Report Window Type in Step 10b

4. **Client Config table** (`m95cmabjfyb94ps`)
   - Client Name, Approval Contact Email, SEO Keywords
   - Query: Once per session in Step 5

**Downstream:** None (MRA is terminal reporting agent)

**Side Effect:** HTML email delivery via Brevo API

---

## Field Computation Details

### Report Type (Trigger-Determined)

**Set by cron trigger:**

| Trigger Schedule | Report Type | Period Window | Window Days |
|------------------|-------------|---------------|-------------|
| Daily 7am UTC | `daily` | Last 48 hours | 2 |
| Monday 7am UTC | `weekly` | Last 7 days | 7 |
| 1st of month 7am UTC | `monthly` | Last 30 days | 30 |

**No AI computation** - directly mapped from trigger type in Step 3.

---

### Total Records (Count from ALA via Step 9)

**CRITICAL ARCHITECTURE (Chat #80 fix):**

MRA does NOT filter ALA by date in the HTTP query. Step 6 fetches ALL ALA records for the client. **Date filtering happens in Step 9 JavaScript.**

**Step 6 query:**
```javascript
const url = 'http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr?where=(Client%20ID,eq,PAK-001)&limit=1000';
```

**Step 9 date filtering:**
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
const filteredRda = [];
for (let i = 0; i < allRda.length; i++) {
  let alaId = String(rec['ALA Record ID'] || '');
  
  if (alaDateMap[alaId]) {
    let alaTs = alaDateMap[alaId].ala_timestamp;
    
    if (alaTs >= periodStartTs && alaTs < periodEndTs) {
      filteredRda.push(rec);
    }
  }
}

const total_records = filteredRda.length;
```

**Purpose:** Display total review volume in report

---

### Avg Star Rating (Calculation from ALA via Step 9)

**Computed during Step 9 date filtering loop:**

```javascript
let star_sum = 0;

for (let i = 0; i < allRda.length; i++) {
  let alaId = String(rec['ALA Record ID'] || '');
  
  if (alaDateMap[alaId]) {
    let alaTs = alaDateMap[alaId].ala_timestamp;
    
    if (alaTs >= periodStartTs && alaTs < periodEndTs) {
      let alaRec = alaDateMap[alaId].ala_record;
      star_sum += parseFloat(alaRec['Star Rating'] || 0);
    }
  }
}

const avg_star_rating = total_rda > 0 ? Math.round((star_sum / total_rda) * 100) / 100 : 0;
```

**Format:** Two decimal places (e.g., 4.27, 3.67)

**Purpose:** Quick sentiment indicator in report

---

### SLA Compliance Rate (Time-to-Approval Calculation)

**Definition:** % of response drafts approved within 48 hours of review posting

**Computed in Step 9:**

```javascript
let sla_compliant = 0;
let published_count = 0;

for (let i = 0; i < filteredRda.length; i++) {
  let rec = filteredRda[i];
  let status = rec['Approval Status'];
  
  if (status === 'Published') {
    published_count++;
    
    let pub_ts = rec['Published Timestamp'] ? new Date(rec['Published Timestamp']).getTime() : 0;
    let rda_ts = rec['RDA Timestamp'] ? new Date(rec['RDA Timestamp']).getTime() : 0;
    
    if (pub_ts > 0 && rda_ts > 0) {
      let hrs = (pub_ts - rda_ts) / 3600000;
      if (hrs <= 48) {
        sla_compliant++;
      }
    }
  }
}

const sla_compliance_rate = published_count > 0 ? Math.round((sla_compliant / published_count) * 10000) / 100 : 0;
```

**Format:** Percentage with 2 decimals (e.g., 87.50%)

**Target:** 90% compliance

---

### T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE (From SIA via Step 11)

**Step 10b query (dynamic URL from Step 10a):**
```javascript
const report_type = s9.report_type || 'weekly';
const url = 'http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mdn68l4lm609fve?where=(Report%20Window%20Type,eq,' + report_type + ')&limit=1000&sort=-Run%20Timestamp';
```

**Step 11 aggregation:**
```javascript
const allClusters = input.list || [];
const latestRunTs = allClusters[0]['Run Timestamp'];
const clusters = allClusters.filter(c => c['Run Timestamp'] === latestRunTs);

let t_negative_count = 0;
let t_ambiguous_count = 0;
let t_positive_count = 0;

for (let i = 0; i < clusters.length; i++) {
  let tier = (clusters[i]['Signal Tier'] || '').trim();
  
  if (tier === 'T-NEGATIVE') t_negative_count++;
  else if (tier === 'T-AMBIGUOUS') t_ambiguous_count++;
  else if (tier === 'T-POSITIVE') t_positive_count++;
}
```

**HTML email display (weekly/monthly):**
```
📊 Signal Distribution
  T-NEGATIVE: 1 signal(s)
  T-AMBIGUOUS: 0 signal(s)
  T-POSITIVE: 1 signal(s)
  Total signals: 2
```

---

### Platform Counts JSON (From ALA via Step 9)

**Computed during Step 9 date filtering loop:**

```javascript
const platform_counts = {};

for (let i = 0; i < allRda.length; i++) {
  let alaId = String(rec['ALA Record ID'] || '');
  
  if (alaDateMap[alaId]) {
    let alaTs = alaDateMap[alaId].ala_timestamp;
    
    if (alaTs >= periodStartTs && alaTs < periodEndTs) {
      let alaRec = alaDateMap[alaId].ala_record;
      let plat = alaRec['Platform'] || 'Unknown';
      platform_counts[plat] = (platform_counts[plat] || 0) + 1;
    }
  }
}
```

**Example JSON:**
```json
{
  "Google": 1,
  "OpenTable": 2,
  "Yelp": 0,
  "TripAdvisor": 0
}
```

**HTML email display (all 3 report types):**
```
📈 Platform Breakdown
  Google: 1
  OpenTable: 2
  Yelp: 0
  TripAdvisor: 0
```

---

### Magic Link Token (Security via Step 12)

**Generation:**
```javascript
const crypto = require('crypto');
const magic_token = crypto.randomBytes(16).toString('hex');  // 32-character hex string

const exp = new Date();
exp.setDate(exp.getDate() + 7);
const token_expires_at = exp.toISOString();
```

**Example token:**
`e557f7d3-b988-484b-a2f8-2c45c8db46ee`

**Dashboard URL (included in email):**
```
https://subtextcx.solofella.com/PAK-001?token=e557f7d3-b988-484b-a2f8-2c45c8db46ee
```

**Expiration:** 7 days from creation

**Validation:** Dashboard server queries MRA table, matches token + client ID + expiration check

---

### Delivery Status (Brevo API Response via Step 19)

**Set based on Brevo API response:**

**Success response:**
```json
{
  "messageId": "abc123-def456"
}
```
→ Step 18 PATCH: `Delivery Status = 'sent'`, `Delivery Timestamp = ISO 8601 UTC`

**Failure response:**
```json
{
  "code": "invalid_parameter",
  "message": "Invalid recipient email"
}
```
→ Error handler: `Delivery Status = 'failed'`, logs error to Error Log field

---

### Error Log (Failure Capture)

**Populated when:**
- Brevo API call fails (Step 19 error)
- NocoDB query fails (Step 6/7b/10b error)
- Aggregation error (Step 9/11 computation error)

**Error does NOT block entire workflow:**
- Error logged to this field
- Email not sent (Delivery Status = 'failed')
- Workflow continues (error handler sends notification email to miguel@solofella.com)

**Example error log entries:**
```
2026-04-28T07:02:15Z - Brevo API error: Invalid recipient email
2026-04-28T07:05:32Z - SIA query error: No records found for Report Window Type = weekly
2026-04-28T07:08:47Z - Step 9 aggregation error: Division by zero (no published records for SLA calculation)
```

---

## Token Budget

**Zero tokens per report.**

**MRA operations:**
- NocoDB queries (read-only, pure HTTP)
- JavaScript aggregation (for loops, arithmetic, string parsing)
- HTML string concatenation (email body formatting)
- Brevo API call (external HTTP POST, no AI)

**No OpenAI. No Anthropic. No LLM anywhere.**

**Cost:** Infrastructure only (~$0 at pilot scale, Brevo free tier 300 emails/day)

---

## Data Flow (23-Node Architecture)

```
Cron Trigger (Daily/Weekly/Monthly) — Step 1a/1b/1c
↓
Merge Triggers — Step 2
↓
Set Report Type & Window — Step 3
↓
Get Client List — Step 4
↓
Generate MRA Run ID + Magic Token — Step 5
↓
Fetch ALL ALA Records (no date filter) — Step 6
↓
Build RDA Query URL — Step 7a
↓
Fetch ALL RDA Records (no date filter) — Step 7b
↓
CRITICAL: Filter by ALA Date + Aggregate Metrics — Step 9
  ├─ Build ALA Date lookup map (parse M/D/YY)
  ├─ Filter RDA by matching ALA Record ID
  ├─ Check if ALA Date in period window
  ├─ Calculate: total_records, avg_star_rating, platform_counts
  ├─ Calculate: tier_breakdown, sla_compliance_rate
  └─ Calculate: all_time_pending (from ALL RDA, not filtered)
↓
Build SIA Query URL (dynamic by report_type) — Step 10a
↓
Fetch SIA Clusters (Report Window Type = report_type) — Step 10b
↓
Process SIA + SEO — Step 11
  ├─ Count signal distribution (T-NEG/T-AMB/T-POS)
  ├─ Aggregate domain totals
  ├─ Build trend_signals array (exclude singletons)
  └─ Calculate SEO keyword hits
↓
Generate Magic Link Token + Expiration — Step 12
↓
Build NocoDB POST Body + Skip Flag — Step 13
↓
IF Node: Daily Reports Skip NocoDB — Step 14
  ├─ TRUE (daily) → Return [] (halt)
  └─ FALSE (weekly/monthly) → Continue
↓
NocoDB POST (Weekly/Monthly Only) — Step 15
↓
Build Brevo Payload — Step 16
↓
Build HTML Email Body (Branded Design) — Step 17
  ├─ Navy-to-Olive gradient header
  ├─ Metric cards with brand colors
  ├─ Platform grid, domain items, trend badges
  ├─ Burnt orange CTA button (#ae4900 bg, #F6F6F6 text)
  └─ htmlContent (not textContent)
↓
PATCH NocoDB Delivery Status — Step 18
↓
Send Email via Brevo API — Step 19
```

**No AI anywhere in flow.**

---

## HTML Email Design (Chat #80 Update)

**SubtextCX Brand Colors:**
- **#002D72** (Navy Blue) — Header gradient, key metrics
- **#475a28** (Olive Green) — Header gradient, positive signals, domain borders
- **#ae4900** (Burnt Orange) — CTA button background, alert boxes, negative signals
- **#F6F6F6** (Light Gray) — Card backgrounds, platform items, CTA button text
- **#A8A9AD** (Medium Gray) — Labels, muted text
- **#111111** (Near Black) — Primary text

**Email Structure:**
1. Header: Navy-to-Olive gradient
2. Metric cards: Light gray bg, navy left border
3. Alert boxes: Light orange bg, burnt orange border (T3 warnings)
4. Platform grid: 2-column layout
5. Domain items: Olive green left border
6. Trend badges: Color-coded by tier
7. CTA button: Burnt orange bg (#ae4900), light gray text (#F6F6F6), no underline
8. Footer: Gray bg, governance note

**Brevo API payload:**
```javascript
{
  sender: { name: 'SubtextCX Reports', email: 'marellano@solofella.com' },
  to: [{ email: approval_email, name: client_name }],
  subject: email_subject,
  htmlContent: email_body  // NOT textContent
}
```

---

## Related Documents

- **HOW Document:** [SCX_MRA_HOW_v2.0.md](SCX_MRA_HOW_v2.0.md) (Chat #80 update)
- **Changelog:** [SCX_MRA_CHANGELOG.md](SCX_MRA_CHANGELOG.md)
- **SIA Schema:** [../SIA/SIA_Schema_v2.md](../SIA/SIA_Schema_v2.md) (3-branch architecture)
- **MCD v7.4.1:** [../../SubtextCX_MCD_v7_4_1.docx](../../SubtextCX_MCD_v7_4_1.docx)
- **Dashboard Spec:** [../../commercial/Dashboard_Freelancer_Brief.md](../../commercial/Dashboard_Freelancer_Brief.md)

---

**End of MRA Schema v2.0**
