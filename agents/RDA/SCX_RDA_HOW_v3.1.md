# SCX_RDA_HOW_v3.2

**Agent Name:** RDA (Response Drafting Agent)  
**Version:** 3.2  
**Last Updated:** Chat #75 · April 2026  
**Model:** claude-sonnet-4-6 — all 5 Claude calls  
**Status:** Verified operational — 28 nodes complete — 87% quality baseline confirmed  
**Pipeline Position:** 6th agent — final per-record automated agent  
**Claude Calls / Record:** 5 — opening, body, SEO/governance, internal brief, audit  
**NocoDB Table:** `mr1v67cszcklwns` — 20 fields  
**Quality Baseline:** 87% · April 2026

---

## Purpose

RDA translates BRA's response strategy into governed draft language calibrated to the response tier and the client's brand voice. It produces two outputs per record: a public-facing response draft and an internal signal brief for the client management team.

**The response must still read as written by a person, not optimized by a machine. Every Claude call in RDA carries this as its governing principle.**

RDA does not decide strategy, classify signals, or prescribe operational actions. The internal brief describes what a signal means — never what to do about it.

**TERMINAL AGENT:** No draft produced by RDA reaches any platform before a human makes an explicit approval decision. Claude's scope ends at draft generation.

---

## RDA Produces

- Public Response Draft — platform-formatted, calibrated to tier and brand voice
- Internal Follow-Up Draft — signal interpretation brief for the client's management team
- Commercial Commitment Flag — deterministic post-call prohibited-term scan (EN + ES)
- Approval Status — Pending / Pending-Elevated (deterministic, pre-human)
- Audit Passed flag + Audit Failed Items — 11-item editorial audit result

## RDA Does NOT

- Decide response strategy — that is BRA
- Re-classify emotion, pain, or tier — all upstream
- Override governance flags — Halt records never reach RDA
- Generate refund offers, vouchers, or discount commitments
- Publish anything — approval gate is structural, not advisory
- Prescribe operational actions in the internal brief
- Fetch the BRA record from NocoDB — rich payload arrives in webhook

---

## Architecture — Five-Call Generation Model

The single-call model produced attention dilution. The five-call model assigns each concern to a focused, isolated Claude call.

| Call | Step | Purpose | Temp | max_tokens |
|---|---|---|---|---|
| 1 | 7b | Opening Constructor — one sentence only | 0 | 150 |
| 2 | 7d | Body Builder — tier completion | 0.5 | 500 |
| 3 | 7f | SEO + Governance — three tasks | 0 | 600 |
| 4 | 7h | Internal Brief — signal interpretation | 0 | 400 |
| 5 | 9c | Audit — 11-item checklist + correction | 0 | 1500 |

---

## Input Sources

### Source 1 — BRA Webhook Payload (28 fields)

**Part A — BRA Strategy Outputs (14 fields)**

| # | Field | Type | Req | RDA Use |
|---|---|---|---|---|
| 1 | BRA Record ID | Integer | YES | Primary FK — idempotency + NocoDB write + PATCH BRA complete |
| 2 | Confirmed Response Tier | T1/T2/T3 | YES | PRIMARY PROMPT SWITCH |
| 3 | Governance Flag | None/Flag | YES | Flag = Pending-Elevated. Halt records never reach RDA. |
| 4 | Human Review Required | Boolean | YES | true → Pending-Elevated |
| 5 | Emotional Axis | Text | YES | Emotional dimension Claude must address |
| 6 | Pain Axis | Text | YES | Operational pain Claude must acknowledge |
| 7 | Stability Axis | LongText | YES | Recurrence framing — informs internal brief |
| 8 | Signal-Based Consideration | LongText | YES | Internal brief seed |
| 9 | Tone Style | Enum (4) | YES | Empathetic-Formal / Empathetic-Warm / Neutral-Professional / Dignity-Restorative |
| 10 | Urgency Level | Enum (3) | YES | Immediate = SLA note in internal draft |
| 11 | Behavioral Interpretation | LongText or null | NO | T2/T3 only |
| 12 | Commercial Risk Flag | Boolean | YES | true → Pending-Elevated |
| 13 | Solution Type | Text | YES | Response category context |
| 14 | lang | Text (en/es) | NO | PRIMARY language — record-level. Client Config is fallback only. |

**Part B — EIP + Trace Pass-Throughs (9 fields)**

| # | Field | Type | RDA Use |
|---|---|---|---|
| 15 | Enriched Emotion Tag | Text | Claude context enrichment for T2/T3 |
| 16 | Enriched Pain Point | Text | Pain acknowledgment precision |
| 17 | Pain Point Sub-Category | Text | T3 dignity framing precision |
| 18 | Intensity Level | Enum (4) | Urgency framing in internal draft |
| 19 | Signal Type | Text | Mixed Signal / Positive / Negative / Ambiguous |
| 20 | HSI / ESS / EIP / ALA Record IDs | Integer | Chain traceability |
| 21 | Emotion Hypothesis | LongText | Narrative emotional state from EIP |
| 22 | Keywords | Text | Signal keywords from ALA |
| 23 | Reviewer Handle | Text | Guest name from ALA — drives opening construction |

### Source 2 — NocoDB Client Config (session-cached GET)

Table ID: `m95cmabjfyb94ps`

| # | Field | Type | Req | RDA Use |
|---|---|---|---|---|
| 1 | Client ID | Text | YES | Audit trail |
| 2 | Tone Descriptors | LongText | YES | 3–5 adjectives |
| 3 | Vocabulary Restrictions | LongText | NO | Words never to use |
| 4 | Formality Level | Number 1–5 | YES | Controls sentence structure |
| 5 | Person Preference | Text | YES | 'we' / brand name / 'our team' |
| 6 | Brand Phrases To Include | LongText | NO | ONE per draft maximum |
| 7 | Brand Phrases To Avoid | LongText | NO | Banned phrases |
| 8 | Language (fallback) | Text (en/es) | NO | FALLBACK ONLY |
| 9 | SEO Keywords | LongText | NO | Weave 1–2 naturally into public draft |
| 10 | Approval Contact Email | Email | YES | Required before first client run |

---

## Tier-Specific Draft Instructions

| Tier | Situation | Public Response | Internal Brief |
|---|---|---|---|
| T1 | Stable. Standard feedback. High confidence. | Warm acknowledgment. Conditional opening lead. 2–3 sentences. Minor criticism acknowledged if present. | Signal interpretation only. Trajectory framing. No operational directives. |
| T2 | Moderate instability. Masked/ambiguous signal. | Hi or Hello + guest name. Name what guest experienced. Preserve ambiguity. Contact invitation required. 2–4 sentences. | Full behavioral context. Trajectory framing. No prescriptive language. |
| T3 | High-stakes dignity. Severe failure. | Hello + guest name. Grave tone. Acknowledge actual harm — not a category. No generic apology. Offer direct private contact. 3–5 sentences. | Full situation brief. Legal/brand risk note if applicable. No operational directives. |

**PROHIBITED (all tiers):** Specific refund amounts · Voucher/complimentary/discount commitments · Financial commitment language · Email address, URL, or placeholder text  
**T3 ONLY:** Never use "We apologize for any inconvenience." Tone is grave — not warm, not transactional.  
**PERMITTED for T2/T3:** Generic contact invitations without named channel.

---

## Five Claude Calls — Prompt Specification

### Call 1 — Opening Constructor (Step 7b) · temp: 0 · max_tokens: 150

**Governing principle:** produce a single opening sentence that feels written by a thoughtful person — not generated by a machine. Every word must earn its place.

**T1 Priority Order** (first condition that matches wins):

1. **NAMED STAFF → PLEASURE LEAD:** `"[Name], hearing that [staff name] made that kind of impression on your visit — that is something our team will carry."`
2. **GUEST WORD LEAD:** `"[Name], that word '[their word]' is exactly what our team works to earn."`
3. **OCCASION LEAD:** `"[Name], a [occasion] that feels [their description] — that is what our team is here for."`
4. **SPECIFIC DETAIL LEAD:** `"[Dish name] deserves that kind of praise, [Name] — our kitchen will hear this."`
5. **BELONGING LEAD:** `"[Name], welcome to the neighborhood — we are glad you found us."`
6. **GRATITUDE LEAD (fallback):** `"Thank you for this, [Name] — it means more than a quick reply can say."`

**T2:** Hi or Hello + guest name. Specific signal in what follows. Measured and careful.  
**T3:** Hello + guest name. Grave and human tone. Acknowledge weight of experience.  
**No guest name:** Open with specific signal detail (T1/T2) or specific nature of harm (T3).

**Critical rules:** One sentence only. Guest name MUST appear when available. Never use "Knowing" as opening word. Never use "landed" as primary acknowledgment verb.

---

### Call 2 — Body Builder (Step 7d) · temp: 0.5 · max_tokens: 500

**Governing principle:** complete a response draft that reads as written by a senior hospitality professional — not assembled from parts.

**CRITICAL OPENING RULE — NON-NEGOTIABLE:** Your response MUST begin with the opening sentence provided, copied exactly word for word. Failure to use this sentence exactly will invalidate the entire draft.

**Tier Completion Rules:**

- **T1:** 1-2 sentences after opening. Total 2-3 sentences. Reinforce specific detail, acknowledge team or occasion, close with something specific. **T1 MIXED SIGNAL RULE:** If review contains minor criticism alongside positive sentiment, dedicate one sentence to acknowledging it — name the issue directly. If opening references staff member by name, that name must appear in body as well.
- **T2:** 1-3 sentences after opening. Total 2-4 sentences. Name what guest experienced. Preserve ambiguity. Contact invitation required — this closing is required for T2.
- **T3:** 2-4 sentences after opening. Total 3-5 sentences. Each sentence serves distinct purpose: acknowledge harm, acknowledge scope, offer contact. Never restate.

**PADDING TEST:** Before finalizing, each sentence after the opening is evaluated — does it add something the previous sentence did not say? If no — delete it.

**Closing Rule:** Must derive from something specific in the review — the occasion, the dish, their frequency, their own words. Never generic. Never repeat a specific noun from the opening.

---

### Call 3 — SEO + Governance (Step 7f) · temp: 0 · max_tokens: 600

**Governing principle:** refine a draft that already reads as written by a person. If an SEO keyword disrupts human quality, omit it.

**Three tasks:**

1. **SEO INTEGRATION:** Weave 1-2 contextually relevant keywords naturally. Never forced. If no keyword fits, omit entirely.
2. **GOVERNANCE CHECK:** Remove email address, URL, named contact channels, placeholder text, refund/voucher/discount language, "We apologize for any inconvenience."
3. **GUEST NAME CHECK:** If guest name available and absent from draft — T1: insert "[Name]," at start of opening. T2/T3: insert "Hello [Name] —" before existing opening content.

---

### Call 4 — Internal Brief (Step 7h) · temp: 0 · max_tokens: 400

**Governing principle:** produce a signal interpretation note that reads as written by a thoughtful analyst — not a system-generated summary. Substance over formula. Every sentence must add meaning.

**What to include:**
- What emotional state the guest was in and what drove it
- What the signal reveals about the guest's relationship with the brand
- The stability framing: what does the recurrence context (first visit, repeat guest, established pattern) reveal about the trajectory of this guest's relationship with the brand?
- Any behavioral risk or loyalty indicator present in the signal

**What to never include:** Recommended actions, operational directives, headers like "Next Steps", prescriptive language, generic filler.

**Length:** 3-5 sentences. Substantive. No padding.

---

### Call 5 — Audit (Step 9c) · temp: 0 · max_tokens: 1500

**Governing principle:** evaluate and correct a draft that must read as written by a person. When correcting failures, preserve the human quality of the original. Never introduce language that sounds machine-generated.

**11-item checklist:**

| # | Item | FAIL Condition + FIX |
|---|---|---|
| 1 | NAMED STAFF | Staff name in review absent from draft. FIX: insert exact name. |
| 2 | REVIEW DEPTH MATCH | Review >80 words receives zero specific references. Short review padded. FIX: add/remove references. |
| 3 | CLOSING SPECIFICITY | Generic formula closing. Closing repeats specific noun from opening. Vague referent. FIX: rewrite closing only. |
| 4 | LOYALTY SIGNAL | Regular guest treated as first-time visitor. FIX: acknowledge ongoing relationship. |
| 5 | CONSTRUCTIVE SUGGESTION | Specific guest suggestion deflected or ignored. FIX: acknowledge by name. |
| 6 | PROHIBITED CONTENT | Email, URL, named contact channels, placeholder text, prohibited phrases. FIX: remove. |
| 7 | OVERUSED VERBS | "landed" as primary acknowledgment verb. FIX: resonated / came through / delivered / felt right / worked. |
| 8 | OVERUSED PHRASES | Five flagged phrases including "means a great deal to our team" / "Knowing" as opening word. FIX: specific alternative. |
| 9 | BRAND PHRASE COUNT | More than one brand phrase from the three-phrase list. FIX: remove all but most natural one. |
| 10 | OPENING REPETITION | Opening repeats structure from recent 2 drafts. Passes automatically if no recent records. FIX: rewrite opening only. |
| 11 | GUEST NAME PRESENT | Guest name available but absent. FIX: T1 insert "[Name]," at start. T2/T3 insert "Hello [Name] —" before opening. |

**Fallback:** If audit JSON parse fails, Step 9d uses generation_draft with audit_passed: false and AUDIT PARSE ERROR. If 3+ items fail: prefix "FULL REVIEW REQUIRED" — human rewrite needed.

---

## Step-by-Step Decomposition — 28 Nodes

> v3 CHANGE: Single Claude call replaced by five focused calls (Steps 7a–7h + Step 8). Old Step 7, Step 8, and Step 9 retired. Step 9a-2 added. Node count: 18 → 28.

### STEP 1 — Webhook Trigger + Payload Validation · Webhook Node · Code Node

```javascript
const p = $input.first().json;
if (!p.bra_record_id) throw new Error('Missing bra_record_id');
const braId = parseInt(p.bra_record_id);
if (isNaN(braId)||braId<=0) throw new Error('bra_record_id must be positive integer');
if (p.governance_flag==='Halt')
  throw new Error('Halt record reached RDA. Record: '+braId);
// All 28 payload fields carried forward key-by-key (no spread operator)
```

### STEP 2 — Idempotency Check · NocoDB GET · IF Node

```
Table: RDA  |  Filter: (BRA Record ID,eq,{{$json.bra_record_id}})  |  Limit: 1
// IF: pageInfo.totalRows > 0
// TRUE  → Set Node (log skip) → END
// FALSE → Continue to Step 3
```

### STEP 3 — Brand Voice Load — Client Config GET · Code Node · NocoDB GET

Session-cached. Zero NocoDB calls after first record.

```javascript
// [INIT-1] NocoDB GET — Client Config table
//   Filter: (Client ID,eq,{{inp.client_id}})
// [INIT-2] Parse and build brand_voice object
// brand_voice carried forward through all nodes
```

### STEP 4 — Already Processed (IF TRUE) · IF Node

Returns empty array to halt execution if idempotency check returns existing record.

### STEP 5 — Brand Voice Object Build · Code Node

```javascript
const rec = $input.first().json;
const bv = rec.brand_voice;
const lang = rec.lang || bv.language || 'en';
// lang_resolved set — record-level lang is PRIMARY
```

### STEP 7a — Build Opening Prompt · Code Node

Builds Claude request body for Call 1. Applies guest name filter: handles null, 'Anonymous', handles containing 'review'. Returns opening_prompt_body as JSON.stringify.

### STEP 7b — Opening Claude Call · HTTP POST → Anthropic API

```
Method: POST  |  URL: https://api.anthropic.com/v1/messages
Auth: Subtext-CX-Anthropic
Headers: anthropic-version: 2023-06-01, Content-Type: application/json
Body: RAW — {{$json.opening_prompt_body}}
model: claude-sonnet-4-6  |  max_tokens: 150  |  temperature: 0
Retry: true  |  Max tries: 3  |  Wait: 5000ms
```

### STEP 7c — Parse Opening + Build Body Prompt · Code Node

```javascript
const raw = response.content?.[0]?.text;
if (!raw || raw.trim().length < 5) throw new Error('Opening call empty.');
const opening_sentence = raw.trim();
// Build body_prompt_body:
// 'OPENING SENTENCE — COPY THIS EXACTLY AS YOUR FIRST LINE:\n"' + opening_sentence + '"'
```

### STEP 7d — Body Claude Call · HTTP POST → Anthropic API

```
model: claude-sonnet-4-6  |  max_tokens: 500  |  temperature: 0.5
Body: RAW — {{$json.body_prompt_body}}
```

### STEP 7e — Parse Body + Build SEO/Governance Prompt · Code Node

Reads body draft. Validates minimum length. Includes GUEST NAME in user prompt for TASK 3.

### STEP 7f — SEO/Governance Claude Call · HTTP POST → Anthropic API

```
model: claude-sonnet-4-6  |  max_tokens: 600  |  temperature: 0
Body: RAW — {{$json.seo_prompt_body}}
```

### STEP 7g — Parse SEO Draft + Build Internal Brief Prompt · Code Node

SEO-governed draft becomes public_response_draft. Builds internal_prompt_body for Call 4.

### STEP 7h — Internal Brief Claude Call · HTTP POST → Anthropic API

```
model: claude-sonnet-4-6  |  max_tokens: 400  |  temperature: 0
Body: RAW — {{$json.internal_prompt_body}}
```

### STEP 8 — Parse Internal Brief + Build Final Output · Code Node

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

### STEP 9a — Fetch ALA Record · NocoDB GET

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr/{{$json.ala_record_id}}
Auth: xc-token
```

### STEP 9a-2 — Fetch Recent RDA Drafts · NocoDB GET

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns
  ?where=(Client%20ID,eq,{{$('Step 2').first().json.client_id}})
  &limit=2&sort=-RDA%20Timestamp
Auth: xc-token
```

### STEP 9b — Build Audit Prompt · Code Node

```javascript
const alaRecord = $('Step 9a - Fetch ALA Record').first().json;
const rawText = alaRecord['Raw Tex'] || alaRecord['Raw Text'] || 'Not available';
const recentRecords = $('Step 9a-2 - Fetch Recent RDA Drafts').first().json;
const inp = $('Step 8 - Build Final Output').first().json;

const recentList = recentRecords.list || [];
for (let i = 0; i < recentList.length; i++) {
  const draft = recentList[i]['Public Response Draft'];
  if (draft && draft.length > 10) {
    const firstSentence = draft.split(/[.!—]/)[0].trim();
    if (firstSentence.length > 5) recentOpenings.push(firstSentence);
  }
}
// audit_system_prompt built as string concatenation (no template literals)
// closingBlock built separately
// audit_system_prompt_final = audit_system_prompt + recentOpeningsBlock + closingBlock
// generation_draft = inp.public_response_draft
// generation_internal = inp.internal_followup_draft
```

### STEP 9c — Audit Claude Call · HTTP POST → Anthropic API

```
model: claude-sonnet-4-6  |  max_tokens: 1500  |  temperature: 0
Body: RAW — {{$json.audit_request_body}}
```

### STEP 9d — Parse Audit Output · Code Node

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

### STEP 10 — Output Validation Pass-Through · Code Node

Validates both drafts minimum length. Pass-through — carries all payload fields forward unchanged.

### STEP 11 — Commercial Commitment Scan · Code Node (deterministic)

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
const flaggedTerms = prohibited.filter(term => bothDrafts.includes(term));
const commercial_commitment_flag = flaggedTerms.length > 0;
```

### STEP 12 — Approval Status Assignment · Code Node (deterministic)

```javascript
const isElevated = (
  row.commercial_commitment_flag === true ||
  row.governance_flag === 'Flag' ||
  row.human_review_required === true ||
  row.confirmed_response_tier === 'T3'
);
const approval_status = isElevated ? 'Pending-Elevated' : 'Pending';
```

### STEP 13 — RDA Run ID + Timestamp · Code Node

```javascript
const rda_run_id = 'RDA-' + d + '-' + t + '-' + idx;
// Format: RDA-YYYYMMDD-HHMMSS-NNN
const rda_timestamp = now.toISOString();
```

### STEP 14 — Build NocoDB POST Body · Code Node

Constructs JSON.stringify body for NocoDB write. 20 RDA table fields. Carries full 28-field payload forward.

### STEP 15 — NocoDB POST — RDA Record · HTTP POST

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns
Auth: xc-token
Body: RAW — {{$json.nocodb_post_body}}
Content-Type: application/json
```

### STEP 16 — Capture RDA Record ID · Code Node

```javascript
const rdaId = $input.first().json?.Id;
if (!rdaId) throw new Error('NocoDB RDA write returned no Record ID. bra_record_id: '
  + $('Step 14 - Build NocoDB POST Body').first().json.bra_record_id);
const inp = $('Step 14 - Build NocoDB POST Body').first().json;
// Reads from Step 14 named reference — not $input — to get full payload
```

### STEP 17 — PATCH BRA → RDA Status Complete · NocoDB PATCH

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mwqejw7swhd2cf4/{{$json.bra_record_id}}
Auth: xc-token
Body: { 'RDA Status': 'Complete' }
```

### STEP 18 — Human Approval Notification · Code Node · Email Node

```javascript
// Subject:
// [ELEVATED — APPROVAL REQUIRED] T# Review Response --- RDA-Run-ID
// [APPROVAL REQUIRED] T# Review Response --- RDA-Run-ID

// Email body: TIER, GUEST, APPROVAL STATUS, AUDIT PASSED,
// URGENCY, EMOTIONAL AXIS, PAIN AXIS, PUBLIC RESPONSE DRAFT,
// INTERNAL FOLLOW-UP DRAFT, SIGNAL-BASED CONSIDERATION,
// BEHAVIORAL CONTEXT (conditional), ELEVATION REASON (conditional)
// APPROVAL INSTRUCTIONS + 48-HOUR SLA

// approval_contact_email safe accessor:
// (typeof inp.brand_voice === 'string' ? JSON.parse(inp.brand_voice) : inp.brand_voice)
//   ?.approval_contact_email || null
```

**PHASE 1 APPROVAL METHOD:** Client operator updates Approval Status directly in NocoDB. No email parsing. No n8n write-back automation.

---

## Node Map — SCX-RDA

```
[01] Webhook (POST /webhook/scx-rda) — responds 200 immediately
[02] Code Node — Payload Validation (bra_record_id + Halt check)
[03] NocoDB GET — Idempotency (BRA Record ID exists in RDA?)
[04] IF Node — RDA record already exists?
     |-- TRUE  → Set Node (log skip) → END
     +-- FALSE →
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
      Pending          → Standard approval email
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

## Approval Status Lifecycle

| Status | Set By | Meaning |
|---|---|---|
| Pending | Step 12 (deterministic) | Draft generated. Awaiting human review. Standard SLA: 48 hours. |
| Pending-Elevated | Step 12 (deterministic) | T3, governance flag, commercial commitment, or human review required. Manager-level review. |
| Approved | Human approver | Draft approved without edits. Cleared for publication. |
| Edited-Approved | Human approver | Human edited draft before approval. Original preserved for audit. |
| Not Accepted | Human approver | Draft not accepted. Reason in Approval Notes. |
| Published | Client operator (manual) | Response published to platform. Published Timestamp set. MRA anchor. |

---

## NocoDB RDA Table — 20 Fields

**Table ID:** `mr1v67cszcklwns`

| Field Name | NocoDB Type | Req | Default | Notes |
|---|---|---|---|---|
| Id | AutoNumber | Auto | — | Primary key |
| RDA Run ID | SingleLineText | YES | — | RDA-YYYYMMDD-HHMMSS-NNN |
| BRA Record ID | Number | YES | — | FK → BRA table |
| ALA Record ID | Number | YES | — | FK → ALA table — direct traceability (re-added v3) |
| RDA Timestamp | DateTime | YES | — | ISO 8601 UTC |
| Confirmed Response Tier | SingleSelect | YES | — | T1/T2/T3 |
| Public Response Draft | LongText | YES | — | Audited or fallback draft |
| Internal Follow-Up Draft | LongText | YES | — | Signal interpretation brief. Internal only. |
| Approval Status | SingleSelect | YES | Pending | Pending / Pending-Elevated / Approved / Edited-Approved / Not Accepted / Published |
| Commercial Commitment Flag | Checkbox | YES | false | Deterministic scan Step 11 — EN + ES |
| Elevation Reason | LongText | NO | null | Pipe-delimited triggers. null if Pending. |
| Flagged Terms | SingleLineText | NO | null | Prohibited terms detected. null if clean. |
| Client ID | SingleLineText | YES | — | Which brand voice config was used |
| lang | SingleLineText | NO | null | en/es |
| Audit Passed | Checkbox | NO | false | true if all 11 audit items pass |
| Audit Failed Items | SingleLineText | NO | null | Item names or AUDIT PARSE ERROR or FULL REVIEW REQUIRED |
| Reviewer Handle | SingleLineText | NO | null | Guest name from ALA |
| Published Timestamp | DateTime | NO | null | null until published. MRA anchor. |
| Approval Notes | LongText | NO | null | Human approver comments. |
| Error Log | LongText | NO | null | null on clean write |

SingleSelect pre-population: Confirmed Response Tier (T1/T2/T3) · Approval Status (6 values — no trailing spaces).

---

## Credentials + Configuration

| Item | Value |
|---|---|
| Workflow Name | SCX-RDA |
| Webhook Path | /webhook/scx-rda |
| Anthropic Credential | Subtext-CX-Anthropic (Header Auth, Name: x-api-key) |
| NocoDB Credential | xc-token (Header Auth, Name: xc-token) |
| Email Credential | Subtext-CX-Email |
| Claude Model | claude-sonnet-4-6 — all 5 Claude calls |
| anthropic-version header | 2023-06-01 — required on every Anthropic API call |
| NocoDB URL (internal) | http://nocodb:8080 — never localhost or external IP |
| Triggered By | SCX-BRA Step 18 — fire-and-forget POST, rich payload. Never fires if Halt. |
| Approval SLA | 48 hours from email send. Locked Chat #42. |
| Phase 1 Approval | Client operator updates Approval Status directly in NocoDB. |
| NocoDB Calls / Record | 4–5: GET idem + INIT (session first) + GET ALA + GET recent RDA + POST RDA + PATCH BRA |
| Infrastructure | DigitalOcean n8n-Solofella — NYC3 — Ubuntu 24.04 — 4GB RAM — IP: 161.35.133.49 |

---

## Quality Baseline + Open Items

### Quality Baseline — 87% · Chat #75 · April 2026

**Working at 87%:**
- Five-call architecture eliminates attention dilution
- Opening construction deterministic with priority ordering
- Guest name enforced at three independent checkpoints (Step 7a, Step 7e TASK 3, Step 9b item 11)
- Named staff enforced at three checkpoints (Step 7a priority 1, Step 7c T1 body instruction, Step 9b item 1)
- T1 mixed-signal records acknowledge minor criticism
- SEO keywords placed naturally or omitted
- Commercial commitment scan EN + ES
- Opening variation mechanism prevents structural repetition across consecutive records

**Known gaps at 87%:**
- Full last names in drafts when reviewer_handle includes surname — first-name extraction not yet implemented
- Very short ambiguous reviews (1–5 words) occasionally produce overcompensated body content
- Formula checklist requires periodic review after each pilot month

**Path to 90%+:** Two to three weeks of live PAK-001 data with Christine's approval patterns.

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
| **SCX-Sheet-Sync OAuth credential failure** | **In Progress (Chat #77)** | **n8n 2.4.6 task runner cannot use OAuth refresh tokens in scheduled workflows. Error: "refreshToken is required" at 7am UTC scheduled run (Apr 20, 2026). Solution: Replace with Google Service Account (non-expiring JSON key). Impacts autonomous operation of approval workflow sheet population. Service Account setup: (1) Create in solofella-cmh-project, (2) Generate JSON key, (3) Share PAK-001 sheet with service account email, (4) Update SCX-Sheet-Sync Step 8 HTTP Request credential, (5) Test next scheduled run.** |

**Spanish commercial scan:** both EN and ES prohibited terms applied regardless of record lang — bilingual safety net. Additional Spanish terms specific to a client's brand or legal context may be needed — review with first Spanish-language pilot client.

**48-hour SLA:** Locked Chat #42. Track Time to Approval (RDA KPI) from RDA Timestamp to Approval Status change. If consistently missed, investigate approval workflow with client.

---

## n8n Code Node Rules

- No spread operator — key-by-key only
- No console.warn — blocked by task runner
- No for...of loops — use index-based for loops
- Boolean IF checks — use Code Node
- Payload parse: `typeof body === 'string' ? JSON.parse(body) : body`
- Claude output — strip markdown fences before JSON.parse
- PATCH = JSON body mode — NOT RAW
- NocoDB RAW Content-Type field = `application/json`
- Wait Between Tries max = 5000ms
- Template literals in audit system prompts replaced with string concatenation — avoids red highlight syntax errors from curly braces

---

## Related Documents

- **Upstream Agent:** BRA — `agents/BRA/`
- **NocoDB Client Config Table:** `m95cmabjfyb94ps`
- **NocoDB Base ID:** `pq249fix22t3ofv`
- **MCD:** SubtextCX_MCD_v7.4 — full project state

---

**End of SCX_RDA_HOW_v3.2**  
*Subtext CX · SCX_RDA_HOW_v3.2 · Chat #75 · April 2026 · Solofella LLC*

---

Review and confirm. When approved I commit to GitHub at `agents/RDA/SCX_RDA_HOW_v3.2.md`.
