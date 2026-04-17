# RDA Schema

**NocoDB Table ID:** `mr1v67cszcklwns`  
**Base ID:** `pq249fix22t3ofv`

---

## Table Fields

| Field Name | Type | Source | Purpose |
|------------|------|--------|---------|
| RDA Record ID | Auto-increment | System | Primary key |
| ALA Record ID | Number | Pass-through | Traceability through entire pipeline |
| BRA Record ID | Number | Pass-through | Upstream reference |
| Hospitality Signal ID | SingleLineText | Inherited from HSI | Unique signal identifier |
| Client ID | SingleLineText | Pass-through | PAK-001, EDO-001, etc. |
| **Response Draft** | LongText | **RDA computation (Claude)** | Final client-facing response text |
| **Internal Followup Draft** | LongText | **RDA computation (Claude)** | Signal meaning description (not shown to client) |
| **Brand Voice Mode** | SingleSelect | Client Config fetch | Standard / Warm / Professional / Casual |
| **Guest Name Present** | Boolean | Conditional check | True if guest name available for personalization |
| **Tier** | SingleSelect | Inherited from BRA | T1 / T2 / T3 |
| **Approval Status** | SingleSelect | Google Sheet sync | Pending / Approved / Modify / Not Accepted |
| Star Rating | Number | Pass-through from ALA | 1-5 rating |
| Review Text | LongText | Pass-through from ALA | Original review (for context) |
| Created At | DateTime | System | Record creation timestamp |

---

## Relationships

**Upstream:** BRA provides initial draft + tier assignment

**Downstream:** 
- **Google Sheet approval workflow** (Apps Script webhook → n8n → PATCH RDA)
- **Dashboard** (read-only display of approval counts, NOT draft text)

---

## Field Computation Details

### Response Draft (Claude Generation)

**Computed by:** Claude claude-sonnet-4-6

**Input:**
- BRA initial draft (template-populated for T1, Claude-generated for T2/T3)
- Brand voice settings from Client Config table
- Guest name (if available)
- Review text, star rating, tier
- HSI Behavioral Narrative (for context)

**System prompt governance:**Refine the BRA draft with brand voice integration. Apply client's brand voice mode.If guest name available:

T1: Use warm acknowledgment variations (6 structural options)
T2/T3: Simple "Hi [name]" or "Hello [name]"
If no guest name:

Open with specific signal detail from review (never generic "thank you for feedback")
CRITICAL: Detect and interpret only. Never prescribe operational actions.
Max 1 brand phrase per draft. Vary phrasing even when using brand terms.Output: Final response draft (150-300 words)

**Example output (T1 with guest name):**

**Input:**
- BRA draft: "Thank you, Sarah, for highlighting the perfectly seared scallops. We're glad our food quality team delivered an experience you enjoyed."
- Brand Voice Mode: Warm
- Client brand phrase: "At Park Avenue Kitchen"

**RDA output:**Sarah, thank you for taking the time to share your experience with the scallops. We're
delighted you enjoyed them. Our team at Park Avenue Kitchen takes pride in sourcing the
finest ingredients and preparing each dish with care. Your feedback about the sear reminds
us why we do what we do—creating moments that matter for our guests. We hope to welcome
you back soon.

**Token cost:** ~3,000 tokens (BRA draft + brand voice context + Claude refinement)

---

### Internal Followup Draft (Governance-Constrained)

**Computed by:** Same Claude call (dual output)

**System prompt:**Provide internal followup context: what should the operator UNDERSTAND about this signal?CRITICAL GOVERNANCE RULE:

Describe signal MEANING only
NEVER recommend actions ("add servers", "change menu")
Focus on guest psychology and pattern recognition
Output: Internal context (50-100 words)

**Example output:**

**Input:**
- Review: "Great food but service was slow. We had a reservation."
- Tier: T2
- Behavioral Narrative: "Guest experienced Autonomy violation (wait time despite reservation). Food Quality satisfied expectations but Service Quality pain point dampened overall experience."

**Internal followup draft:**This guest's frustration stems from unmet expectations—they made a reservation to control
their timeline, but the wait violated that control. The Food Quality satisfaction prevented
a T3 classification, but the Service signal is a retention risk. Pattern: weekend capacity
or reservation system not honored during high-traffic periods.

**Bad example (violates governance):**Add more servers during weekend dinner. ← PRESCRIPTIVE, NOT ALLOWED

**Token cost:** Included in ~3,000 token RDA call (dual output, single API call)

---

### Brand Voice Mode (Client Config Fetch)

**Query before Claude call:**

```javascriptconst clientConfig = await nocodb.query({
table: 'Client_Config',
filter: client_id = '${clientId}'
});const brandVoiceMode = clientConfig.list[0].brand_voice_mode; // Standard / Warm / Professional / Casual
const brandPhrase = clientConfig.list[0].brand_phrase; // e.g., "At Park Avenue Kitchen"

**Applied in Claude prompt:**
- **Standard:** Professional, service-focused
- **Warm:** Personal, relationship-building
- **Professional:** Formal, quality commitment
- **Casual:** Friendly, approachable

**Brand phrase usage:** Max 1 per draft, with variation constraint

---

### Guest Name Present (Conditional Logic)

**Check:**

```javascriptconst guestName = items[0].json.reviewer_handle; // From ALAconst guestNamePresent = guestName && guestName.length > 0 && guestName !== 'Anonymous';

**If TRUE:**
- **T1:** Apply 6 structural greeting variations (gratitude lead, pleasure lead, guest word lead, specific detail lead, occasion lead, belonging lead)
- **T2/T3:** Simple "Hi [name]," or "Hello [name],"

**If FALSE:**
- Open with specific signal detail from review
- Example: "We're glad the ribeye exceeded expectations..." (NOT "Thank you for your feedback")

---

### Approval Status (Google Sheet Sync)

**Initial value (set by RDA):** `Pending`

**Updated by Google Sheet Apps Script webhook:**

When client edits approval sheet:
1. Apps Script onEdit fires webhook to n8n
2. n8n SCX-Sheet-Approval workflow PATCHes RDA table
3. Updates `approval_status` field: `Approved` / `Modify` / `Not Accepted`

**Workflow logic:**

```javascript// Google Sheet columns: Date | Platform | Stars | Review | Proposed (protected) | Edited | Status | RDA-ID// onEdit event
if (editedColumn === 'Status' || editedColumn === 'Edited') {
const status = row.Status; // Approved / Modify / Not Accepted
const rdaId = row['RDA-ID'];// Webhook to n8n
UrlFetchApp.fetch('https://n8n-solofella.domain/webhook/scx-sheet-approval', {
method: 'POST',
contentType: 'application/json',
payload: JSON.stringify({ rda_id: rdaId, status: status })
});
}

**n8n PATCH:**

```javascriptawait nocodb.patch({
table: 'RDA',
recordId: rdaId,
data: { approval_status: status }
});

---

## Token Budget

**~3,000 tokens per record**

**Breakdown:**
- BRA draft input: ~500 tokens (already generated)
- Brand voice config fetch: 0 tokens (NocoDB query)
- Claude API call (dual output): ~2,500 tokens
  - Response Draft: ~2,000 tokens
  - Internal Followup Draft: ~500 tokens
- Guest name conditional logic: 0 tokens (deterministic)

---

## Open Issues (From Changelog)

### Issue 1: Internal Brief Quality (Not Yet Fixed)

**Problem:** `internal_followup_draft` sometimes includes operational prescriptions

**Current:** "Operator should add more servers during weekend rush" ← VIOLATES GOVERNANCE

**Fix needed:** Strengthen system prompt - describe signal meaning only, no "add/change/fix" language

**Priority:** High (governance violation)

---

### Issue 2: Brand Voice Variation (Not Yet Fixed)

**Problem:** Brand phrases repeat too frequently (currently allows up to 3 per draft)

**Current:** "At Park Avenue Kitchen, we... At Park Avenue Kitchen, our team... At Park Avenue Kitchen..."

**Fix needed:** Reduce to 1 phrase max, add variation constraint to system prompt

**Example fix:**System prompt addition:
"Use client brand phrase maximum ONCE per draft. If used, vary the phrasing:

'At [Brand]...' → 'Our team at [Brand]...' → '[Brand] is committed to...' → 'We at [Brand]...'"


**Priority:** Medium (prevents over-repetition, but not a governance violation)

---

## Data FlowBRA webhook → RDA
↓
INIT-1 to INIT-3: Receive + parse payload
Validate: BRA draft present, tier assigned
↓
Node 4-9: Brand Voice Integration
Query Client Config table (brand voice mode + phrase)
Check guest name availability
↓
Node 10-16: Claude API call (dual output)
System prompt: Governance-constrained refinement
Input: BRA draft + brand voice + guest name + review context
Output 1: Response Draft (client-facing)
Output 2: Internal Followup Draft (operator context)
↓
Node 17: Write to RDA NocoDB table
Initial approval_status = 'Pending'
↓
Node 18: Pass to Google Sheet
Write to client-specific approval sheet
Protected "Proposed" column contains draft
Client edits in "Edited" column, sets "Status"
↓
Apps Script webhook → n8n SCX-Sheet-Approval → PATCH RDA
Update approval_status based on client action

---

## Related Documents

- **HOW Document:** [SCX_RDA_HOW_v3.1.md](SCX_RDA_HOW_v3.1.md)
- **Changelog:** [SCX_RDA_CHANGELOG.md](SCX_RDA_CHANGELOG.md)
- **Upstream Agent:** [../BRA/SCX_BRA_HOW_v3.2.md](../BRA/SCX_BRA_HOW_v3.2.md)
- **Google Sheet Approval Spec:** Documented in MCD v7.4 (6 locked fixes)

---

**End of RDA Schema**
