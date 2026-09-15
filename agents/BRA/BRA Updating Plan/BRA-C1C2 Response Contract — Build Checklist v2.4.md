Chat #165 · September 14, 2026

# BRA-C1/C2 Response Contract — Build Checklist v2.4

**Supersedes v2.3. Only change from v2.3: Phase 0's T1 failure-path item is replaced with the concrete, enumerable repair/escalate list from Chat #162. Everything else is unchanged — reproduced in full so this is the single document to build against, not a diff to reassemble.**

---

## Phase 0 — Schema & Ontology Freeze
*No nodes built. Spec-locking only.*

- [ ] Response Contract schema confirmed final — all fields, types, enums
- [ ] Contract Field Manifest created — single authoritative table listing every field, producer, validator, RDA-visibility
- [ ] `primary_claim_type` / `verification_state` / `highest_risk_class` three-axis model confirmed
- [ ] `strictest_epistemic_posture` derivation matrix reviewed — provisional pending Phase 1
- [ ] `escalate_to_human` operational meaning defined explicitly — must mean something stronger than default human approval
- [ ] Invariant confirmed: `highest_risk_class: high_liability` + `governance_flag: None` = hard-fail
- [ ] `halt_reason` field added, separate from `governance_flag` — distinguishes technical failure from genuine safety signal
- [ ] `source_type` conditional provenance rules confirmed
- [ ] `relevance_confidence` scale confirmed as `high|medium|low`
- [ ] Version fields confirmed: `contract_builder_version`, `validator_version`, `ruleset_version`, `template_library_version`
- [ ] Storage location for version fields and `decision_trace` named explicitly — must be compatible with "no new NocoDB columns"
- [ ] `decision_trace` scope confirmed — persisted for debugging, not sent to RDA's generation step
- [ ] **T1 failure-path logic — concrete repair/escalate list (replaces the placeholder "deterministic repair or escalate" phrase from v2.3):**

  **Mechanically repairable — one attempt, deterministic, no Claude call:**
  1. Missing/null `closing_objective` — re-pull from the selected template's default
  2. Missing/null `commercial_restriction_types` when `commercial_risk_flag` is true — re-derive from the fixed flag-to-types mapping
  3. `primary_claim_type` present but `strictest_epistemic_posture` missing — re-run the Phase 0 lookup matrix
  4. Any of `contract_builder_version` / `validator_version` / `ruleset_version` / `template_library_version` missing — re-insert the fixed build-time constant
  5. `schema_version` missing or wrong — re-insert the fixed constant

  **Never repaired — escalate to T2/T3 immediately:**
  1. `highest_risk_class: high_liability` without a matching `governance_flag: Halt` or without a template/regex trail justifying it
  2. `verification_state` is anything other than `not_applicable` or `guest_report_only` on a T1 record — T1 has no mechanism to independently verify or dispute a claim, so this state is itself evidence of an upstream inconsistency
  3. `must_reflect` array empty or missing — content-extraction failure, not a field-value failure; no rule can invent content
  4. Any defect type not on the repairable list above — unknown failure modes escalate by default, never guessed at

  **Organizing principle:** repairable = the correct value is fully determined by other fields already on the record (arithmetic). Escalate = requires judging the review's content, or resolves a genuine contradiction between two things that should agree and don't (interpretation).

  **Retry discipline:** one repair attempt only. If the repaired contract still fails validation, that is an automatic escalation — never a second repair attempt. This mirrors T2/T3's N=1 retry, but as a different mechanism (rule-based patch vs. LLM rebuild), not the same mechanism reused blindly.

---

## Phase 1 — Offline Golden Test Set

- [ ] Golden set locked at 72 records, overlapping coverage minima — not additive mutually-exclusive buckets:

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

- [ ] One record may satisfy multiple coverage rows
- [ ] False-positive test case included ("refused to sit in dirty section")
- [ ] Corroborated/disputed pairs specifically constructed
- [ ] **New, added for Phase 0's repair list:** at least 5 records deliberately constructed with each of the 5 repairable defect types, and at least 1 record per escalate-only defect type, so Phase 4/5 can test the repair/escalate split directly — not just the contract content itself
- [ ] Expected Response Contract manually authored for every record, using the Phase 0 field manifest
- [ ] Expected contracts reviewed by Miguel before use as ground truth

---

## Phase 2 — T2/T3 Shadow-Mode Contract Generation
*Shadow only.*

- [ ] Step 9c/9f current live code read fresh
- [ ] Step 9c prompt extended per the Phase 0 field manifest
- [ ] `claim_type` classifier added, aligned with existing `Halt` conditions
- [ ] User prompt extended with existing source fields
- [ ] Step 9f validation extended per the manifest's exact field count
- [ ] Enum validation updated for all new fields
- [ ] `max_tokens: 500` benchmarked against golden set, explicitly bound to the existing hard limit
- [ ] Original 7 pre-existing BRA fields checked for quality degradation
- [ ] Parse/completeness/contradiction rate measured against golden set
- [ ] Full observability instrumentation begins here: builder version, validator version, contract output, validation result, failure class, failure reason, retry/repair attempted, retry/repair result, escalation reason, Claude latency/token usage
- [ ] Confirmed shadow-only, no production impact

---

## Phase 3 — T2/T3 Validator, Shadow Mode

- [ ] Validator node built after Step 9f
- [ ] Structural failure checks implemented
- [ ] Provenance checks: `direct_guest_statement` requires valid `source_span`; `upstream_inference` requires `upstream_origin`; `business_verified_fact` requires a verification reference; exact-quote `source_span` checked against actual source text
- [ ] `commercial_restriction_types` array explicitly schema-validated
- [ ] Semantic hard-failure checks implemented (high_liability+None, disputed+direct_apology)
- [ ] Semantic ambiguity path routes to narrow LLM check or human review
- [ ] N=1 retry implemented for T2/T3 — failure reason passed to rebuild
- [ ] Hard-stop path: `governance_flag: Halt`, `halt_reason` set, `human_review_required: true`
- [ ] On hard-stop, full `failed_rules[]` array appended to the execution log alongside the Halt flag
- [ ] Pass/contradiction-catch rate measured against golden set

---

## Phase 4 — T1 Deterministic Builder + Escalation Filter
*Built against frozen Phase 0 schema.*

- [ ] Step 9a/9b current live code read fresh
- [ ] Contract-builder node inserted after Step 9b
- [ ] Fields derived from template metadata + Step 7 axes
- [ ] Verify T1 contracts retain review-specific anchors — check whether `must_reflect` carries concrete review details; add deterministic extraction from existing upstream fields if not
- [ ] `must_not_introduce` includes prohibition set + commercial items when flagged
- [ ] Regex pre-filter restructured as concept-family rulesets (policy/procedure, factual staff action, discrimination/dignity, health/allergy, physical safety, misconduct/violence, financial/fraud, legal/regulatory, privacy), each with positive patterns, exclusions, context requirements
- [ ] Pre-filter made bilingual EN/ES — native Spanish terms validated against real review language
- [ ] Pre-filter optimized toward high recall over precision
- [ ] Confirmed: filter escalates to T2/T3, does not reclassify in-place
- [ ] Term groups tested against golden set (false-positive + false-negative, both languages)
- [ ] **T1's repair/escalate list from Phase 0 implemented exactly as specified** — the 5 repairable cases wired to their deterministic fixes, the 4 escalate cases wired to route to T2/T3, unknown defects default to escalate
- [ ] One-repair-attempt-only discipline implemented — no second repair try under any condition
- [ ] Determinism verified using inputs + all four version fields, not `hsi_record_id` alone
- [ ] Escalation rate tracked as ongoing metric

---

## Phase 5 — T1 Validator

- [ ] Same failure-class logic from Phase 3 applied to T1 output shape, **using T1's repair/escalate list, never the T2/T3 retry mechanism**
- [ ] All 5 repairable defect types tested against their golden-set examples — confirm each repairs correctly
- [ ] All 4 escalate-only defect types tested against their golden-set examples — confirm each escalates, none get incorrectly repaired
- [ ] Provenance checks applied to T1's deterministically-sourced fields
- [ ] Pass rate measured against golden set for T1-path records

---

## Phase 6 — RDA Side-by-Side Comparison
*Shadow only.*

- [ ] Shadow contract written to a temporary execution log or audit field alongside each generated draft pair
- [ ] RDA generates two drafts per record: current behavior vs. contract-governed
- [ ] Comparison run against golden set and a live production sample (shadow only)
- [ ] Comparison blinded — reviewer scores without knowing which draft is which; reveal only after scoring
- [ ] Each draft scored independently: factual grounding, acknowledgment adequacy, naturalness, brand fit, governance correctness
- [ ] Over-admission test: no unverified/disputed facts asserted as true
- [ ] Under-acknowledgment test: no evasiveness on obvious/corroborated failures
- [ ] Human sign-off obtained before Phase 7

---

## Phase 7 — Payload Integration Behind Feature Flag

- [ ] Step 18 current live code read fresh
- [ ] Contract fields added per the Phase 0 field manifest
- [ ] `dominant_pole` and `masked_emotion_hypothesis` added
- [ ] Feature flag `response_contract_enabled` implemented, default **false**
- [ ] Real end-to-end integration test executed with a sentinel synthetic contract — every value compared field-by-field, any missing/changed field is a test failure
- [ ] RDA's fallback behavior explicitly verified: `response_contract_enabled === false` or null fields → graceful legacy parsing, not failure
- [ ] No NocoDB schema change — consistent with the Phase 0 storage-location decision
- [ ] Step 19 retry fix NOT bundled in

---

## Phase 8 — Controlled Rollout

- [ ] Feature flag enabled for a limited subset only
- [ ] All four version fields confirmed logging to the Phase 0-designated storage location
- [ ] Rollback tested: flag off restores exact prior behavior
- [ ] Failure observability continuing from Phase 2's instrumentation
- [ ] Explicit stop/rollback conditions defined before rollout: unsupported factual admission on a disputed/high-risk record; required contract field disappears at the BRA→RDA boundary; a validator failure reaches RDA as a normal pass; `escalate_to_human` ignored downstream; malformed contract passes validation
- [ ] Quantitative review triggers defined: large increase in human rejection rate; excessive false-Halt rate; unexpected T1 escalation-rate spike
- [ ] Subset reviewed against these explicit criteria before expanding
- [ ] Full rollout only after subset sign-off

---

## Open items

- T1 cost-complexity ceiling — future review trigger, not a blocker. Revisit if: rules become hard to reason about; false-negative rate plateaus; changes repeatedly break unrelated patterns; maintenance effort exceeds a small classifier's; a meaningful share of T1 escalates anyway

## Locked decisions — unchanged

- Payload-only to RDA, no new NocoDB columns
- Step 19 fix stays a separate tracked item
- No new Template Library `claim_type` column
- Multi-claim reviews: Option B (record-level worst-case fields)
- Regex/exception filter escalates, does not reclassify in-place

---

**Status: this is the version to build against.** Rather than call this "final" again, the honest statement is: this is settled at the level of detail it's been examined at so far — Phase 0 through Phase 8, including the T1 repair mechanism down to individual defect types. If a later question surfaces a level of detail nobody's asked about yet (the way Chat #162's question did), that becomes v2.5, named as an addition, not hidden inside a silent revision.
