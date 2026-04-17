# BRA Schema

**NocoDB Table ID:** `mwqejw7swhd2cf4`  
**Base ID:** `pq249fix22t3ofv`

---

## Table Fields

| Field Name | Field ID | Type | Source | Purpose |
|------------|----------|------|--------|---------|
| BRA Record ID | (auto) | Auto-increment | System | Primary key |
| ALA Record ID | | Number | Pass-through | Traceability through entire pipeline |
| HSI Record ID | | Number | Pass-through | Upstream reference |
| Hospitality Signal ID | | SingleLineText | Inherited from HSI | Unique signal identifier |
| Client ID | | SingleLineText | Pass-through | PAK-001, EDO-001, etc. |
| **Tier Assignment** | | SingleSelect | **BRA computation (deterministic)** | T1 / T2 / T3 based on severity + star rating |
| **Template ID** | | SingleLineText | **BRA computation (for T1 only)** | Template Library reference (NULL for T2/T3) |
| **Response Draft** | | LongText | **BRA computation** | Template-populated (T1) or Claude-generated (T2/T3) |
| **Generation Method** | | SingleSelect | **BRA computation** | Template / Claude |
| **Error Log** | **c3crsub617hnuee** | LongText | Error capture | Template selection failures or Claude API errors |
| Created At | (auto) | DateTime | System | Record creation timestamp |

---

## Relationships

**Upstream:** HSI provides behavioral narratives and signal severity

**Downstream:** RDA receives BRA draft and refines with brand voice

**Side effect:** BRA updates HSI table's `BRA Status` field via PATCH

---

## Field Computation Details

### Tier Assignment (Deterministic)

**Computed by:** Code Node (no AI)

**Input:**
- Signal Severity Score (from HSI, 1-10 scale)
- Star Rating (from ALA pass-through, 1-5 scale)

**Logic:**

```javascript
const severity = items[0].json.signalSeverityScore;
const starRating = items[0].json.starRating;

let tier;

if (severity <= 3 && starRating >= 4) {
  tier = 'T1'; // Positive, low-severity
} else if (severity >= 8 || starRating <= 2) {
  tier = 'T3'; // Negative, high-severity
} else {
  tier = 'T2'; // Ambiguous, moderate-severity (catch-all)
}
```

**Possible values:**
- **T1:** Low-severity positive (most common, ~70-80%)
- **T2:** Moderate-severity ambiguous (~15-20%)
- **T3:** High-severity negative (~5-10%)

**Why this matters:**
- T1 → Template engine (0 tokens)
- T2/T3 → Claude API (~2,500 tokens each)

---

### Template ID (For T1 Records Only)

**Computed by:** Deterministic selection from Template Library

**Logic:**

```javascript
// Only for T1 tier
if (tier === 'T1') {
  // Query Template Library
  const templates = await nocodb.query({
    table: 'Template_Library',
    filter: `tier = 'T1' AND domain = '${domain}'`
  });
  
  // Generate deterministic seed from HSI Record ID
  const seed = hashCode(hsiRecordId);
  
  // Select template using modulo
  const selectedIndex = seed % templates.length;
  const selectedTemplate = templates[selectedIndex];
  
  templateId = selectedTemplate.template_id; // e.g., "T1-FoodQuality-003"
  
  // Increment usage count in Template Library
  await nocodb.patch({
    table: 'Template_Library',
    recordId: selectedTemplate.id,
    data: { usage_count: selectedTemplate.usage_count + 1 }
  });
}
```

**Example Template IDs:**
- `T1-FoodQuality-003`
- `T1-ServiceQuality-012`
- `T1-Ambiance-007`

**For T2/T3 records:** Template ID = `NULL` (Claude-generated, no template)

---

### Response Draft (Hybrid Generation)

**Two generation paths based on Tier Assignment:**

#### Path A: T1 Template Population (70-80% of records)

**Process:**
1. Fetch template from Template Library using Template ID
2. Extract placeholders: `{guest_name}`, `{specific_detail}`, `{domain}`
3. Replace with actual values from review data

**Code:**

```javascript
let draft = selectedTemplate.template_text;

// Replace {guest_name}
if (guestName) {
  draft = draft.replace('{guest_name}', guestName);
} else {
  // Remove greeting clause if no name
  draft = draft.replace('Thank you, {guest_name}, for', 'Thank you for');
}

// Replace {specific_detail}
const specificDetail = extractSpecificDetail(reviewText, domain);
draft = draft.replace('{specific_detail}', specificDetail);

// Replace {domain}
draft = draft.replace('{domain}', domain.toLowerCase());
```

**Example:**

**Template text:**
Thank you, {guest_name}, for highlighting {specific_detail}. We're glad our {domain}
team delivered an experience you enjoyed. Your feedback helps us maintain the standards
that matter most to our guests.

**Populated draft (guest name = "Michael", specific detail = "the perfectly grilled ribeye", domain = "Food Quality"):**
Thank you, Michael, for highlighting the perfectly grilled ribeye. We're glad our food
quality team delivered an experience you enjoyed. Your feedback helps us maintain the
standards that matter most to our guests.

**Token cost:** 0 (pure string replacement)

---

#### Path B: T2/T3 Claude Generation (20-30% of records)

**Process:**
1. Construct Claude API request with governance-constrained prompt
2. Include: Review text, Behavioral Narrative (HSI), Severity Score, Star Rating
3. Claude generates custom response (150-250 words)

**System prompt:**
You are a hospitality response drafting assistant. Generate a response to the guest review.
CRITICAL GOVERNANCE RULES:

Acknowledge the guest's experience as described
Express genuine appreciation for feedback
NEVER prescribe operational fixes
NEVER make promises about future changes
Focus on understanding and acknowledgment

Inputs:

Review: {reviewText}
Behavioral Narrative: {behavioralNarrative}
Severity: {severity}
Star Rating: {starRating}

Output: 150-250 words, professional yet empathetic, acknowledge without solving.

**Example Claude output (T3):**

**Input:**
- Review: "Waited 45 minutes despite reservation. Food was cold when it arrived. Completely unacceptable."
- Star Rating: 1
- Severity: 9
- Behavioral Narrative: "Guest experienced Autonomy violation (loss of control over schedule) with high-severity Service Quality and Food Quality pain points. Polite phrasing despite 1-star rating suggests guest values civility but is deeply dissatisfied."

**Claude draft:**
We're truly sorry your visit didn't meet the experience you deserved. A 45-minute wait
despite having a reservation, followed by cold food, falls far short of our standards.
We understand how frustrating it must have been to lose control of your evening timeline,
especially when you'd taken the time to make a reservation. The combination of the wait
and the temperature issue compounded what should have been a seamless dining experience.
Your feedback is important to us, and we appreciate you taking the time to share it, even
though the experience was disappointing. It helps us understand where we fell short and
what needs our attention.
Thank you for giving us the opportunity to learn from this.

**Token cost:** ~2,500 tokens

---

### Generation Method (Metadata)

**Possible values:**
- **Template:** Draft generated via T1 template engine (0 tokens)
- **Claude:** Draft generated via Claude API (T2 or T3, ~2,500 tokens)

**Use case:** Track distribution of template vs AI generation for cost analysis

**Example query (for MRA reporting):**

```javascript
const stats = await nocodb.query({
  table: 'BRA',
  aggregate: 'COUNT',
  groupBy: 'generation_method',
  filter: `client_id = 'PAK-001' AND created_at >= '2026-04-01'`
});

// Result:
// { Template: 225, Claude: 75 }
// Interpretation: 75% templates (zero cost), 25% Claude (token cost)
```

---

### Error Log (Failure Capture)

**Field ID:** `c3crsub617hnuee`

**Populated when:**
- Template Library query returns zero templates (domain not covered)
- Template placeholder replacement fails (malformed template)
- Claude API call fails (rate limit, timeout, invalid response)

**Error does NOT block pipeline:**
- Error logged to this field
- Downstream agents (RDA, MRA) can detect error and flag for manual review
- BRA continues processing other records

**Example error log entries:**
2026-04-17T14:32:11Z - Template query failed: No templates found for domain 'Parking' tier 'T1'

2026-04-17T15:08:42Z - Claude API error: Rate limit exceeded (429 response)

2026-04-17T16:21:03Z - Template placeholder error: {specific_detail} not found in review text

---

## Template Library Reference

**NocoDB Table ID:** `mafv9by73ebama7`

**BRA queries this table for T1 template selection.**

**Template Library Schema:**

| Field | Type | Purpose |
|-------|------|---------|
| Template ID | SingleLineText | Unique identifier (e.g., T1-FoodQuality-003) |
| Tier | SingleSelect | T1 (future: T2 if needed, never T3) |
| Domain | SingleSelect | Food Quality, Service Quality, etc. |
| Template Text | LongText | Response text with placeholders |
| Brand Voice Mode | SingleSelect | Standard / Warm / Professional / Casual |
| Usage Count | Number | Incremented each time template selected |

**Current size:** 50+ templates

**Coverage:** 13 domains × T1 tier × 4 brand voice modes (not all combinations exist yet)

**Maintenance:** Add templates when pilot feedback shows gaps in coverage

---

## Token Budget

**Per-record cost (by tier):**

| Tier | Generation Method | Token Cost | Distribution | Weighted Cost |
|------|-------------------|------------|--------------|---------------|
| T1 | Template | 0 | 75% | 0 |
| T2 | Claude | 2,500 | 20% | 500 |
| T3 | Claude | 2,500 | 5% | 125 |
| **Avg** | **Hybrid** | | **100%** | **~625** |

**At 300 reviews/month:**
- T1: 225 records × 0 = 0 tokens
- T2: 60 records × 2,500 = 150K tokens
- T3: 15 records × 2,500 = 37.5K tokens
- **Total BRA: 187.5K tokens/month**

**If all-Claude (no templates):**
- 300 records × 2,500 = 750K tokens/month
- **Hybrid saves: 562.5K tokens/month (75% reduction)**

---

## HSI Table Update (Side Effect)

**BRA PATCHes HSI table after draft generation:**

**Target field:** `BRA Status` (field ID: `cl1250sz39sm45l` in HSI table)

**Updated values:**
- `T1-Generated` (template used)
- `T2-Generated` (Claude used)
- `T3-Generated` (Claude used)
- `Error` (BRA processing failed)

**Logic:**

```javascript
await nocodb.patch({
  table: 'HSI',
  recordId: hsiRecordId,
  data: {
    bra_status: tier === 'T1' ? 'T1-Generated' : tier === 'T2' ? 'T2-Generated' : 'T3-Generated'
  }
});
```

**Purpose:** Track which HSI records have been processed by BRA (used by MRA for reporting)

---

## Data Flow
HSI webhook → BRA
↓
INIT-1 to INIT-3: Receive + parse payload
↓
Node 4-6: Tier Assignment (deterministic)
Input: Severity Score + Star Rating
Output: T1 / T2 / T3
↓
IF T1:
Node 7-11: Template Engine
Query Template Library (domain match)
Select template (deterministic seed)
Replace placeholders {guest_name}, {specific_detail}, {domain}
Token cost: 0
↓
Node 18: PATCH HSI table (BRA Status = 'T1-Generated')
↓
Node 19: Write to BRA table + pass to RDA webhook
IF T2 or T3:
Node 12-17: Claude API
System prompt: Governance-constrained response generation
Input: Review + Behavioral Narrative + Severity
Output: Custom draft (150-250 words)
Token cost: ~2,500
↓
Node 18: PATCH HSI table (BRA Status = 'T2-Generated' or 'T3-Generated')
↓
Node 19: Write to BRA table + pass to RDA webhook

---

## Related Documents

- **HOW Document:** [SCX_BRA_HOW_v3.2.md](SCX_BRA_HOW_v3.2.md)
- **Changelog:** [SCX_BRA_CHANGELOG.md](SCX_BRA_CHANGELOG.md)
- **Upstream Agent:** [../HSI/SCX_HSI_HOW_v3.md](../HSI/SCX_HSI_HOW_v3.md)
- **Downstream Agent:** [../RDA/SCX_RDA_HOW_v3.1.md](../RDA/SCX_RDA_HOW_v3.1.md)
- **Template Library:** NocoDB table `mafv9by73ebama7`

---

**End of BRA Schema**
