# SCX_HSI_HOW_v4.0

**Agent Name:** HSI (Human Signal Intelligence)
**Version:** 4.0
**Last Updated:** July 24, 2026
**Model:** claude-sonnet-4-6
**Status:** Operational — 32 nodes — Known issues documented below
**Note:** This is the first full technical HOW for HSI. v3.0 was a short conceptual overview; several things it described do not exist in the live workflow. Treat this document as authoritative over v3.0 wherever they conflict.

---

## Purpose

HSI (Human Signal Intelligence) receives the synthesized emotional + structural signal from ESS and converts it into a behavioral interpretation — what the combined emotion, expression mode, and pain point pattern actually means about guest experience and loyalty trajectory. It does not re-classify emotion or pain points (EIP and ESS already resolved those). It interprets.

HSI is the agent closest to human judgment in the pipeline. It determines whether a record requires human review before proceeding to drafting, and assigns the preliminary response tier that governs how BRA and RDA handle the record.

**Governance constraint (permanent):** HSI interprets signal meaning only. It never prescribes operational actions, never produces response language, and never re-classifies upstream outputs.

---

## Input Sources

**Trigger:** Webhook POST from ESS at path `/scx-hsi`

**Receives from ESS payload (~30 fields):**

From EIP pass-through:
- `eip_record_id`
- `ala_record_id`
- `enriched_emotion_tag`
- `core_emotion`
- `pain_point_sub_category`
- `pain_point_domain_confirmed`
- `need_state`
- `cognitive_driver`
- `polarity_balance`
- `dominant_pole`
- `intensity_level`
- `certainty_score`
- `ambiguity_flag`
- `masked_emotion_flag`
- `collapse_flag`
- `signal_type`
- `signal_weight`
- `emotion_hypothesis`
- `pain_hypothesis`
- `keywords`
- `reviewer_handle`
- `review_date`
- `platform`
- `star_rating`
- `lang`
- `client_id`

From ESS computation:
- `ess_record_id` (required — validated at Step 2)
- `canon_flag`
- `expression_mode`
- `emotional_clarity`
- `narrative_alignment_score`
- `structural_confidence_score`
- `canon_warnings`
- `prior_run_id`
- `prior_table`

**HSI does NOT query Pain Point Master or Emotion Dictionary.** All classification data is inherited from EIP via ESS pass-through.

---

## Processing Logic — Full Node-by-Node (32 Nodes)

### Connection Path

```
Webhook → Step 2 → Step 3 → If →
  [TRUE: Step 4b — dead end]
  [FALSE: Step 4c] → Step 5 → Step 6 → Step 6 IF →
    [TRUE: Step 6a → Step 6b]
    [FALSE: Step 6c → Step 6b]
  → Step 7 → Step 8 → Step 9 → Step 10 → Step 11 → Step 12
  → Step 13 → Step 13b → Step 14 → Step 15 → Step 16 → Step 17
  → Step 18 → Step 19 → Step 20 → Step 21 → Step 22 → Step 23
  → Step 24 → Step 25 [dead end]
```

---

### Intake & Idempotency (Nodes 1–6)

**Node 1 — Webhook**
Receives POST from ESS at path `scx-hsi`. Returns hardcoded `{"status":"received"}` immediately. No auth required.

**Node 2 — Step 2: Payload Validation** (Code)
Parses `input.body` (handles string or object). Requires `ess_record_id` present and a positive integer — throws if absent or invalid. Extracts all ~30 fields with `|| null` fallbacks. Exception: `canon_flag` is passed through raw with no null fallback (deliberate — canon gate at Step 5 needs its exact value).

**Node 3 — Step 3: Idempotency Check** (HTTP GET)
Queries HSI's own table `mb8nv8t3nk6xzed` with filter `WHERE ESS Record ID = ess_record_id`, limit 1. Checks whether this ESS record has already been processed by HSI.

**Node 4 — If (Step 4a)**
Condition: `pageInfo.totalRows > 0`.
- TRUE branch → duplicate detected.
- FALSE branch → new record, proceed.

**Node 5 — Step 4b: Skip Exit** (Set, TRUE branch)
Sets `skip_reason: "Duplicate: ESS Record ID already processed by HSI"` with `includeOtherFields: true`. Dead end — no downstream trigger.

**Node 6 — Step 4c: FALSE Branch Gate** (Code, FALSE branch)
Rebuilds the full ~30-field record from `$('Step 2 - Payload Validation')`. Required because the IF node does not pass data automatically on the FALSE branch. Returns `[]` on any unexpected condition per standard pipeline IF-gate pattern.

---

### Canon Gate & Reviewer History (Nodes 7–12)

**Node 7 — Step 5: Field Validation + Canon Gate** (Code)
Two checks:

1. **Canon gate:** Throws `'Canon Flag = false reached HSI'` unless `canon_flag` is `true`, `1`, or the string `'true'`. Handles all three forms defensively — boolean may arrive as any of these from ESS.

2. **Field presence validation:** Requires 9 fields non-null and non-empty: `expression_mode`, `emotional_clarity`, `narrative_alignment_score`, `structural_confidence_score`, `enriched_emotion_tag`, `intensity_level`, `signal_type`, `emotion_hypothesis`, `pain_hypothesis`. Throws if any are absent.

**Node 8 — Step 6: Prior Record Count** (Code)
Checks `reviewer_handle`. If empty or starts with `"Anonymous-"`, short-circuits with `prior_record_count: 0, is_named_reviewer: false` and skips the NocoDB lookup entirely. Otherwise sets `prior_record_count: null, is_named_reviewer: true` to flag for a real lookup downstream.

**Node 9 — Step 6a: Prior Record GET** (HTTP GET)
Queries HSI's own table `mb8nv8t3nk6xzed` with filter `WHERE Reviewer Handle = reviewer_handle`, limit 100. Intended to retrieve how many prior HSI records exist for this reviewer.

**Node 10 — Step 6b: Prior Record Count Capture** (Code)
Reads `response.pageInfo?.totalRows` as `count`. References `$('Step 6 - Prior Record Count')` for all other fields. Sets `prior_record_count: count` and carries through `is_named_reviewer`. Receives from both Step 6a (TRUE path) and Step 6c (FALSE path).

**Node 11 — Step 6 IF: Is Named Reviewer?**
Compares `{{$json.is_named_reviewer}}` equals the string `"True"` (capital T), with `caseSensitive: true` explicitly set.
- TRUE branch → Step 6a (real lookup)
- FALSE branch → Step 6c (anonymous gate)

⚠️ **Known Issue — likely routing bug:** `is_named_reviewer` is a genuine JS boolean set by Step 6. A JS boolean `true` stringifies to lowercase `"true"`, not `"True"`. With case-sensitivity on, this condition likely never evaluates true. Result: every record — named reviewer or not — routes to Step 6c. See Open Items #3.

**Node 12 — Step 6c: Anonymous Gate** (Code, FALSE branch)
Unconditionally hardcodes `prior_record_count: 0, is_named_reviewer: false`, regardless of what Step 6 determined. If the Step 6 IF bug is confirmed, this node processes every record.

---

### Deterministic Synthesis: S4A1–S4A6 (Nodes 13–18)

No LLM calls in this section. All computation is deterministic JavaScript.

**Node 13 — Step 7: S4A1 Context Classification** (Code)
Buckets `expression_mode` + `emotional_clarity` into `context_bucket`:

| Expression Mode | Emotional Clarity | context_bucket |
|---|---|---|
| Explicit | Clear or Diffuse | `High-Clarity-Explicit` |
| Implicit | Clear | `High-Clarity-Implicit` |
| Masked | any | `Masked-Ambiguous` |
| Implicit | non-Clear | `Masked-Ambiguous` |
| Conflicted | any | `Conflicted` |
| Performative | any | `Performative` |
| any other | any | `Low-Clarity` |

Note: `Absent` expression mode falls silently into `Low-Clarity` catch-all. See Open Items #7 (minor semantic gap, not a hard bug).

**Node 14 — Step 8: S4A2 Signal Reliability** (Code)
Computes `reliability_tier` from the average of `narrative_alignment_score` and `structural_confidence_score`:
- ≥ 0.70 → `Strong`
- ≥ 0.50 → `Moderate`
- else → `Weak`

Also sets `polarity_shift` (boolean): flags a mismatch between categorical `dominant_pole` and continuous `polarity_balance`:
- Positive pole but `polarity_balance < 0.40` → `true`
- Negative pole but `polarity_balance > 0.70` → `true`
- else → `false`

**Node 15 — Step 9: S4A3 Temporal Context** (Code)
Uses `prior_record_count` and `is_named_reviewer` (both potentially locked at 0/false due to Step 6 IF bug) to derive:

- `recurrence_signal`:
  - 0 prior → `First Occurrence`
  - 1–3 prior → `Repeat Signal`
  - 4+ prior → `Systemic Pattern`
- `pattern_type_recurrence`: `Sporadic` (0–3 prior) or `Systemic` (4+)
- `temporal_context`: narrative string combining recurrence signal with loyalty/churn framing

⚠️ If Step 6 IF bug is confirmed: this node permanently outputs `First Occurrence` / `Sporadic` for every record. Repeat-guest and systemic-pattern detection would never function in production. See Open Items #3.

**Node 16 — Step 10: S4A4 Masked Emotion Prep** (Code)
Builds `masked_context` object:
- `masked`: boolean (from `masked_emotion_flag`)
- `surface_emotion`: equals `enriched_emotion_tag`
- `hypothesis_needed`: `true` only when `masked && expression_mode === 'Masked'`
- `ess_notes_context`: `(canon_warnings || '').toString().substring(0, 200)`

⚠️ Minor cosmetic issue: `canon_warnings` is an array. `.toString()` produces a bare comma-joined string rather than readable formatting. Functional but not human-readable. See Open Items #7.

**Node 17 — Step 11: S4A5 Behavioral Context Synthesis** (Code)
Pure template-string synthesis. Combines `enriched_emotion_tag`, `expression_mode`, `need_state`, `cognitive_driver`, `pain_point_sub_category`, `signal_type` into a single `behavioral_context` sentence. No logic branches. No known issues.

**Node 18 — Step 12: S4A6 Interpretation Confidence** (Code)
Weighted formula:

```
confidence_score =
  clarityMap[emotional_clarity]   × 0.35
  + narrative_alignment_score     × 0.25
  + structural_confidence_score   × 0.40
  - (ambiguity_flag ? 0.10 : 0)
  - (intensity_level === 'Critical' ? 0.05 : 0)
  clamped to [0, 1]
```

Clarity weights:
- Clear → 1.00
- Diffuse → 0.75
- Fragmented → 0.40
- Ambiguous → 0.20

Produces:
- `confidence_score` (0–1 decimal)
- `interpretation_confidence` tier: ≥ 0.72 → `High` | ≥ 0.50 → `Medium` | else → `Low`

---

### Claude Interpretation Call (Nodes 19–24)

**Node 19 — Step 13: Build Claude Prompt** (Code)
Constructs `hsi_system_prompt` and `hsi_user_prompt`.

**System prompt defines:**
- Role: "You are the HSI (Human Signal Intelligence) agent" — the sole agent permitted to generate meaning from the signal
- Explicit prohibitions: do not re-classify emotion or pain points; do not suggest business actions; do not produce response language
- Required output: exactly 4 JSON keys (no markdown fences, no preamble)
- Language rule: if `lang === 'es'`, narrative field content in Spanish; field names always in English

**Four required output fields:**

| Field | Description |
|---|---|
| `signal_synthesis_summary` | 2–3 sentence behavioral narrative: how emotion + pain point impacted guest experience and what unmet need drove the signal |
| `contextual_linguistic_framing` | How the guest's language revealed or concealed their actual emotional state |
| `temporal_signal_insight` | What recurrence and timing tell us about loyalty trajectory and churn risk |
| `masked_emotion_hypothesis` | One sentence if `hypothesis_needed` is true; null otherwise |

**User prompt feeds:**
`context_bucket`, `reliability_tier`, `interpretation_confidence`, `behavioral_context`, `temporal_context`, `masked_context` (JSON-stringified), `lang`, plus supporting EIP fields: `enriched_emotion_tag`, `intensity_level`, `signal_type`, `emotion_hypothesis`, `pain_hypothesis`, `pain_point_sub_category`.

**Node 20 — Step 13b: Build Claude Request Body** (Code)
```
model: claude-sonnet-4-6
max_tokens: 600
temperature: 0.3
```

**Node 21 — Step 14: Claude API Call** (HTTP POST)
Endpoint: `https://api.anthropic.com/v1/messages`
Headers: `anthropic-version: 2023-06-01`, `Content-Type: application/json`
Credential: `x-api-key` via httpHeaderAuth (credential id `uMahlx4nOC5YJh0Z`)
`retryOnFail: true`, `waitBetweenTries: 5000`

**Node 22 — Step 15: Output Parsing + Validation** (Code)
Strips markdown fences. `JSON.parse`s result. Validates:
- `signal_synthesis_summary`, `contextual_linguistic_framing`, `temporal_signal_insight` must all be non-empty strings of length ≥ 10 — throws if any fail
- `masked_emotion_hypothesis` treated as optional (`|| null`), consistent with the prompt's conditional instruction

**Node 23 — Step 16: Preliminary Response Tier** (Code)
Deterministic tier assignment. Evaluated in order — first match wins.

**T3 conditions (any one sufficient):**
- `signal_type === 'Dignity-Risk Signal'`
- `intensity_level === 'Critical'`
- `masked_emotion_flag && expression_mode === 'Masked' && interpretation_confidence === 'Low'`
- `pain_point_sub_category.includes('Trust, Dignity & Belonging')`

⚠️ **Known Issue — last T3 clause likely dead code:** `'Trust, Dignity & Belonging'` is a canonical domain name (stored in `pain_point_domain_confirmed`). The live check is against `pain_point_sub_category`, which holds a specific enriched pain point phrase (e.g., "Bill takes too long to arrive") — not a domain name. This substring check almost certainly never matches. Coverage for Trust/Dignity records is partially retained via the `signal_type === 'Dignity-Risk Signal'` check, but this clause does not add what it appears to intend. See Open Items #4.

**T2 conditions (if not T3, any one sufficient):**
- `signal_type` is `'Masked Negative Signal'`, `'Ambiguous Negative Signal'`, or `'Mixed Signal'`
- `signal_type === 'Negative Signal'` AND `intensity_level` is `'High'` or `'Moderate'`
- `ambiguity_flag` is true
- `interpretation_confidence === 'Medium'`
- `expression_mode === 'Conflicted'`

**T1:** All other records.

**Node 24 — Step 17: Human Review Required** (Code)
```
human_review_required =
  tier === 'T3'
  || interpretation_confidence === 'Low'
  || structural_confidence_score < 0.50
  || reliability_tier === 'Weak'
```
This is the concrete implementation of the pipeline's Human Approval Gate principle. It is a computed boolean written to NocoDB and passed to BRA — not merely a policy statement. Records where this is `true` are flagged before any draft is generated.

---

### Write & Handoff (Nodes 25–32)

**Node 25 — Step 18: Run ID + Timestamp** (Code)
Generates:
- `hsi_run_id`: format `HSI-YYYYMMDD-HHMMSS-mmm` (pure timestamp, e.g. `HSI-20260724-143052-417`)
- `hsi_timestamp`: ISO string

Note: This format differs from v3.0's domain-encoded example. The pure timestamp format is what the live workflow produces. See Corrections section.

**Node 26 — Step 19: Build NocoDB POST Body** (Code)
Assembles write body for HSI table. Fields written:

| Field | Value |
|---|---|
| HSI Run ID | `hsi_run_id` |
| ESS Record ID | `ess_record_id` |
| ALA Record ID | `ala_record_id` |
| HSI Timestamp | `hsi_timestamp` |
| Signal Synthesis Summary | from Claude |
| Contextual Linguistic Framing | from Claude |
| Temporal Signal Insight | from Claude |
| Masked Emotion Hypothesis | from Claude (or null) |
| Interpretation Confidence | `interpretation_confidence` tier |
| Preliminary Response Tier | T1 / T2 / T3 |
| Human Review Required | boolean |
| Pattern Type - Recurrence | `pattern_type_recurrence` |
| Prior Record Count | `prior_record_count` |
| Reviewer Handle | `reviewer_handle` |
| lang | `lang` |
| Client ID | `client_id` |
| BRA Status | `"Pending"` (hardcoded initial value) |
| Error Log | null |

**Node 27 — Step 20: NocoDB POST HSI Record** (HTTP POST)
Writes to `mb8nv8t3nk6xzed`. Body mode: JSON. Credential: `xc-token` (id `DT9tnRgqYpPc3rXo`).

**Node 28 — Step 21: Capture HSI Record ID** (Code)
Reads `response.Id` from the NocoDB POST response. Throws if absent. Rebuilds the full field set referencing `$('Step 18 - Run ID + Timestamp')` so downstream nodes have access to all computed values.

**Node 29 — Step 22: Build Patch Body** (Code)
Computes `hsi_patch_body: JSON.stringify({'HSI Status': 'Complete'})`.

⚠️ **Known Issue — dead code:** This computed value is never referenced by Step 23, which hardcodes its own body directly. Same pattern confirmed in EIP Step 17a and ESS Step 16. See Open Items #6.

**Node 30 — Step 23: NocoDB PATCH ESS Complete** (HTTP PATCH)
PATCHes **ESS's table** `m5yektnbtxf8evk` by `ess_record_id`. Body: hardcoded `{"HSI Status": "Complete"}` (ignores Step 22's computed value). This closes the ESS status loop: ESS set `HSI Status: "Ready"` when it handed off; HSI sets it to `"Complete"` when its own processing is done. The field lives on the ESS record.

Note: ESS uses `"Ready"` as its outbound placeholder status while all other agents use `"Pending"`. Cosmetic inconsistency only — see Open Items #7.

**Node 31 — Step 24: Build BRA Payload** (Code)
References `$("Step 21 - Capture HSI Record ID")` directly. Builds the BRA-bound payload:

Trace fields:
- `hsi_record_id`, `ess_record_id`, `eip_record_id`, `ala_record_id`
- `prior_run_id: hsi_run_id`, `prior_table: 'HSI'`

HSI outputs:
- `preliminary_response_tier`, `human_review_required`
- `signal_synthesis_summary`, `temporal_signal_insight`, `masked_emotion_hypothesis`, `contextual_linguistic_framing`
- `interpretation_confidence`, `pattern_type_recurrence`, `prior_record_count`
- `lang`, `client_id`

EIP pass-through fields (~20) that BRA requires for template selection and draft context.

⚠️ **Stale code comment:** The node contains the comment `// Plain object — NOT JSON.stringify (Chat #72: BRA trigger uses JSON body mode)` immediately followed by a line that calls `JSON.stringify(braPayload)`. The actual behavior — raw body mode, manually stringified — is correct and consistent with all other inter-agent trigger nodes in the pipeline. The comment is wrong, not the logic.

**Node 32 — Step 25: BRA Trigger** (HTTP POST)
POSTs to `http://161.35.133.49:5678/webhook/scx-bra`.
`timeout: 5000`, `retryOnFail: false` (explicitly set), `alwaysOutputData: true`.

⚠️ **Known Issue — missing fault tolerance:** No `onError: continueRegularOutput`. With `retryOnFail: false` also set, a BRA webhook failure will fail the entire HSI execution — even though the HSI NocoDB record was already successfully written one step earlier. This is the fourth confirmed instance of this gap across the pipeline (EIP→ESS, ESS→HSI, HSI→BRA all lack it; only ALA→EIP has it). See Open Items #6.

---

## NocoDB Schema

**Table ID:** `mb8nv8t3nk6xzed`
**Internal URL:** `http://nocodb:8080`
**Base ID:** `pq249fix22t3ofv`

| Field Name | Field ID | Type | Source |
|---|---|---|---|
| Id | c0607r51v6u2t4q | ID | NocoDB auto |
| CreatedAt | c3za5vlujb4alur | CreatedTime | NocoDB auto |
| UpdatedAt | cz2dmzt1ucg5w6x | LastModifiedTime | NocoDB auto |
| nc_created_by | cojdiy56zdqqusp | CreatedBy | NocoDB auto |
| nc_updated_by | c0ye8j7kdod9w82 | LastModifiedBy | NocoDB auto |
| nc_order | c0c2avqeuwfd6o4 | Order | NocoDB auto |
| HSI Run ID | cst26kxts2rr59m | SingleLineText | HSI Step 18 |
| ESS Record ID | csut9uben8tf0h9 | Number | Pass-through |
| HSI Timestamp | c7imf3d8vqs3dxe | DateTime | HSI Step 18 |
| Signal Synthesis Summary | c4lkamc5k4t3381 | LongText | Claude (Step 15) |
| Contextual Linguistic Framing | cfs1emzygey3yhv | LongText | Claude (Step 15) |
| Temporal Signal Insight | cdai0gzkdpa2r50 | LongText | Claude (Step 15) |
| Masked Emotion Hypothesis | cyp573flx0em4k9 | LongText | Claude (Step 15) or null |
| Interpretation Confidence | c5l84vnnoaur7v9 | SingleSelect | HSI Step 12 |
| Preliminary Response Tier | c8sfgaqc4238rrq | SingleSelect | HSI Step 16 |
| Human Review Required | cd4krvwpiplvk43 | Checkbox | HSI Step 17 |
| Pattern Type - Recurrence | ca6ugd9x5087en0 | SingleSelect | HSI Step 9 |
| Prior Record Count | cawsxs2cp1putua | Number | HSI Step 9/6b |
| Reviewer Handle | c9qkkdbbc2fjaj2 | SingleLineText | Pass-through |
| lang | cz6lcaxs89vgupz | SingleLineText | Pass-through |
| BRA Status | cl1250sz39sm45l | SingleSelect | Set to "Pending" by HSI; PATCHed by BRA |
| Error Log | c3yggyzs95hc1na | LongText | Error capture |
| ALA Record ID | c152r1sz5jn9hw7 | Number | Pass-through |
| Client ID | co62ectct8c3z5m | SingleLineText | Pass-through |

---

## S4A1–S4A6 Reference

### S4A1 — Context Classification (Step 7)
Input: `expression_mode` + `emotional_clarity`
Output: `context_bucket` (6 values: High-Clarity-Explicit, High-Clarity-Implicit, Masked-Ambiguous, Conflicted, Performative, Low-Clarity)
Purpose: Single categorical input to Claude prompt summarizing the signal's legibility.

### S4A2 — Signal Reliability (Step 8)
Input: `narrative_alignment_score`, `structural_confidence_score`, `dominant_pole`, `polarity_balance`
Output: `reliability_tier` (Strong / Moderate / Weak), `polarity_shift` (boolean)
Purpose: Assesses how much to trust the upstream ESS/EIP interpretation.

### S4A3 — Temporal Context (Step 9)
Input: `prior_record_count`, `is_named_reviewer`
Output: `recurrence_signal`, `pattern_type_recurrence`, `temporal_context`
Purpose: Converts reviewer history into a recurrence classification that informs Claude's churn/loyalty framing.
⚠️ Output may be permanently `First Occurrence` / `Sporadic` for all records if Step 6 IF bug is confirmed.

### S4A4 — Masked Emotion Prep (Step 10)
Input: `masked_emotion_flag`, `enriched_emotion_tag`, `expression_mode`, `canon_warnings`
Output: `masked_context` object
Purpose: Packages masked-emotion context for the Claude prompt's conditional hypothesis instruction.

### S4A5 — Behavioral Context Synthesis (Step 11)
Input: `enriched_emotion_tag`, `expression_mode`, `need_state`, `cognitive_driver`, `pain_point_sub_category`, `signal_type`
Output: `behavioral_context` (single sentence)
Purpose: Produces a concise human-readable context string that grounds the Claude call.

### S4A6 — Interpretation Confidence (Step 12)
Input: `emotional_clarity`, `narrative_alignment_score`, `structural_confidence_score`, `ambiguity_flag`, `intensity_level`
Output: `confidence_score` (0–1), `interpretation_confidence` (High / Medium / Low)
Purpose: Quantifies how much confidence HSI has in the behavioral interpretation it is about to request from Claude. Feeds tier assignment (Step 16) and human review gate (Step 17).

---

## Claude Interpretation Output Fields

| Field | Presence | Min Length | Spanish Support |
|---|---|---|---|
| signal_synthesis_summary | Required | ≥ 10 chars | Yes — content in Spanish if `lang=es` |
| contextual_linguistic_framing | Required | ≥ 10 chars | Yes |
| temporal_signal_insight | Required | ≥ 10 chars | Yes |
| masked_emotion_hypothesis | Conditional (null if `hypothesis_needed=false`) | none | Yes |

Field names always in English regardless of `lang`.

---

## Preliminary Response Tier Logic

Evaluated deterministically in Step 16 after Claude call. First match wins.

**T3 — Dignity-Restoration:**
Any of: `signal_type = 'Dignity-Risk Signal'` | `intensity_level = 'Critical'` | `masked_emotion_flag AND expression_mode = 'Masked' AND interpretation_confidence = 'Low'` | `pain_point_sub_category contains 'Trust, Dignity & Belonging'` *(likely dead — see Open Items #4)*

**T2 — Calibrated:**
Any of: `signal_type` in {Masked Negative Signal, Ambiguous Negative Signal, Mixed Signal} | `signal_type = 'Negative Signal' AND intensity_level` in {High, Moderate} | `ambiguity_flag = true` | `interpretation_confidence = 'Medium'` | `expression_mode = 'Conflicted'`

**T1 — Standard:** All other records.

---

## Human Review Gate

Implemented in Step 17. This is the concrete pipeline mechanism for the governance principle — not a policy statement.

```
human_review_required = true when:
  tier === 'T3'
  OR interpretation_confidence === 'Low'
  OR structural_confidence_score < 0.50
  OR reliability_tier === 'Weak'
```

Value written to HSI NocoDB and passed to BRA in the trigger payload. BRA uses it for draft governance. RDA respects it at the Approval Status stage.

---

## Token Budget

**Estimated ~700–900 tokens per record** (revised from v3.0's ~1,800 figure, which predated the real prompt structure).

- S4A1–S4A6 (Steps 7–12): 0 tokens — deterministic Code Nodes
- Step 13 Build Prompt: 0 tokens — Code Node
- Step 14 Claude API Call: ~600–800 tokens (max_tokens ceiling: 600; system + user prompt input ~200–300 tokens estimated)
- All other nodes: 0 tokens

⚠️ Open Item #7: Token budget not re-measured against the actual live prompt this cycle. The figure above is an estimate based on `max_tokens: 600` and observed prompt structure. A live measurement is needed for the final confirmed figure.

---

## Key Design Decisions

**Why HSI does not re-query Pain Point Master or Emotion Dictionary:**
EIP already resolved all classification. Re-querying would add ~20K tokens per record of redundant cost with no new information. HSI receives the resolved outputs as pass-through fields.

**Why Claude instead of GPT for HSI:**
HSI requires narrative synthesis and governance-constrained prose — how emotion + pain point pattern translates into guest behavioral interpretation. Claude claude-sonnet-4-6 produces more coherent multi-field narrative output under the detect/interpret-only constraint than GPT at equivalent token budgets.

**Why four Claude output fields instead of two:**
v3.0 described two outputs (Behavioral Narrative, Operator Insight). The live system generates four specialized fields: synthesis summary, linguistic framing, temporal insight, and masked hypothesis. This separation improves prompt specificity (each field has a narrow, defined scope) and makes downstream BRA/RDA usage cleaner — each field maps directly to a specific use in the draft context.

**Why `human_review_required` is a computed boolean rather than a policy flag:**
The four-condition gate (T3 tier, Low confidence, low structural confidence, Weak reliability) captures multiple independent failure modes. Any one condition is sufficient to require human review before BRA proceeds. This ensures the governance principle has teeth in the code, not just in documentation.

**Why `prior_record_count` uses a 100-record limit:**
The GET at Step 6a uses `limit: 100`. At current operational volume (10–15 records/day per location), this ceiling is not a constraint. Revisit if volume increases significantly.

---

## Architectural Corrections to v3.0

These are not incremental drift. Several v3.0 concepts do not exist in the live workflow.

1. **Acronym:** v3.0 used "Hospitality Signal Intelligence." The live Claude system prompt (Step 13) states "Human Signal Intelligence." This document uses "Human Signal Intelligence" as authoritative.

2. **Severity score does not exist.** v3.0 described a 1–10 severity score (Low=3, Medium=6, High=9 base, with modifiers). The live S4A1–S4A6 nodes compute a completely different set of outputs: context bucket, reliability tier, temporal context, masked emotion prep, behavioral context string, and `interpretation_confidence` on a 0–1 scale. There is no severity score anywhere in the live workflow.

3. **Claude outputs are four fields, not two.** v3.0's "Behavioral Narrative" and "Operator Insight" do not exist as field names in the live system. The real outputs are `signal_synthesis_summary`, `contextual_linguistic_framing`, `temporal_signal_insight`, `masked_emotion_hypothesis`.

4. **Run ID format:** v3.0's example `HSI-ServiceQuality-High-20260417-003` (domain + severity encoded) does not match reality. Live format: `HSI-YYYYMMDD-HHMMSS-mmm` (pure timestamp).

5. **Node count:** v3.0 stated 30 nodes. Live workflow has 32 nodes.

6. **v3.0 omits approximately half the workflow** — the entire idempotency check, canon gate, reviewer history lookup, tier classification, human review gate, NocoDB write, ESS PATCH, and BRA handoff sections are absent from v3.0.

---

## Open Items — Require Follow-Up

These cannot be resolved from code inspection alone. Documented explicitly rather than silently guessed.

**Open Item #1 — Acronym decision (closed for this document):**
"Human Signal Intelligence" adopted as authoritative based on the live system prompt. "Hospitality Signal Intelligence" (v3.0) is stale. No further action needed unless the naming is officially reversed.

**Open Item #2 — Last Updated reference:**
Chat number omitted per session instruction. Date only used.

**Open Item #3 — Step 6 IF likely routing bug (not yet live-tested):**
High confidence this is real based on code inspection: JS boolean `true` vs. string `"True"` with case-sensitivity on. Not yet confirmed by live test. **Recommended test:** Submit one record with a non-anonymous `reviewer_handle` that already has prior HSI records. Confirm whether Step 6a's GET actually fires. If the bug is confirmed, fix is a single IF node condition change (remove case-sensitivity or change comparator to `"true"` lowercase). Downstream impact if confirmed: `prior_record_count` is 0 for every record, `recurrence_signal` is `First Occurrence` for every record — repeat-guest and systemic-pattern detection non-functional in production.

**Open Item #4 — Step 16 Trust/Dignity T3 clause field mismatch:**
The clause `pain_point_sub_category.includes('Trust, Dignity & Belonging')` checks the wrong field. `'Trust, Dignity & Belonging'` is a domain name — it belongs in a check against `pain_point_domain_confirmed`, not `pain_point_sub_category`. Decision needed: treat as bug fix (change field reference) or behavior change requiring sign-off. Partial coverage of the same intent is retained via `signal_type === 'Dignity-Risk Signal'`.

**Open Item #5 — v3.0 severity score provenance:**
Unknown whether the 1–10 severity score was ever implemented and later replaced, or was aspirational and never built. Affects whether future documentation should note a removed feature. Requires input from whoever authored v3.0.

**Open Item #6 — Two pipeline-wide cleanup candidates:**
- **Dead patch-body pattern:** Step 22 computes a value never used by Step 23. Same pattern confirmed in EIP and ESS. Three agents affected. Recommend a single cleanup pass across all three simultaneously rather than per-agent fixes.
- **Missing `onError: continueRegularOutput` on inter-agent triggers:** Confirmed absent on EIP→ESS, ESS→HSI, HSI→BRA. Present only on ALA→EIP. A BRA webhook failure currently fails the entire HSI execution even after the HSI record is already written. Decision needed: fix as a pipeline-wide pass across all three, or per-agent as each is touched.

**Open Item #7 — Minor items not requiring immediate action:**
- Token budget not re-measured against live prompt structure. Estimate above is based on `max_tokens: 600` ceiling.
- `Absent` expression mode falls into `Low-Clarity` catch-all in S4A1 rather than its own bucket. Minor semantic gap.
- `canon_warnings.toString()` produces bare comma-joined array instead of readable formatting in Step 10. Cosmetic only.
- ESS uses `"Ready"` as its HSI-handoff status while all other agents use `"Pending"`. Cosmetic inconsistency.
- Step 24 code comment ("NOT JSON.stringify") contradicts the line immediately below it (which does call `JSON.stringify`). Logic is correct; comment is stale.

---

## Downstream Handoff

**HSI → ESS (PATCH):**
- PATCHes ESS table `m5yektnbtxf8evk` by `ess_record_id`
- Sets `HSI Status: "Complete"` on the ESS record

**HSI → BRA (webhook trigger):**
Endpoint: `http://161.35.133.49:5678/webhook/scx-bra`
Payload includes: all trace IDs, HSI outputs (`preliminary_response_tier`, `human_review_required`, four Claude fields, `interpretation_confidence`, `pattern_type_recurrence`, `prior_record_count`, `lang`, `client_id`), and ~20 EIP pass-through fields BRA requires.

**HSI → SIA:**
SIA reads directly from HSI's NocoDB table on its own schedule. No webhook from HSI to SIA.

---

## n8n Workflow Details

**Workflow Name:** SCX-HSI
**Trigger:** Webhook POST from ESS at `/scx-hsi`
**Auth:** None on webhook node
**NocoDB credential:** `xc-token` (httpHeaderAuth, id `DT9tnRgqYpPc3rXo`)
**Anthropic credential:** `x-api-key` (httpHeaderAuth, id `uMahlx4nOC5YJh0Z`)
**NocoDB internal URL:** `http://nocodb:8080` — never localhost
**Claude API URL:** `https://api.anthropic.com/v1/messages`
**Claude manual headers:** `anthropic-version: 2023-06-01` + `Content-Type: application/json`
**Model:** `claude-sonnet-4-6`, `max_tokens: 600`, `temperature: 0.3`
**SplitInBatches:** Not used in HSI — ESS triggers HSI one record at a time
**n8n version:** 2.4.6 self-hosted

**Critical build rules (permanent, apply to all future edits):**
- No spread operators in Code Nodes — key-by-key construction only
- IF node FALSE branch requires `return []` gate
- PATCH uses JSON body mode, not RAW
- `require('http')` works in Code Nodes — `$http`/`$helpers`/fetch blocked
- `for` loops over index preferred; avoid `for...of`, `continue`, `const` inside loop bodies
- `pageInfo.totalRows` not `list.length` for NocoDB record counts
- Agent trigger body: `typeof body === 'string' ? JSON.parse(body) : body`

---

## Related Documents

- **v3.0 (superseded):** `SCX_HSI_HOW_v3.md`
- **Upstream (ESS):** `SCX_ESS_HOW` (current version)
- **Upstream (EIP):** `SCX_EIP_HOW` (current version)
- **Downstream (BRA):** `SCX_BRA_HOW_v3.2.md`
- **Downstream (SIA):** `SCX_SIA_HOW` (reads HSI NocoDB directly)
- **Schema Registry:** `mqv1znpza948pm9`
- **MCD:** SubtextCX_MCD (current version)

---

*SCX_HSI_HOW_v4.0 · Human Signal Intelligence · Solofella LLC · July 24, 2026*
*First full technical HOW. Supersedes v3.0 in all respects.*

