# SCX_RDA_HOW_v4.1

**VRYOH INTELLIGENCE · SOLOFELLA LLC**
**HOW DOCUMENT — RDA**
**Response Drafting Agent**
Complete Step Decomposition · Node Logic · Code · Prompts · Field Contracts
v4.1 · July 3, 2026 · Chat #96

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0 | Chat #49 — Initial build |
| v2.0 | Chat #58 — Audit remediation + NocoDB rationalisation. 20 → 18 nodes. |
| v3.0 | Chat #74–75 — Five-call generation architecture. 18 → 23 nodes. |
| v3.1 | Chat #75 — Five-prompt synchronicity audit. 11-item audit checklist. Quality baseline 87%. |
| v3.2 | Chat #77 — SCX-Sheet-Sync OAuth credential failure documented. No RDA changes. |
| v4.0 | Chat #88–89 — Spanish language deployment complete. Multi-client architecture (6 active clients). Brand voice schema rebuilt. Client Config 22 fields. Step 6 full rewrite. Step 7a lang_resolved fix + anti-repetition + CARRY LEAD RESTRICTION. Steps 7c, 7e, 7g prompt rewrites. Step 9b 11-item audit. Product renamed VRYOH Intelligence. |
| **v4.1** | **Chat #96 — Spanish deployment reverted to English-only baseline after live test showed language-mixing and coherence failures. Phase 3 Signal Enrichment layer added (Steps 6a–6e) — EIP/Emotion Dictionary/Pain Point Master data injected into draft generation for the first time. Step 9e deterministic compliance checker added. Priority-template fixed sentences removed from Step 7a. Governance vs. Style architectural principle established from real PAK-001 client-history calibration. Diagnose-only communication protocol locked. Multiple real bugs found and fixed (6c/6d stale-reference lookup failure; 7a undefined-variable error; 7e duplicate declaration; 9b syntax error; 7c missing concatenation operator, unresolved as of this document).** |

---

## Summary Grid

| Property | Value |
|----------|-------|
| **Workflow Name** | SCX-RDA |
| **Product Name** | VRYOH Intelligence |
| **Model** | claude-sonnet-4-6 — all Claude calls |
| **Pipeline Position** | 7th agent — final per-record automated agent |
| **Claude Calls / Record** | 5 — opening, body, SEO/gov, internal brief, audit |
| **Trigger Source** | BRA Webhook — rich payload (Halt never fires) |
| **Languages** | **English-only (reverted Chat #96). Spanish deployment complete Chat #88–89, deactivated Chat #96 — Phase 4 status.** |
| **Signal Enrichment** | **NEW — Steps 6a–6e. EIP Cognitive Driver/Need State + Emotion Dictionary Common Expressions + Pain Point Master Operational/Emotional Signal/Sample Expression, consolidated into `signal_enrichment_brief`.** |
| **Deterministic Compliance Layer** | **NEW — Step 9e. Auto-corrects "landed"/"landing"; flags (does not auto-insert) missing T2 contact-invitation and near-miss banned phrases.** |
| **Post-Call Scan** | Deterministic prohibited-term scan (English-only, Spanish array removed) |
| **Total n8n Nodes** | 28 main (23 v4.0 baseline + 4 new signal-enrichment nodes + 1 new compliance-check node) |
| **NocoDB Table Fields** | 20 — RDA table schema **FROZEN Chat #96, no new columns permitted** |
| **Human Approval** | MANDATORY — no draft reaches any platform without human decision |
| **Approval SLA** | 48 hours (locked Chat #42) |
| **Active Clients** | PAK-001, AJI-001, AJI-002, AJI-003, AJI-004, AJI-005 |
| **Client Config Fields** | 25 fields (includes Sheet ID/Sheet Tab Name for SCX-Sheet-Sync) |
| **Opening Variation** | Step 9a-2: 2 recent drafts fetched per client. Priority-template sentences removed Chat #96 — brand voice now shapes construction directly. |
| **Audit Checklist** | 12 items — added Embedded Criticism Confirmation (Chat #96) |
| **Governance/Style Principle** | **NEW — locked Chat #96. See Section 12.** |
| **Doc Version** | v4.1 — July 3, 2026 |

---

## 1. AGENT PURPOSE

*(Unchanged from v4.0 — restated for completeness.)*

### RDA — Response Drafting Agent

Translates BRA's response strategy into governed draft language calibrated to the response tier and the client's brand voice. RDA produces two outputs per record: a public-facing response and an internal signal brief for the client management team. RDA does not decide what to do — it decides how to say it.

**The response must still read as written by a person, not optimized by a machine.** Every Claude call in RDA carries this as its governing principle.

**TERMINAL AGENT:** RDA is the final per-record automated agent. No draft produced by RDA reaches any platform before a human makes an explicit approval decision. Claude's scope ends at draft generation.

**GOVERNING PRINCIPLE — QUALITY:** Fewer, broader rules that Claude applies with judgment — not exhaustive lists. **Chat #96 addition: rules should also be classified by enforcement layer — mechanical/deterministic rules belong in code, judgment rules belong in the Claude prompt. Mixing both categories in one prompt was identified this session as a contributing cause of rule-competition failures.**

### RDA PRODUCES
*(Unchanged from v4.0.)*
- Public Response Draft
- Internal Follow-Up Draft
- Commercial Commitment Flag
- Approval Status
- Audit Passed flag + Audit Failed Items (now 12-item checklist)

### RDA DOES NOT
*(Unchanged from v4.0.)*

---

## 2. INPUT CONTRACT

### Source 1 — BRA Webhook Payload

Unchanged from v4.0's 28-field contract. **`lang` field is retained in the contract but its practical value is always `"en"` as of Chat #96** — Spanish routing deactivated, not removed from schema (Phase 4 re-activation would reuse this field).

### Source 2 — NocoDB Client Config

Field list unchanged from v4.0 (18 brand-voice fields + Sheet ID/Sheet Tab Name + Id/CreatedAt/UpdatedAt/nc_ audit fields = 25 total real columns, confirmed directly against live table Chat #96).

**Correction to v4.0 terminology:** v4.0 labeled field #4 `vocabulary_restrictions`. **Real, confirmed column name and mapped key is `response_negative_experience`** — `vocabulary_restrictions` does not exist anywhere in the real Client Config schema or the object Step 6 builds. This mislabeling in v4.0 propagated into Step 7c's prompt as `bv.vocabulary_restrictions`, a genuine bug (confirmed always resolving to `'Not specified'`), fixed Chat #96 by replacing the manual per-field BRAND VOICE BRIEF block with `brand_voice_brief` (Section 3).

### Source 3 — NEW (Chat #96): EIP Table Direct Fetch

RDA now fetches directly from the EIP table (`mhicpnrahaesxmy`) by `eip_record_id`, rather than relying on ESS/HSI/BRA pass-through, which was confirmed to drop `Cognitive Driver` and `Need State` somewhere in that chain.

### Source 4 — NEW (Chat #96): Emotion Dictionary + Pain Point Master, Targeted Lookup

RDA performs a single-row lookup (not full-dictionary injection, which is EIP's method) against:
- Emotion Dictionary (`mrrscb955j1d2i7`) WHERE `Core Emotion` = EIP's live `Enriched Emotion Tag` value
- Pain Point Master (`meavqh37mdqgl4d`) WHERE `Enriched Pain Point` = EIP's live `Enriched Pain Point` value

Confirmed 1:1 match between EIP's SingleSelect option values and both dictionaries' key fields.

---

## 3. SIGNAL ENRICHMENT LAYER — STEPS 6a–6e (NEW, Chat #96)

**Purpose:** deliver the actual hospitality-specific differentiator into draft generation. Prior to this addition, RDA generated drafts using only HSI/BRA's compressed categorical labels (`emotional_axis`, `pain_axis`, `enriched_emotion_tag` as a bare tag) — never the rich dictionary content (Common Expressions, Cognitive Driver, Need State, Operational/Emotional Signal, Sample Expression) that EIP's own architecture is built to produce.

### Step 6a — Brand Voice Consolidation
Parses `brand_voice` object (built at Step 6, unchanged from v4.0) into a single formatted string, `brand_voice_brief` — 12 fields: Register, Core Driver, Regional Accent, Personality, Formality, Person Preference, Response to Negative Experience, Guest Should Feel, Recovery Protocol, Differentiator, Brand Phrases Include, Brand Phrases Avoid. Replaces the previous pattern of each downstream node (7c, 7e, 7g) independently re-parsing `bv.*` fields — confirmed in v4.0's code as duplicated logic across three separate nodes.

```javascript
const inp = $input.first().json;
const bv = inp.brand_voice || {};
// ... builds briefLines array, one per field ...
const brandVoiceBrief = briefLines.join('\n');
const out = {};
const inpKeys = Object.keys(inp);
for (let i = 0; i < inpKeys.length; i++) { out[inpKeys[i]] = inp[inpKeys[i]]; }
out.brand_voice_brief = brandVoiceBrief;
return [{ json: out }];
```

### Step 6b — Fetch EIP Signal Data
```
GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mhicpnrahaesxmy?where=(Id,eq,{{ $('Step 6a — Brand Voice Consolidation').first().json.eip_record_id }})&limit=1
```
Pulls `Cognitive Driver` and `Need State` directly from EIP's own table row — confirmed present there but confirmed NOT surviving the ESS→HSI→BRA pass-through chain into RDA's payload.

### Step 6c — Fetch Emotion Dictionary Row
```
GET .../mrrscb955j1d2i7?where=(Core%20Emotion,eq,{{ $('Step 6b — Fetch EIP Signal Data').first().json.list[0]['Enriched Emotion Tag'] }})&limit=1
```
Pulls `Common Expressions` for the matched emotion.

**BUG FOUND AND FIXED, Chat #96:** original implementation referenced `$('Step 6a...').first().json.enriched_emotion_tag` — a stale value that traveled unmodified from Step 2 through the entire chain, not a fresh value. Confirmed via live test: this stale reference caused a real 0-row lookup failure (`Enriched Emotion Tag: "Ambivalence"`, live in EIP, present and confirmed in the dictionary table, but the query using the stale reference still returned `totalRows: 0`). Fixed by re-pointing the lookup to Step 6b's live-fetched value. Confirmed deployed and working after fix.

### Step 6d — Fetch Pain Point Master Row
```
GET .../meavqh37mdqgl4d?where=(Enriched%20Pain%20Point,eq,{{ $('Step 6b — Fetch EIP Signal Data').first().json.list[0]['Enriched Pain Point'] }})&limit=1
```
Same stale-reference bug found and fixed identically. Pulls `Operational Signal`, `Emotional Signal`, `Sample User Expression`.

### Step 6e — Build Signal Enrichment Brief
Consolidates 6b/6c/6d results into `signal_enrichment_brief`, a single string:
```
COGNITIVE DRIVER: [...]
NEED STATE: [...]
COMMON EXPRESSIONS: [...]
OPERATIONAL SIGNAL: [...]
EMOTIONAL SIGNAL: [...]
SAMPLE EXPRESSION: [...]
```
Passes through all upstream fields unchanged, per the same key-by-key pattern as Step 6a.

**Confirmed node execution order:** 6a → 6b → 6c → 6d → 6e → 7a, strictly sequential (required — 6c/6d depend on 6b's output).

**Field lifecycle:** `signal_enrichment_brief` is injected into 7a's `SIGNAL ENRICHMENT` line and 7c's `SIGNAL DETAIL` block. Passed through unused by 7e (confirmed no task in 7e needs it). Used again in 7g's `internal_user_prompt`. **Deliberately does not travel past Step 8** — no consumer exists downstream, and per the RDA schema-freeze rule, no new NocoDB column stores it.

**No new NocoDB columns created anywhere in this layer** — locked rule, Chat #96: RDA table schema is frozen; all Phase 3 data travels in-pipeline only.

---

## 4. TIER-SPECIFIC DRAFT INSTRUCTIONS

Structurally unchanged from v4.0's table. **Content changes, Chat #96:**

- **T1 opening greeting:** v4.0/original design = no greeting word, direct name address (`"[Name], ..."`). **Confirmed and reconfirmed correct across multiple Chat #96 test batches** — this is intentional tier differentiation, not a defect, despite initial appearance of inconsistency.
- **T2/T3 opening greeting:** unchanged — "Hi/Hello [Name] —".
- **T2 contact-invitation requirement:** confirmed **still unreliable** — recurring failure across 6+ consecutive test batches this session, unresolved as of this document (Section 11).

### Absolute Governance Constraints — All Tiers

v4.0's blanket list is **superseded by the Governance vs. Style distinction established Chat #96** (Section 12) — several items in v4.0's "always prohibited" list are confirmed, via real client history, to be legitimately client-specific rather than universal (see Section 12 for the corrected framework).

---

## 5. SIGNAL-DETECTION ARCHITECTURE, STEP 7a (REVISED Chat #96)

**Removed:** the fixed-sentence Priority 1-6 templates from v4.0 (`→ PLEASURE LEAD:`, `→ GUEST WORD LEAD:`, etc.) — confirmed to constrain brand-voice differentiation, since every client received structurally identical sentences differing only by inserted words, regardless of Register/Formality/Personality differences between e.g. PAK-001 and Aji Ceviche.

**Replaced with:**
```
BRAND VOICE — THIS SHAPES YOUR SENTENCE:
Use Register, Core Driver, Regional Accent, Personality, and Differentiator... to decide the tone, rhythm, and wording...

GUEST NAME: Always address the guest by name when available.

SIGNAL CHECKLIST — identify which applies, in this order, stop at first match. This tells you WHAT to notice — brand voice tells you HOW to write it:
OPENING PRIORITY: Guest Name → Named Staff → Specific Detail → Fallback (T1 only)...
Detect Occasion and Belonging for use in the body only, never in the opening.
```

**Confirmed dead instruction, unresolved:** "Detect Occasion and Belonging for use in the body only" has no observable effect — 7a's Claude call returns one plain-text sentence with no structured output schema, so there is no channel for a detected value to reach Step 7c. Two remediation paths discussed, neither built: (a) give 7a a structured-output contract, or (b) rely on 7c's own detection since 7c may already receive the raw review text (unconfirmed — dependent on whether 7c's prompt includes `raw_text`/`clean_text`, not verified this session).

**Recurring bug pattern in this file specifically:** two separate undefined-variable errors introduced this session when copying instruction blocks between 7a/7c/7e (variable declared as `rec` in one file, referenced as `inp` in a copied line, and vice versa). Both instances found and fixed; flagged here as a pattern to check on any future edit to this file.

---

## 6. STEP 7c — DRAFT BODY BUILDER (REVISED, Chat #96)

**Confirmed live structure (Quality Rules rewrite):** Role → Opening Rule → Tier Rules → Tone → Brand Voice → **Quality Rules (NEW section)** → Style Standard → Output.

**Quality Rules, new this session:**
```
- Detect Occasion when the review mentions a birthday, anniversary, celebration, date night, holiday, or special visit...
- Detect Belonging when the guest signals discovering the restaurant, being new to the area...
- Treat all review details as historical unless the input explicitly confirms they are current.
- Commercial References [prices, discounts, menu tiers, seasonal offerings, promotions, temporary offers]...
- Never repeat, imply current availability of, or encourage a future visit around a Commercial Reference.
- Timeless Closing Anchors are: (1) occasion, (2) guest relationship/loyalty, (3) overall experience, (4) favorite food/beverage (non-promotional), (5) non-commercial guest wording.
- Always choose the highest-priority Timeless Closing Anchor available.
- Never use a Commercial Reference as a closing anchor.
```

**CONFIRMED, UNRESOLVED BUG:** missing string-concatenation operator between the Quality Rules block's final line and the `Style Standard:` section header:
```javascript
'- Never use a generic closing or repeat a specific noun already used in the opening.\n'
'Style Standard:\n' +
```
No `+` joins these two literals. **Style Standard and Output sections are very likely never actually concatenated into the live `body_system_prompt` string.** Leading hypothesis for observed cross-guest phrase repetition in multiple test batches (the instruction meant to prevent this — "vary sentence structure and avoid repeated phrases across guests" — may never reach Claude). Flagged, not yet fixed as of this document.

**Confirmed real failure, Commercial Reference rule:** live test showed a draft closing on "the $60 menu will be waiting when you're ready to work through the rest of it" — a direct Commercial Reference violation. Diagnosed root cause: affirmative instructions ("reinforce review specifics," "use guest wording") compete with and can override the negative prohibition when both point to the same candidate detail — a general, documented LLM behavior pattern, not unique to this build. Two remediations proposed, neither built: (a) rewrite as an explicit filter-then-select procedure ("ignore Commercial References first, then choose from what remains"), (b) move entity classification (Commercial Event vs. Guest Occasion vs. neutral detail) to a deterministic pre-processing step in code, so generation only writes rather than also classifying.

**Confirmed conceptual gap:** "Occasion" detection language does not distinguish a personal guest occasion (birthday) from a commercial/promotional event (Restaurant Week) — both can satisfy the same detection wording, confirmed by a real test where "Restaurant Week" triggered Commercial Reference misuse via Occasion-adjacent framing.

**Confirmed working, added this session:** Embedded Criticism Confirmation rule (does not confirm a guest's negative characterization as accepted fact — added after a real case where a draft response validated a guest's claim that a specific seat was objectively the worst in the city). Differentiator instruction revised to require demonstration through specific detail rather than literal category-label insertion — confirmed working in later test batches after initial failures showed "chef-driven dining" inserted as a literal phrase twice in one batch.

---

## 7. STEPS 7e/7g/8/9b/9d — PASS-THROUGH CHAIN (Chat #96 updates)

All five nodes confirmed carrying `brand_voice_brief` end-to-end (7a→9d, retroactively added across every node this session). `signal_enrichment_brief` confirmed carried 7a→7c→7e→7g, intentionally not carried past 7g. `guest_name` confirmed carried 7a→7c→7e→7g→8→9b→9d→10-13 (per user-confirmed deployment; not independently re-verified node-by-node for 11-13).

### Step 9b — 12-Item Audit (was 11 in v4.0)
Item 12 added: **EMBEDDED CRITICISM CONFIRMATION** — flags if the draft restates a guest's negative characterization as accepted fact. Native Spanish LANGUAGE block added but **currently inert** (system is English-only as of Chat #96).

**Unresolved gap, both v4.0 and this version:** rule "opening/closing repetition vs. recent client drafts" remains audit-only (item 10) — no code-level or generation-time equivalent exists, meaning it's only caught after the fact, not prevented or auto-corrected.

### Step 9d — Closing/Signature
**v4.0's tier-aware EN/ES closing rotation was lost during the Chat #96 English-only revert** (the pre-Spanish original file this reverted to had no closing-signature logic at all). **Re-added Chat #96**, English-only:
```javascript
const closings = inp.confirmed_response_tier === 'T1'
  ? ['Warm regards', 'Warmly', 'Cheers!', 'Have the best day!']
  : ['Warm regards', 'Sincerely', 'With appreciation'];
```
Spanish arrays (`closingsES_T1`/`closingsES_T2T3`) deliberately not carried forward as dead code, per explicit instruction — Phase 4 reactivation would re-add them at that time, not before.

---

## 8. STEP 9e — DETERMINISTIC COMPLIANCE CHECK (NEW, Chat #96)

**Placement:** between Step 9d and Step 10.

**Rationale:** direct response to test evidence showing the same 2-3 specific violations recurring across 6+ batches, despite prompt-level rule statements and an LLM-based audit call (9b/9c) both theoretically covering them. Moves what's mechanically checkable out of the audit's judgment-dependent process into deterministic code.

**Function:**
```javascript
// Auto-corrects, safe single-word swap
if (landedRegex.test(draft)) { draft = draft.replace(landedRegex, 'came together'); }
// Flags only — does NOT auto-insert a fixed sentence
if (tier === 'T2' && !contactRegex.test(draft)) {
  deterministicFailures.push('T2 MISSING CONTACT INVITATION (flagged — not auto-corrected)');
  humanReviewRequired = true;
}
```

**Design principle, explicit:** auto-inserting a fixed sentence for the missing-contact-invitation case was deliberately avoided — this would reintroduce the exact templating/sameness problem that the Priority 2-6 removal (Section 5) was built to solve. A human catching and writing the correction once is preferable to code silently inserting the same canned line on every affected record.

**Status:** user-confirmed deployed. **Not independently field-verified this session** — no live NocoDB field values (`human_review_required`, `audit_failed_items`) were shown to confirm the flag actually fires; only draft text was reviewed across available test batches, which cannot show this by itself since the design intentionally leaves the draft text unchanged for this specific violation.

**Known gap:** does not yet check rule #10 from the audit checklist (opening/closing repetition) or the Commercial Reference violation — both confirmed recurring, neither added to this node yet.

---

## 9. SPANISH DEPLOYMENT — COMPLETE HISTORY (Chat #86-89, deactivated Chat #96)

*(Per explicit instruction, full historical record preserved for Phase 4 reference.)*

**Deployment (Chat #86-89):** ALA Step 6 language detection activated — CSV `lang` column drives `lang` field, English default. EIP Step 5 Dictionary Load Check added `emotion_table_id`/`pain_table_id` routing: ES = `mhot4w62tupht71`/`mwmiyyoucuhsms8`, EN = `mrrscb955j1d2i7`/`meavqh37mdqgl4d`. EIP INIT nodes made dynamic (URL selection, lang-conditional field mapping). EIP domain validation bypassed for `lang=es` records. ESS Step 7 language instruction added. RDA Step 6 mapped Client Name field. RDA Step 9d built lang-conditional closing rotation (ES: Atentamente/Cordialmente/Hasta pronto/Con gusto) with generic brand signature via Client Name. RDA Step 7a: `lang_resolved` fix (`(inp.lang || 'en').toLowerCase().trim()` — never derived from `bv.language`, since that field returns comma-joined bilingual values like `"En, Es"` for some clients, which would break routing). Anti-repetition rule and CARRY LEAD RESTRICTION added to reduce cross-record "hearing that" overuse. Client Config required 3 new fields at the time (Recovery Protocol, Brand Voice Anchors, Commitment Restrictions — later confirmed the actual live field names differ slightly: RECOVERY PROTOCOL, Guest Feeling, Commitments NEVER, etc.).

**HSI/BRA/SIA/MRA required no changes** for Spanish support.

**Live quality benchmarks established at deployment (Chat #89):** Luis (AJI-001, ES, T1) — Spanish T1 benchmark. Arlette Acosta (AJI-001, EN, T1). John Diaz (PAK-001, EN, T1). Barkev (PAK-001, EN, T1).

**Deactivation (Chat #96):** a live test batch, run after the Phase 1/2 optimization work this session, showed two Spanish-primary records with genuine language-mixing within single drafts (opening in one language, body switching to the other mid-draft) and at least one instance of the banned pipeline-term "señal" leaking into output despite an explicit prohibition. Root-cause reasoning discussed: Claude's language output is an emergent, token-level, probabilistic property — not a hard-enforced switch — and competing instructions (content-shaping rules stated alongside a late-positioned LANGUAGE instruction) can cause the language constraint to lose priority under certain conditions, especially for phrases whose English-language completion is more statistically dominant in the model's training distribution than a natural Spanish equivalent.

**Decision:** revert the full 7a→9d chain to the original, pre-Spanish, English-only source code (confirmed 85-96% historically effective per user-provided figures, two different numbers given at different points, not independently reconciled), then retrofit `brand_voice_brief` and Phase 3 signal enrichment on top of that restored baseline — rather than attempting to patch the Spanish-era build further.

**Phase 4 status: deferred, not abandoned.** Re-activation trigger, per the original locked roadmap: "only if Phase 2 test shows Spanish still AI-detectable... decision based on evidence not prediction." That evidence now exists (the language-mixing failures) but no re-activation attempt has been made this session — Phase 4 remains a distinct, deferred future item, with three architectural options discussed but not selected: (a) fully separate EN/ES system prompts per node (reduces competing-instruction risk, since the model would never hold an English rule-set while producing Spanish tokens), (b) post-generation deterministic language-detection + regenerate-on-mismatch (a safety net, not a prevention method), (c) few-shot examples shown natively in the target language rather than described abstractly in English (documented as more reliable for format/language adherence than prose instruction alone).

---

## 10. NODE MAP

```
[01] Webhook (POST /webhook/scx-rda)
[02] Code Node — Payload Validation
[03] NocoDB GET — Idempotency
[04] IF Node — Already processed?
[05] NocoDB GET — Brand Voice Load (Client Config)
[06] Code Node — Build Brand Voice Object
[06a] Code Node — Brand Voice Consolidation → brand_voice_brief          ← NEW Chat #96
[06b] HTTP GET — Fetch EIP Signal Data (Cognitive Driver, Need State)    ← NEW Chat #96
[06c] HTTP GET — Fetch Emotion Dictionary Row (Common Expressions)       ← NEW Chat #96, bug-fixed
[06d] HTTP GET — Fetch Pain Point Master Row (Operational/Emotional/Sample) ← NEW Chat #96, bug-fixed
[06e] Code Node — Build Signal Enrichment Brief                          ← NEW Chat #96
[07a] Code Node — Build Opening Prompt (Priority templates removed, English-only, revert baseline)
[07b] HTTP POST — Opening Claude Call
[07c] Code Node — Parse Opening + Build Body Prompt (Quality Rules added; concat-operator bug unresolved)
[07d] HTTP POST — Body Claude Call
[07e] Code Node — Parse Body + Build SEO/Gov Prompt (Embedded Criticism rule added)
[07f] HTTP POST — SEO/Gov Claude Call
[07g] Code Node — Parse SEO Draft + Build Internal Brief Prompt
[07h] HTTP POST — Internal Brief Claude Call
[08]  Code Node — Build Final Output
[09a] NocoDB GET — Fetch ALA Record
[09a-2] NocoDB GET — Fetch Recent RDA Drafts
[09b] Code Node — Build 12-Item Audit Prompt (item 12 added; Spanish block inert)
[09c] HTTP POST — Audit Claude Call
[09d] Code Node — Parse Audit + Append Closing Signature (re-added, English-only, Chat #96)
[09e] Code Node — Deterministic Compliance Check                        ← NEW Chat #96
[10]  Code Node — Output Validation
[11]  Code Node — Commercial Commitment Scan (English-only, Spanish array removed Chat #96)
[12]  Code Node — Approval Status Assignment
[13]  Code Node — RDA Run ID + Timestamp
[14]  Code Node — Build NocoDB POST Body
[15]  NocoDB POST — Write RDA Record
[16]  Code Node — Capture RDA Record ID
[17]  NocoDB PATCH — BRA RDA Status → Complete
[18]  Code Node — Build Approval Notification + Email Node

TOTAL: 28 main nodes (23 v4.0 baseline + 5 new: 6a/6b/6c/6d/6e + 9e)
Claude fires at [07b], [07d], [07f], [07h], [09c] only — 5 calls per record, unchanged count
```

---

## 11. APPROVAL STATUS LIFECYCLE

Unchanged from v4.0.

---

## 12. GOVERNANCE vs. STYLE — ARCHITECTURAL PRINCIPLE (NEW, Chat #96)

Established through direct analysis of 5 real PAK-001 responses, all published on Google, all written by Eddy (F&B Director) — not hypothetical examples.

**Confirmed pattern across all 5 real responses:** thanks-formula opener (5/5), a repeated appreciation phrase ("we greatly appreciate...") (4/5), generic sign-off (5/5), low specificity on most (4/5). **This is the client's authentic, consistently demonstrated voice** — not an anomaly, and directly conflicts with several of RDA's generic style rules (which were built to eliminate exactly this pattern as "generic AI-sounding" output).

**One example (a genuine 1-star, severe-harm review) showed Eddy personally identifying himself and sharing a real work email** — initially assessed as a governance violation, then re-diagnosed: this is Eddy's deliberate, configured recovery approach for severe cases, consistent with PAK's own Recovery Protocol field value ("The Transparent Owner — Direct accountability, zero defensive language"), not an accidental leak.

**Resulting principle, locked:**

**Universal governance (never client-configurable, identical for every brand):**
- No financial commitments (refunds, discounts, vouchers, specific compensation).
- No temporal/commercial hallucination (implying current availability of an expired promotion).
- Tier-severity matching (apology language must match the severity of harm described — confirmed real violation in the calibration data: a generic apology given for a review describing food being aggressively thrown and a simple accommodation request being refused).

**Client-configurable (per brand, driven by Client Config + real historical approved drafts):**
- Opening/closing formula tolerance and specificity level.
- Contact-channel specificity for T2/T3 (generic vs. personal identification + email) — **trigger is signal severity (tier), never star rating** — using star count as the trigger would defeat the product's core masked-emotion-detection differentiator, since star rating and true signal severity are confirmed to diverge (a 5-star review can carry a real embedded complaint; a 1-star review's actual severity should be judged by HSI's signal analysis, not the number alone).
- Closing Anchor Priority (which detail a brand emphasizes first — occasion, relationship, food, guest wording) — **proposed, no implementation decided.** No corresponding Client Config field currently exists for this; would require either a new Client Config column (a different table from RDA's frozen schema, so not blocked by the RDA-specific freeze rule) or derivation from an existing field (e.g., inferring priority from Core Driver's value).

**Direct connection to the implementation roadmap:** this finding is the concrete justification for implementation item #11 (client-specific approved-example aggregation, already on the locked 11-item list) — rather than writing generic hypothetical examples, each client's own real approved/edited historical drafts should become that client's actual few-shot reference material.

---

## 13. NOCODB RDA TABLE — 20 FIELDS, SCHEMA FROZEN

Field list unchanged from v4.0. **New rule, locked Chat #96: no additional columns will be added to this table going forward.** All new data (Phase 3 signal fields, deterministic-check results, `brand_voice_brief`, `guest_name`) travels in-pipeline only, never persisted as a new column.

---

## 14. ACTIVE CLIENT CONFIGURATION

Unchanged from v4.0 — 6 active clients (PAK-001, AJI-001 through AJI-005). **Language column now practically always "en" for all clients** — AJI's EN+ES designation in v4.0's table reflects the deactivated Spanish capability, not current live behavior.

---

## 15. CREDENTIALS + CONFIGURATION

Unchanged from v4.0.

---

## 16. STANDING PROCESS RULES (LOCKED Chat #96)

- **DIAGNOSE-ONLY RULE:** when identifying a problem, gap, or issue, state the diagnosis only. Do not propose, write, or suggest a fix/solution/code unless explicitly requested. Applies every time, not just once.
- **web_fetch DISTRUST:** `web_fetch` on this repo repeatedly returned only the bare URL instead of real file content across many attempts this session; do not treat `web_fetch` results as verified without explicitly checking the raw returned content in the same response before commenting on it. Direct user-pasted code remains the primary source of truth.
- **RDA table schema FROZEN** (Section 13).
- Every response begins `Chat #[N] · [Date]`.
- Quality target: RDA output should approach 90%. Cost is not a blocking constraint if additional calls/complexity are needed to reach that target — can be absorbed into pricing.

---

## 17. QUALITY BASELINE + OPEN ITEMS — Chat #96

### What is working:
- Signal Enrichment layer (6a-6e) deployed, real bug found and fixed (stale-reference lookup failure), confirmed producing real dictionary content in live tests ✅
- Tier-based greeting differentiation (T1 direct-name vs. T2/T3 Hello/Hi) confirmed correct across multiple batches ✅
- Closing-signature rotation restored after being lost in the revert, confirmed tier-aware in live output ✅
- Embedded Criticism Confirmation rule confirmed working — no further instances of confirming a guest's negative characterization as fact, after the fix ✅
- Differentiator instruction fix confirmed reducing (not eliminating) literal slogan-insertion ✅
- "landed"/"landing" no longer observed in the most recent test batch (post Step 9e deployment — though causal link not isolated from other simultaneous changes) ✅

### Known, unresolved gaps as of this document:
| Item | Status |
|------|--------|
| Step 7c missing `+` operator (Style Standard/Output likely unreachable) | Diagnosed, not fixed |
| T2 missing contact-invitation | Recurring 6+ batches; Step 9e flags but doesn't correct; root prompt cause unresolved |
| Occasion/Belonging dead instruction in 7a | No structured output channel exists |
| Commercial Reference vs. positive-instruction conflict | Diagnosed; procedural rewrite and code-classification alternatives proposed, neither built |
| Occasion vs. Commercial Event conflation | No distinguishing logic exists |
| Closing Anchor Priority as client-configurable | No Client Config field exists yet; implementation undecided |
| Rule #10/#35 (cross-record repetition) | Audit-only; no deterministic/generation-time equivalent |
| 7a/7c mutual consistency after latest edits | Not independently re-verified |
| Step 9e field-level verification | User-confirmed deployed; not independently confirmed via live NocoDB field values |

---

**VRYOH Intelligence · SCX_RDA_HOW_v4.1 · Chat #96 · July 3, 2026 · Solofella LLC**
