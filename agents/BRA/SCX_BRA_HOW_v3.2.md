# SCX_BRA_HOW_v3.2

**Agent Name:** BRA (Brand Response Agent)  
**Version:** 3.2  
**Last Updated:** Chat #18 · March 11, 2026  
**Model:** Hybrid (T1 deterministic, T2/T3 Claude claude-sonnet-4-6)  
**Status:** Verified operational - 19 nodes complete

---

## Purpose

BRA generates initial response drafts based on HSI behavioral narratives and signal severity. It uses a **hybrid approach**: deterministic template selection for T1 (majority of reviews ~70-80%), and Claude API calls for T2/T3 (minority ~20-30%).

**BRA is NOT the final draft agent.** BRA creates initial drafts that RDA refines with brand voice integration.

---

## Input Source

**Upstream Agent:** HSI (Hospitality Signal Intelligence)

**Receives from HSI webhook:**
- Hospitality Signal ID
- Signal Severity Score (1-10)
- Behavioral Narrative
- Operator Insight
- Domain (Food Quality, Service Quality, etc.)
- ALA Record ID (traceability)
- Client ID
- Review Text (original)
- Star Rating

**BRA does NOT query dictionaries or other agent tables.**

---

## Processing Logic

### Node Flow (19 Nodes Total)

**INIT Nodes (1-3):** Webhook receive from HSI, payload parse

**Tier Assignment (4-6):** Deterministic Code Node
- Severity 1-3 + Star Rating ≥4 → **T1** (positive, low-severity)
- Severity 4-7 OR Star Rating 3 → **T2** (ambiguous, moderate-severity)
- Severity 8-10 OR Star Rating ≤2 → **T3** (negative, high-severity)

**T1 Branch (7-11):** Template Engine (No AI)
- Query Template Library NocoDB table
- Filter by: `tier = 'T1'` AND `domain = [Domain from HSI]`
- Select template using deterministic logic:
  - Seed = hash(HSI Record ID) → consistent selection per review
  - Prevents same template for all T1 reviews
- Populate template with: Guest name (if available), specific detail from review, domain
- **Token cost: 0** (no AI call)

**T2/T3 Branch (12-17):** Claude API (Governance-Constrained)
- System prompt: Generate response draft based on HSI behavioral narrative
- Governance constraint: Acknowledge guest experience, express appreciation, do NOT prescribe operational fixes
- Input: Review text, Behavioral Narrative, Operator Insight, Severity Score
- Output: Response draft (150-250 words)
- **Token cost: ~2,500 per record**

**BRA Status Update (18):** PATCH HSI NocoDB table
- Update `BRA Status` field to: `T1-Generated` / `T2-Generated` / `T3-Generated`

**Output (19):** Write to BRA NocoDB table, pass to RDA webhook

---

## NocoDB Schema

**Table ID:** `mwqejw7swhd2cf4`

**Fields:**
- BRA Record ID (auto-increment, primary key)
- ALA Record ID (traceability)
- HSI Record ID (upstream reference)
- Hospitality Signal ID (from HSI)
- Client ID
- **Tier Assignment** (T1 / T2 / T3, deterministic from severity + star rating)
- **Template ID** (for T1 records only, NULL for T2/T3)
- **Response Draft** (BRA output: template-populated for T1, Claude-generated for T2/T3)
- **Generation Method** (Template / Claude)
- **Error Log** (field ID: `c3crsub617hnuee`) - Captures failures without blocking pipeline
- Created At (timestamp)

---

## Tier Assignment Logic (Deterministic)

**Runs in Code Node before template selection or Claude call:**

```javascript
const severity = items[0].json.signalSeverityScore; // From HSI
const starRating = items[0].json.starRating; // From ALA pass-through

let tier;

// T1: Positive, low-severity
if (severity <= 3 && starRating >= 4) {
  tier = 'T1';
}
// T3: Negative, high-severity
else if (severity >= 8 || starRating <= 2) {
  tier = 'T3';
}
// T2: Ambiguous, moderate-severity (catch-all)
else {
  tier = 'T2';
}

return { tier };
```

**Distribution (observed in pilot tests):**
- **T1:** ~70-80% of reviews (most guests satisfied, minor issues)
- **T2:** ~15-20% (mixed experiences, moderate concerns)
- **T3:** ~5-10% (serious problems, low ratings)

**Why this matters:** T1 uses zero AI tokens (deterministic templates). T2/T3 use Claude.

**Pipeline token efficiency:** 70-80% of reviews cost 0 tokens at BRA layer.

---

## T1 Template Engine (Deterministic)

### Template Library Structure

**NocoDB Table ID:** `mafv9by73ebama7`

**Fields per template:**
- Template ID (T1-FoodQuality-001, T1-ServiceQuality-012, etc.)
- Tier (T1 only in current system)
- Domain (Food Quality, Service Quality, Ambiance, etc.)
- Template Text (with placeholders: `{guest_name}`, `{specific_detail}`, `{domain}`)
- Brand Voice Mode (Standard / Warm / Professional / Casual)
- Usage Count (incremented each time template selected)

**Example template:**
Template ID: T1-FoodQuality-003
Tier: T1
Domain: Food Quality
Brand Voice Mode: Warm
Template Text:
"Thank you, {guest_name}, for highlighting {specific_detail}. We're glad our {domain}
team delivered an experience you enjoyed. Your feedback helps us maintain the standards
that matter most to our guests."

---

### Template Selection Algorithm

**Deterministic (not random):**

```javascript
// Query Template Library for T1 + matching domain
const templates = await nocodb.query({
  table: 'Template_Library',
  filter: `tier = 'T1' AND domain = '${domain}'`
});

// Generate seed from HSI Record ID (ensures same template for same review on re-run)
const seed = hashCode(hsiRecordId); // Simple hash function

// Select template using seed modulo template count
const selectedIndex = seed % templates.length;
const selectedTemplate = templates[selectedIndex];

// Increment usage count (track template distribution)
await nocodb.patch({
  table: 'Template_Library',
  recordId: selectedTemplate.id,
  data: { usage_count: selectedTemplate.usage_count + 1 }
});
```

**Why deterministic seed?**
- Re-running same review always selects same template (reproducibility)
- Distributes templates across reviews (prevents "everyone gets Template #1")
- No randomness = audit trail is clear

---

### Template Placeholder Replacement

**After template selection, populate placeholders:**

```javascript
let draft = selectedTemplate.templateText;

// Replace {guest_name} if available
if (guestName) {
  draft = draft.replace('{guest_name}', guestName);
} else {
  // Remove greeting clause if no name
  draft = draft.replace('Thank you, {guest_name}, for', 'Thank you for');
}

// Replace {specific_detail} with actual review detail
const specificDetail = extractSpecificDetail(reviewText, domain);
draft = draft.replace('{specific_detail}', specificDetail);

// Replace {domain} with actual domain
draft = draft.replace('{domain}', domain.toLowerCase());

return { responseDraft: draft, generationMethod: 'Template' };
```

**Example output:**

**Input:**
- Template: "Thank you, {guest_name}, for highlighting {specific_detail}..."
- Guest name: "Sarah"
- Specific detail: "the perfectly seared scallops"
- Domain: "Food Quality"

**Output:**
Thank you, Sarah, for highlighting the perfectly seared scallops. We're glad our food
quality team delivered an experience you enjoyed. Your feedback helps us maintain the
standards that matter most to our guests.

**Token cost: 0** (no AI call, pure string replacement)

---

## T2/T3 Claude Generation (Governance-Constrained)

**Runs only for reviews with moderate-to-high severity or low star ratings.**

### System Prompt (Governance-Critical)
You are a hospitality response drafting assistant. Generate a response to the guest review.
CRITICAL GOVERNANCE RULES:

Acknowledge the guest's experience as described in the Behavioral Narrative
Express genuine appreciation for their feedback
NEVER prescribe operational fixes ("we'll add more servers", "we'll change the menu")
NEVER make promises about future changes
Focus on understanding and acknowledgment, not problem-solving

Inputs:

Review text: {reviewText}
Behavioral Narrative: {behavioralNarrative} (describes guest psychology, not what to fix)
Operator Insight: {operatorInsight} (helps you understand context, don't reference it in draft)
Signal Severity: {severity} (1-10 scale)
Star Rating: {starRating}

Output format:

150-250 words
Professional yet empathetic tone
Address guest directly (use "you" and "your")
If guest name available, open with direct greeting
Focus on acknowledgment and appreciation, not solutions


**Example Claude output (T3 record):**

**Input:**
- Review: "The wait was unacceptable. We had a reservation and still waited 40 minutes. When the food finally came, it was cold. Ruined our anniversary dinner."
- Star Rating: 1
- Behavioral Narrative: "Guest experienced Autonomy violation (loss of control over schedule) combined with high-severity Service Quality and Food Quality pain points. The anniversary context amplified emotional impact. Masked frustration (polite phrasing despite 1-star rating) suggests guest values civility but is deeply dissatisfied."

**Claude output:**
We're truly sorry your anniversary dinner didn't meet the experience you deserved.
A 40-minute wait despite having a reservation, followed by food arriving cold, is
not the standard we hold ourselves to—and we understand how that impacted what
should have been a meaningful celebration.
Your feedback about the wait time and food temperature is important. We recognize
that when a special occasion is involved, every detail matters even more, and we
fell short on both counts during your visit.
We appreciate you taking the time to share this with us, even though the experience
was disappointing. Your perspective helps us understand where we need to improve,
particularly during high-traffic periods and for reservations marking important moments.
Thank you for giving us the opportunity to learn from this. We hope you were able to
celebrate your anniversary in a way that honored the occasion, despite the challenges
you encountered with us.

**Governance check:** No prescriptions. Acknowledges experience, expresses understanding, thanks for feedback. Does NOT say "we'll add more servers" or "we'll fix the kitchen timing."

**Token cost:** ~2,500 tokens (review text + behavioral narrative + Claude response generation)

---

## Token Budget

**Per record (average across T1/T2/T3 distribution):**

| Tier | % of Reviews | Token Cost per Record | Weighted Cost |
|------|--------------|----------------------|---------------|
| **T1** | **75%** | **0** (template) | **0** |
| **T2** | **20%** | **2,500** (Claude) | **500** |
| **T3** | **5%** | **2,500** (Claude) | **125** |
| **Average** | **100%** | | **~625 tokens** |

**At 300 reviews/month:**
- T1 records: 225 × 0 = 0 tokens
- T2 records: 60 × 2,500 = 150K tokens
- T3 records: 15 × 2,500 = 37.5K tokens
- **Total BRA cost: 187.5K tokens/month**

**If all records used Claude (no hybrid):**
- 300 × 2,500 = 750K tokens/month
- **Hybrid saves: 562.5K tokens/month (75% reduction)**

---

## Key Design Decisions

### Why Hybrid (Template + Claude)?

**Decision made:** Chat #18 · March 11, 2026

**Problem:** Using Claude for ALL reviews would cost ~750K tokens/month at pilot scale.

**Solution:** Template engine for T1 (majority), Claude for T2/T3 (minority).

**Rationale:**
- **T1 reviews (70-80%):** Low-severity, positive experiences. Templates work well.
  - Guest already satisfied → acknowledgment + appreciation sufficient
  - Low risk of generic-sounding response (guest is happy, not analyzing quality)
  
- **T2/T3 reviews (20-30%):** Moderate-to-high severity, mixed or negative. Claude required.
  - Nuanced situations require custom responses
  - Guest dissatisfied → template would feel dismissive
  - High risk of brand damage if response feels robotic

**Trade-off:** Accept template predictability for T1 to save 75% of BRA token cost.

**Quality gate:** RDA refines ALL drafts (T1, T2, T3) with brand voice, so templates get personalized downstream.

---

### Why BRA Doesn't Generate Final Drafts

**BRA = initial draft only.**

**RDA = final draft with brand voice integration.**

**Separation rationale:**
- **BRA:** Fast, cost-efficient initial draft (templates for T1, governance-constrained Claude for T2/T3)
- **RDA:** Slower, higher-quality refinement (brand voice, conditional guest name logic, governance re-check)

**If BRA generated final drafts:**
- Would need to query Client Config for brand voice settings (adds complexity to hybrid logic)
- Would need to implement conditional guest name handling (duplicates RDA work)
- T1 templates would need brand-specific variations (template library explodes from 50 templates to 200+)

**Current architecture cleaner:** BRA focuses on tier-appropriate content generation, RDA focuses on brand voice personalization.

---

## Downstream Handoff

**BRA → RDA:**
- Response Draft (template-populated for T1, Claude-generated for T2/T3)
- Tier Assignment (T1/T2/T3)
- Hospitality Signal ID (for traceability)
- Generation Method (Template / Claude)
- All HSI data (Behavioral Narrative, Severity Score, etc.)

**RDA refines BRA draft with:**
- Brand voice integration (client-specific phrases, tone adjustments)
- Conditional guest name handling (personalized greetings)
- Final governance check (ensure no prescriptive language)

---

## Related Documents

- **Changelog:** [SCX_BRA_CHANGELOG.md](SCX_BRA_CHANGELOG.md)
- **Schema:** [BRA_Schema.md](BRA_Schema.md)
- **Upstream Agent:** [../HSI/SCX_HSI_HOW_v3.md](../HSI/SCX_HSI_HOW_v3.md)
- **Downstream Agent:** [../RDA/SCX_RDA_HOW_v3.1.md](../RDA/SCX_RDA_HOW_v3.1.md)
- **Template Library:** NocoDB table `mafv9by73ebama7`

---

## n8n Workflow Details

**Workflow Name:** SCX-BRA  
**Trigger:** Webhook POST from HSI  
**Credentials:** NocoDB xc-token, Anthropic API key (for T2/T3 only)

**Critical Rules:**
- Tier assignment runs BEFORE branching (deterministic Code Node)
- T1 branch: Template query + string replacement (zero AI calls)
- T2/T3 branch: Claude API call with governance prompt
- BRA Status field in HSI table updated via PATCH (not INSERT)
- Error Log captures template selection failures or Claude API errors
- Claude API manual headers: `anthropic-version:2023-06-01` + `Content-Type:application/json`

---

**End of BRA HOW v3.2**
