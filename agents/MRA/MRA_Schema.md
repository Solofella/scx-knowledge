# MRA Schema

**NocoDB Table ID:** `mybieof2em75t6e`  
**Base ID:** `pq249fix22t3ofv`

---

## Table Fields

| Field Name | Field ID | Type | Source | Purpose |
|------------|----------|------|--------|---------|
| MRA Run ID | `czljrhljau7a8iv` | Auto-increment | System | Primary key |
| Report Type | `cyqw1ki969a0vq0` | SingleSelect | MRA computation | 48hr-Summary / Weekly-Brief / Monthly-Report |
| Client ID | `chp1m03x1v1t3mv` | SingleLineText | Query parameter | PAK-001, EDO-001, etc. |
| Total Records | `cr3fr2dpxdncllz` | Number | MRA aggregation | Count of reviews processed in period |
| Avg Star Rating | `cohbxugfjq590n1` | Number | MRA aggregation | Average rating across period (1-5 scale) |
| SLA Rate | `cauxg0fskprxy70` | Number | MRA computation | % drafts approved within 48hr |
| T-NEGATIVE | `c7qu39qw9puutdq` | Number | SIA aggregation | Count of negative-tier reviews |
| T-AMBIGUOUS | `corkoggtozkumne` | Number | SIA aggregation | Count of ambiguous-tier reviews |
| T-POSITIVE | `cg0u8la22vt06lc` | Number | SIA aggregation | Count of positive-tier reviews |
| Delivery Status | `ci6hytuki6u9i2j` | SingleSelect | Email API response | Sent / Failed / Pending |
| Magic Link Token | `can5u62qbnmin1x` | SingleLineText | MRA generation | Secure token for dashboard access (32-byte hex) |
| Error Log | `c5b2bybjo0equeb` | LongText | Error capture | Email delivery failures, query errors, aggregation issues |
| Platform Counts JSON | `cixk8vliloo1rao` | JSON | ALA aggregation | Review counts by platform (Google, Yelp, OpenTable, etc.) |
| Created At | (auto) | DateTime | System | Report generation timestamp |

---

## Relationships

**Upstream (Read-Only Queries):**
- **SIA table:** Signal distribution metrics, trend direction
- **RDA table:** Response draft counts, approval status, velocity
- **ALA table:** Review counts, star ratings, platform distribution
- **Client Config table:** Client display name, email address

**Downstream:** None (MRA is end-of-pipeline reporting)

**Side Effect:** Email delivery via Brevo API (external service)

---

## Field Computation Details

### Report Type (Trigger-Determined)

**Set by cron trigger:**

| Trigger Schedule | Report Type | Period Covered |
|------------------|-------------|----------------|
| Daily 6am UTC | `48hr-Summary` | Last 48 hours |
| Monday 7am UTC | `Weekly-Brief` | Last 7 days |
| 1st of month 7am UTC | `Monthly-Report` | Last 30 days |

**No AI computation** - directly mapped from trigger type

---

### Total Records (Count from ALA)

**Query:**
```javascript
const alaRecords = await nocodb.query({
  table: 'ALA',
  filter: `client_id = '${clientId}' AND date_posted >= '${startDate}' AND date_posted <= '${endDate}'`
});

const totalRecords = alaRecords.pageInfo.totalRows; // Use pageInfo, not list.length
```

**Purpose:** Display total review volume in report

---

### Avg Star Rating (Calculation from ALA)

**Query:**
```javascript
const alaRecords = await nocodb.query({
  table: 'ALA',
  filter: `client_id = '${clientId}' AND date_posted >= '${startDate}' AND date_posted <= '${endDate}'`
});

let totalStars = 0;
for (let i = 0; i < alaRecords.list.length; i++) {
  totalStars += alaRecords.list[i].star_rating;
}

const avgStarRating = (totalStars / alaRecords.list.length).toFixed(1); // e.g., 4.2
```

**Format:** One decimal place (e.g., 4.2, 3.8)

**Purpose:** Quick sentiment indicator in report

---

### SLA Rate (Time-to-Approval Calculation)

**Definition:** % of response drafts approved within 48 hours of review posting

**Query:**
```javascript
const rdaRecords = await nocodb.query({
  table: 'RDA',
  filter: `client_id = '${clientId}' AND created_at >= '${startDate}' AND approval_status = 'Approved'`
});

let within48hr = 0;

for (let i = 0; i < rdaRecords.list.length; i++) {
  const draft = rdaRecords.list[i];
  
  // Get review posted date from ALA
  const alaRecord = await nocodb.query({
    table: 'ALA',
    filter: `id = ${draft.ala_record_id}`
  });
  
  const reviewPosted = new Date(alaRecord.list[0].date_posted);
  const draftApproved = new Date(draft.approved_at);
  
  const hoursToApproval = (draftApproved - reviewPosted) / (1000 * 60 * 60);
  
  if (hoursToApproval <= 48) {
    within48hr++;
  }
}

const slaRate = Math.round((within48hr / rdaRecords.list.length) * 100); // Integer percentage
```

**Target:** 90% compliance

**Dashboard color coding:**
- Green: ≥90%
- Yellow: 80-89%
- Red: <80%

---

### T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE (From SIA)

**Query:**
```javascript
const siaRecords = await nocodb.query({
  table: 'SIA',
  filter: `client_id = '${clientId}' AND date_range_start >= '${startDate}' AND date_range_end <= '${endDate}'`
});

let tNegative = 0;
let tAmbiguous = 0;
let tPositive = 0;

for (let i = 0; i < siaRecords.list.length; i++) {
  const record = siaRecords.list[i];
  
  if (record.signal_tier === 'T-NEGATIVE') {
    tNegative += record.count;
  } else if (record.signal_tier === 'T-AMBIGUOUS') {
    tAmbiguous += record.count;
  } else if (record.signal_tier === 'T-POSITIVE') {
    tPositive += record.count;
  }
}
```

**Email display:**
Signal Distribution:

Negative: 12 (18%)
Ambiguous: 23 (34%)
Positive: 32 (48%)


**Percentage calculation:**
```javascript
const total = tNegative + tAmbiguous + tPositive;
const negPct = Math.round((tNegative / total) * 100);
const ambPct = Math.round((tAmbiguous / total) * 100);
const posPct = Math.round((tPositive / total) * 100);
```

---

### Platform Counts JSON (From ALA)

**Query:**
```javascript
const alaRecords = await nocodb.query({
  table: 'ALA',
  filter: `client_id = '${clientId}' AND date_posted >= '${startDate}'`
});

const platformCounts = {};

for (let i = 0; i < alaRecords.list.length; i++) {
  const platform = alaRecords.list[i].platform;
  platformCounts[platform] = (platformCounts[platform] || 0) + 1;
}

const platformCountsJSON = JSON.stringify(platformCounts);
```

**Example JSON:**
```json
{
  "Google": 45,
  "Yelp": 18,
  "OpenTable": 4
}
```

**Email display:**
Platform Breakdown:

Google: 45
Yelp: 18
OpenTable: 4


---

### Magic Link Token (Security)

**Generation:**
```javascript
const crypto = require('crypto');
const token = crypto.randomBytes(32).toString('hex'); // 64-character hex string
```

**Example token:**
a7f3c8e9d2b1f4a6c5e8d9f2b3a7c4e6d8f9a2b5c7e4f6a9d3b8c5e7f2a4b6c8

**Dashboard URL:**
https://subtextcx.venuiq.com/PAK-001?token=a7f3c8e9d2b1f4a6c5e8d9f2b3a7c4e6d8f9a2b5c7e4f6a9d3b8c5e7f2a4b6c8

**Expiration:** 7 days from creation (enforced in dashboard validation logic)

**Single-use option (future):** Add `used` boolean field, mark true after first access

---

### Delivery Status (Brevo API Response)

**Set based on Brevo API response:**

**Success response:**
```json
{
  "messageId": "abc123-def456",
  "status": "sent"
}
```
→ MRA sets: `Delivery Status = 'Sent'`

**Failure response:**
```json
{
  "error": "Invalid recipient email",
  "code": 400
}
```
→ MRA sets: `Delivery Status = 'Failed'`, logs error to Error Log field

**Pending (rare):**
- Email queued in Brevo but not yet sent
- MRA sets: `Delivery Status = 'Pending'`

---

### Error Log (Failure Capture)

**Populated when:**
- Brevo API call fails (invalid email, rate limit, network timeout)
- NocoDB query fails (table not found, invalid filter)
- Aggregation error (division by zero, missing data)

**Error does NOT block processing:**
- Error logged to this field
- Email not sent (Delivery Status = 'Failed')
- Workflow continues (next client or next report type)

**Example error log entries:**
2026-04-17T06:02:15Z - Brevo API error: Invalid recipient email 'invalid@example'

2026-04-17T06:05:32Z - SIA query error: No records found for date range 2026-04-10 to 2026-04-17

2026-04-17T06:08:47Z - Aggregation error: Division by zero (no RDA records for SLA calculation)

---

## Token Budget

**Zero tokens per report.**

**MRA operations:**
- NocoDB queries (read-only, no AI)
- JavaScript aggregation (`for` loops, arithmetic)
- String concatenation (email body formatting)
- Brevo API call (external HTTP request, no AI)

**No OpenAI. No Anthropic. No LLM.**

**Cost:** Infrastructure only (NocoDB queries, Brevo free tier)

---

## Data Flow
Cron Trigger (Daily 6am / Monday 7am / Monthly 1st 7am)
↓
INIT-2: Determine report type (48hr / Weekly / Monthly)
↓
INIT-3: Set date range based on report type
↓
Node 4: Query SIA table
Aggregate: T-NEG, T-AMB, T-POS counts
↓
Node 5: Query RDA table
Calculate: SLA rate, approval status distribution
↓
Node 6: Query ALA table
Aggregate: Total reviews, avg star rating, platform counts
↓
Node 7: Compose email body (plain text formatting)
Generate: Magic link token (crypto.randomBytes)
↓
Node 8: Brevo API call
Send email to client contact
Capture delivery status
↓
Write to MRA NocoDB table (report metadata + delivery status)

**No AI anywhere in flow.**

---

## Email Template Structure

**Subject line:**
SubtextCX [Report Type] - [Client Name] - [Date]

**Body format:**
SubtextCX [Report Type]
[Client Display Name]
[Date]
[Metrics Section - 5-7 bullet points]
View full dashboard: [Magic Link URL]

Powered by SubtextCX

**Example 48hr Summary:**
SubtextCX 48-Hour Summary
Park Avenue Kitchen by David Burke
April 17, 2026
Reviews Processed: 8

Google: 5
Yelp: 2
OpenTable: 1

Response Drafts Generated: 8

T1 (Positive): 6
T2 (Ambiguous): 1
T3 (Negative): 1

Approval Status:

Approved: 5
Pending Review: 3

Average Response Velocity: 14 hours
View dashboard: https://subtextcx.venuiq.com/PAK-001?token=[64-char-token]

Powered by SubtextCX

---

## Related Documents

- **HOW Document:** [SCX_MRA_HOW_v1.0.md](SCX_MRA_HOW_v1.0.md)
- **Changelog:** [SCX_MRA_CHANGELOG.md](SCX_MRA_CHANGELOG.md)
- **SIA (similar zero-cost model):** [../SIA/SCX_SIA_HOW_v3.md](../SIA/SCX_SIA_HOW_v3.md)
- **Dashboard Spec:** [../../commercial/Dashboard_Freelancer_Brief.md](../../commercial/Dashboard_Freelancer_Brief.md)

---

**End of MRA Schema**
