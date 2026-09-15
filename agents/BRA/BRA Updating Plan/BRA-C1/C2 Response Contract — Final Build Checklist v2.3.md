Chat #158 · September 10, 2026

# BRA-C1/C2 Response Contract — Final Build Checklist v2.3

**Incorporates GPT's 13 adopted findings + Gemini's 2 findings from the final checklist audit (Chat #156-157). This supersedes v2.2's checklist. Ready to build against.**

---

## Phase 0 — Schema & Ontology Freeze
*No nodes built. Spec-locking only.*

- [ ] Response Contract schema confirmed final — all fields, types, enums
- [ ] **Contract Field Manifest created** — single authoritative table listing every field (`must_reflect`, `may_reflect`, `must_not_introduce`, `commercial_restriction_types`, `primary_claim_type`, `verification_state`, `highest_risk_class`, `strictest_epistemic_posture`, `closing_objective`), who produces it, who validates it, whether RDA receives it — checked against the actual schema so "6 fields" / "7 fields" language never drifts from reality again *(GPT #2)*
- [ ] `primary_claim_type` / `verification_state` / `highest_risk_class` three-axis model confirmed
- [ ] `strictest_epistemic_posture` derivation matrix reviewed — provisional pending Phase 1
- [ ] **`escalate_to_human` operational meaning defined explicitly** — since VRYOH already requires human approval for everything, this value must mean something stronger than default approval (e.g. public drafting suppressed, or draft marked non-publishable) or the enum carries no real distinction *(GPT #12)*
- [ ] Invariant confirmed: `highest_risk_class: high_liability` + `governance_flag: None` = hard-fail
- [ ] **`halt_reason` field added, separate from `governance_flag`** — distinguishes a technical failure (`contract_validation_failure`) from a genuine safety signal (`high_liability_guest_signal`); same workflow action, different meaning, matters for reporting *(GPT #11)*
- [ ] `source_type` conditional provenance rules confirmed
- [ ] `relevance_confidence` scale confirmed as `high|medium|low`
- [ ] Version fields confirmed: `contract_builder_version`, `validator_version`, `ruleset_version`, `template_library_version`
- [ ] **Storage location for version fields and `decision_trace` named explicitly** — "no new NocoDB columns" and "log 4 version fields per record" were locked independently without confirming compatibility; name the actual location (existing JSON field, execution log, or other) before Phase 8 depends on it *(GPT #8)*
- [ ] `decision_trace` scope confirmed: created by builder, available during shadow tests, persisted for debugging — **not sent to RDA's generation step**, since RDA should consume the governed decision, not BRA's internal reasoning *(GPT #9)*
- [ ] **T1 failure-path logic redefined** — N=1 retry only applies to T2/T3 (a constrained LLM rebuild using the failure reason can plausibly succeed). T1 is deterministic: same input + same rules = same invalid output on retry. T1's failure path is instead: deterministic repair if the defect is mechanically fixable, otherwise escalate to the T2/T3 Claude path or human review — never a blind identical rebuild *(GPT #3 — real logic defect, not polish)*

---

## Phase 1 — Offline Golden Test Set

- [ ] **Golden set locked at 72 records, overlapping coverage minima — not additive mutually-exclusive buckets** *(GPT's model, adopted over Gemini's internally-inconsistent 63/70)*:

| Coverage requirement | Minimum |
|---|---:|
| Experiential | 10 |
| Policy/procedural | 10 |
| Factual allegation | 15 |
| — guest-report-only | 8 |
| — corroborated | 6 |
| — disputed | 6 |
| High-liability | 10 |
| Commercial-boundary | 6 |
| Mixed-epistemic / multi-claim | 8 |
| Spanish | ≥30 of 72 |

- [ ] One record may satisfy multiple coverage rows — this resolves the prior contradiction without forcing an artificial taxonomy
- [ ] False-positive test case included ("refused to sit in dirty section")
- [ ] Corroborated/disputed pairs specifically constructed — same underlying claim, once confirmed, once contested
- [ ] Expected Response Contract manually authored for every record, using the Phase 0 field manifest as the template
- [ ] Expected contracts reviewed by Miguel before use as ground truth

---

## Phase 2 — T2/T3 Shadow-Mode Contract Generation
*Shadow only — RDA doesn't consume it yet.*

- [ ] Step 9c/9f current live code read fresh
- [ ] Step 9c prompt extended per the Phase 0 field manifest — exact field count, no drift
- [ ] `claim_type` classifier added, aligned with existing `Halt` conditions
- [ ] User prompt extended with existing source fields
- [ ] Step 9f validation extended per the manifest's exact field count
- [ ] Enum validation updated for all new fields
- [ ] `max_tokens: 500` benchmarked against golden set — **explicitly bound to the existing hard limit, not just measured in isolation** *(Gemini's addition)*
- [ ] Original 7 pre-existing BRA fields checked for quality degradation
- [ ] Parse/completeness/contradiction rate measured against golden set
- [ ] **Full observability instrumentation begins here, not Phase 8** — for every shadow run, log builder version, validator version, contract output, validation result, failure class, failure reason, retry attempted, retry result, escalation reason, and Claude latency/token usage *(GPT #6 — moved earlier)*
- [ ] Confirmed shadow-only, no production impact

---

## Phase 3 — T2/T3 Validator, Shadow Mode

- [ ] Validator node built after Step 9f
- [ ] Structural failure checks implemented
- [ ] **Provenance checks added explicitly**: `direct_guest_statement` requires non-empty, valid `source_span`; `upstream_inference` requires `upstream_origin`; `business_verified_fact` requires a verification reference; an exact-quote `source_span` is checked against the actual source review text *(GPT #10 + Gemini's `source_span` omission — same fix, both flagged it)*
- [ ] `commercial_restriction_types` array explicitly schema-validated, not just implied by `must_not_introduce` *(Gemini's omission)*
- [ ] Semantic hard-failure checks implemented (high_liability+None, disputed+direct_apology)
- [ ] Semantic ambiguity path routes to narrow LLM check or human review
- [ ] N=1 retry implemented **for T2/T3 only** — failure reason passed to rebuild
- [ ] Hard-stop path confirmed: `governance_flag: Halt`, `halt_reason` set appropriately, `human_review_required: true`
- [ ] **On hard-stop, the full `failed_rules[]` array appended to the execution log alongside the Halt flag** — not just a bare flag with no detail *(Gemini's addition)*
- [ ] Pass/contradiction-catch rate measured against golden set

---

## Phase 4 — T1 Deterministic Builder + Escalation Filter
*Built against frozen Phase 0 schema.*

- [ ] Step 9a/9b current live code read fresh
- [ ] Contract-builder node inserted after Step 9b
- [ ] Fields derived from template metadata + Step 7 axes
- [ ] **Verify T1 contracts retain review-specific anchors, not just generic template-category content** — check whether `must_reflect` can carry concrete details (staff member named, specific dish, explicit complaint) from fields already available upstream; if not present today, add deterministic extraction of these from existing raw-review/upstream fields, no AI required *(GPT #5)*
- [ ] `must_not_introduce` includes prohibition set + commercial items when flagged
- [ ] **Regex pre-filter restructured as concept-family rulesets, not a flat word list** — each family (policy/procedure, factual staff action, discrimination/dignity, health/allergy, physical safety, misconduct/violence, financial/fraud, legal/regulatory, privacy) gets positive patterns, exclusions, and context requirements *(GPT #13)*
- [ ] **Pre-filter made bilingual EN/ES** — native Spanish concept terms validated against real review language, not mechanically translated (e.g. política, me negaron, no me dejaron, discriminación, racista, alergia, me cobraron dos veces, abogado, demanda, policía, agresión, robo) *(GPT #14 — real gap, English-only was a miss)*
- [ ] Pre-filter optimized toward high recall over high precision initially — a false positive only adds scrutiny to a cheap T1 record; a false negative lets a sensitive record through the fast path *(GPT #13)*
- [ ] Confirmed: filter escalates to T2/T3, does not reclassify in-place
- [ ] Term groups tested against golden set (false-positive + false-negative rate, both languages)
- [ ] **Determinism verified using inputs + all four version fields, not `hsi_record_id` alone** — same input + same ruleset version + same template-library version + same contract-builder version → identical contract; the ID alone is insufficient since the template library can change independently *(GPT #4)*
- [ ] **T1 failure path implemented per Phase 0's redefinition** — deterministic repair or escalation, not blind identical retry
- [ ] Escalation rate tracked as ongoing metric

---

## Phase 5 — T1 Validator

- [ ] Same failure-class logic from Phase 3 applied to T1 output shape, **using T1's corrected failure path (Phase 0/4), not the T2/T3 retry logic**
- [ ] Provenance checks applied to T1's deterministically-sourced fields as well
- [ ] Pass rate measured against golden set for T1-path records

---

## Phase 6 — RDA Side-by-Side Comparison
*Shadow only.*

- [ ] **Shadow contract written to a temporary execution log or audit field alongside each generated draft pair** — without this, a human reviewer has no way to confirm RDA actually followed the contract's boundaries versus coincidentally producing similar text *(Gemini's real, non-overlapping finding)*
- [ ] RDA generates two drafts per record: current behavior vs. contract-governed
- [ ] Comparison run against golden set and a live production sample (shadow only)
- [ ] **Comparison blinded** — reviewer scores drafts without knowing which is legacy vs. contract-governed; randomize ordering; reveal only after scoring, to avoid confirmation bias *(GPT #15)*
- [ ] Each draft scored independently on: factual grounding, acknowledgment adequacy, naturalness, brand fit, governance correctness
- [ ] Over-admission test: confirmed no unverified/disputed facts asserted as true
- [ ] Under-acknowledgment test: confirmed no evasiveness on obvious/corroborated failures
- [ ] Human sign-off obtained before Phase 7

---

## Phase 7 — Payload Integration Behind Feature Flag

- [ ] Step 18 current live code read fresh
- [ ] Contract fields added per the Phase 0 field manifest
- [ ] `dominant_pole` and `masked_emotion_hypothesis` added
- [ ] Feature flag `response_contract_enabled` implemented, default **false**
- [ ] **Real end-to-end integration test executed, not a chat-based confirmation** — BRA constructs a known synthetic contract with sentinel values (e.g. `primary_claim_type: factual_allegation`, `verification_state: disputed`, `highest_risk_class: elevated`, distinctive array contents), Step 18 serializes it, the actual webhook payload is captured, RDA receives and parses it, every value is compared field-by-field against the source — any missing or changed field is a test failure *(GPT #7 — direct precedent: this exact failure mode already happened once in this project)*
- [ ] **RDA's fallback behavior explicitly verified**: when `response_contract_enabled === false`, or contract fields are null, RDA's intake degrades gracefully to legacy parsing rather than failing *(Gemini's real, non-overlapping finding)*
- [ ] No NocoDB schema change (payload-only) — **consistent with the Phase 0 storage-location decision for version fields**
- [ ] Step 19 retry fix NOT bundled in

---

## Phase 8 — Controlled Rollout

- [ ] Feature flag enabled for a limited client/environment subset only
- [ ] All four version fields confirmed logging to the Phase 0-designated storage location
- [ ] Rollback tested: flag off restores exact prior behavior, no workflow surgery required
- [ ] Failure observability confirmed continuing from Phase 2's instrumentation — build failure, validator rejection, rebuild/escalation outcome, human escalation reason each logged distinctly
- [ ] **Explicit stop/rollback conditions defined before rollout, not left to subjective review** — immediate stop if: a confirmed unsupported factual admission occurs on a disputed/high-risk record; a required contract field disappears at the BRA→RDA boundary; a validator failure reaches RDA as a normal pass; `escalate_to_human` is ignored downstream; a malformed contract passes validation *(GPT #16)*
- [ ] Quantitative review triggers defined: large increase in human rejection rate; excessive false-Halt rate; unexpected spike in T1 escalation rate
- [ ] Subset reviewed against these explicit criteria before expanding
- [ ] Full rollout only after subset sign-off

---

## Open items — now resolved or reframed

- ~~Golden set per-bucket minimum count~~ → **Resolved: 72 total, overlapping coverage table above**
- ~~Regex term list completeness~~ → **Reframed: "T1 exception-detector EN/ES coverage and false-negative rate"** — an ongoing engineering target, not a one-time word-list task
- T1 cost-complexity ceiling — still a future review trigger, not a blocker. Concrete signals to revisit: rules become hard to reason about; false-negative rate plateaus despite added special cases; changes repeatedly break unrelated patterns; maintenance effort exceeds a small classifier's; a meaningful share of T1 escalates anyway

---

## Locked decisions — unchanged from v2.2

- Contract data stays payload-only to RDA — no new NocoDB columns (now with an explicit storage-location answer for version fields, per Phase 0)
- Step 19's missing retry/onError fix is a separate tracked item
- Template Library does not get a new `claim_type` column
- Multi-claim reviews use Option B (record-level worst-case fields)
- Regex/exception filter escalates to T2/T3 on match — does not reclassify in-place

---

**This is the version to build against.** 15 findings from this round incorporated (13 GPT, 2 Gemini), each traceable to its source. No further audit round scheduled — per GPT's own recommendation, the next real information should come from actually running Phase 0-1, not another conceptual pass.
