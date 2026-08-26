Chat #89 · June 30, 2026Chat #89 · June 30, 2026

v3.2 fully read. Now producing the updated HOW document reflecting all changes made in Chat #88 and Chat #89.

---

# SCX_RDA_HOW_v4.0

**VRYOH INTELLIGENCE · SOLOFELLA LLC**
**HOW DOCUMENT — RDA**
**Response Drafting Agent**
Complete Step Decomposition · Node Logic · Code · Prompts · Field Contracts
v4.0 · June 30, 2026 · Chat #89

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0 | Chat #49 — Initial build |
| v2.0 | Chat #58 — Audit remediation + NocoDB rationalisation. 20 → 18 nodes. |
| v3.0 | Chat #74–75 — Five-call generation architecture. 18 → 23 nodes. |
| v3.1 | Chat #75 — Five-prompt synchronicity audit. 11-item audit checklist. Quality baseline 87%. |
| v3.2 | Chat #77 — SCX-Sheet-Sync OAuth credential failure documented. No RDA changes. |
| v4.0 | Chat #88–89 — Spanish language deployment complete. Multi-client architecture (6 active clients: PAK-001, AJI-001–005). Brand voice schema rebuilt (v3.2 → current). Client Config 22 fields. Step 6 full rewrite. Step 7a lang_resolved fix + anti-repetition + CARRY LEAD RESTRICTION. Steps 7c, 7e, 7g prompt rewrites. Step 9b: item 1 BODY REQUIREMENT, item 3 disguised generics, item 9 dynamic brand phrases, missing + operator fixed, bv type safety. Product renamed VRYOH Intelligence. |

---

## Summary Grid

| Property | Value |
|----------|-------|
| **Workflow Name** | SCX-RDA |
| **Product Name** | VRYOH Intelligence |
| **Model** | claude-sonnet-4-6 — all 5 Claude calls |
| **Pipeline Position** | 7th agent — final per-record automated agent |
| **Claude Calls / Record** | 5 — opening, body, SEO/gov, internal brief, audit |
| **Trigger Source** | BRA Webhook — rich payload (Halt never fires) |
| **Languages** | English (EN) + Spanish (ES) — record-level routing via lang field |
| **Post-Call Scan** | Deterministic prohibited-term scan — EN + ES |
| **Total n8n Nodes** | 23 main + 2 brand voice init + 3 error = 28 |
| **NocoDB Table Fields** | 20 — RDA-produced data + audit fields |
| **Human Approval** | MANDATORY — no draft reaches any platform without human decision |
| **Approval SLA** | 48 hours (locked Chat #42) |
| **Active Clients** | PAK-001, AJI-001, AJI-002, AJI-003, AJI-004, AJI-005 |
| **Client Config Fields** | 22 fields including Brand Personality, Recovery Protocol, Register, Core Driver, Regional Accent, Differentiator, Guest Feeling, Commitment Restrictions |
| **Opening Variation** | Step 9a-2: 2 recent drafts fetched per client. ANTI-REPETITION RULE in Step 7a. |
| **Audit Checklist** | 11 items — compliance + formula + opening + guest name |
| **Doc Version** | v4.0 — June 30, 2026 |

---

## 1. AGENT PURPOSE

### RDA — Response Drafting Agent

Translates BRA's response strategy into governed draft language calibrated to the response tier and the client's brand voice. RDA produces two outputs per record: a public-facing response and an internal signal brief for the client management team. RDA does not decide what to do — it decides how to say it.

**The response must still read as written by a person, not optimized by a machine.** Every Claude call in RDA carries this as its governing principle.

**TERMINAL AGENT:** RDA is the final per-record automated agent. No draft produced by RDA reaches any platform before a human makes an explicit approval decision. Claude's scope ends at draft generation.

**GOVERNING PRINCIPLE — QUALITY:**
Fewer, broader rules that Claude applies with judgment — not exhaustive lists of permitted and prohibited constructions. Every rule in every prompt establishes a principle. Case-specific enumeration is avoided.

### RDA PRODUCES

- Public Response Draft — platform-formatted, calibrated to tier and brand voice, with deterministic closing signature
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

### Source 1 — BRA Webhook Payload (28 fields)

| # | Field | Type | Req | RDA Use |
|---|-------|------|-----|---------|
| 1 | BRA Record ID | Integer | YES | Primary FK — idempotency + NocoDB write + PATCH BRA |
| 2 | Confirmed Response Tier | T1/T2/T3 | YES | PRIMARY PROMPT SWITCH |
| 3 | Governance Flag | None/Flag | YES | Flag = Pending-Elevated. Halt never reaches RDA. |
| 4 | Human Review Required | Boolean | YES | true → Pending-Elevated |
| 5 | Emotional Axis | Text | YES | Emotional dimension Claude addresses in public draft |
| 6 | Pain Axis | Text | YES | Operational pain Claude acknowledges |
| 7 | Stability Axis | LongText | YES | Recurrence framing — internal brief trajectory |
| 8 | Signal-Based Consideration | LongText | YES | Internal brief seed |
| 9 | Tone Style | Enum (4) | YES | Empathetic-Formal / Empathetic-Warm / Neutral-Professional / Dignity-Restorative |
| 10 | Urgency Level | Enum (3) | YES | Immediate = SLA note in internal draft |
| 11 | Behavioral Interpretation | LongText/null | NO | T2/T3 only |
| 12 | Commercial Risk Flag | Boolean | YES | true → Pending-Elevated |
| 13 | Solution Type | Text | YES | Response category context |
| 14 | lang | Text (en/es) | YES | PRIMARY language instruction — from ALA CSV lang column |
| 15 | Enriched Emotion Tag | Text | NO | Claude context enrichment |
| 16 | Enriched Pain Point | Text | NO | Pain acknowledgment precision |
| 17 | Pain Point Sub-Category | Text | NO | T3 dignity framing |
| 18 | Intensity Level | Enum (4) | NO | Urgency framing in internal draft |
| 19 | Signal Type | Text | NO | Mixed / Positive / Negative / Ambiguous |
| 20–23 | HSI / ESS / EIP / ALA Record IDs | Integer | YES | Chain traceability |
| 24 | Emotion Hypothesis | LongText | NO | Narrative emotional state from EIP |
| 25 | Keywords | Text | NO | Signal keywords from ALA |
| 26 | Reviewer Handle | Text | NO | Guest name — drives Step 7a opening construction |
| 27 | Prior Run ID | Text | NO | BRA run reference |
| 28 | Client ID | Text | YES | Routes to correct Client Config record |

### Source 2 — NocoDB Client Config (22 fields, fetched at Step 5)

**v4.0 CHANGE:** Client Config schema significantly rebuilt. MultiSelect fields return arrays — ms() helper required. Three SingleSelect fields added (Register, Core Driver, Regional Accent). Four new brand voice fields added (Recovery Protocol, Guest Feeling, Commitment Restrictions, Differentiator). Sheet ID and Sheet Tab Name added for SCX-Sheet-Sync routing (RDA ignores these two fields).

| # | Field | NocoDB Type | RDA Use |
|---|-------|-------------|---------|
| 1 | Client ID | SingleLineText | Routing key |
| 2 | Client Name | SingleLineText | Brand signature in closing |
| 3 | Brand Personality | MultiSelect → ms() | tone_descriptors |
| 4 | Response for a negative experience | SingleSelect | vocabulary_restrictions |
| 5 | Formality Level | SingleSelect | formality_level |
| 6 | Person Preference | SingleSelect | person_preference |
| 7 | Brand Phrases To Include | MultiSelect → ms() | brand_phrases_include |
| 8 | Brand Phrases To Avoid | MultiSelect → ms() | brand_phrases_avoid |
| 9 | Language | MultiSelect → ms() | language (bv.language — NOT used for lang_resolved) |
| 10 | Approval Contact Email | Email | Step 18 email routing |
| 11 | SEO Keywords | LongText | Step 7e SEO integration |
| 12 | THE RECOVERY PROTOCOL | SingleSelect | recovery_protocol |
| 13 | Guests feeling after reading a response | MultiSelect → ms() | guest_feeling |
| 14 | Commitments should NEVER be made | MultiSelect → ms() | commitment_restrictions |
| 15 | Differentiator | MultiSelect → ms() | differentiator |
| 16 | THE REGISTER | SingleSelect | the_register |
| 17 | THE CORE DRIVER | SingleSelect | the_core_driver |
| 18 | THE REGIONAL ACCENT | SingleSelect | the_regional_accent |
| 19 | Sheet ID | SingleLineText | SCX-Sheet-Sync only — RDA ignores |
| 20 | Sheet Tab Name | SingleLineText | SCX-Sheet-Sync only — RDA ignores |

**CRITICAL:** `lang_resolved` is NEVER derived from `bv.language`. It is always `(inp.lang || 'en').toLowerCase().trim()` — the record-level lang field from the ALA CSV pipeline payload. bv.language returns "En, Es" for bilingual clients — this would break all language routing.

---

## 3. TIER-SPECIFIC DRAFT INSTRUCTIONS

| Tier | Situation | Public Response | Internal Brief |
|------|-----------|-----------------|----------------|
| **T1** | Stable. Standard feedback. | 2–3 sentences total including opening. Opening lead selected by priority order. Closing derives from review content — never generic. Minor criticism acknowledged if present. | Signal interpretation. Trajectory framing. Behavioral risk or loyalty indicator. No operational directives. |
| **T2** | Moderate instability. Masked/ambiguous signal. | 2–4 sentences total. "Hola [Name] —" in Spanish, "Hi/Hello [Name] —" in English. Name what the guest experienced specifically. Contact invitation required as final sentence. | Full behavioral context. Trajectory framing. Masked emotion hypothesis if present. No prescriptive language. |
| **T3** | High-stakes dignity. Severe failure. | 3–5 sentences total. "Hola [Name] —" in Spanish, "Hello [Name] —" in English. Grave tone. Acknowledge actual harm — not category. Direct private contact invitation. | Full situation brief. Legal/brand risk note if applicable. No operational directives. |

### Absolute Governance Constraints — All Tiers

**PROHIBITED:** Specific refund amounts · voucher / complimentary / discount commitments · specific follow-up timelines not pre-approved · email address · URL · placeholder text · named contact channels.

**T3 ONLY:** Never "We apologize for any inconvenience." · Dignity-Restorative tone · Hello/Hola + guest name required.

---

## 4. FIVE-CALL CLAUDE ARCHITECTURE

### Call 1 — Opening Constructor (Step 7b) · temp: 0 · max_tokens: 150

```
Your role: produce a single opening sentence that feels written by a thoughtful person — not generated by a machine. Every word must earn its place.

You are the Opening Constructor for VRYOH RDA. Your only job is to produce the single opening sentence of a public response draft for a hospitality review.

You do not write the full response. You produce one sentence only.

ANTI-REPETITION RULE — read this before anything else:
Check the RECENT OPENINGS listed in the user prompt. If the construction you would naturally select shares a lead word, grammatical structure, or phrasing pattern with any recent opening — skip that priority and move to the next. No two responses for the same client may open the same way. This rule overrides the priority order below.

TIER RULES:

For T1 positive signals:
Select the opening construction based on what the review contains. Apply the priority order below — stop at the first match not recently used.

  Priority 1 — NAMED STAFF: Review names a specific staff member (server, host, bartender, chef).
    Default construction:
    → "[Name], hearing that [staff name] made that kind of impression on your visit — that is something our team will carry."
    CARRY LEAD RESTRICTION: Use this construction only when the guest's primary signal is emotional warmth with no specific action or moment described. If the review describes a specific quality, behavior, or moment involving the staff member, use the next available alternative:
    → "[Staff name] deserves to hear this, [Name] — and he/she will."
    → "[Name], the way [staff name] [specific action from review] — that does not go unnoticed."
    → "[Name], [staff name] will hear what you wrote — and it will mean something."
    Select whichever has not been used recently. Never repeat a structure already in the recent openings list.

  Priority 2 — GUEST WORD LEAD: Guest quotes or emphasizes a specific word (e.g. "always", "impeccable", "perfect", "world-class", "fantastic", "best", "recommended", "recomendado").
    → "[Name], that word '[their word]' is exactly what our team works to earn."

  Priority 3 — OCCASION LEAD: Review describes a special occasion (birthday, anniversary, celebration, date night).
    → "[Name], a [occasion] that feels [their description] — that is what our team is here for."

  Priority 4 — SPECIFIC DETAIL LEAD: Guest describes a specific dish or drink that stood out.
    → "[Dish name] deserves that kind of praise, [Name] — our kitchen will hear this."

  Priority 5 — BELONGING LEAD: Guest signals they are new to the area or discovering the restaurant for the first time.
    → "[Name], welcome to the neighborhood — we are glad you found us."

  Priority 6 — GRATITUDE LEAD (fallback): Review is warm and positive but matches none of the above.
    → "Thank you for this, [Name] — it means more than a quick reply can say."

  IF no guest name is available: open with a specific detail from the review instead of a name.

For T2 signals:
Open with "Hi [Name] —" or "Hello [Name] —" in English. In Spanish, open with "Hola [Name] —". What follows must reflect the specific signal — not a formula. Tone is measured and careful.
  IF no guest name is available: open with a specific detail from the signal.

For T3 signals:
Open with "Hello [Name] —" in English. In Spanish, open with "Hola [Name] —". Tone is grave and human — not warm, not transactional. Acknowledge the weight of what the guest experienced.
  IF no guest name is available: open with the specific nature of the harm described.

CRITICAL RULES:
- Return one sentence only. No preamble. No explanation. No JSON wrapper.
- Guest name must appear in the opening sentence for all tiers when a name is available.
- The sentence must end with a period, em dash continuation, or natural punctuation.
- Never use "Knowing" as an opening word.
- Never use "landed" as the primary acknowledgment verb.
- Never open with "what you described" or "lo que describes" — these are body constructions, not opening constructions.

LANGUAGE: If lang_resolved is "es", write the opening sentence in Spanish. Apply all tier rules and priority order in Spanish. The quality standard — specific, human, non-formulaic — is identical.
```

**User prompt fields:** RESPONSE TIER, GUEST NAME, SIGNAL TYPE, TONE STYLE, EMOTION HYPOTHESIS, KEYWORDS FROM REVIEW, ENRICHED EMOTION, BEHAVIORAL CONTEXT (conditional), RECENT OPENINGS (from Step 9a-2).

### Call 2 — Body Builder (Step 7d) · temp: 0.5 · max_tokens: 500

```
Your role: complete a response draft that reads as written by a senior hospitality professional — not assembled from parts. The response must feel specific to this guest, this visit, and this moment.

You are the Draft Body Builder for VRYOH RDA. Your job: complete a hospitality response draft using a pre-constructed opening sentence as the first line.

CRITICAL OPENING RULE — NON-NEGOTIABLE:
Your response MUST begin with the opening sentence provided in the user prompt, copied exactly word for word. Do not paraphrase it. Do not improve it. Do not rewrite it. Your first word must be the first word of that opening sentence. Failure to use this sentence exactly will invalidate the entire draft.

INTERNAL LANGUAGE PROHIBITION:
Never use internal pipeline terminology in any public response:
- "señal" used as a classification noun referring to the guest's review — e.g. "tu señal", "esa señal", "la señal del cliente" — replace with "reseña", "visita", "experiencia", or "comentario". Natural idiomatic Spanish usage of "señal" as a metaphor is acceptable; "review", "visit", or "experience" in English.
- "tier", "T1", "T2", "T3" — never reference pipeline classification language.
- "pipeline", "audit", "draft" — never reference system process language.
The response must read as written by a hospitality professional, not a system operator.
When a named individual is referenced, use the correct singular pronoun — he/she based on context. Never default to "they" for a single named person unless the guest's review explicitly signals a non-binary preference.

TIER COMPLETION RULES:

  T1: Write exactly 1-2 sentences after the opening. Total draft including opening: 2-3 sentences maximum. This is a ceiling — not a target.
    The sub-rules below are filters — apply only what the review actually requires.
    T1 SUB-RULES:
      - If the opening references a staff member by name and the body has room, include that name naturally once. Do not add a sentence just to satisfy this.
      - If the review contains a minor criticism alongside overall positive sentiment, acknowledge it by name in one sentence. Do not generalize. Do not add this sentence if no criticism exists.
      - The final sentence must close on something specific to this guest's review — the occasion, the dish, their own words, their visit pattern. If nothing specific exists, a brief sincere acknowledgment is sufficient.

  T2: Write exactly 1-3 sentences after the opening. Total draft including opening: 2-4 sentences maximum. This is a ceiling — not a target.
    Each sentence must add a dimension the previous sentence did not cover. If you cannot add a new dimension — stop.
    Name what the guest experienced specifically — not a category of experience.
    Acknowledge without asserting full understanding. Preserve ambiguity where it exists.
    The final sentence must be a direct invitation to contact using generic language only — no email address, URL, or named channel. This is the only sentence in T2 where a generic construction is permitted.

  T3: Write exactly 2-4 sentences after the opening. Total draft including opening: 3-5 sentences maximum.
    Each sentence must serve one distinct purpose in this order: (1) acknowledge the harm specifically, (2) acknowledge the scope or impact, (3) offer direct private contact.
    Never restate. Never use a generic apology. No "We apologize for any inconvenience."
    Private contact invitation must use generic language only — no email address, URL, or named channel.

PADDING TEST:
After writing, read each sentence after the opening. Ask: if this sentence were removed, would the guest notice something missing? If no — delete it. One well-crafted sentence is better than three that circle the same idea.

TONE: Apply tone_style exactly as provided in the user prompt.
  Empathetic-Formal: Warm but structured. Formal register. No contractions.
  Empathetic-Warm: Conversational, human, genuine. Contractions allowed.
  Neutral-Professional: Measured, factual. Acknowledge without amplifying.
  Dignity-Restorative: Specific, human, non-formulaic. Acknowledge the actual experience. Never generic.

BRAND VOICE:
  - Write in first-person plural using person_preference consistently throughout.
  - Match formality_level: 1 = very casual, 5 = very polished.
  - Include at most ONE phrase from brand_phrases_include per draft. ONE is absolute. Only when it emerges naturally — never as filler. If no phrase fits naturally, omit entirely.
  - Never use any word or phrase from brand_phrases_avoid or vocabulary_restrictions.
  - Never invent brand differentiators, menu items, or cultural claims not present in the guest's review or the BRAND VOICE BRIEF.

HUMAN NATURALNESS:
  - Write as a thoughtful, senior hospitality professional would — not as a system producing output.
  - Each response must feel written specifically for that guest, that visit, that moment.
  - For T2 and T3: name what the guest experienced directly in the body — not just the opening.
  - Short responses are better than padded responses. Say one thing well. A two-sentence response that earns every word is better than a four-sentence response that fills space.

CLOSING RULE:
  The closing sentence must derive from something the guest actually said, described, or experienced in their review — the occasion, the dish, their visit pattern, their own words.
  Never repeat a specific noun already used in the opening sentence.
  If a brand phrase from brand_phrases_include is used as the closing, it must be immediately followed by something specific to this guest — never stand alone.
  Never invent specific content to satisfy this rule. If no closing referent exists in the review, close with one brief sincere sentence — not a formula, not an invented detail.

  The following constructions always fail this rule — regardless of surrounding context:
  - "we hope your next visit brings [anything]" — non-specific
  - "we hope the food brings you back" — non-specific
  - "we hope to see you again soon" — non-specific
  - "Esperamos verte de nuevo pronto" — non-specific in Spanish
  - "we look forward to welcoming you back" — non-specific
  - "we can't wait to do it again" — non-specific
  - Any closing that names content not present in the guest's review — invention, not specificity

  Test before finalizing: what specific word or phrase from THIS guest's review does this closing refer to? If the answer is nothing — replace with a brief sincere acknowledgment.

RETURN the complete draft as plain text only — no JSON, no preamble, no explanation.
Start with the opening sentence exactly as provided, then continue the response.

LANGUAGE: If the LANGUAGE field in the user prompt is "es", write the entire response in Spanish. Apply all tier, tone, brand voice, and governance rules in Spanish. The response must read as written by a senior hospitality professional in Spanish — not as a translation from English.
```

**User prompt fields:** OPENING SENTENCE (exact), RESPONSE TIER, TONE STYLE, LANGUAGE, BRAND VOICE BRIEF (13 fields), RESPONSE STRATEGY (5 BRA fields), SIGNAL DETAIL (emotion hypothesis, keywords), BEHAVIORAL CONTEXT (conditional).

### Call 3 — SEO + Governance (Step 7f) · temp: 0 · max_tokens: 600

```
Your role: refine a draft that already reads as written by a person. Your changes must preserve that quality — never mechanize it. If a SEO keyword disrupts the human quality of the draft, omit it.

You are the SEO and Governance Layer for VRYOH RDA. You receive a completed hospitality response draft. Your job is three tasks only.

Apply tasks in order: SEO first, then Governance on the result, then Guest Name check on the final result.

TASK 1 — SEO INTEGRATION:
If seo_keywords are provided, weave 1-2 of the most contextually relevant keywords naturally into the draft. Keywords must appear as part of a genuine sentence — never listed, never forced, never at the expense of tone or human naturalness. If no keyword fits naturally without disrupting the draft, do not add any.

TASK 2 — GOVERNANCE CHECK:
Verify the draft contains none of the following. If any are present, remove or replace them:
- Any email address
- Any URL
- Any placeholder text such as [contact], [name], [email]
- The phrase "We apologize for any inconvenience"
- Any specific refund amounts, voucher offers, discount commitments, or financial commitments
- Any reference to named contact channels such as "our website", "our profile", "our direct messaging"
Generic contact invitations such as "please reach out to us directly" or "we would welcome the opportunity to speak with you" are permitted for T2 and T3 records — do not remove these.

TASK 3 — GUEST NAME CHECK:
If the GUEST NAME field is not "Not available" and the guest name does not appear anywhere in the draft, insert it in the opening line only:
  T1: insert "[Name]," at the start of the opening sentence.
  T2 and T3: insert "Hello [Name] —" before the existing opening content.
If the name is already present anywhere in the draft — do not touch the opening.

RETURN the final draft as plain text only — no JSON, no preamble, no explanation.
If no changes are needed, return the draft exactly as received.

LANGUAGE: The draft may be in any language. Apply all three tasks in the same language as the draft. Do not translate. Return the draft in its original language.
```

**User prompt fields:** DRAFT TO REVIEW, RESPONSE TIER, GUEST NAME, SEO KEYWORDS.

### Call 4 — Internal Brief (Step 7h) · temp: 0 · max_tokens: 400

```
Your role: produce a signal interpretation note that reads as written by a thoughtful analyst — not a system-generated summary. Substance over formula. Every sentence must add meaning.

You are the Internal Brief Writer for VRYOH RDA. Your job: produce a signal interpretation note for the client's management team describing what this guest signal means — not what to do about it.

WHAT TO INCLUDE:
- What emotional state the guest was in and what drove it — always required.
- What the signal reveals about the guest's relationship with the brand — always required.
- Any behavioral risk or loyalty indicator present in the signal — include only if present in the signal data. If absent, omit.
- The stability framing: what does the recurrence context reveal about the trajectory of this guest's relationship with the brand — include only if recurrence context is available. If this is a first recorded signal with no prior history, state that explicitly in one sentence. Do not fabricate trajectory.

WHAT TO NEVER INCLUDE:
- Recommended actions or operational directives
- Headers like "Recommended Actions", "Next Steps", or "Action Items"
- Prescriptive language telling the team what to do
- Generic filler that restates the obvious

BRAND CONTEXT USAGE:
- Recovery Protocol describes how this client handles negative or mixed signals. Use it to frame the emotional tone of the brief — not as an action directive.
- Commitment Restrictions are what this client never promises publicly. Do not reference these in the brief — they are operational guardrails, not signal intelligence.

LENGTH: 3-5 sentences. Substantive. No padding.

RETURN the internal brief as plain text only — no JSON, no preamble, no explanation.

LANGUAGE: If the LANGUAGE field in the user prompt is "es", write the internal brief in Spanish. Apply all governance and substance rules in Spanish.
```

**User prompt fields:** RESPONSE TIER, SIGNAL TYPE, INTENSITY LEVEL, URGENCY, LANGUAGE, BRAND CONTEXT (Recovery Protocol, Commitment Restrictions), SIGNAL ANALYSIS (6 BRA/EIP fields), BEHAVIORAL CONTEXT (conditional).

### Call 5 — Audit (Step 9c) · temp: 0 · max_tokens: 1500

**11-item checklist:**

| # | Item | FAIL Condition + FIX |
|---|------|---------------------|
| 1 | NAMED STAFF | Staff name in review absent from draft. BODY REQUIREMENT: name must appear in at least one body sentence — not only in the opening. FIX: insert name naturally into one body sentence. Do not rewrite opening. |
| 2 | REVIEW DEPTH MATCH | Review >80 words receives zero specific references. Short review padded. FIX: add/remove specific references. |
| 3 | CLOSING SPECIFICITY | Generic formula standing alone with no specific referent. Closing repeats specific noun from opening. Disguised generics: "we hope to see you back at the table again soon" / "we'd love to make the second one just as memorable" / "more of the same" / "again soon". Exception: brand phrase not standalone if immediately followed by specific guest content. FIX: rewrite closing only. |
| 4 | LOYALTY SIGNAL | Regular guest (always / every time / weekly / each visit) treated as first-time visitor. FIX: acknowledge ongoing relationship. |
| 5 | CONSTRUCTIVE SUGGESTION | Specific guest suggestion deflected or ignored. FIX: acknowledge by name. |
| 6 | PROHIBITED CONTENT | Email, URL, named contact channels, placeholder text, "We apologize for any inconvenience." Generic contact invitations permitted for T2/T3. FIX: remove. |
| 7 | OVERUSED VERBS | "landed" as primary acknowledgment verb. FIX: replace with resonated / came through / delivered / felt right / worked / made an impression. |
| 8 | OVERUSED PHRASES | "means a great deal to our team" / "our team works hard to" / "all of it" / "that kind of evening" / "exactly what we work toward" / "Knowing" as opening word / "makes our day" / "more of the same". FIX: replace with specific alternative. |
| 9 | BRAND PHRASE COUNT | More than one phrase from BRAND PHRASES list (provided dynamically in user prompt) appears in same draft. Spanish draft: evaluate only Spanish-language equivalents. FIX: remove all but most natural one. |
| 10 | OPENING REPETITION | Opening uses same grammatical structure or lead word as recent drafts. Passes automatically if no recent records. FIX: rewrite opening only. |
| 11 | GUEST NAME PRESENT | Guest name available but absent from draft. FIX: T1: insert "[Name]," at start. T2/T3: insert "Hello [Name] —" before opening. |

**Correction rules:** correct only the failing element — never rewrite passing elements. Never introduce content not in the original review. Language of correction must match language of draft.

**JSON return structure:**
```json
{
  "audit_passed": true,
  "failed_items": ["ITEM NAME"],
  "final_public_response_draft": "complete corrected or unchanged draft"
}
```

**User prompt fields:** ORIGINAL GUEST REVIEW, PROPOSED PUBLIC RESPONSE DRAFT, GUEST NAME, RESPONSE TIER, TONE STYLE, BRAND PHRASES (dynamic from bv.brand_phrases_include).

---

## 5. STEP-BY-STEP DECOMPOSITION

### STEP 1 — WEBHOOK + PAYLOAD VALIDATION
Webhook receives BRA trigger. Code Node parses payload. Validates bra_record_id is positive integer. Throws error if governance_flag === 'Halt'. Carries all 28 fields forward key-by-key.

### STEP 2 — PAYLOAD VALIDATION (Step 2)
Validates all required fields present. Returns clean payload object.

### STEP 3 — IDEMPOTENCY CHECK
NocoDB GET — RDA table filtered by BRA Record ID. If record exists (pageInfo.totalRows > 0) — returns empty array to halt execution. No error thrown.

### STEP 4 — ALREADY PROCESSED
IF node — gates FALSE branch with return [].

### STEP 5 — BRAND VOICE LOAD
NocoDB GET — Client Config table filtered by client_id. Returns single Client Config record.

### STEP 6 — BUILD BRAND VOICE OBJECT
**v4.0 FULL REWRITE.** ms() helper handles MultiSelect arrays. All 18 brand voice fields mapped. lang_resolved derived from inp.lang — never from bv.language.

```javascript
const ms = function(val) {
  return Array.isArray(val) ? val.join(', ') : (val || null);
};

const brand_voice = {
  Client_ID:               rec['Client ID'],
  'Client Name':           rec['Client Name'] || null,
  tone_descriptors:        ms(rec['Brand Personality']),
  vocabulary_restrictions: rec['Response for a negative experience'] || null,
  formality_level:         rec['Formality Level'] || '3 — Balanced',
  person_preference:       rec['Person Preference'] || null,
  brand_phrases_include:   ms(rec['Brand Phrases To Include']),
  brand_phrases_avoid:     ms(rec['Brand Phrases To Avoid']),
  language:                ms(rec['Language']) || 'en',
  seo_keywords:            rec['SEO Keywords'] || null,
  approval_contact_email:  rec['Approval Contact Email'] || null,
  recovery_protocol:       rec['THE RECOVERY PROTOCOL - How you handle negative or mixed reviews *'] || null,
  guest_feeling:           ms(rec['Guests feeling after reading a response']),
  commitment_restrictions: ms(rec['Commitments should NEVER be made']),
  differentiator:          ms(rec['Differentiator']),
  the_register:            rec['THE REGISTER - Your default communication style *'] || null,
  the_core_driver:         rec['THE CORE DRIVER - What your responses champion most *'] || null,
  the_regional_accent:     rec['THE REGIONAL ACCENT - Your cultural and geographic tone *'] || null
};
```

### STEP 7a — BUILD OPENING PROMPT
**v4.0 CHANGES:** lang_resolved now `(inp.lang || 'en').toLowerCase().trim()` — bv.language fallback removed. ANTI-REPETITION RULE added. CARRY LEAD RESTRICTION added. "what you described"/"lo que describes" banned in CRITICAL RULES. "Hola" explicit for Spanish T2/T3. bv declaration removed (was unused — caused n8n red highlight).

### STEP 7b — OPENING CLAUDE CALL
temp: 0 · max_tokens: 150 · retryOnFail: true · waitBetweenTries: 5000

### STEP 7c — PARSE OPENING + BUILD BODY PROMPT
**v4.0 CHANGES:** bv type safety added. body_system_prompt full rewrite — INTERNAL LANGUAGE PROHIBITION, pronoun fix, tier ceilings, T1 SUB-RULES as filters, PADDING TEST reframed, CLOSING RULE with prohibited constructions and closing test. Brand Voice Brief expanded to 13 fields.

```javascript
const bv = typeof inp.brand_voice === 'string' ? JSON.parse(inp.brand_voice) : inp.brand_voice;
```

### STEP 7d — BODY CLAUDE CALL
temp: 0.5 · max_tokens: 500 · retryOnFail: true · waitBetweenTries: 5000

### STEP 7e — PARSE BODY + BUILD SEO/GOVERNANCE PROMPT
**v4.0 CHANGES:** bv type safety added. seo_system_prompt updated — VRYOH branding, task sequencing instruction, TASK 3 guest name presence check before insertion.

```javascript
const bv = typeof inp.brand_voice === 'string' ? JSON.parse(inp.brand_voice) : inp.brand_voice;
```

### STEP 7f — SEO/GOVERNANCE CLAUDE CALL
temp: 0 · max_tokens: 600 · retryOnFail: true · waitBetweenTries: 5000

### STEP 7g — PARSE SEO DRAFT + BUILD INTERNAL BRIEF PROMPT
**v4.0 CHANGES:** bv type safety added. internal_system_prompt full rewrite — WHAT TO INCLUDE conditional handling, BRAND CONTEXT USAGE block, VRYOH branding.

```javascript
const bv = typeof inp.brand_voice === 'string' ? JSON.parse(inp.brand_voice) : inp.brand_voice;
```

### STEP 7h — INTERNAL BRIEF CLAUDE CALL
temp: 0 · max_tokens: 400 · retryOnFail: true · waitBetweenTries: 5000

### STEP 8 — BUILD FINAL OUTPUT
Reads internal brief from Step 7h. Validates public_response_draft and internal_followup_draft. Assembles final output object.

### STEP 9a — FETCH ALA RECORD
NocoDB GET — ALA table by ala_record_id. Returns Raw Text for audit user prompt. Field: `alaRecord['Raw Tex'] || alaRecord['Raw Text'] || 'Not available'`

### STEP 9a-2 — FETCH RECENT RDA DRAFTS
NocoDB GET — RDA table filtered by client_id. limit=2. sort=-RDA Timestamp. Returns recent public drafts for opening variation check.

### STEP 9b — BUILD AUDIT PROMPT
**v4.0 CHANGES:** bv declaration added with type safety. Missing + operator in closingBlock fixed. item 1 BODY REQUIREMENT added. item 3 disguised generics added + brand phrase exception. item 8 "makes our day" + "more of the same" added. item 9 hardcoded PAK-001 phrases replaced with dynamic BRAND PHRASES from bv.brand_phrases_include. BRAND PHRASES field added to audit_user_prompt. VRYOH branding.

### STEP 9c — AUDIT CLAUDE CALL
temp: 0 · max_tokens: 1500 · retryOnFail: true · waitBetweenTries: 5000

### STEP 9d — PARSE AUDIT OUTPUT
Reads audit response. Parses JSON. Fallback to generation_draft if parse fails. Deterministic closing signature appended:

```javascript
const closingsEN = ['Warm regards', 'Warmly', 'Cheers!', 'Have the best day!'];
const closingsES = ['Atentamente', 'Cordialmente', 'Hasta pronto', 'Con gusto'];
const closings = (inp.lang_resolved || 'en').toLowerCase() === 'es' ? closingsES : closingsEN;
const closingIndex = inp.bra_record_id % closings.length;
const closing = closings[closingIndex];
const brandSignature = bv?.['Client Name'] || inp.client_id || 'VRYOH Intelligence';
const finalDraft = publicDraft.trim() + '\n\n' + closing + ',\n' + brandSignature;
```

### STEPS 10–18
Unchanged from v3.2. Commercial Commitment Scan (EN + ES), Approval Status Assignment, RDA Run ID, NocoDB POST, Capture Record ID, PATCH BRA Status, Approval Notification Email.

---

## 6. NODE MAP

```
[01] Webhook (POST /webhook/scx-rda)
[02] Code Node — Payload Validation
[03] NocoDB GET — Idempotency
[04] IF Node — Already processed?
     TRUE → END | FALSE → continue
[05] NocoDB GET — Brand Voice Load (Client Config)
[06] Code Node — Build Brand Voice Object (v4.0 rewrite)
[07a] Code Node — Build Opening Prompt (v4.0: lang_resolved fix, anti-repetition)
[07b] HTTP POST — Opening Claude Call (temp 0, max_tokens 150)
[07c] Code Node — Parse Opening + Build Body Prompt (v4.0: bv type safety, prompt rewrite)
[07d] HTTP POST — Body Claude Call (temp 0.5, max_tokens 500)
[07e] Code Node — Parse Body + Build SEO/Gov Prompt (v4.0: bv type safety, task sequencing)
[07f] HTTP POST — SEO/Gov Claude Call (temp 0, max_tokens 600)
[07g] Code Node — Parse SEO Draft + Build Internal Brief (v4.0: bv type safety, prompt rewrite)
[07h] HTTP POST — Internal Brief Claude Call (temp 0, max_tokens 400)
[08]  Code Node — Build Final Output
[09a] NocoDB GET — Fetch ALA Record
[09a-2] NocoDB GET — Fetch Recent RDA Drafts (2 per client)
[09b] Code Node — Build 11-Item Audit Prompt (v4.0: bv, item 1/3/8/9 fixes, + operator fix)
[09c] HTTP POST — Audit Claude Call (temp 0, max_tokens 1500)
[09d] Code Node — Parse Audit + Append Closing Signature
[10]  Code Node — Output Validation
[11]  Code Node — Commercial Commitment Scan (EN + ES)
[12]  Code Node — Approval Status Assignment
[13]  Code Node — RDA Run ID + Timestamp
[14]  Code Node — Build NocoDB POST Body
[15]  NocoDB POST — Write RDA Record
[16]  Code Node — Capture RDA Record ID
[17]  NocoDB PATCH — BRA RDA Status → Complete
[18]  Code Node — Build Approval Notification + Email Node

TOTAL: 23 main + 2 brand voice init + 3 error = 28 nodes
Claude fires at [07b], [07d], [07f], [07h], [09c] only — 5 calls per record
```

---

## 7. APPROVAL STATUS LIFECYCLE

| Status | Set By | Meaning |
|--------|--------|---------|
| Pending | Step 12 | Draft generated. Standard 48-hour SLA. |
| Pending-Elevated | Step 12 | T3, governance flag, commercial commitment, or human review required. |
| Approved | Human | Cleared for publication. |
| Edited-Approved | Human | Human edited before approving. |
| Not Accepted | Human | Draft not accepted. |
| Published | Client operator | Terminal state. Published Timestamp set. MRA anchor. |

---

## 8. NOCODB RDA TABLE — 20 FIELDS

Unchanged from v3.2. See v3.2 for full field list.

**Key note:** `lang` field in RDA table is written from `inp.lang_resolved` — always the record-level pipeline value, never the Client Config Language MultiSelect value.

---

## 9. ACTIVE CLIENT CONFIGURATION

| Client ID | Brand | Locations | Language | Recovery Protocol |
|-----------|-------|-----------|----------|-------------------|
| PAK-001 | Park Avenue Kitchen by David Burke | Manhattan, NY | EN | The Transparent Owner |
| AJI-001 | Aji Ceviche Bar | Orlando, FL | EN + ES | The Gracious Host |
| AJI-002 | Aji Ceviche Bar | Casselberry, FL | EN + ES | The Gracious Host |
| AJI-003 | Aji Ceviche Bar | Sarasota, FL | EN + ES | The Gracious Host |
| AJI-004 | Aji Ceviche Bar | Tampa, FL | EN + ES | The Gracious Host |
| AJI-005 | Aji Ceviche Bar | St. Petersburg, FL | EN + ES | The Gracious Host |

---

## 10. CREDENTIALS + CONFIGURATION

| Item | Value |
|------|-------|
| Workflow Name | SCX-RDA |
| Webhook Path | /webhook/scx-rda |
| Anthropic Credential | x-api-key (Header Auth) |
| NocoDB Credential | xc-token (Header Auth) |
| Claude Model | claude-sonnet-4-6 — all 5 calls |
| anthropic-version header | 2023-06-01 — required on every Anthropic API call |
| NocoDB URL (internal) | http://nocodb:8080 — never localhost |
| Triggered By | SCX-BRA Step 18 — fire-and-forget POST |
| Triggers | None — terminal agent |
| Approval SLA | 48 hours from email send |
| Google Sheets Delivery | SCX-Sheet-Sync — Service Account credential (googleApi) |
| Infrastructure | DigitalOcean NYC3 · Ubuntu 24.04 · 4GB RAM · 161.35.133.49 |

---

## 11. QUALITY BASELINE + OPEN ITEMS

### Quality Baseline — Chat #89 · June 30, 2026

**What is working:**
- Spanish language fully deployed and routing correctly ✅
- lang_resolved fix eliminates English fallback for AJI Spanish records ✅
- Multi-client brand voice dynamic at runtime — 6 clients active ✅
- Recovery Protocol, Register, Core Driver, Regional Accent active in prompts ✅
- Anti-repetition rule operational for sequential records ✅
- CARRY LEAD RESTRICTION reduces "hearing that" overuse ✅
- Closing test principle — "what word from THIS review does this refer to?" — working ✅
- Named staff body requirement in audit catching omissions ✅
- Dynamic BRAND PHRASES in audit item 9 — client-specific ✅
- Closing signature (EN/ES) + Client Name brand signature confirmed working ✅

**Known gaps:**
- "Hearing" overuse within simultaneous batch — architectural constraint. ANTI-REPETITION RULE works for sequential records only. Batch records processed simultaneously have no NocoDB history to check against. Option B (batch_position field) deferred to post-pilot.
- First-name extraction from full reviewer handles — not yet implemented.
- Very short reviews (1–5 words) occasionally over-compensated in body.

**Quality benchmarks established Chat #89:**
- Luis (AJI-001, ES, T1) — Spanish T1 benchmark
- Arlette Acosta (AJI-001, EN, T1) — English T1 benchmark
- John Diaz (PAK-001, EN, T1) — clean T1
- Barkev (PAK-001, EN, T1) — clean T1 with strong GUEST WORD LEAD and specific closing

### Open Items

| Item | Status | Notes |
|------|--------|-------|
| Step 7a — CARRY LEAD RESTRICTION | Deployed Chat #89 | Verify live via GitHub |
| Step 7c — bv type safety | Deployed Chat #89 | Verify live via GitHub |
| Step 7e — bv type safety | Deployed Chat #89 | Verify live via GitHub |
| Step 9b — full rewrite | Deployed Chat #89 | Verify live via GitHub |
| Batch anti-repetition | Open | Architectural fix — batch_position field. Post-pilot. |
| First-name extraction | Open | Full surnames appearing in drafts. Step 7a user prompt filter needed. |
| PAK-001 go-live | ~July 15, 2026 | Eddie (Operations Director) contact |
| AJI-001–005 pilot | Live — June 25, 2026 | 45-day trial, 5 locations |
| MRA build | On hold | Pending pilot data |
| Phase 1b auto-ingestion | Planned | Google Business Profile API + Yelp Fusion API |
| Revenue Impact Decoder | Planned | Prerequisites pending |

---

**VRYOH Intelligence · SCX_HOW_RDA_v4.0 · Chat #89 · June 30, 2026 · Solofella LLC**
