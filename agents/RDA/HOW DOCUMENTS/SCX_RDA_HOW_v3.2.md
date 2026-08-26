# SCX_RDA_HOW_v3.2

**SUBTEXT CX · SOLOFELLA LLC**  
**HOW DOCUMENT — RDA**  
**Response Drafting Agent**  
Complete Step Decomposition · Node Logic · Code · Prompts · Field Contracts  
v3.2 · April 21, 2026 · Chat #77

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0 | Initial build — Chat #49 / Session #6 / March 20, 2026 |
| v2.0 | Chat #58 — Audit remediation + NocoDB rationalisation: rich BRA payload, lang handling corrected, Spanish commercial scan terms added, NocoDB table reduced from 20 to 16 fields. Node count: 20 → 18. |
| v3.0 | Chat #74-75 — Five-call generation architecture (Steps 7a-7h). Single Claude call replaced by Opening Constructor, Body Builder, SEO/Governance, Internal Brief, and Audit calls. Node count: 18 → 23. |
| v3.1 | Chat #75 (Apr 2026) — Five-prompt synchronicity audit. Governing principle added to all five Claude prompts. Priority ordering for T1 conditional leads. Guest name three-layer safety net (Steps 7a, 7e, 9b). Audit expanded to 11-item checklist. Opening variation mechanism (Step 9a-2 + audit item 10). Quality baseline confirmed at 87%. |
| v3.2 | Chat #77 (Apr 21, 2026) — Added Open Item: SCX-Sheet-Sync OAuth credential failure. Google Service Account solution documented. No changes to RDA workflow itself. |

---

**PURPOSE:** This document specifies every step, sub-step, code block, prompt, field, and output for the RDA (Response Drafting Agent) n8n workflow SCX-RDA. A developer or Miguel must be able to build SCX-RDA from this document alone with zero prior context.

---

## Summary Grid

| Property | Value |
|----------|-------|
| **Workflow Name** | SCX-RDA |
| **Model** | claude-sonnet-4-6 — all 5 Claude calls |
| **Pipeline Position** | 6th agent — final per-record automated agent |
| **Claude Calls / Record** | 5 — opening, body, SEO/gov, internal brief, audit |
| **Trigger Source** | BRA Webhook — rich payload (Halt never fires) |
| **Post-Call Scan** | Deterministic prohibited-term scan — EN + ES |
| **Total n8n Nodes** | 23 main + 2 brand voice init + 3 error = 28 |
| **NocoDB Table Fields** | 20 — RDA-produced data + audit fields |
| **Human Approval** | MANDATORY — no draft reaches any platform without human decision |
| **Approval SLA** | 48 hours (locked Chat #42) |
| **Quality Baseline** | 87% — Chat #75 April 2026 |
| **Client Config** | Brand Voice retrieved via NocoDB GET at session init |
| **Opening Variation** | Step 9a-2: 2 recent drafts fetched per record per client |
| **Audit Checklist** | 11 items — compliance + formula + opening + guest name |
| **Doc Version** | v3.2 — April 21, 2026 |

---

## 1. AGENT PURPOSE

### RDA — Response Drafting Agent

Translates BRA's response strategy into governed draft language calibrated to the response tier and the client's brand voice. RDA produces two outputs per record: a public-facing response and an internal signal brief for the client management team. RDA does not decide what to do — it decides how to say it.

**The response must still read as written by a person, not optimized by a machine.** Every Claude call in RDA carries this as its governing principle.

**TERMINAL AGENT:** RDA is the final per-record automated agent. No draft produced by RDA reaches any platform before a human makes an explicit approval decision. Claude's scope ends at draft generation.

### RDA PRODUCES

- Public Response Draft — platform-formatted, calibrated to tier and brand voice
- Internal Follow-Up Draft — signal interpretation brief for the client's management team
- Commercial Commitment Flag — deterministic post-call prohibited-term scan (EN + ES)
- Approval Status — Pending / Pending-Elevated (deterministic, pre-human)
- Audit Passed flag + Audit Failed Items — 11-item editorial audit result

### RDA DOES NOT

- Decide response strategy — that is BRA
- Re-classify emotion, pain, or tier — all upstream
- Override governance flags — Halt records never reach RDA
- Generate refund offers, vouchers, or discount commitments
- Publish anything — approval gate is structural, not advisory
- Prescribe operational actions in the internal brief
- Fetch the BRA record from NocoDB — rich payload arrives in webhook

---

## 2. INPUT CONTRACT — TWO SOURCES

**v3 ARCHITECTURE:** BRA sends a rich payload — all BRA strategy fields and EIP pass-throughs arrive in the webhook body. RDA does NOT fetch the BRA record from NocoDB. Brand Voice is retrieved via Client Config NocoDB GET at session init and carried forward through all 23 nodes via explicit key-by-key assignment. No spread operator used anywhere in the pipeline.

### Source 1 — BRA Webhook Payload

#### Part A — BRA Strategy Outputs (14 fields)

| # | Field | Type | Req | RDA Use |
|---|-------|------|-----|---------|
| 1 | BRA Record ID | Integer | YES | Primary FK — idempotency + NocoDB write + PATCH BRA complete |
| 2 | Confirmed Response Tier | T1/T2/T3 | YES | PRIMARY PROMPT SWITCH — determines which Claude instruction set applies |
| 3 | Governance Flag | None/Flag | YES | Flag = Pending-Elevated. Halt records never reach RDA — suppressed at BRA. |
| 4 | Human Review Required | Boolean | YES | true → Pending-Elevated approval status |
| 5 | Emotional Axis | Text | YES | Emotional dimension Claude must address in public draft |
| 6 | Pain Axis | Text | YES | Operational pain Claude must acknowledge |
| 7 | Stability Axis | LongText | YES | Recurrence framing — informs internal brief trajectory framing |
| 8 | Signal-Based Consideration | LongText | YES | Internal brief seed — replaces Human Action Suggestion from v2 |
| 9 | Tone Style | Enum (4) | YES | Draft register selector — Empathetic-Formal / Empathetic-Warm / Neutral-Professional / Dignity-Restorative |
| 10 | Urgency Level | Enum (3) | YES | Immediate = SLA note in internal draft |
| 11 | Behavioral Interpretation | LongText or null | NO | T2/T3 only — full signal context for internal draft |
| 12 | Commercial Risk Flag | Boolean | YES | true → Pending-Elevated regardless of other flags |
| 13 | Solution Type | Text | YES | Response category context — included in internal brief |
| 14 | lang | Text (en/es) | NO | PRIMARY language instruction — record-level tag from ALA via pipeline |

#### Part B — EIP + Trace Pass-Throughs (from workflow memory)

| # | Field | Type | Req | RDA Use |
|---|-------|------|-----|---------|
| 15 | Enriched Emotion Tag | Text | NO | Claude context enrichment for T2/T3 drafts |
| 16 | Enriched Pain Point | Text | NO | Pain acknowledgment precision in public draft |
| 17 | Pain Point Sub-Category | Text | NO | T3 dignity framing precision |
| 18 | Intensity Level | Enum (4) | NO | Urgency framing in internal draft |
| 19 | Signal Type | Text | NO | Mixed Signal / Positive / Negative / Ambiguous |
| 20 | HSI / ESS / EIP / ALA Record IDs | Integer | YES | Chain traceability — carried in memory, stored in RDA NocoDB table (v3 addition) |
| 21 | Emotion Hypothesis | LongText | NO | Narrative emotional state description from EIP |
| 22 | Keywords | Text | NO | Signal keywords from ALA — used by Claude for specificity |
| 23 | Reviewer Handle | Text | NO | Guest name from ALA — drives opening construction in Step 7a |

### Source 2 — NocoDB Client Configuration (session-cached GET)

Brand Voice is retrieved at session init — before the Claude prompt is constructed. If already loaded, zero NocoDB calls. A new client requires only a new NocoDB Client Config record. No structural code changes.

| # | Field | Type | Req | RDA Use |
|---|-------|------|-----|---------|
| 1 | Client ID | Text | YES | Stored in RDA NocoDB record — audit trail for which brand voice was used |
| 2 | Tone Descriptors | LongText | YES | 3–5 adjectives. e.g. 'chef-driven, warm, polished, unpretentious, New York' |
| 3 | Vocabulary Restrictions | LongText | NO | Words never to use in public responses |
| 4 | Formality Level | Number 1–5 | YES | 1=highly casual, 5=highly formal. Controls sentence structure. |
| 5 | Person Preference | Text | YES | 'we' / brand name / 'our team' — controls first-person reference |
| 6 | Brand Phrases To Include | LongText | NO | Preferred phrases. ONE per draft maximum. Only when contextually natural. |
| 7 | Brand Phrases To Avoid | LongText | NO | Banned phrases — stronger than Vocabulary Restrictions |
| 8 | Language (fallback) | Text (en/es) | NO | FALLBACK ONLY — record-level lang from pipeline takes priority |
| 9 | SEO Keywords | LongText | NO | Weave 1–2 naturally into public draft via Step 7e |
| 10 | Approval Contact Email | Email | YES | Who receives the approval email from Step 18. Required before first client run. |

---

## 3. TIER-SPECIFIC DRAFT INSTRUCTIONS

The Confirmed Response Tier is the primary Claude prompt switch. T1, T2, and T3 receive different instruction sets — not just different tone — because they represent fundamentally different response situations. Tier is set by HSI and confirmed by BRA. RDA never re-classifies.

| Tier | Situation | Public Response Instructions | Internal Brief Instructions |
|------|-----------|------------------------------|----------------------------|
| **T1** | Stable. Standard feedback. High confidence. | Warm acknowledgment. Conditional opening lead based on signal content (named staff, guest word, occasion, dish, belonging, gratitude). 2–3 sentences max. Specific to this guest. Minor criticism acknowledged if present. | Signal interpretation only. Trajectory framing. Behavioral risk or loyalty indicator. No operational directives. |
| **T2** | Moderate instability. Masked/ambiguous signal. | Measured opening with Hi or Hello + guest name. Name what the guest experienced. Acknowledge without asserting full understanding. Preserve ambiguity. Contact invitation required. 2–4 sentences. | Full behavioral context from BRA. Trajectory framing. Masked emotion hypothesis if present. No prescriptive language. |
| **T3** | High-stakes dignity. Severe failure. Discrimination or humiliation language. | Hello + guest name. Grave tone. Restore dignity through specificity. Acknowledge actual harm — not a category. No generic apology. Offer direct private contact. 3–5 sentences. | Full situation brief. Legal/brand risk note if applicable. Trajectory framing. No operational directives. |

### Absolute Governance Constraints — All Tiers

**PROHIBITED:**
- Specific refund amounts or offers
- Voucher / complimentary / discount commitments
- Specific follow-up timelines not pre-approved
- Any language that could bind the business to a financial commitment
- Any email address, URL, or placeholder text

**ACCEPTABLE:**
- Empathetic acknowledgment
- Commitment to internal review
- Invitation for direct communication (generic language, no named channel)
- Non-specific follow-up
- General expression of concern

**T3 ONLY:**
- Never use 'We apologize for any inconvenience.'
- Dignity-Restorative tone requires specific, human acknowledgment of the experience described.
- Opening must be Hello + guest name.
- Tone is grave — not warm, not transactional.

---

## 4. FIVE-CALL CLAUDE ARCHITECTURE — PROMPT SPECIFICATION

**v3 ARCHITECTURE:** The single-call model used in v1–v2 produced attention dilution. The five-call model assigns each concern to a focused, isolated Claude call. Each call carries a governing principle: produce output that reads as written by a thoughtful person — not generated by a machine.

### Call 1 — Opening Constructor (Step 7b) · temp: 0 · max_tokens: 150

**Governing principle:** produce a single opening sentence that feels written by a thoughtful person — not generated by a machine. Every word must earn its place.

**System prompt:**

```
Your role: produce a single opening sentence that feels written by a thoughtful
person — not generated by a machine. Every word must earn its place.

You are the Opening Constructor for SubtextCX RDA. Your only job is to produce
the single opening sentence of a public response draft for a hospitality review.
You do not write the full response. You produce one sentence only.

TIER RULES:

For T1 positive signals: select the opening construction based on what the
review contains. Apply the priority order below when multiple conditions present:

  Priority 1 — NAMED STAFF: If the review names a specific staff member:
    → PLEASURE LEAD: "[Name], hearing that [staff name] made that kind of
      impression on your visit — that is something our team will carry."
  Priority 2 — GUEST WORD LEAD: If the guest quotes or emphasizes a specific
    word (e.g. "always", "impeccable", "perfect", "world-class", "fantastic"):
    → "[Name], that word '[their word]' is exactly what our team works to earn."
  Priority 3 — OCCASION LEAD: birthday, anniversary, celebration, date night:
    → "[Name], a [occasion] that feels [their description] — that is what our
      team is here for."
  Priority 4 — SPECIFIC DETAIL LEAD: specific dish or drink that stood out:
    → "[Dish name] deserves that kind of praise, [Name] — our kitchen will
      hear this."
  Priority 5 — BELONGING LEAD: new to area or discovering restaurant for
    the first time:
    → "[Name], welcome to the neighborhood — we are glad you found us."
  Priority 6 — GRATITUDE LEAD (fallback): warm and positive, none of the above:
    → "Thank you for this, [Name] — it means more than a quick reply can say."
  IF no guest name available: open with a specific detail from the signal.

For T2 signals: open with Hi or Hello followed by the guest name. What follows
must reflect the specific signal — not a formula. Tone is measured and careful.
  IF no guest name available: open with a specific detail from the signal.

For T3 signals: open with Hello followed by the guest name. Tone is grave and
human — not warm, not transactional. Acknowledge the weight of what the guest
experienced. IF no guest name available: open with the specific nature of the
harm described.

CRITICAL RULES:
- Return one sentence only. No preamble. No explanation. No JSON wrapper.
- The guest name MUST appear in the opening sentence for all tiers when available.
- The sentence must end with a period, em dash continuation, or natural punctuation.
- Never use "Knowing" as an opening word.
- Never use "landed" as the primary acknowledgment verb.
```

**User prompt fields:** RESPONSE TIER, GUEST NAME, SIGNAL TYPE, TONE STYLE, EMOTION HYPOTHESIS, KEYWORDS FROM REVIEW, ENRICHED EMOTION, BEHAVIORAL CONTEXT (conditional).

### Call 2 — Body Builder (Step 7d) · temp: 0.5 · max_tokens: 500

**Governing principle:** complete a response draft that reads as written by a senior hospitality professional — not assembled from parts. The response must feel specific to this guest, this signal, and this moment.

**System prompt (key sections):**

```
CRITICAL OPENING RULE — NON-NEGOTIABLE:
Your response MUST begin with the opening sentence provided in the user prompt,
copied exactly word for word. Do not paraphrase it. Do not improve it. Do not
rewrite it. Your first word must be the first word of that opening sentence.
Failure to use this sentence exactly will invalidate the entire draft.

TIER COMPLETION RULES:
  T1: Write exactly 1-2 sentences after the opening. Reinforce the specific
  detail, acknowledge the team or occasion, close with something specific.
  Total: 2-3 sentences. Never restate the opening.
  T1 MIXED SIGNAL RULE: If review contains minor criticism alongside positive
  sentiment, dedicate one sentence to acknowledging it — name the issue directly.
  If opening references a staff member, that name must appear in body as well.

  T2: 1-3 sentences after opening. Name what the guest experienced. Preserve
  ambiguity. Contact invitation required. Total: 2-4 sentences.

  T3: 2-4 sentences after opening. Acknowledge actual harm — not a category.
  No generic apology. Offer direct private contact. Total: 3-5 sentences.

PADDING TEST: Before finalizing, read each sentence after the opening.
Ask: does this sentence add something the previous sentence did not say?
If no — delete it.

CLOSING RULE: Must derive from something specific in this review — the
occasion, the dish, their frequency, their own words. Never generic.
Never repeat a specific noun already used in the opening.
```

**User prompt fields:** OPENING SENTENCE (exact), RESPONSE TIER, TONE STYLE, LANGUAGE, BRAND VOICE BRIEF (all 7 fields), RESPONSE STRATEGY (5 BRA fields), SIGNAL DETAIL (emotion hypothesis, keywords), BEHAVIORAL CONTEXT (conditional).

### Call 3 — SEO + Governance (Step 7f) · temp: 0 · max_tokens: 600

**Governing principle:** refine a draft that already reads as written by a person. Changes must preserve that quality — never mechanize it. If an SEO keyword disrupts human quality, omit it.

```
TASK 1 — SEO INTEGRATION:
Weave 1-2 contextually relevant keywords naturally. Never forced. Never at the
expense of tone. If no keyword fits naturally, do not add any.

TASK 2 — GOVERNANCE CHECK:
Remove: email address, URL, named contact channels (website, profile, direct
messaging), placeholder text, refund/voucher/discount language,
'We apologize for any inconvenience.'
Generic contact invitations ('please reach out to us directly', 'we would
welcome the opportunity to speak with you') are permitted for T2 and T3.

TASK 3 — GUEST NAME CHECK:
If guest name available and absent from draft:
  T1: insert '[Name],' at start of opening sentence.
  T2/T3: insert 'Hello [Name] —' before existing opening content.
Do not rewrite anything else.
```

**User prompt fields:** DRAFT TO REVIEW, RESPONSE TIER, GUEST NAME, SEO KEYWORDS.

### Call 4 — Internal Brief (Step 7h) · temp: 0 · max_tokens: 400

**Governing principle:** produce a signal interpretation note that reads as written by a thoughtful analyst — not a system-generated summary. Substance over formula. Every sentence must add meaning.

```
WHAT TO INCLUDE:
- What emotional state the guest was in and what drove it
- What the signal reveals about the guest's relationship with the brand
- The stability framing: what does the recurrence context (first visit, repeat
  guest, established pattern) reveal about the trajectory of this guest's
  relationship with the brand?
- Any behavioral risk or loyalty indicator present in the signal

WHAT TO NEVER INCLUDE:
- Recommended actions or operational directives
- Headers like 'Recommended Actions', 'Next Steps', or 'Action Items'
- Prescriptive language telling the team what to do
- Generic filler that restates the obvious

LENGTH: 3-5 sentences. Substantive. No padding.
```

**User prompt fields:** RESPONSE TIER, SIGNAL TYPE, INTENSITY LEVEL, URGENCY, LANGUAGE, SIGNAL ANALYSIS (6 BRA/EIP fields), BEHAVIORAL CONTEXT (conditional).

### Call 5 — Audit (Step 9c) · temp: 0 · max_tokens: 1500

**Governing principle:** evaluate and correct a draft that must read as written by a person. When correcting failures, preserve the human quality of the original. Never introduce language that sounds machine-generated.

**11-item checklist:**

| # | Item | FAIL Condition + FIX |
|---|------|---------------------|
| 1 | NAMED STAFF | Staff name in review absent from or replaced in draft. FIX: insert exact name. |
| 2 | REVIEW DEPTH MATCH | Review >80 words receives zero specific references. Short review padded. FIX: add/remove specific references. |
| 3 | CLOSING SPECIFICITY | Generic formula closing. Closing repeats specific noun from opening. Vague referent ('all of it', 'that same experience'). FIX: rewrite closing only. |
| 4 | LOYALTY SIGNAL | Regular guest (always / every time / weekly / each visit) treated as first-time visitor. FIX: acknowledge ongoing relationship. |
| 5 | CONSTRUCTIVE SUGGESTION | Specific guest suggestion deflected with generic language or ignored. FIX: acknowledge by name. |
| 6 | PROHIBITED CONTENT | Email, URL, named contact channels, placeholder text, 'We apologize for any inconvenience.' FIX: remove. Generic contact invitations permitted for T2/T3. |
| 7 | OVERUSED VERBS | 'landed' as primary acknowledgment verb. FIX: replace with resonated / came through / delivered / felt right / worked / made an impression. |
| 8 | OVERUSED PHRASES | Five flagged phrases: 'means a great deal to our team' / 'our team works hard to' / 'all of it' / 'that kind of evening' / 'exactly what we work toward' / 'Knowing' as opening word. FIX: replace with specific alternative. |
| 9 | BRAND PHRASE COUNT | More than one of: 'Thank you for dining with us' / 'We'd love to welcome you back' / 'We take great pride in every guest experience.' FIX: remove all but the most natural one. |
| 10 | OPENING REPETITION | Opening uses same grammatical structure or lead word as recent 2 drafts. Passes automatically if no recent records. FIX: rewrite opening only. |
| 11 | GUEST NAME PRESENT | Guest name available but absent from draft. FIX: T1: insert '[Name],' at start. T2/T3: insert 'Hello [Name] —' before opening. Do not rewrite anything else. |

**Fallback behavior:** if audit JSON parse fails, Step 9d uses generation_draft from Step 8 with audit_passed: false and 'AUDIT PARSE ERROR' in audit_failed_items. If 3+ audit items fail, prefix is 'FULL REVIEW REQUIRED —' signaling human rewrite needed.

---

## 5. STEP-BY-STEP DECOMPOSITION — 23 NODES

**v3 CHANGE:** Single Claude call replaced by five focused calls (Steps 7a–7h + Step 8). Old Step 7 (Build Claude Prompt), Step 8 (Build Claude Request Body), and Step 9 (Claude API Call) retired. Step 9a-2 added for recent openings fetch. Audit nodes (Step 9b-9d) retained and updated. Node count: 18 → 23.

### STEP 1 — WEBHOOK TRIGGER + PAYLOAD VALIDATION
**n8n:** Webhook Node · Code Node

```javascript
const p = $input.first().json;
if (!p.bra_record_id) throw new Error('Missing bra_record_id');
const braId = parseInt(p.bra_record_id);
if (isNaN(braId)||braId<=0) throw new Error('bra_record_id must be positive integer');
if (p.governance_flag==='Halt')
  throw new Error('Halt record reached RDA. Record: '+braId);
// All 28 payload fields carried forward key-by-key (no spread operator)
return [{ json: {
  bra_record_id: braId,
  confirmed_response_tier: p.confirmed_response_tier,
  governance_flag: p.governance_flag||'None',
  human_review_required: p.human_review_required||false,
  lang: p.lang||null,
  // ... all remaining 23 fields key-by-key
} }];
```

### STEP 2 — IDEMPOTENCY CHECK
**n8n:** NocoDB GET · IF Node

```
// NocoDB GET — RDA Idempotency
Table: RDA  |  Filter: (BRA Record ID,eq,{{$json.bra_record_id}})  |  Limit: 1
// IF: pageInfo.totalRows > 0
// TRUE  → Set Node (log skip) → END
// FALSE → Continue to Step 3
```

### STEP 3 — BRAND VOICE LOAD — CLIENT CONFIG GET
**n8n:** Code Node · NocoDB GET (session-cached)

Session-cached. Zero NocoDB calls after first record. Pattern identical to EIP dictionary load and BRA template load.

```javascript
// Code Node: Brand Voice Load Check
const inp = $input.first().json;
// [INIT-1] NocoDB GET — Client Config table
//   Filter: (Client ID,eq,{{inp.client_id}})
//   Returns: single Client Config record
// [INIT-2] Code Node — parse and build brand_voice object:
//   brand_voice = {
//     Client_ID, tone_descriptors, vocabulary_restrictions,
//     formality_level, person_preference, brand_phrases_include,
//     brand_phrases_avoid, language, seo_keywords, approval_contact_email
//   }
// brand_voice carried forward in payload through all 23 nodes
```

### STEP 4 — BRAND VOICE LOAD — ALREADY PROCESSED (IF TRUE)
**n8n:** IF Node · halt branch

If idempotency check returns existing record: returns empty array to halt execution. Does not throw error. Record is already in NocoDB.

### STEP 5 — BRAND VOICE OBJECT BUILD
**n8n:** Code Node

Parses Client Config NocoDB response. Constructs clean brand_voice object. Resolves lang: record-level lang is PRIMARY, Client Config language is fallback only.

```javascript
const rec = $input.first().json;
const bv = rec.brand_voice;
const lang = rec.lang || bv.language || 'en';
// brand_voice object attached to payload
// lang_resolved set for downstream use
```

### STEP 7a — BUILD OPENING PROMPT
**n8n:** Code Node

Builds the Claude request body for Call 1 (Opening Constructor). Applies guest name filter: handles null, 'Anonymous', handles containing 'review'. Constructs opening_prompt_body as JSON.stringify. Carries all 28 payload fields forward.

### STEP 7b — OPENING CLAUDE CALL
**n8n:** HTTP POST → Anthropic API

```
Method: POST  |  URL: https://api.anthropic.com/v1/messages
Auth: Subtext-CX-Anthropic
Headers: anthropic-version: 2023-06-01, Content-Type: application/json
Body: RAW — {{$json.opening_prompt_body}}
model: claude-sonnet-4-6  |  max_tokens: 150  |  temperature: 0
Retry: true  |  Max tries: 3  |  Wait: 5000ms
```

### STEP 7c — PARSE OPENING + BUILD BODY PROMPT
**n8n:** Code Node

Reads opening sentence from Step 7b response. Validates it is not empty or too short. Builds body_prompt_body for Call 2 (Body Builder). Passes opening_sentence as explicit field for downstream nodes.

```javascript
const raw = response.content?.[0]?.text;
if (!raw || raw.trim().length < 5) throw new Error('Opening call empty.');
const opening_sentence = raw.trim();
// Build body_prompt_body with opening_sentence in user prompt:
// 'OPENING SENTENCE — COPY THIS EXACTLY AS YOUR FIRST LINE:\n"' + opening_sentence + '"'
```

### STEP 7d — BODY CLAUDE CALL
**n8n:** HTTP POST → Anthropic API

```
model: claude-sonnet-4-6  |  max_tokens: 500  |  temperature: 0.5
Body: RAW — {{$json.body_prompt_body}}
```

### STEP 7e — PARSE BODY + BUILD SEO/GOVERNANCE PROMPT
**n8n:** Code Node

Reads body draft from Step 7d. Validates minimum length. Builds seo_prompt_body for Call 3. Passes raw_body_draft as field. Includes GUEST NAME in user prompt for TASK 3 guest name check.

### STEP 7f — SEO/GOVERNANCE CLAUDE CALL
**n8n:** HTTP POST → Anthropic API

```
model: claude-sonnet-4-6  |  max_tokens: 600  |  temperature: 0
Body: RAW — {{$json.seo_prompt_body}}
```

### STEP 7g — PARSE SEO DRAFT + BUILD INTERNAL BRIEF PROMPT
**n8n:** Code Node

Reads SEO-governed draft from Step 7f. This becomes public_response_draft. Builds internal_prompt_body for Call 4 (Internal Brief). Carries opening_sentence and public_response_draft forward.

### STEP 7h — INTERNAL BRIEF CLAUDE CALL
**n8n:** HTTP POST → Anthropic API

```
model: claude-sonnet-4-6  |  max_tokens: 400  |  temperature: 0
Body: RAW — {{$json.internal_prompt_body}}
```

### STEP 8 — PARSE INTERNAL BRIEF + BUILD FINAL OUTPUT
**n8n:** Code Node (was Step 7i)

Reads internal brief from Step 7h. Validates minimum length. Validates public_response_draft is present. Assembles final output object with public_response_draft and internal_followup_draft as clean string fields. This replaces what the old Step 9 (Claude API Call) used to produce.

```javascript
const internal_followup_draft = raw.trim();
if (!inp.public_response_draft || inp.public_response_draft.length < 20)
  throw new Error('public_response_draft missing at Step 8. ala_record_id: '+inp.ala_record_id);
return [{ json: {
  // all 28 payload fields key-by-key
  public_response_draft: inp.public_response_draft,
  internal_followup_draft: internal_followup_draft
} }];
```

### STEP 9a — FETCH ALA RECORD
**n8n:** NocoDB GET

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr/{{$json.ala_record_id}}
Auth: xc-token
// Returns: ALA record including Raw Text field for audit user prompt
```

### STEP 9a-2 — FETCH RECENT RDA DRAFTS
**n8n:** NocoDB GET

**v3 ADDITION:** Fetches 2 most recent public response drafts for this client. Used in Step 9b audit item 10 (OPENING REPETITION). Prevents structural repetition across consecutive records for the same client.

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns
  ?where=(Client%20ID,eq,{{$('Step 2').first().json.client_id}})
  &limit=2&sort=-RDA%20Timestamp
Auth: xc-token
```

### STEP 9b — BUILD AUDIT PROMPT
**n8n:** Code Node

Reads ALA raw text from Step 9a. Reads recent drafts from Step 9a-2 and extracts first sentence of each for opening repetition check. Reads public_response_draft and internal_followup_draft from Step 8. Validates both. Builds 11-item audit system prompt using string concatenation (no template literals — avoids n8n red highlight syntax errors from curly braces). Constructs audit_request_body.

```javascript
// Key reads:
const alaRecord = $('Step 9a - Fetch ALA Record').first().json;
const rawText = alaRecord['Raw Tex'] || alaRecord['Raw Text'] || 'Not available';
const recentRecords = $('Step 9a-2 - Fetch Recent RDA Drafts').first().json;
const inp = $('Step 8 - Build Final Output').first().json;

// Recent openings extraction:
const recentList = recentRecords.list || [];
for (let i = 0; i < recentList.length; i++) {
  const draft = recentList[i]['Public Response Draft'];
  if (draft && draft.length > 10) {
    const firstSentence = draft.split(/[.!—]/)[0].trim();
    if (firstSentence.length > 5) recentOpenings.push(firstSentence);
  }
}

// audit_system_prompt built as string concatenation
// closingBlock (CORRECTION RULES + CRITICAL JSON RULES) built separately
// audit_system_prompt_final = audit_system_prompt + recentOpeningsBlock + closingBlock

// Return fields: generation_draft = inp.public_response_draft
//                generation_internal = inp.internal_followup_draft
```

### STEP 9c — AUDIT CLAUDE CALL
**n8n:** HTTP POST → Anthropic API

```
model: claude-sonnet-4-6  |  max_tokens: 1500  |  temperature: 0
Body: RAW — {{$json.audit_request_body}}
```

### STEP 9d — PARSE AUDIT OUTPUT
**n8n:** Code Node

Reads audit Claude response. Parses JSON. Fallback: if parse fails, uses inp.generation_draft with AUDIT PARSE ERROR note. Threshold: if 3+ items fail, prefixes audit_failed_items with 'FULL REVIEW REQUIRED'. Internal brief always reads from inp.generation_internal — never from audit response.

```javascript
let publicDraft = inp.generation_draft; // fallback
if (raw) {
  try {
    const parsed = JSON.parse(clean);
    if (parsed.final_public_response_draft && parsed.final_public_response_draft.length >= 20) {
      publicDraft = parsed.final_public_response_draft;
      const failedCount = (parsed.failed_items || []).length;
      auditFailedItems = failedCount >= 3
        ? 'FULL REVIEW REQUIRED — ' + (parsed.failed_items || []).join(', ')
        : (parsed.failed_items || []).join(', ') || null;
    }
  } catch(e) {
    auditFailedItems = 'AUDIT PARSE ERROR — invalid JSON from audit call';
  }
}
// internal_followup_draft: inp.generation_internal (never from audit response)
```

### STEP 10 — OUTPUT VALIDATION PASS-THROUGH
**n8n:** Code Node

Validates public_response_draft and internal_followup_draft are present and meet minimum length. Pass-through node — carries all payload fields forward unchanged.

### STEP 11 — COMMERCIAL COMMITMENT SCAN
**n8n:** Code Node (deterministic, post-call)

Scans both drafts for prohibited commercial terms. Deterministic — not left to Claude self-governance. Runs before NocoDB write.

```javascript
const prohibitedEN = [
  'refund','voucher','complimentary','discount','credit',
  'compensation','reimburse','waive','free of charge','no charge'
];
const prohibitedES = [
  'reembolso','descuento','cortesia','gratuito','gratis',
  'sin costo','sin cargo','compensacion'
];
// Both lists applied regardless of lang — bilingual safety net
const bothDrafts = [public.toLowerCase(), internal.toLowerCase()].join(' ');
const flaggedTerms = prohibited.filter(term => bothDrafts.includes(term));
const commercial_commitment_flag = flaggedTerms.length > 0;
```

### STEP 12 — APPROVAL STATUS ASSIGNMENT
**n8n:** Code Node (deterministic)

```javascript
const isElevated = (
  row.commercial_commitment_flag === true ||
  row.governance_flag === 'Flag' ||
  row.human_review_required === true ||
  row.confirmed_response_tier === 'T3'
);
const approval_status = isElevated ? 'Pending-Elevated' : 'Pending';
```

### STEP 13 — RUN ID + TIMESTAMP
**n8n:** Code Node

```javascript
const rda_run_id = 'RDA-' + d + '-' + t + '-' + idx;
// Format: RDA-YYYYMMDD-HHMMSS-NNN
const rda_timestamp = now.toISOString();
```

### STEP 14 — BUILD NOCODB POST BODY
**n8n:** Code Node

Constructs JSON.stringify body for NocoDB write. Includes all 20 RDA table fields. Carries full 28-field payload forward for downstream nodes.

### STEP 15 — NOCODB POST — RDA RECORD
**n8n:** NocoDB POST

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns
Auth: xc-token
Body: RAW — {{$json.nocodb_post_body}}
Content-Type: application/json
```

### STEP 16 — CAPTURE RDA RECORD ID
**n8n:** Code Node

```javascript
const rdaId = $input.first().json?.Id;
if (!rdaId) throw new Error('NocoDB RDA write returned no Record ID. bra_record_id: '
  + $("Step 14 - Build NocoDB POST Body").first().json.bra_record_id);
const inp = $('Step 14 - Build NocoDB POST Body').first().json;
// Reads from Step 14 named reference — not $input — to get full payload
// Adds rda_record_id: rdaId to return object
```

### STEP 17 — PATCH BRA → RDA STATUS COMPLETE
**n8n:** NocoDB PATCH

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mwqejw7swhd2cf4/{{$json.bra_record_id}}
Auth: xc-token
Body: { 'RDA Status': 'Complete' }
```

### STEP 18 — HUMAN APPROVAL NOTIFICATION
**n8n:** Code Node · Email Node

The Human Approval Gate is structural, not advisory. Claude's scope ends at draft generation. No approved draft exists until a human makes an explicit decision.

```javascript
// Subject:
// [ELEVATED — APPROVAL REQUIRED] T# Review Response --- RDA-Run-ID
// [APPROVAL REQUIRED] T# Review Response --- RDA-Run-ID

// Email body structure:
// TIER: T1/T2/T3
// GUEST: reviewer_handle || 'Anonymous'
// APPROVAL STATUS: Pending / Pending-Elevated
// AUDIT PASSED: Yes | No — [failed item names]
// URGENCY, EMOTIONAL AXIS, PAIN AXIS
// PUBLIC RESPONSE DRAFT: [full draft]
// INTERNAL FOLLOW-UP DRAFT: [full brief]
// SIGNAL-BASED CONSIDERATION: [from BRA]
// BEHAVIORAL CONTEXT: [conditional — T2/T3 only]
// ELEVATION REASON: [conditional]
// APPROVAL INSTRUCTIONS: Update Approval Status in NocoDB RDA table.
// 48-HOUR SLA APPLIES.

// approval_contact_email: safe accessor for string or object brand_voice
// (typeof inp.brand_voice === 'string' ? JSON.parse(inp.brand_voice) : inp.brand_voice)
//   ?.approval_contact_email || null
```

**PHASE 1 APPROVAL METHOD:** Client operator updates Approval Status directly in NocoDB interface. No email parsing. No n8n write-back automation. Phase 2 can add Google Sheet approval interface with Apps Script webhook back to n8n.

---

## 6. NODE MAP — SCX-RDA

```
[01] Webhook (POST /webhook/scx-rda) — responds 200 immediately
[02] Code Node — Payload Validation (bra_record_id + Halt check)
[03] NocoDB GET — Idempotency (BRA Record ID exists in RDA?)
[04] IF Node — RDA record already exists?
     |–– TRUE  → Set Node (log skip) → END
     +–– FALSE →
[05] Code Node — Brand Voice Load Check
     | (if not loaded: [INIT-1] NocoDB GET Client Config → [INIT-2] parse + cache)
[06] Code Node — Build Brand Voice Object
[07a] Code Node — Build Opening Prompt
[07b] HTTP POST — Opening Claude Call (temp 0, max_tokens 150)
[07c] Code Node — Parse Opening + Build Body Prompt
[07d] HTTP POST — Body Claude Call (temp 0.5, max_tokens 500)
[07e] Code Node — Parse Body + Build SEO/Governance Prompt
[07f] HTTP POST — SEO/Governance Claude Call (temp 0, max_tokens 600)
[07g] Code Node — Parse SEO Draft + Build Internal Brief Prompt
[07h] HTTP POST — Internal Brief Claude Call (temp 0, max_tokens 400)
[08]  Code Node — Parse Internal Brief + Build Final Output
[09a] NocoDB GET — Fetch ALA Record (raw review text for audit)
[09a-2] NocoDB GET — Fetch 2 Recent RDA Drafts (opening variation)
[09b] Code Node — Build 11-Item Audit Prompt
[09c] HTTP POST — Audit Claude Call (temp 0, max_tokens 1500)
[09d] Code Node — Parse Audit Output (with fallback)
[10]  Code Node — Output Validation Pass-Through
[11]  Code Node — Commercial Commitment Scan (EN + ES)
[12]  Code Node — Approval Status Assignment
[13]  Code Node — RDA Run ID + Timestamp
[14]  Code Node — Build NocoDB POST Body
[15]  NocoDB POST — Write RDA Record (20 fields)
[16]  Code Node — Capture RDA Record ID
[17]  NocoDB PATCH — BRA RDA Status → Complete
[18]  Code Node — Build Approval Notification
      Email Node — Send Approval Email
      Pending         → Standard approval email
      Pending-Elevated → Elevated approval email with elevation reason

── Error Handler ─────────────────────────────────────────────
[ERR1] Error Trigger
[ERR2] Code Node — Build error record + email body
[ERR3] NocoDB POST (RDA error) + PATCH (BRA RDA Status=Error) + Email

TOTAL: 23 main + 2 brand voice init + 3 error = 28 nodes
Claude fires at [07b], [07d], [07f], [07h], [09c] only (5 calls per record)
Commercial Commitment Scan at [11] is deterministic — not Claude
Human Approval Gate at [18] is structural — Claude scope ends at [08]
```

---

## 7. APPROVAL STATUS LIFECYCLE

Approval Status is the authoritative record of where each draft stands. It is the only field that moves after RDA writes the initial record. Published Timestamp is null until approval is recorded AND client operator publishes — it is the MRA anchor.

| Status | Set By | Meaning + Next Action |
|--------|--------|----------------------|
| **Pending** | Step 12 (deterministic) | Draft generated. Awaiting human review. Standard SLA: 48 hours. No elevation triggers present. |
| **Pending-Elevated** | Step 12 (deterministic) | One or more elevation conditions: T3, governance flag, commercial commitment, human review required. Manager-level review required. |
| **Approved** | Human approver | Draft approved without edits. Cleared for publication. |
| **Edited-Approved** | Human approver | Human edited draft before approval. Edited version stored. Original preserved for audit. Cleared for publication. |
| **Not Accepted** | Human approver | Draft not accepted. Reason in Approval Notes. Record closed. |
| **Published** | Client operator (manual) | Response published to platform. Published Timestamp set. Terminal state. MRA anchor. |

---

## 8. NOCODB RDA TABLE — 20 FIELDS

**v3 CHANGE:** ALA Record ID, HSI Record ID, ESS Record ID, EIP Record ID re-added to RDA table (removed in v2, restored in v3 for direct traceability). Audit Passed (Checkbox) and Audit Failed Items (SingleLineText) added. Total: 16 → 20 fields.

| Field Name | NocoDB Type | Req | Default | Notes |
|------------|-------------|-----|---------|-------|
| Id | AutoNumber | Auto | — | Primary key |
| RDA Run ID | SingleLineText | YES | — | RDA-YYYYMMDD-HHMMSS-NNN |
| BRA Record ID | Number | YES | — | FK → BRA table |
| ALA Record ID | Number | YES | — | FK → ALA table — direct traceability (re-added v3) |
| RDA Timestamp | DateTime | YES | — | ISO 8601 UTC |
| Confirmed Response Tier | SingleSelect | YES | — | T1/T2/T3 — kept for approver UX |
| Public Response Draft | LongText | YES | — | Audited or fallback draft. Approver may edit. |
| Internal Follow-Up Draft | LongText | YES | — | Signal interpretation brief. Internal only. |
| Approval Status | SingleSelect | YES | Pending | Pending / Pending-Elevated / Approved / Edited-Approved / Not Accepted / Published |
| Commercial Commitment Flag | Checkbox | YES | false | Deterministic scan Step 11 — EN + ES |
| Elevation Reason | LongText | NO | null | Pipe-delimited elevation triggers. null if Pending. |
| Flagged Terms | SingleLineText | NO | null | Prohibited terms detected. null if clean. |
| Client ID | SingleLineText | YES | — | Which brand voice config was used |
| lang | SingleLineText | NO | null | en/es. Record-level pipeline lang tag. |
| Audit Passed | Checkbox | NO | false | NEW v3 — true if all 11 audit items pass |
| Audit Failed Items | SingleLineText | NO | null | NEW v3 — item names or AUDIT PARSE ERROR or FULL REVIEW REQUIRED |
| Reviewer Handle | SingleLineText | NO | null | Guest name from ALA reviewer handle |
| Published Timestamp | DateTime | NO | null | null until published. MRA anchor. |
| Approval Notes | LongText | NO | null | Human approver comments. Rejection reason codes. |
| Error Log | LongText | NO | null | null on clean write |

**SingleSelect pre-population:** Confirmed Response Tier (3: T1/T2/T3) · Approval Status (6 values as above).

### NocoDB Client Config Table Schema (runtime configuration)

| Field Name | NocoDB Type | Notes |
|------------|-------------|-------|
| Client ID | SingleLineText | Unique client identifier |
| Client Name | SingleLineText | Display name for audit reference |
| Tone Descriptors | LongText | 3–5 adjectives. e.g. 'chef-driven, warm, polished, unpretentious, New York' |
| Vocabulary Restrictions | LongText | Words never to use. e.g. 'per your request, moving forward, at this time' |
| Formality Level | Number (1–5) | 1=highly casual, 5=highly formal. Default: 3 |
| Person Preference | SingleLineText | 'we' / 'our team' / brand name. e.g. 'our team' |
| Brand Phrases To Include | LongText | Preferred phrases. ONE per draft maximum. Only when contextually natural. |
| Brand Phrases To Avoid | LongText | Banned phrases. Stronger than Vocabulary Restrictions. |
| Language (fallback) | SingleLineText | 'en' or 'es'. FALLBACK ONLY. Record-level lang takes priority. Default: en. |
| SEO Keywords | LongText | Weave 1–2 naturally in public draft via Step 7e. Never forced. |
| Approval Contact Email | Email | Who receives the approval email from Step 18. Required before first client run. |

---

## 9. CREDENTIALS + CONFIGURATION

| Item | Value |
|------|-------|
| Workflow Name | SCX-RDA |
| Webhook Path | /webhook/scx-rda |
| Anthropic Credential | Subtext-CX-Anthropic (Header Auth, Name: x-api-key) |
| NocoDB Credential | xc-token (Header Auth, Name: xc-token) |
| Email Credential | Subtext-CX-Email (approval email sender) |
| Claude Model | claude-sonnet-4-6 — all 5 Claude calls |
| anthropic-version header | 2023-06-01 — required on every Anthropic API call |
| NocoDB URL (internal) | http://nocodb:8080 — never localhost or external IP |
| Triggered By | SCX-BRA Step 18 — fire-and-forget POST, rich payload. Never fires if Halt. |
| Triggers | No downstream agent. RDA is the terminal automated agent. Human approval loop only. |
| Approval SLA | 48 hours from email send. Locked Chat #42. |
| Phase 1 Approval Method | Client operator updates Approval Status directly in NocoDB. No email parsing. |
| NocoDB Calls / Record | 4–5: GET idem + INIT (session first only) + GET ALA + GET recent RDA + POST RDA + PATCH BRA |
| Active Clients | PAK-001 (Park Avenue Kitchen by David Burke) — Live. EDO-001 — Q2 2026. AJI-001 — Exploratory. |
| Infrastructure | DigitalOcean n8n-Solofella — NYC3 — Ubuntu 24.04 — 4GB RAM — IP: 161.35.133.49 |

---

## 10. QUALITY BASELINE + OPEN ITEMS

### Quality Baseline — 87% · Chat #75 · April 2026

**What is working at 87%:**

Five-call architecture eliminates attention dilution from single-call model. Opening construction is deterministic with priority ordering. Guest name enforced at three independent checkpoints (Step 7a, Step 7e TASK 3, Step 9b item 11). Named staff enforced at three checkpoints (Step 7a priority 1, Step 7c T1 body instruction, Step 9b item 1). T1 mixed-signal records acknowledge minor criticism. SEO keywords placed naturally or omitted. Commercial commitment scan covers EN + ES. Opening variation mechanism prevents structural repetition across consecutive records per client. Internal brief describes trajectory of guest relationship — not operational actions.

**Known gaps at 87%:**

Full last names in drafts when reviewer_handle includes surname. First-name extraction not yet implemented in Step 7a user prompt. Very short ambiguous reviews (1–5 words) occasionally produce overcompensated body content. Formula checklist in Step 9b requires periodic review after each pilot month as new default patterns emerge.

**Path to 90%+:** two to three weeks of live PAK-001 data with Christine's approval patterns revealing which specific failure types persist at operational volume.

### Open Items

| Item | Status | Notes |
|------|--------|-------|
| First-name extraction from reviewer_handle | Open | Full surnames appearing in drafts. Requires filter in Step 7a user prompt. |
| MRA build | On hold | Pending first pilot data. Table created. 5 pre-build items remain. |
| BCA (Batch Controller Agent) | On hold | Option B design (reads from ALA NocoDB). Post-pilot. |
| subtextcx.com landing page | Not started | Required before outbound prospect contact. |
| Google Sheet approval interface | Phase 2 | Apps Script onEdit → n8n webhook → PATCH RDA NocoDB. |
| OpenTable partner application | Pending | Submit at partners.opentable.com. |
| TripAdvisor developer program | Pending | Apply when ready. |
| EDO-001 Client Config | Not created | Q2 2026 pilot. |
| Phase 1b auto-ingestion | Planned | Google Business Profile API + Yelp Fusion API builds. |
| **SCX-Sheet-Sync OAuth credential failure** | **In Progress (Chat #77)** | **n8n 2.4.6 task runner cannot use OAuth refresh tokens in scheduled workflows. Error: "refreshToken is required" at 7am UTC scheduled run (Apr 20, 2026). Solution: Replace with Google Service Account (non-expiring JSON key). Impacts autonomous operation of approval workflow sheet population. Service Account setup: (1) Create in solofella-cmh-project, (2) Generate JSON key, (3) Share PAK-001 sheet with service account email, (4) Update SCX-Sheet-Sync Step 8 HTTP Request credential, (5) Test next scheduled run. This issue affects the Google Sheet approval workflow that surfaces RDA drafts to Christine, but does not impact RDA workflow itself.** |

**Spanish commercial scan:** both EN and ES prohibited terms applied regardless of record lang — bilingual safety net. Additional Spanish terms specific to a client's brand or legal context may be needed — review with first Spanish-language pilot client.

**48-hour SLA:** Locked Chat #42. Track Time to Approval (RDA KPI) from RDA Timestamp to Approval Status change. If consistently missed, investigate approval workflow with client.

---

**Subtext CX · SCX_HOW_RDA_v3.2 · Chat #77 · April 21, 2026 · Solofella LLC**
