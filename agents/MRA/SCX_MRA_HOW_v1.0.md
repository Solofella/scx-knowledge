# SCX_MRA_HOW_v1.0

**Agent Name:** MRA (Management Reporting Agent)  
**Version:** 1.0  
**Last Updated:** Chat #75 · April 13, 2026  
**Model:** Pure JavaScript (zero LLM calls)  
**Status:** Verified operational - 8 nodes complete

---

## Purpose

MRA generates scheduled intelligence briefs for clients on three cadences:
1. **48-hour response draft summary** (daily 6am delivery)
2. **Weekly intelligence brief** (Monday 7am delivery)  
3. **Monthly comprehensive report** (1st of month 7am delivery)

**MRA uses pure JavaScript** - zero AI calls, zero token cost (same architecture as SIA).

---

## Input Sources

**MRA does NOT receive webhooks.** Runs on schedule, queries tables directly.

**Tables queried:**

1. **SIA table** (`mdn68l4lm609fve`)
   - Signal distribution (T-NEG/T-AMB/T-POS percentages)
   - Domain breakdown
   - Trend direction (UP/DOWN/STABLE)

2. **RDA table** (`mr1v67cszcklwns`)
   - Response draft counts
   - Approval status distribution
   - Time-to-approval metrics

3. **ALA table** (`m57efwbtrvwohhr`)
   - Total review count in period
   - Average star rating
   - Platform distribution (Google, Yelp, etc.)

4. **Client Config table** (`m95cmabjfyb94ps`)
   - Client display name
   - Email contact address
   - Report frequency preferences

---

## Processing Logic

### Node Flow (8 Nodes Total)

**Trigger (Node 1):** Three separate cron schedules
- Daily 6am UTC → 48hr summary
- Monday 7am UTC → weekly brief
- 1st of month 7am UTC → monthly report

**INIT Nodes (2-3):** Determine report type, client list, date range

**Data Aggregation (4-6):** Pure JavaScript
- Query SIA table (signal distribution, trends)
- Query RDA table (draft counts, approval status, velocity)
- Query ALA table (review counts, avg rating, platform breakdown)
- Calculate SLA rate (% drafts approved within 48hr)
- Calculate month-over-month deltas (if prior data exists)

**Email Composition & Delivery (7-8):**
- Format metrics into plain text email body
- Generate magic link token for dashboard access
- Send via Brevo API (transactional email service)
- Log delivery status to MRA NocoDB table

---

## NocoDB Schema

**Table ID:** `mybieof2em75t6e`

**Fields:**

| Field Name | Field ID | Type | Purpose |
|------------|----------|------|---------|
| MRA Run ID | `czljrhljau7a8iv` | Auto-increment | Primary key |
| Report Type | `cyqw1ki969a0vq0` | SingleSelect | 48hr-Summary / Weekly-Brief / Monthly-Report |
| Client ID | `chp1m03x1v1t3mv` | SingleLineText | PAK-001, EDO-001, etc. |
| Total Records | `cr3fr2dpxdncllz` | Number | Count of reviews in period |
| Avg Star Rating | `cohbxugfjq590n1` | Number | Average rating across period |
| SLA Rate | `cauxg0fskprxy70` | Number | % drafts approved within 48hr |
| T-NEGATIVE | `c7qu39qw9puutdq` | Number | Count of negative-tier reviews |
| T-AMBIGUOUS | `corkoggtozkumne` | Number | Count of ambiguous-tier reviews |
| T-POSITIVE | `cg0u8la22vt06lc` | Number | Count of positive-tier reviews |
| Delivery Status | `ci6hytuki6u9i2j` | SingleSelect | Sent / Failed / Pending |
| Magic Link Token | `can5u62qbnmin1x` | SingleLineText | Secure token for dashboard access |
| Error Log | `c5b2bybjo0equeb` | LongText | Email delivery failures, data errors |
| Platform Counts JSON | `cixk8vliloo1rao` | JSON | Review counts by platform |
| Created At | (auto) | DateTime | Record creation timestamp |

---

## Report Types & Content

### 1. 48-Hour Response Draft Summary

**Schedule:** Daily at 6am UTC

**Content:**
- Reviews received in last 48 hours
- Response drafts generated (count by tier: T1/T2/T3)
- Approval status (Approved / Pending / Modified)
- Average response velocity (hours from review posted to draft approved)

**Example email:**
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
View dashboard: https://subtextcx.venuiq.com/PAK-001?token=[secure_token]

---

### 2. Weekly Intelligence Brief

**Schedule:** Monday at 7am UTC

**Content:**
- Reviews processed last 7 days
- Signal distribution (T-NEG/T-AMB/T-POS percentages from SIA)
- Top 3 domains by volume
- Trending signals (UP/DOWN/STABLE vs prior week from SIA)
- Response draft performance (approval rate, avg velocity)

---

### 3. Monthly Comprehensive Report

**Schedule:** 1st of month at 7am UTC

**Content:**
- Reviews processed last 30 days
- Month-over-month comparison
- Domain analysis (top 5 domains, trends, top pain points from SIA)
- Response performance (approval rate, SLA compliance)
- Platform distribution

---

## Token Budget

**Zero tokens per report.**

**MRA operations:**
- NocoDB queries (read-only)
- JavaScript aggregation (`for` loops, calculations)
- Email composition (string concatenation)
- Brevo API call (no AI)

**Cost:** Infrastructure only (NocoDB queries, Brevo free tier up to 300 emails/day)

---

## Key Design Decisions

### Why Pure JavaScript (No AI)?

**MRA's task is reporting aggregated data:**
- Read SIA metrics (already aggregated)
- Read RDA counts (simple queries)
- Calculate percentages and averages (deterministic math)
- Format into email text (string templates)

**No interpretation needed** - just presenting numbers in structured format.

**Benefits:**
- Zero token cost (scales infinitely)
- Fast execution (<2 seconds for all 3 report types)
- Deterministic output (no LLM variability)
- No API rate limits

**Same architecture as SIA** - proven zero-cost model.

---

### Magic Link Security

**Token generation:**
```javascript
const crypto = require('crypto');
const token = crypto.randomBytes(32).toString('hex');
const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 7 days
```

**Dashboard validation:**
- User clicks: `subtextcx.venuiq.com/PAK-001?token=abc123...`
- Server queries MRA table: match token + client ID
- If valid + not expired: grant access
- If invalid: redirect to error page

**Security:**
- 7-day expiration
- Client ID validation
- Single-use option (mark token as used after first access)

---

### SLA Rate Calculation

**Definition:** % of response drafts approved within 48 hours of review posting

```javascript
const drafts = await nocodb.query({
  table: 'RDA',
  filter: `client_id = '${clientId}' AND created_at >= '${startDate}'`
});

let within48hr = 0;

for (let i = 0; i < drafts.length; i++) {
  const hoursToApproval = calculateHours(drafts[i].review_posted, drafts[i].approved_at);
  if (hoursToApproval <= 48) within48hr++;
}

const slaRate = Math.round((within48hr / drafts.length) * 100);
```

**Target:** 90% compliance

---

## Email Delivery (Brevo)

**Service:** Brevo transactional email API

**Free tier:** 300 emails/day (sufficient for pilot scale)

**Email format:**
- Plain text (avoid spam filters)
- Summary metrics (5-7 bullet points)
- Magic link to dashboard
- Footer: "Powered by SubtextCX"

**Delivery tracking:** Brevo logs delivery status, MRA stores in Delivery Status field

---

## Related Documents

- **Changelog:** [SCX_MRA_CHANGELOG.md](SCX_MRA_CHANGELOG.md)
- **Schema:** [MRA_Schema.md](MRA_Schema.md)
- **SIA (similar zero-cost model):** [../SIA/SCX_SIA_HOW_v3.md](../SIA/SCX_SIA_HOW_v3.md)
- **Dashboard Spec:** [../../commercial/Dashboard_Freelancer_Brief.md](../../commercial/Dashboard_Freelancer_Brief.md)

---

## n8n Workflow Details

**Workflow Name:** SCX-MRA  
**Triggers:** 3 separate cron schedules (daily 6am, Monday 7am, monthly 1st 7am)  
**Credentials:** NocoDB xc-token, Brevo API key

**Critical Rules:**
- All aggregation in Code Nodes (pure JavaScript)
- No AI API calls anywhere
- Index-based `for` loops (no `for...of`)
- `pageInfo.totalRows` for counts (not `list.length`)

---

**End of MRA HOW v1.0**
