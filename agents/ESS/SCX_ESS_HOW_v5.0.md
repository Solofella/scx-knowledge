# SCX_ESS_HOW_v5.0

**Agent Name:** ESS (Emotional Signal Stabilizer)
**Version:** 5.0
**Last Updated:** Chat #98 · July 24, 2026
**Model:** claude-sonnet-4-6
**Status:** Verified operational · 23 nodes · As-built from live workflow inspection
**Supersedes:** v4.0 (Chat #74, April 4, 2026) — contained multiple architectural errors; see Corrections section

---

## Purpose

ESS receives emotion classifications from EIP and analyzes HOW the guest expresses those emotions. It does not re-classify emotions — EIP already resolved those. ESS determines:

- **Expression Mode** — the style of emotional expression (6 categories)
- **Emotional Clarity** — how clearly the emotion is communicated (4 categories)
- **Narrative Alignment Score** — how coherently the emotion tag, pain point, signal type, and dominant pole fit together (LLM output, 0.0–1.0)
- **Structural Confidence Score** — deterministic composite reliability score (Code Node, 0.0–1.0)

ESS also runs a **Canonical Integrity Check** on EIP's output before any Claude call is made. Records that fail hard validation are written to ESS's NocoDB table as error records and never reach Claude or HSI.

**ESS inherits all emotion data from EIP via payload. Zero dictionary re-query cost.**

---

## Input Source

**Upstream Agent:** EIP (Emotional Intelligence Processor)
**Trigger:** Webhook POST from EIP to path `/scx-ess`

**Fields received from EIP payload (~25 fields):**

| Field | Notes |
|---|---|
| eip_record_id | Required — positive integer. Hard validation gate. |
| ala_record_id | Traceability |
| client_id | Multi-client routing |
| lang | Language of review (en / es) |
| enriched_emotion_tag | Resolved by EIP |
| core_emotion | Resolved by EIP |
| pain_point_sub_category | Resolved by EIP |
| pain_point_domain_confirmed | Resolved by EIP |
| need_state | Resolved by EIP |
| cognitive_driver | Resolved by EIP |
| polarity_balance | Float 0.0–1.0 |
| dominant_pole | Positive / Negative / Mixed |
| intensity_level | Low / Moderate / High / Critical |
| certainty_score | Float 0.0–1.0 |
| ambiguity_flag | Boolean |
| masked_emotion_flag | Boolean |
| collapse_flag | Boolean, defaults false |
| signal_type | One of 6 EIP values |
| signal_weight | Numeric |
| emotion_hypothesis | Text |
| pain_hypothesis | Text |
| keywords | Text |
| reviewer_handle | Pass-through from ALA |
| review_date | Pass-through from ALA |
| platform | Pass-through from ALA |
| star_rating | Pass-through from ALA |

**ESS does NOT query the Emotion Dictionary.** All emotion data inherited from EIP payload.

⚠️ **v4 CORRECTION:** v4 listed "Enriched Emotion Breakdown JSON" as a received field. This field does not exist — EIP never produces or sends it. Removed.

---

## Processing Logic

### Node Flow (23 Nodes — Execution Order)

**INIT (Nodes 1–2)**

**Node 1 — Webhook**
Receives POST from EIP at path `scx-ess`. Returns hardcoded `{"status":"received"}`.

**Node 2 — Step 2: Payload Validation** (Code)
Parses `input.body` handling both string and pre-parsed object cases:
`typeof input.body === 'string' ? JSON.parse(input.body) : input.body`
Requires `eip_record_id` present and a positive integer — throws if absent or invalid.
Extracts all ~25 fields with `|| null` fallbacks.
Computes `eip_run_id: p.prior_run_id || p.eip_run_id || null` — accepts either key name defensively.

---

**IDEMPOTENCY (Nodes 3–5)**

**Node 3 — Step 3: Idempotency Check** (HTTP GET)
Queries ESS table `m5yektnbtxf8evk` WHERE `EIP Record ID = eip_record_id`, limit 1.

**Node 4a — Step 4a: IF Already Processed?**
Condition: `pageInfo.totalRows > 0`

**Node 4b — Step 4b: Skip Exit** (Set, TRUE branch)
Sets `skip_reason: "Duplicate: EIP Record ID already processed by ESS"`, `includeOtherFields: true`.
Dead end — no downstream connection.

**Node 4c — Step 4c: FALSE Branch Gate** (Code, FALSE branch)
If `totalRows > 0` returns `[]`. Otherwise rebuilds full ~25-field record from Step 2.
Belt-and-suspenders guard against IF node firing both branches.

---

**CANONICAL INTEGRITY CHECK (Nodes 5–7)**

**Node 5 — Step 5: Canonical Integrity Check** (Code)

Hard checks — any failure pushes to `hardErrors` array:
- `enriched_emotion_tag`, `pain_point_sub_category`, `cognitive_driver` must be non-empty
- `polarity_balance`, `certainty_score` must parse as float in [0.0, 1.0]
- `dominant_pole` must be one of: `Positive` / `Negative` / `Mixed`
- `intensity_level` must be one of: `Low` / `Moderate` / `High` / `Critical`
- `signal_type` must be one of the 6 EIP values: `Positive Signal` / `Negative Signal` / `Masked Negative Signal` / `Ambiguous Negative Signal` / `Dignity-Risk Signal` / `Mixed Signal`
- `ambiguity_flag`, `masked_emotion_flag` must be boolean-ish (`true` / `false` / `1` / `0`)

Soft checks — append to `softWarnings`, do not hard-fail:
- `emotion_hypothesis` or `pain_hypothesis` missing → warning string containing `-0.10`
- Either present but under 10 characters → warning string containing `-0.05`

⚠️ **The `-0.10` / `-0.05` substrings in warning strings are functionally significant** — Step 11 scans `canon_warnings` for these literal substrings to compute soft penalties. They are not decorative labels.

Output fields:
- `canon_pass` — **string** `'true'` or `'false'` (not boolean)
- `failure_reason` — hardErrors joined with ` | `, or null
- `canon_warnings` — array of warning strings, or null

⚠️ **v4 CORRECTION:** v4 described this node as verifying "Core Emotion maps correctly to Enriched Emotion Tag," "Need State matches Emotion Dictionary entry," "Cognitive Driver aligns with emotion type." The real node does none of these cross-referential checks. It validates field presence, numeric ranges, and enum membership only.

**Node 6 — Step 6: IF Canon Pass?**
Condition: `$json.canon_pass` equals string `"true"`.
TRUE → Claude classification path.
FALSE → error exit path.

**Node 6a — Step 6a: NocoDB POST ESS Error Record** (HTTP POST, FALSE branch)
Writes to `m5yektnbtxf8evk`:
- `ESS Run ID`: `"ERROR-{eip_record_id}"`
- `Canon Flag`: false
- `Integrity Failure Reason`: failure_reason
- `HSI Status`: `"Error"`
- `Error Log`: `"Canon hard fail — record rejected before Claude call"`

No Claude call ever happens for a canon-failed record.

**Node 6b — Step 6b: NocoDB PATCH EIP Error** (HTTP PATCH)
PATCHes `mhicpnrahaesxmy/{eip_record_id}` → `{"ESS Status": "Error"}`.
Dead end after this.

---

**CLAUDE CLASSIFICATION (Nodes 7–10)**

⚠️ **v4 CORRECTION — CRITICAL:** v4 described TWO separate Claude calls (Steps 9–16 for Expression Mode, Steps 17–21 for Emotional Clarity) and called this a "Critical Rule." The live workflow makes **ONE combined Claude call** returning all three outputs together.

**Node 7 — Step 7: Build Claude Prompt** (Code)

`ess_system_prompt` instructs Claude to:
- Classify HOW emotion is expressed and assess narrative coherence
- NOT interpret meaning or speculate about motivation (EIP already resolved that)
- Return **exactly 3 keys**, JSON only:
  - `expression_mode`: one of the 6 valid values (see categories below)
  - `emotional_clarity`: one of the 4 valid values (see categories below)
  - `narrative_alignment_score`: decimal 0.0–1.0 — measures whether Enriched Emotion Tag logically aligns with Pain Point Sub-Category, Signal Type, and Dominant Pole simultaneously. 1.0 = perfect coherence, 0.0 = none.
- **Language rule:** input fields may be in any language. Output values must always be in English regardless of input language.

`ess_user_prompt` feeds: `enriched_emotion_tag`, `core_emotion`, `cognitive_driver`, `need_state`, `pain_point_sub_category`, domain, `signal_type`, `dominant_pole`, `polarity_balance`, `certainty_score`, `masked_emotion_flag`, `ambiguity_flag`, `emotion_hypothesis`, `pain_hypothesis`, and `lang` if present.

**Node 8 — Step 8: Build Claude Request Body** (Code)
```
model: claude-sonnet-4-6
max_tokens: 100
temperature: 0.3
system: <ess_system_prompt>
messages: [{role: user, content: ess_user_prompt}]
```

**Node 9 — Step 9: Claude API Call** (HTTP POST)
Endpoint: `https://api.anthropic.com/v1/messages`
Headers: `anthropic-version: 2023-06-01` + `Content-Type: application/json`
Auth: httpHeaderAuth x-api-key, credential id `uMahlx4nOC5YJh0Z`
`retryOnFail: true`, `waitBetweenTries: 5000`

**Node 10 — Step 10: Output Parsing + Validation** (Code)
Strips markdown code fences before JSON.parse (regex removes ` ```json ` and ` ``` ` variants — necessary because this Claude call does not use forced-JSON/structured response mode).
Validates:
- `expression_mode` against 6 valid values
- `emotional_clarity` against 4 valid values
- `narrative_alignment_score` parses as float in [0, 1]

Throws on any violation.

---

**SCORING + ASSEMBLY (Nodes 11–13)**

**Node 11 — Step 11: Structural Confidence Score** (Code — fully deterministic)

Formula:
```
score = (certainty_score × 0.40)
      + (narrative_alignment_score × 0.35)
      + (canon_bonus × 0.25)
```

`canon_bonus` is a **hardcoded constant `1.0`**. Only canon-passing records ever reach this node (hard fails exited via Step 6a/6b), so this term always contributes a fixed 0.25. It is intentional-by-consequence, not a live variable input — but document it precisely to prevent a future refactor from assuming it varies.

Then subtract:
- `0.10` if `ambiguity_flag` is true
- `0.05` if `masked_emotion_flag` is true
- Soft penalty: scan `canon_warnings` strings for literal substrings `-0.10` and `-0.05`, sum matched penalties, cap total soft penalty at `-0.15`

Final score clamped to [0, 1].

⚠️ **v4 CORRECTION:** v4 attributed the deterministic role to Narrative Alignment Score on a 1–10 scale. Narrative Alignment Score is actually an LLM output on a 0.0–1.0 scale. Structural Confidence Score is the deterministic node — not documented in v4 at all.

**Node 12 — Step 12: Run ID + Timestamp** (Code)
`ess_run_id` format: `ESS-YYYYMMDD-HHMMSS-mmm`
`ess_timestamp`: ISO string

**Node 13 — Step 13: Build NocoDB POST Body** (Code)
Builds write body. `Canon Flag` correctly converted to real boolean: `inp.canon_pass === true || inp.canon_pass === 'true'`

---

**OUTPUT + HANDOFF (Nodes 14–19)**

**Node 14 — Step 14: NocoDB POST ESS Record** (HTTP POST)
Writes to `m5yektnbtxf8evk`. See NocoDB Schema section.

**Node 15 — Step 15: Capture ESS Record ID** (Code)
Reads `response.Id`, throws if absent.
Rebuilds full field set referencing `$('Step 12 - Run ID + Timestamp')` — not its immediate predecessor.

**Node 16 — Step 16: Build Patch Body** (Code)
⚠️ **Dead code.** Computes `ess_patch_body: JSON.stringify({"ESS Status": "Complete"})`. This value is never used — Step 17 ignores it and hardcodes its own body. No active bug, but a maintenance trap.

**Node 17 — Step 17: NocoDB PATCH EIP Complete** (HTTP PATCH)
PATCHes `mhicpnrahaesxmy/{eip_record_id}` → hardcoded `{"ESS Status": "Complete"}`.
Ignores Step 16's computed value.

**Node 18 — Step 18: Build HSI Payload** (Code)
References `$('Step 15 - Capture ESS Record ID')` directly — bypasses Step 17 PATCH response (not needed).

Payload contents:

*Trace fields:* `ess_record_id`, `eip_record_id`, `ala_record_id`, `prior_run_id: ess_run_id`, `prior_table: 'ESS'`

*ESS outputs:* `canon_flag: inp.canon_pass` (raw string — see Known Issues), `expression_mode`, `emotional_clarity`, `narrative_alignment_score`, `structural_confidence_score`, `canon_warnings`, `lang`, `client_id`

*EIP pass-through (~20 fields):* `enriched_emotion_tag`, `core_emotion`, `pain_point_sub_category`, `pain_point_domain_confirmed`, `intensity_level`, `polarity_balance`, `dominant_pole`, `certainty_score`, `ambiguity_flag`, `masked_emotion_flag`, `collapse_flag`, `cognitive_driver`, `need_state`, `signal_type`, `signal_weight`, `emotion_hypothesis`, `pain_hypothesis`, `keywords`, `reviewer_handle`, `review_date`, `platform`, `star_rating`

⚠️ **v4 CORRECTION:** v4 listed 5 fields sent to HSI. Real payload carries 30+ fields.

**Node 19 — Step 19: HSI Trigger** (HTTP POST)
POSTs to `http://161.35.133.49:5678/webhook/scx-hsi`
`timeout: 5000`, `alwaysOutputData: true`
⚠️ **No `onError: continueRegularOutput` configured** — see Known Issues.
Dead end.

---

**Full Connection Path:**
```
Webhook → Step 2 → Step 3 → Step 4a
  → [TRUE: Step 4b — dead end]
  → [FALSE: Step 4c] → Step 5 → Step 6
      → [FALSE: Step 6a → Step 6b — dead end]
      → [TRUE: Step 7] → Step 8 → Step 9 → Step 10
          → Step 11 → Step 12 → Step 13 → Step 14
          → Step 15 → Step 16 → Step 17 → Step 18
          → Step 19 — dead end
```

---

## NocoDB Schema

**Table ID:** `m5yektnbtxf8evk`

| Field | Column ID | Type | Notes |
|---|---|---|---|
| Id | ci522qw486krwni | ID | System |
| CreatedAt | cyh8yw8uep5eg84 | CreatedTime | System |
| UpdatedAt | cl3l0lxkse9y27z | LastModifiedTime | System |
| nc_created_by | cmm3yi5evarudvv | CreatedBy | System |
| nc_updated_by | c01uzap0epgaz6w | LastModifiedBy | System |
| nc_order | c60q2bd2zlmw8zm | Order | System |
| ESS Run ID | cz10rqt8qb11me9 | SingleLineText | Format: ESS-YYYYMMDD-HHMMSS-mmm |
| EIP Record ID | cxxrixifog227hm | Number | Upstream FK |
| ALA Record ID | c6ky763s3kzga8l | Number | Traceability |
| Client ID | cjuk7l7y5ewkmoy | SingleLineText | |
| Lang | czbux2rv4ecyf65 | SingleLineText | en / es |
| ESS Timestamp | c3imcci8be9teht | DateTime | ISO string |
| Canon Flag | cjmcyb8r40ynfrs | Checkbox | Boolean — true on success path |
| Integrity Failure Reason | cmbdug7nk3d4jvb | LongText | Null on success path |
| Canon Warnings | cbx90vqu5u1a9bd | LongText | JSON-stringified array or null |
| Expression Mode | ctlnh0565qellsu | SingleSelect | 6 valid values |
| Emotional Clarity | cav648icks13mm2 | SingleSelect | 4 valid values |
| Narrative Alignment Score | cvguh9tp2j1uhoh | Decimal | LLM output 0.0–1.0 |
| Structural Confidence Score | caqkeq5cqv5z2qd | Decimal | Deterministic 0.0–1.0 |
| HSI Status | c8xgg338z4dmlps | SingleSelect | Ready / Error |
| Error Log | cp6apx7kkr0w9a8 | LongText | Null on success path |

⚠️ **v4 CORRECTION:** v4 listed `Core Emotion` and `Need State` as stored fields "for reference." Neither exists in the real table. Both travel in-memory via payload only. This has implications for Phase 3 Part A tracing (cognitive_driver + need_state pass-through gap).

---

## Expression Mode Categories (6 Types)

### Explicit
Guest directly states emotion in clear language.
- "I was frustrated with the long wait"
- "We were delighted by the presentation"

Signal: High interpretability. Guest is self-aware and articulate.

### Implicit
Emotion conveyed through context, not directly stated.
- "The server never checked on us" (implies neglect/frustration)
- "We won't be returning" (implies disappointment without naming it)

Signal: Moderate interpretability. Requires inference from facts.

### Masked
Guest uses positive or neutral language to conceal negative emotion. Cross-referenced against star rating.
- "Everything was fine" (2-star review)
- "Not bad" (damning with faint praise)

Signal: Low interpretability. **This is SubtextCX's competitive differentiator.**

### Performative
Emotion expressed for social effect, not genuine internal state.
- "OMG BEST MEAL EVER!!!" (social media amplification)
- "Absolutely unacceptable" (formal complaint language, amplified for effect)

Signal: Guest performing emotion for an audience.

### Conflicted
Multiple contradictory emotions expressed simultaneously.
- "The food was amazing but the service ruined it"
- "Great atmosphere, just wish the portions were bigger"

Signal: Mixed experience. Requires nuanced response strategy.

### Absent
No detectable emotional content. Purely factual or transactional.
- "We ordered the salmon. It arrived in 20 minutes."
- "Parking available in rear lot."

Signal: Informational only. Low engagement.

---

## Emotional Clarity Categories (4 Types)

### Clear
Guest's emotional state is unambiguous and well-articulated. Consistent emotion across review, specific examples, Expression Mode aligns with star rating.

### Diffuse
Emotion present but spread across multiple unfocused themes. Multiple emotions without hierarchy, general tone without precision.
Example: "Everything was good, nice vibe, enjoyed it"

### Fragmented
Emotional narrative broken or inconsistent. Contradictory statements, emotion shifts mid-review.
Example: "Great service, terrible food, loved the decor, won't return"

### Ambiguous
Emotional state unclear or deliberately obscured. Masked emotion, irony, sarcasm, or performative language masking true feeling.
Example: 2-star review: "It was fine"

---

## Narrative Alignment Score (LLM Output — 0.0–1.0)

⚠️ **v4 CORRECTION:** v4 described this as a deterministic Code Node on a 1–10 scale. It is an LLM output, part of the single combined Claude call, on a 0.0–1.0 decimal scale.

**What it measures:** Whether `enriched_emotion_tag`, `pain_point_sub_category`, `signal_type`, and `dominant_pole` logically cohere with each other simultaneously.

- **1.0** — perfect coherence across all four fields
- **0.0** — no coherence

**Use case:** Feeds into Structural Confidence Score (weight 0.35). Also used by HSI to contextualize signal interpretation confidence.

---

## Structural Confidence Score (Deterministic — 0.0–1.0)

**New section — not documented in v4.**

Computed in Step 11. Fully deterministic Code Node — no AI involvement.

**Formula:**
```
base = (certainty_score × 0.40)
     + (narrative_alignment_score × 0.35)
     + (1.0 × 0.25)           ← canon_bonus, always 1.0

penalties:
  -0.10  if ambiguity_flag = true
  -0.05  if masked_emotion_flag = true
  -[sum] soft penalties parsed from canon_warnings strings
         (scan for '-0.10' and '-0.05' substrings, sum, cap at -0.15)

final = clamp(base - penalties, 0, 1)
```

**Note on canon_bonus:** The 0.25 contribution is a fixed constant, not a live variable. Only canon-passing records reach Step 11 — hard failures exit via Steps 6a/6b. This is intentional-by-consequence. A future refactor should not assume this term varies.

**Downstream use:** HSI receives this score as a reliability signal for weighting its own interpretation confidence.

---

## Token Budget

⚠️ **Gap — not re-measured this cycle.** v4 estimated ~1,200 tokens/record based on two Claude calls. The real architecture makes one combined Claude call with `max_tokens: 100`. The actual per-record token cost is materially different from v4's figure and needs a fresh measurement against the live prompt structure before this estimate is updated.

---

## Key Design Decisions

### One Combined Claude Call, Not Two

v4 specified two separate Claude calls as a "Critical Rule." The live workflow uses one call returning all three outputs (`expression_mode`, `emotional_clarity`, `narrative_alignment_score`) in a single 3-key JSON response. This reduces latency and token cost per record.

### Hard Canon Gate Before Claude

Records that fail the Canonical Integrity Check never reach Claude. They are written to ESS NocoDB as error records and trigger an EIP status PATCH. This prevents Claude from processing malformed EIP output and producing unreliable classifications.

### Why Claude for ESS

ESS requires nuanced interpretation of expression style — detecting masked emotions, identifying performative language, assessing narrative coherence. Claude excels at subtle linguistic analysis and context-aware interpretation. GPT handles EIP's structured classification task; Claude handles ESS's interpretive task.

### ESS Does Not Re-Query Emotion Dictionary

EIP already resolved all emotion classifications. Re-querying would waste ~13K tokens per record. ESS inherits everything it needs from the EIP payload.

### Language-Agnostic Output

Review text and EIP fields may arrive in any language. Claude's system prompt explicitly requires output field values in English regardless of input language. The `lang` field is passed to Claude for context only.

---

## Known Issues

**1. canon_pass / canon_flag string-vs-boolean inconsistency**
Step 13 (ESS NocoDB write) correctly converts `canon_pass` to a real boolean. Step 18 (HSI payload) sends `canon_flag: inp.canon_pass` as the raw string `'true'`. This works today because: (a) Step 18 is only reachable after Step 6's TRUE branch, so `canon_pass` is always `'true'` at that point; (b) HSI's intake explicitly accepts `true` / `1` / `'true'` defensively. It works by two undocumented conventions coincidentally agreeing — not an enforced contract. Fragile under future edits to either side.

**2. Step 16 dead code**
Step 16 computes `ess_patch_body` that Step 17 never reads. Step 17 hardcodes its own body. No active bug. Maintenance trap — mirrors the same dead code pattern in EIP Step 17a/17.

**3. Step 19 missing fault tolerance**
HSI Trigger has no `onError: continueRegularOutput`. An HSI outage would fail the entire ESS execution even after a fully successful canon check, Claude call, and NocoDB write. `alwaysOutputData: true` is present but is not equivalent protection. This is a pipeline-wide pattern: ALA→EIP has fault tolerance; EIP→ESS and ESS→HSI both lack it.

**4. canon_bonus hardcoded constant**
`structural_confidence_score`'s canon_bonus is always 1.0 — contributing a fixed 0.25 to every score. Intentional-by-consequence (only passing records reach Step 11), not a bug. Document it precisely so a future refactor doesn't assume it varies.

---

## Open Items — Require Decisions, Not Resolvable From Code

1. **Chat number for "Last Updated"** — this document records Chat #98, July 24, 2026. Confirm if that's correct for the version stamp.

2. **Core Emotion / Need State omission from ESS NocoDB** — v4 specified storing these fields "for reference." The real table does not store them. Is this an intentional architecture decision (they travel in-memory only) or a regression that should be restored? Relevant to Phase 3 Part A.

3. **canon_pass string/boolean hardening** — should Step 18 be updated to perform the same boolean conversion Step 13 does, or is the current convention explicitly documented and left as-is?

4. **Step 19 fault tolerance** — should `onError: continueRegularOutput` be added to ESS Step 19 now, or bundled with the same fix on EIP Step 18 as a single pipeline-wide consistency pass?

5. **Token budget re-measurement** — the v4 figure of ~1,200 tokens/record is based on a two-call architecture that no longer exists. Needs fresh measurement against the live single-call prompt.

6. **canon_flag / HSI contract** — HSI's defensive multi-type check (`true` / `1` / `'true'`) for `canon_flag`: is this the documented permanent contract between ESS and HSI, or was it defensive coding against an anticipated edge case? Needs clarification from whoever designed HSI's intake.

---

## Downstream Handoff

**ESS → HSI** (30+ fields)

*Trace:* `ess_record_id`, `eip_record_id`, `ala_record_id`, `prior_run_id`, `prior_table`

*ESS outputs:* `canon_flag`, `expression_mode`, `emotional_clarity`, `narrative_alignment_score`, `structural_confidence_score`, `canon_warnings`, `lang`, `client_id`

*EIP pass-through:* `enriched_emotion_tag`, `core_emotion`, `pain_point_sub_category`, `pain_point_domain_confirmed`, `intensity_level`, `polarity_balance`, `dominant_pole`, `certainty_score`, `ambiguity_flag`, `masked_emotion_flag`, `collapse_flag`, `cognitive_driver`, `need_state`, `signal_type`, `signal_weight`, `emotion_hypothesis`, `pain_hypothesis`, `keywords`, `reviewer_handle`, `review_date`, `platform`, `star_rating`

**HSI uses ESS output to:**
- Weight behavioral narrative emphasis
- Adjust signal interpretation confidence via Structural Confidence Score
- Contextualize pain point severity against Expression Mode and Emotional Clarity

---

## Related Documents

- SCX_ESS_CHANGELOG.md
- ESS_Schema.md
- Upstream: SCX_EIP_HOW_v5.md
- Downstream: SCX_HSI_HOW_v3.md (pending update)
- SCX_PreBuild_Protocol_v1.0

---

## n8n Workflow Details

**Workflow Name:** SCX-ESS
**Trigger:** Webhook POST from EIP at `/scx-ess`
**Node count:** 23
**NocoDB table (own):** m5yektnbtxf8evk
**NocoDB table (patches):** mhicpnrahaesxmy (EIP table — ESS Status field)
**NocoDB base:** `http://nocodb:8080`, base ID `pq249fix22t3ofv`
**NocoDB credential:** httpHeaderAuth, header name `xc-token`, credential ID `DT9tnRgqYpPc3rXo`
**Anthropic credential:** httpHeaderAuth, header name `x-api-key`, credential ID `uMahlx4nOC5YJh0Z`
**Claude model:** claude-sonnet-4-6 · max_tokens: 100 · temperature: 0.3
**Retry on Claude call:** `retryOnFail: true`, `waitBetweenTries: 5000`

**Critical rules:**
- Canonical validation runs BEFORE Claude call — hard failures never reach Claude
- One combined Claude call returns all three outputs (expression_mode, emotional_clarity, narrative_alignment_score)
- Narrative Alignment Score is an LLM output (0.0–1.0), NOT deterministic
- Structural Confidence Score is the deterministic node (Step 11)
- Output field values always in English regardless of input language
- Strip markdown fences before JSON.parse on Claude response (no forced-JSON mode)
- PATCH body mode for NocoDB PATCH nodes (not RAW)
- NocoDB URL inside n8n: `http://nocodb:8080` — NEVER localhost

---

*SCX_ESS_HOW v5.0 · Chat #98 · July 24, 2026 · Solofella LLC*
*As-built from live n8n workflow inspection · Supersedes v4.0 (Chat #74, April 4, 2026)*
