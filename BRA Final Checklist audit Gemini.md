Chat #158 · September 14, 2026

```
CLAIM: The Phase 0–8 Implementation Checklist and Workflow Diagram match locked v2.2 specifications, but contain 2 checklist omissions, 1 phase dependency risk, and 2 unquantified parameters requiring explicit values before Phase 0 execution.
EVIDENCE FOR: Direct inspection of user-provided prompt text, checklist items, and architecture diagram descriptions.
EVIDENCE AGAINST: Phase 0 checklist omits source_span validation rules and commercial_restriction_types arrays; Phase 6 lacks an explicit shadow logging node.
CONCLUSION: ⚠️ MEDIUM — Technical Audit Complete. Findings and concrete parameter resolutions detailed below.

```

---

### 1. Checklist Content Audit (Inventions & Omissions)

#### A. Omitted Items (Present in v2.2 Plan, Dropped from Checklist)

1. **`source_span` Verification Rule (Phase 0/3):** The v2.2 schema mandates an exact review substring (`source_span`) for items sourced via `direct_guest_statement`. Phase 0 omits a explicit check verifying `source_span` is non-empty when `source_type === "direct_guest_statement"`.
2. **`commercial_restriction_types` Array Validation (Phase 0):** Phase 0 mentions `must_not_introduce` generally, but omits explicit schema validation for the `commercial_restriction_types` array (`refund`, `discount`, `price_promise`, `compensation`, `menu_tier_pricing`).

#### B. Introduced Items / Discrepancies

* **Phase 3 Latency Benchmark Item:** The checklist item for Phase 3 mentions measuring token output, but does not explicitly bind it to the `max_tokens: 500` hard limit established in earlier prompt optimization passes.

---

### 2. Phase Ordering & Dependency Audit

```
                          [ PHASE DEPENDENCY TOPOLOGY ]

Phase 0: Schema Freeze ──► Phase 1: Golden Corpus (N=70) ──► Phase 2: T2/T3 Shadow Prompt
                                                                      │
                                                                      ▼
Phase 5: T1 Validator ◄── Phase 4: T1 Builder + Escalation ◄── Phase 3: T2/T3 Shadow Validator
      │
      ▼
Phase 6: Side-by-Side Verification ──► Phase 7: Payload Integration ──► Phase 8: Feature Flag Rollout

```

* **Phase 7 vs Phase 8 Sequence:** ✅ **CORRECT**. Phase 7 (Payload Integration at Step 18) correctly precedes Phase 8 (Feature Flag Rollout). If the feature flag were toggled before RDA intake (Step 18) was updated to parse the 7 contract fields, RDA would drop the contract payload entirely.
* **Ordering Dependency Gap:** In **Phase 6 (RDA Side-by-Side Comparison)**, the checklist requires dual drafts to be generated, but lacks a checklist item to write shadow contracts to a temporary execution log or NocoDB audit field. Without logging the shadow contract alongside the draft, human auditors in Phase 6 cannot evaluate whether RDA accurately followed the contract boundaries.

---

### 3. Resolution of Open Items

#### Item A: Concrete Golden Set Per-Bucket Distribution ($N=70$ Total Floor)

To meet the 60–80 total target ($35\text{ EN} / 35\text{ ES}$):

| Bucket ID | Target Category | EN Count | ES Count | Total | Target Risk Scenario |
| --- | --- | --- | --- | --- | --- |
| **B1** | Pure Experiential / Praise | 5 | 5 | 10 | T1 item focus, warmth, speed. |
| **B2** | Policy / Procedural Dispute | 5 | 5 | 10 | Seating rules, waitlist, capacity, posted signs. |
| **B3** | Factual Allegation (Single) | 8 | 7 | 15 | Wait-in-car, seating refusal, staff arguments. |
| **B4** | Corroborated / Disputed Pairs | 5 | 5 | 10 | Same factual claim: 5 verified true vs 5 disputed by staff. |
| **B5** | High Liability / Safety | 5 | 5 | 10 | Legal threats, discrimination, food safety, injuries. |
| **B6** | Commercial Boundary | 4 | 4 | 8 | Explicit requests for refunds, discounts, free items. |
| **TOTAL** |  | **32** | **31** | **63** | *(Exceeds 60 floor; leaves buffer up to 80).* |

#### Item B: Concrete T1 Regex Keyword Pre-Filter List

Expand the T1 escalation pre-filter keyword array (running pre-contract to trigger T1-to-T2 escalation) to the following 24 explicit string patterns:

```javascript
const T1_ESCALATION_REGEX = /manager\s+said|policy|fire\s+code|police|refused|signage|posted|denied|lawsuit|lawyer|attorney|discriminat\w*|allergic|allergy|injury|assault|theft|stole|stolen|unlawful|illegal|infestation|food\s+poisoning|safety/i;

```

---

### 4. Missing Step & Safeguard Audit

1. **Explicit RDA Schema Fallback (Phase 7):** Add a checklist item to Phase 7 verifying RDA's intake node defaults gracefully to legacy parsing when `response_contract_enabled === false` or if the payload fields are `null`.
2. **Validator Error Log Format (Phase 3/5):** Add a checklist item to Phase 3/5 specifying that when $N=1$ retry fails, the detailed failure array (`failed_rules[]`) must be appended to the NocoDB execution log string alongside `governance_flag: Halt`.

---
