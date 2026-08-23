# SCX_BRA_HOW_v3.3

**Agent Name:** BRA (Brand Response Agent)  
**Version:** 3.3 (Chat #74–#77 integration)  
**Last Updated:** Chat #77 · April 19, 2026  
**Model:** Hybrid (T1 deterministic template, T2/T3 Claude claude-sonnet-4-6)  
**Status:** Complete & verified — 19 nodes operational · All Chat #74 payload additions integrated  
**Quality Baseline:** 87% (verified in RDA audit; BRA component solid at upstream)

---

## 1. PURPOSE

BRA — **Brand Response Architect** — generates response strategy and initial draft for every review after HSI assigns behavioral tier and severity. 

BRA operates under a **locked governance principle:** DETECT and INTERPRET signal only. NEVER prescribe operational actions. The client's management team applies their operational judgment to BRA's strategic recommendations.

BRA uses a **hybrid approach:**
- **T1 (~70-80% of reviews):** Deterministic template selection using seeded hash — zero AI cost
- **T2/T3 (~20-30%):** Claude API with governance-constrained prompt — ~2,500 tokens/record

**Critical:** BRA is NOT the final draft agent. BRA creates initial strategy-informed drafts. **RDA refines BRA output with brand voice, guest name logic, and final governance audit.** This separation ensures BRA stays lean (token-efficient, fast) while RDA owns the personalization layer.

---

## 2. INPUT CONTRACT — UPSTREAM SOURCE (HSI WEBHOOK)

**Trigger:** HSI Webhook POST (fire-and-forget)

**Payload structure — 28 fields total:**

### BRA Strategy Outputs from HSI (14 fields)

| # | Field | Type | Req | BRA Use |
|---|-------|------|-----|---------|
| 1 | HSI Record ID | Integer | YES | Primary FK — idempotency + NocoDB write + HSI PATCH |
| 2 | Preliminary Response Tier | T1/T2/T3 | YES | **PRIMARY PROMPT SWITCH** — deterministic or Claude branch |
| 3 | Signal Severity Score | 1–10 (num) | YES | Context for Claude prompt (T2/T3) — urgency framing |
| 4 | Behavioral Narrative | LongText | YES | Rich HSI analysis — BRA feeds this to Claude for T2/T3 |
| 5 | Behavioral Interpretation | LongText | NO | T2/T3 only — full signal context for governance-constrained prompt |
| 6 | Stability Axis | LongText | YES | Recurrence framing — informs response strategy |
| 7 | Emotional Axis | Text | YES | Primary emotional dimension response must acknowledge |
| 8 | Pain Axis | Text | YES | Operational pain point — BRA acknowledges in draft |
| 9 | Urgency Level | Enum (Immediate/Standard) | YES | Controls SLA framing in draft |
| 10 | Commercial Risk Flag | Boolean | YES | true = escalate to RDA Pending-Elevated |
| 11 | Solution Category | Text | NO | Context for BRA framing (e.g., "acknowledgment only" vs. "service recovery") |
| 12 | Masked Emotion Hypothesis | LongText | NO | T2/T3 only — Claude context |
| 13 | Signal Type | Text | YES | Positive/Negative/Ambiguous/Mixed — affects template selection |
| 14 | Domain | Text | YES | Food Quality, Service Quality, Ambiance, etc. — **TEMPLATE LOOKUP KEY** |

### EIP + Trace Pass-Throughs (from pipeline memory — Chat #74 additions)

| # | Field | Type | Req | BRA Use | **Chat #74 Addition** |
|---|-------|------|-----|---------|----------------------|
| 15 | Emotion Hypothesis | LongText | NO | Claude context enrichment T2/T3 drafts | ✅ NEW v3.3 |
| 16 | Keywords | Text | NO | Specificity signals for Claude — prevents generic language | ✅ NEW v3.3 |
| 17 | Enriched Emotion Tag | Text | NO | EIP dictionary enriched label — Claude context | |
| 18 | Enriched Pain Point | Text | NO | EIP dictionary enriched label — pain acknowledgment precision | |
| 19 | Intensity Level | Enum (Critical/High/Moderate/Low) | NO | Urgency framing in BRA draft | |
| 20 | ALA Record ID | Integer | YES | Traceability chain — stored in BRA NocoDB | |
| 21 | EIP Record ID | Integer | YES | Direct EIP traceability — stored in BRA NocoDB | |
| 22 | ESS Record ID | Integer | YES | Emotional Signal Stabilizer link — stored in BRA NocoDB | |
| 23 | Client ID | Text | YES | **LOCKED Chat #74** — required field, travels through all agents | ✅ CRITICAL v3.3 |
| 24 | Reviewer Handle | Text | NO | Guest name from ALA — used in greeting logic RDA Layer | ✅ NEW v3.3 |
| 25 | lang | Text (en/es) | YES | Language tag — record-level, deterministic branch selector | |
| 26 | Star Rating | Number (1–5) | YES | Severity context — affects tier routing | |
| 27 | Raw Text (from ALA) | LongText | YES | Original review — specific detail extraction for templates | |
| 28 | (Reserved) | | | | |

**CRITICAL ARCHITECTURE NOTE (Chat #74):**
- **client_id must originate in ALA CSV source.** It cannot be added mid-pipeline.
- **emotion_hypothesis, keywords, reviewer_handle** travel in the rich webhook payload — no additional NocoDB GETs required.
- **All 28 fields carry forward key-by-key** (no spread operator) through all 19 BRA nodes into RDA via Step 18 webhook.

---

## 3. TIER ASSIGNMENT LOGIC — DETERMINISTIC (Chat #74 HSI FIX)

**This is the PRIMARY prompt switch.** Tier routing happens in HSI (not BRA), but BRA respects the tier and routes execution to either template engine (T1) or Claude (T2/T3).

### Routing From HSI

**Chat #74 Tier Routing Fix:** Mixed Signal always routes to T2. Negative Signal (High or Moderate intensity) always routes to T2. Only Positive and Negative Low default to T1.

| Signal Type | Intensity | Routing | BRA Branch |
|-------------|-----------|---------|-----------|
| **Positive Signal** | Any | T1 | Template Engine |
| **Negative Signal** | **Critical** | **T3** | **Claude API** |
| **Negative Signal** | **High** | **T2** | **Claude API** |
| **Negative Signal** | **Moderate** | **T2** | **Claude API** |
| **Negative Signal** | Low | T1 | Template Engine |
| **Mixed Signal** | Any | **T2** | **Claude API** |
| **Ambiguous Signal** | Moderate–Critical | T2–T3 | Claude API |

**Distribution (from pilot batches):**
- T1: ~70-80% (Positive Low/Moderate, Negative Low)
- T2: ~15-20% (Mixed, Negative Moderate, Ambiguous)
- T3: ~5-10% (Negative Critical)

**BRA's role:** Accept HSI tier assignment as FINAL. Do not recalculate. Route to appropriate processing branch (template or Claude).

---

## 4. T1 TEMPLATE ENGINE (DETERMINISTIC, ZERO AI COST)

### Template Library Overview

**NocoDB Table ID:** `mafv9by73ebama7`  
**Record Count:** 21 templates (10 Negative, 6 Positive, 5 Mixed)  
**Version:** v3.0 (locked)

**Schema:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Template ID | Text | YES | T1-FoodQuality-001, T1-ServiceQuality-012, etc. |
| Template Name | Text | YES | Human-readable label for audit trail |
| Signal Category | SingleSelect | YES | Positive / Negative / Mixed |
| Domain | SingleSelect | YES | Food Quality, Service Quality, Ambiance, Staff Behavior, etc. (locked 13 domains) |
| Core | LongText | YES | Base response text with {placeholders} |
| Utility | LongText | NO | Internal notes on when template is most effective |
| Signal-Based Consideration | LongText | NO | Context (never written to output — internal reasoning only) |

**Example Template Record:**

| Field | Value |
|-------|-------|
| Template ID | T1-FoodQuality-003 |
| Template Name | "Specific Dish Praise" |
| Signal Category | Positive |
| Domain | Food Quality |
| Core | "Thank you, {guest_name}, for highlighting {specific_detail}. The {domain} at {restaurant_name} matters deeply to our team. We're proud it delivered that experience, and your feedback helps us stay accountable to the standards our guests expect." |
| Utility | "Best for reviews praising a specific dish or menu item. Personalizes acknowledgment. Works for 4–5 star reviews." |

---

### Template Selection Algorithm (Deterministic Hash Seeding)

**Step 1: Query template library**
```
Filter: (Signal Category eq "Positive") AND (Domain eq "{domain_from_hsi}")
```

**Step 2: Generate seed from HSI Record ID**
```javascript
const seed = hashCode(hsiRecordId); // Deterministic hash function
```

**Why hash seeding?**
- Re-running same review = always same template (audit reproducibility)
- Distributes templates evenly (prevents template #1 for all reviews)
- No randomness = deterministic, predictable, auditable

**Step 3: Select template**
```javascript
const selectedIndex = Math.abs(seed) % templates.length;
const selectedTemplate = templates[selectedIndex];
```

**Step 4: Placeholder replacement**

```javascript
let draft = selectedTemplate.core;

// {guest_name} — safe null check
if (reviewerHandle && reviewerHandle !== 'Anonymous' && !reviewerHandle.includes('review')) {
  draft = draft.replace('{guest_name}', reviewerHandle);
} else {
  draft = draft.replace('Thank you, {guest_name}, for', 'Thank you for');
}

// {specific_detail} — extracted from review text, domain-sensitive
const detail = extractSpecificDetail(rawText, domain);
draft = draft.replace('{specific_detail}', detail);

// {restaurant_name} — client-specific (from Client Config)
draft = draft.replace('{restaurant_name}', clientName);

// {domain} — lowercased
draft = draft.replace('{domain}', domain.toLowerCase());

return { responseDraft: draft, generationMethod: 'Template', templateId: selectedTemplate.id };
```

**Example flow:**

**Inputs:**
- HSI Record ID: 4521
- Domain: "Food Quality"
- Reviewer Handle: "Sarah M"
- Raw Text: "The scallops were absolutely perfect..."
- Client Name: "Park Avenue Kitchen by David Burke"

**Hash Seed:** hashCode(4521) = 1847293

**Template count for domain:** 8 templates

**Selected Index:** 1847293 % 8 = 5 → Template ID: T1-FoodQuality-005

**Replacement:**
```
Template: "Thank you, {guest_name}, for highlighting {specific_detail}..."
↓
Output: "Thank you, Sarah M, for highlighting the perfectly seared scallops..."
```

**Token cost: 0** (pure string manipulation, no AI call)

---

## 5. T2/T3 CLAUDE GENERATION (GOVERNANCE-CONSTRAINED)

**Runs only when HSI assigns T2 or T3.** All governance constraints locked — Claude cannot prescribe operational actions.

### System Prompt (Locked Governance)

```
You are the Brand Response Architect for SubtextCX. Your role is to generate a
response strategy and initial draft based on the guest's behavioral signal.

LOCKED GOVERNANCE PRINCIPLE:
SubtextCX DETECTS and INTERPRETS signal. It does NOT prescribe operational actions.
Your response acknowledges the guest's experience, expresses genuine appreciation
for their feedback, and demonstrates understanding of what they experienced.

You do NOT:
- Recommend specific operational fixes ("we'll add more servers", "we'll retrain staff")
- Make promises about future changes
- Suggest specific follow-up actions or timelines
- Use language that commits the business to any action

You DO:
- Acknowledge what the guest experienced as they described it
- Express genuine appreciation for their willingness to share feedback
- Demonstrate understanding of the emotional/operational dimensions
- Invite further dialogue if appropriate (general language, no specific channels)
- Demonstrate respect for the guest's time and perspective

INPUTS:
- Guest Behavioral Narrative: [HSI's full analysis of guest psychology + pain points]
- Emotional Axis: [Primary emotional state the guest was in]
- Pain Axis: [Operational dimension the guest experienced]
- Emotion Hypothesis: [EIP's enriched emotion analysis — what the guest likely felt]
- Keywords: [Specific words/phrases guest emphasized — use these for specificity]
- Signal Severity: [1-10 scale — urgency context only]
- Star Rating: [Proxy for satisfaction level]

OUTPUT FORMAT:
- Length: 150-250 words
- Tone: Professional, empathetic, human (never transactional)
- Address guest directly (use "you" and "your")
- If guest name available in system, open with direct greeting
- If Urgency is "Immediate", flag 48-hour follow-up commitment (no operational timeline)

SPECIFICITY RULE:
Use the Keywords provided to reference specific details from the guest's experience.
Never generic ("we heard your feedback"). Always specific ("the 40-minute wait despite
your reservation, and the cold food when it arrived").

TONE CALIBRATION BY TIER:
T2 (Mixed/Ambiguous): Measured and careful acknowledgment. Preserve appropriate ambiguity.
  Do NOT over-apologize. Do NOT commit to specific fixes.
T3 (Negative/Dignity-Risk): Grave and human tone. Acknowledge the weight of the experience.
  No generic apologies. Focus on restoring dignity through specific, authentic acknowledgment.

CRITICAL RULES:
- NO refund/voucher/discount offers
- NO financial commitments or compensation language
- NO named contact channels or email addresses
- NO placeholder text
- NO "we apologize for any inconvenience" (especially T3)
```

**User Prompt Template:**

```
GUEST BEHAVIORAL NARRATIVE:
[From HSI Behavioral Narrative field]

EMOTIONAL AXIS:
[From HSI Emotional Axis — what the guest felt]

PAIN AXIS:
[From HSI Pain Axis — operational pain point]

EMOTION HYPOTHESIS:
[From EIP — enriched emotion analysis]

KEYWORDS FROM REVIEW:
[From ALA keywords field — specific words guest emphasized]

STAR RATING: [1-5]
SIGNAL SEVERITY: [1-10]
URGENCY LEVEL: [Immediate/Standard]
REVIEWER HANDLE: [Guest name if available]

Generate a response draft that acknowledges this guest's experience with authentic,
specific understanding. Do not prescribe operational fixes. Focus on genuine acknowledgment
and appreciation for their feedback.
```

### Claude Call Parameters

| Parameter | Value |
|-----------|-------|
| Model | claude-sonnet-4-6 |
| max_tokens | 500 (T2/T3) |
| temperature | 0.3 (measured tone) |
| Retry | true |
| Max tries | 3 |
| Wait between tries | 5000ms |

### Output Example (T3 Record)

**Inputs:**
- Star Rating: 1
- Behavioral Narrative: "Guest experienced Autonomy violation (40-min wait despite reservation) combined with Service Quality and Food Quality failures. Anniversary context amplified emotional impact. Polite phrasing masks deep dissatisfaction."
- Emotion Hypothesis: "Frustration masked by civility. Guest values courtesy but is genuinely disappointed. High risk of negative word-of-mouth due to occasion context."
- Keywords: "unacceptable", "waited 40 minutes", "reservation", "cold", "anniversary"

**Claude Output:**

```
We're truly sorry your anniversary celebration didn't receive the thoughtful attention
it deserved. A 40-minute wait despite your reservation, followed by food arriving cold,
disrupted what should have been a meaningful evening.

We understand the specific weight of having a special occasion compromised by service
failures on both timing and food quality. That combination of disappointments — the loss
of control over your schedule and the physical experience of cold food — represents a
genuine failure on our part.

Your feedback about where we fell short is important to us, both because of the occasion
and because these are foundational elements of hospitality. We appreciate your willingness
to share this with us, even though the experience was disappointing.

We hope the rest of your anniversary was able to feel meaningful despite the challenges
you encountered at our restaurant. Thank you for taking the time to communicate your experience.
```

**Governance check:**
- ✅ Acknowledges specific failures (40-min wait, cold food, anniversary)
- ✅ Demonstrates understanding of emotional impact
- ✅ Thanks for feedback
- ✅ NO operational prescriptions ("we'll hire more staff", "we'll fix timing")
- ✅ NO financial offers or commitments
- ✅ NO generic apology ("we apologize for any inconvenience")

**Token cost:** ~2,500 tokens per T2/T3 record

---

## 6. TOKEN EFFICIENCY ANALYSIS (Chat #74 Verified)

### Cost Breakdown

| Tier | % Records | Cost/Record | Records/Month (300 avg) | Monthly Cost |
|------|-----------|------------|------------------------|--------------|
| T1 | 75% | 0 (template) | 225 | 0 tokens |
| T2 | 20% | 2,500 (Claude) | 60 | 150K tokens |
| T3 | 5% | 2,500 (Claude) | 15 | 37.5K tokens |
| **TOTAL** | **100%** | **~625 avg** | **300** | **~187.5K tokens** |

### Comparison: All-Claude vs. Hybrid

| Model | Monthly Cost (300 reviews) | Annual Cost |
|-------|----------------------------|------------|
| All-Claude (no templates) | 750K tokens | 9M tokens |
| Hybrid (BRA v3.3) | 187.5K tokens | 2.25M tokens |
| **Savings** | **562.5K tokens (75% reduction)** | **6.75M tokens** |

**Why this matters:** At scale (1,000+ reviews/month), hybrid saves 2.25M+ tokens annually. Token cost is priced into client tiers, but hybrid ensures token spend aligns with value created (high-tier clients get more Claude, low-tier clients benefit from template efficiency).

---

## 7. STEP-BY-STEP NODE DECOMPOSITION (19 Nodes)

### INIT NODES (1–3)

**Node 1: Webhook Trigger**
- Path: `/webhook/scx-bra`
- Auth: None
- Returns: 200 OK immediately
- Payload: 28-field rich object from HSI

**Node 2: Payload Validation**
- Validates: hsi_record_id (integer, > 0), confirmed_response_tier (T1/T2/T3), client_id (required — Chat #74)
- Throws on: Halt flag (never reaches BRA), missing client_id, invalid tier
- Carries all 28 fields forward key-by-key (no spread operator)

**Node 3: Idempotency Check**
- NocoDB GET: BRA table, filter `(HSI Record ID eq {{$json.hsi_record_id}})`
- If exists: log skip, END
- If not exists: continue to Step 4

### TIER ROUTING (4–5)

**Node 4: Tier Assignment → Branch**
- IF: confirmed_response_tier === "T1" → route to T1 Branch (Nodes 6–10)
- IF: confirmed_response_tier === "T2" or "T3" → route to T2/T3 Branch (Nodes 11–16)

### T1 BRANCH (Nodes 6–10)

**Node 6: Query Template Library**
- NocoDB GET: Template_Library (mafv9by73ebama7)
- Filter: `(Signal_Category eq "{signalType}") AND (Domain eq "{domain}")`
- Returns: array of template records

**Node 7: Template Selection (Deterministic Hash)**
```javascript
const seed = hashCode(items[0].json.hsi_record_id);
const selectedIndex = Math.abs(seed) % templatesArray.length;
const selectedTemplate = templatesArray[selectedIndex];
return [{ json: selectedTemplate }];
```

**Node 8: Placeholder Replacement**
```javascript
let draft = selectedTemplate.core;
// Null-safe guest name check
if (reviewerHandle && reviewerHandle !== 'Anonymous' && !reviewerHandle.includes('review')) {
  draft = draft.replace('{guest_name}', reviewerHandle);
} else {
  draft = draft.replace('Thank you, {guest_name}, for', 'Thank you for');
}
// Domain + detail + client name replacements
draft = draft.replace('{specific_detail}', extractSpecificDetail(rawText, domain));
draft = draft.replace('{domain}', domain.toLowerCase());
draft = draft.replace('{restaurant_name}', clientName);

return [{ json: {
  response_draft: draft,
  generation_method: 'Template',
  template_id: selectedTemplate.id,
  token_cost: 0
} }];
```

**Node 9: Increment Template Usage Count**
- NocoDB PATCH: Template_Library record (selected template ID)
- Update: usage_count += 1
- Audit trail: tracks template distribution across reviews

**Node 10: Merge T1 Output with Payload**
- Attaches response_draft, generation_method, template_id to payload
- Carries all 28 original fields + T1 outputs forward
- Routes to Node 17 (Status Update)

### T2/T3 BRANCH (Nodes 11–16)

**Node 11: Build Claude Request Body**
- Constructs JSON request with:
  - System prompt (locked governance)
  - User prompt (Behavioral Narrative, Keywords, Emotion Hypothesis, etc.)
  - client_id for audit trail
  - Model: claude-sonnet-4-6, max_tokens: 500, temp: 0.3

**Node 12: Claude API Call**
- POST to https://api.anthropic.com/v1/messages
- Credential: Subtext-CX-Anthropic (Header Auth: x-api-key)
- Manual headers: anthropic-version:2023-06-01, Content-Type:application/json
- Retry: true, max tries: 3, wait: 5000ms

**Node 13: Parse Claude Response**
```javascript
const raw = items[0].json?.content?.[0]?.text;
if (!raw || raw.trim().length < 50) throw new Error('Claude response too short');
const responseDraft = raw.trim();
return [{ json: {
  response_draft: responseDraft,
  generation_method: 'Claude',
  token_cost: 2500 // approximate
} }];
```

**Node 14: Validate T2/T3 Draft**
- Checks: minimum 150 words, no refund/voucher language, no placeholder text
- Throws on validation failure → Error Handler

**Node 15: Merge T2/T3 Output with Payload**
- Attaches response_draft, generation_method to payload
- Carries all 28 original fields forward
- Routes to Node 17

**Node 16: Error Node** (conditional)
- Captures Claude API failures
- Logs error to BRA Error Log table
- Halts record (does not write BRA record)

### TERMINAL NODES (17–19)

**Node 17: Update HSI BRA Status**
- NocoDB PATCH: HSI table, filter `(HSI_Record_ID eq {{bra_record_id}})`
- Update: `BRA_Status` field → `T1-Generated` / `T2-Generated` / `T3-Generated`
- Acknowledges BRA completion upstream

**Node 18: Build BRA NocoDB Record**
```javascript
// All fields carried from payload
const braRecord = {
  hsi_record_id: payload.hsi_record_id,
  ala_record_id: payload.ala_record_id,
  eip_record_id: payload.eip_record_id,
  ess_record_id: payload.ess_record_id,
  client_id: payload.client_id,  // Chat #74 locked
  confirmed_response_tier: payload.confirmed_response_tier,
  response_draft: payload.response_draft,  // T1 template OR T2/T3 Claude
  generation_method: payload.generation_method,
  template_id: payload.template_id || null,  // T1 only
  emotional_axis: payload.emotional_axis,
  pain_axis: payload.pain_axis,
  stability_axis: payload.stability_axis,
  behavioral_interpretation: payload.behavioral_interpretation,
  created_at: new Date().toISOString()
};
```

**Node 19: Write BRA Record + Trigger RDA**
- POST to NocoDB: BRA table (mwqejw7swhd2cf4)
- Capture returned Record ID: bra_record_id
- Fire webhook to SCX-RDA Step 1 (fire-and-forget, full payload in body)
- Response: 200 OK

---

## 8. NOCODB BRA TABLE SCHEMA

**Table ID:** `mwqejw7swhd2cf4`  
**Record count:** Grows with every review (cumulative)  
**Purpose:** Immutable log of BRA generation — audit trail for all reviews processed

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Id | AutoNumber | Auto | Primary key, auto-increment |
| HSI Record ID | Number | YES | FK → HSI. Idempotency key. |
| ALA Record ID | Number | YES | Traceability to original review |
| EIP Record ID | Number | YES | Traceability to emotion analysis |
| ESS Record ID | Number | YES | Traceability to signal stabilization |
| Client ID | SingleLineText | YES | **Chat #74 locked** — which brand/client this review belongs to |
| Confirmed Response Tier | SingleSelect | YES | T1 / T2 / T3 — from HSI (never recalculated by BRA) |
| Response Draft | LongText | YES | Template-populated (T1) or Claude-generated (T2/T3) |
| Generation Method | SingleSelect | YES | "Template" (T1) / "Claude" (T2/T3) |
| Template ID | SingleLineText | NO | T1 only. Links to Template Library record. NULL for T2/T3. |
| Emotional Axis | Text | YES | What the guest felt — context for RDA |
| Pain Axis | Text | YES | Operational pain — context for RDA |
| Stability Axis | LongText | YES | Recurrence framing — context for RDA |
| Behavioral Interpretation | LongText | NO | Full behavioral context — T2/T3 only |
| Commercial Risk Flag | Checkbox | YES | From HSI — escalates record to RDA Pending-Elevated |
| Emotion Hypothesis | LongText | NO | **Chat #74 addition** — EIP enriched emotion analysis |
| Keywords | SingleLineText | NO | **Chat #74 addition** — specific words guest emphasized |
| Reviewer Handle | SingleLineText | NO | **Chat #74 addition** — guest name from ALA (used by RDA) |
| Created At | DateTime | YES | Timestamp — audit trail |
| Error Log | LongText | NO | Captures generation failures (template query, Claude API) |
| BRA Status Override | SingleLineText | NO | Manual override for testing — normally NULL |

**Indexes:**
- Primary: Id
- Secondary: HSI Record ID (idempotency checks)
- Secondary: Client ID (multi-tenant filtering)
- Secondary: Created At (temporal queries)

---

## 9. GOVERNANCE LOCKS (Chat #74 Verified)

### Locked Principles

**Principle 1: DETECT + INTERPRET only**
- BRA generates response STRATEGY and DRAFT
- BRA acknowledges guest experience and provides signal interpretation
- BRA does NOT prescribe "do X" operational actions
- Exception: RDA internal brief can describe signal meaning (still not prescriptive)

**Principle 2: Tier is final**
- HSI assigns preliminary_response_tier
- BRA respects that tier — never recalculates
- Tier routing determines template vs. Claude, nothing else

**Principle 3: Template seeding is deterministic**
- Same HSI Record ID always produces same template (seed = hash)
- Prevents "randomness" in audit trail
- Enables reproducibility + transparency

**Principle 4: Pass-through fields carry forward**
- client_id, emotion_hypothesis, keywords, reviewer_handle all travel in webhook
- No mid-pipeline additions allowed (Field Traceability locked Chat #74)
- RDA receives full context in a single payload

**Principle 5: Token cost is a feature**
- Hybrid model (templates + Claude) is by design, not a workaround
- T1 efficiency funds T2/T3 quality
- Token spend priced into client tiers

---

## 10. DOWNSTREAM HANDOFF TO RDA

**Trigger:** Node 19 webhook POST to SCX-RDA

**Payload includes:**
- All 28 original fields (client_id, lang, emotion_hypothesis, keywords, reviewer_handle, etc.)
- response_draft (BRA output — template OR Claude)
- generation_method ("Template" OR "Claude")
- template_id (T1 only)
- bra_record_id (NocoDB Record ID)

**RDA's role:**
1. Refine BRA draft with brand voice (Client Config retrieval)
2. Implement guest name logic (opens with greeting if name available)
3. Final governance audit (11-item checklist)
4. Generate internal brief (signal interpretation for client management)
5. Approval gate (human must approve before publication)

**RDA does NOT:**
- Recalculate tier
- Replace strategy
- Change emotional tone
- Re-derive pain points

RDA refines — it does not redesign.

---

## 11. ERROR HANDLING

**Error trigger points:**
- Node 6: Template library query returns empty (domain + signal type has no templates)
- Node 12: Claude API failure (timeout, 429, 5xx)
- Node 14: Validation failure (draft too short, contains prohibited terms)

**Error handler workflow:**
1. **Error Capture:** Code Node logs error details (node name, message, payload)
2. **Error Record:** POST to BRA Error Log (NocoDB, field ID: c3crsub617hnuee)
3. **Error Notification:** Email to Miguel + log to n8n Error Table
4. **Status Update:** PATCH HSI `BRA_Status` → "Error"
5. **Terminal:** Record halted (not written to BRA NocoDB)

**Recovery:** Record re-queued manually OR template library is updated (if domain gap) OR retry workflow re-run with fresh Claude API call.

---

## 12. CONFIGURATION & CREDENTIALS

| Item | Value |
|------|-------|
| Workflow Name | SCX-BRA |
| Webhook Path | /webhook/scx-bra |
| Anthropic Credential | Subtext-CX-Anthropic (Header Auth, Name: x-api-key) |
| NocoDB Credential | xc-token (Header Auth, Name: xc-token) |
| Claude Model | claude-sonnet-4-6 |
| Temperature (T2/T3) | 0.3 |
| Max Tokens (T2/T3) | 500 |
| NocoDB URL (internal) | http://nocodb:8080 |
| NocoDB Base ID | pq249fix22t3ofv |
| BRA Table ID | mwqejw7swhd2cf4 |
| Template Library Table ID | mafv9by73ebama7 |
| Triggered By | HSI Webhook POST (Node 19) |
| Downstream | SCX-RDA Webhook (Node 19) |

---

## 13. QUALITY + OPEN ITEMS

**Quality baseline (from RDA audit):** BRA output component is solid — 87% baseline achieved at RDA layer. BRA correctly routes T1/T2/T3, templates are specific, Claude prompts are governance-aligned.

**Known strengths:**
- ✅ Deterministic template seeding prevents audit trail ambiguity
- ✅ Hybrid model achieves 75% token cost reduction
- ✅ Governance principle consistently applied (DETECT/INTERPRET only)
- ✅ All Chat #74 payload additions (emotion_hypothesis, keywords, reviewer_handle, client_id) successfully carry through BRA → RDA
- ✅ T1 templates personalized with guest name + specific detail (3-layer safety net)

**Known limitations:**
- ~15% of T2 records occasionally over-generalize emotional context (requires Claude prompt refinement post-pilot)
- Template library could expand to 25–30 templates per domain (currently 21 total) for even finer granularity
- Commercial risk flag occasionally missed in edge cases (Governance Flag + Commercial Risk Flag should be redundant in Step 2 validation)

**Open items:**
- [ ] Post-pilot template library expansion (25–30 templates)
- [ ] Claude T2/T3 prompt refinement after 3 months PAK-001 data
- [ ] Spanish template library parity (current: 21 EN templates only)
- [ ] Template usage analytics dashboard (which templates generating best RDA audit scores?)

---

## 14. RELATED DOCUMENTS

- **Upstream Agent:** [../HSI/SCX_HOW_HSI_v3.0.md](../HSI/SCX_HOW_HSI_v3.0.md)
- **Downstream Agent:** [../RDA/SCX_HOW_RDA_v3.1.md](../RDA/SCX_HOW_RDA_v3.1.md)
- **Template Library:** NocoDB table `mafv9by73ebama7` (21 templates, 13 fields)
- **MCD:** [../../../SubtextCX_MCD_v7_4_1.docx](Master Continuity Document)
- **BRA Changelog:** [SCX_BRA_CHANGELOG.md](included below)

---

## CHANGELOG

### v3.3 (Chat #77 · April 19, 2026)

**Major additions:**
- ✅ Integrated Chat #74 additions: emotion_hypothesis, keywords, reviewer_handle, client_id payload fields
- ✅ Updated tier routing logic per HSI Chat #74 fix: Mixed Signal → T2 (always), Negative High/Mod → T2
- ✅ Expanded governance documentation (locked DETECT/INTERPRET principle)
- ✅ Added Chat #74 field traceability table (28-field payload structure)
- ✅ Clarified template seeding as deterministic (no randomness)
- ✅ Updated error handling pipeline
- ✅ Added quality baseline (87%, verified at RDA audit layer)
- ✅ Documented all 19 nodes with Chat #74 context

**Minor updates:**
- Formalized T2/T3 Claude prompt structure (governance rules explicit)
- Expanded token efficiency analysis (75% cost reduction)
- Added Spanish support roadmap

---

### v3.2 (Chat #73 · March 25, 2026)

- Full pipeline completion: 19 nodes verified
- Hybrid model (template + Claude) locked
- Template Library v3.0 (21 templates)
- T1 deterministic seeding implemented

---

### v3.1 (Chat #50 · February 2026)

- Initial hybrid architecture design
- Template + Claude branching logic

---

**Subtext CX · BRA HOW v3.3 · Chat #77 · April 19, 2026 · Solofella LLC**

---

## END OF UPDATED BRA HOW DOCUMENT v3.3

✅ **Status:** Complete. All Chat #74 additions integrated. Tier routing HSI fix reflected. Governance principle locked. 28-field payload structure documented. Error handling detailed. Quality baseline noted.

**This updated v3.3 should now be committed to:**
```
https://github.com/Solofella/scx-knowledge/blob/main/agents/BRA/SCX_BRA_HOW_v3.3.md
```
