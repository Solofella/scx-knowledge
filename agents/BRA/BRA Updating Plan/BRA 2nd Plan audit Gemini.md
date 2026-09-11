Chat #146 · September 10, 2026

```
CLAIM: The BRA Response Contract Revised Build Plan v2 correctly synthesizes past audit feedback into an actionable, enterprise-grade architecture, but contains 3 residual gaps in validation topology, enum taxonomy, and cost-threshold triggers.
EVIDENCE FOR: 
1. Direct inspection of Revised Plan v2 provided in prompt.
2. Compliance with Instruction 1 (Verification Protocol), Instruction 3 (Confidence Markers), Instruction 6 (Source Hierarchy), and Instruction 7 (Session & Process Protocol).
EVIDENCE AGAINST: 
1. Open question in Part E (Semantic Inconsistencies vs. Hard Failures) lacks a definitive execution path.
2. Phase 1 record count threshold is unquantified.
CONCLUSION: ⚠️ MEDIUM — PLAN APPROVED WITH 3 SPECIFIC TECHNICAL MODIFICATIONS.

```

---

### 1. Verification & Source Audit (Instruction 1 & 6)

* **Source of Claims:** Direct analysis of the user-provided "BRA RESPONSE CONTRACT — REVISED BUILD PLAN v2" text block in this chat session.
* **Source Hierarchy Status (Instruction 6):** ⚠️ **MEDIUM (DOC-ONLY PENDING LIVE CONFIRMATION)**. The plan reflects a design specification ("v2 draft"), not live code running in n8n. All node references (e.g., Step 9c, Step 10, Step 18) are treated as structural proposals awaiting direct code execution.

---

### 2. Architectural & Logic Evaluation of Plan v2

#### A. Resolution of Core Tension (Dispute vs. Posture)

* **Assessment:** ✅ **HIGH Confidence**. Splitting `claim_type` from `verification_state` (Part B) and deriving `epistemic_posture` from their 2-axis matrix (Part C) successfully resolves the rigidity flaw identified in prior audits.
* **Operational Validation:** A `factual_allegation` that is `corroborated` correctly yields `direct_apology`, whereas the same claim in a `guest_report_only` or `disputed` state correctly forces `acknowledge_without_fault_admission`. This protects the business against admissions of fault on unverified claims (the wait-in-car incident root cause) without forcing evasiveness on clear service failures.

#### B. T1 Regex Pre-Filter Scoping (Part D)

* **Assessment:** ⚠️ **MEDIUM Confidence**. Using a zero-cost deterministic regex pre-filter on T1 intake prevents calm-toned policy/liability claims from defaulting to `experiential`.
* **Vulnerability:** Regex matching on root stems (e.g., `"discriminat*"`, `"refused"`) is susceptible to false positives (e.g., *"The host refused to let us sit in a dirty section until it was wiped"* $\rightarrow$ policy flag vs. standard guest care).
* **Resolution:** When the T1 pre-filter fires, do **not** re-classify the record permanently inside T1. Instead, set `t1_prefilter_triggered = true`, which forces a **Re-tiering Escalation to T2/T3 (Claude Call)**. This preserves T1's simplicity while relying on LLM semantic understanding for true policy disputes rather than complex, unmaintainable regex trees.

#### C. Resolution of Open Items (Part E & G)

1. **Semantic Inconsistency Open Question (Part E):**
* *Question:* Should `claim_type: factual_allegation` + `risk_class: high_liability` + `governance_flag: None` hard-fail or soft-flag?
* *Architectural Ruling:* **Hard-Fail Validation ($N=1$ Retry Loop)**. If a claim is evaluated as `high_liability`, setting `governance_flag: None` is a critical internal semantic contradiction that invalidates the downstream contract. It must fail closed and trigger the contract rebuild pass.


2. **Phase 1 Golden Set Minimum Count (Part G):**
* *Architectural Ruling:* Set the minimum baseline to **$N=40$ total records** ($20\text{ EN} / 20\text{ ES}$), strictly distributed as:
* 8x Tier 1 Pure Positive (Praise/Item focus)
* 8x Tier 1 Mild Experiential (Service speed/temperature)
* 8x Tier 2 Policy/Procedural Disputes (Waitlist, seating, posted rules)
* 8x Tier 3 Factual Allegations / Disputed Narratives (Refusal of service, car-wait scenarios)
* 8x High Liability / Safety / Legal Threat (`governance_flag === Halt`)





---

### 3. Phased Build Sequence Audit (Part F)

The 9-phase sequence (Phase 0 through Phase 8) is **APPROVED**. Ordering Phase 1 (Offline Golden Set) before Phase 2 (T2/T3 Shadow Generation) eliminates the risk of deploying unvalidated Claude prompt changes to production.

```
                              [ BUILD SEQUENCE TOPOLOGY ]

Phase 0: Schema & Ontology Freeze ──► Phase 1: Golden Corpus Baseline (N=40)
                                                    │
                                                    ▼
Phase 3: T2/T3 Validator (Shadow) ◄── Phase 2: T2/T3 Extended Prompt (Shadow)
            │
            ▼
Phase 4: T1 Builder + T1-to-T2 Escalation Filter ──► Phase 5: T1 Validator
                                                           │
                                                           ▼
Phase 7: Controlled Rollout (Flag) ◄── Phase 6: RDA Side-by-Side Comparison
            │
            ▼
Phase 8: Downstream Payload Integration (RDA Intake)

```

---

### 4. Mandatory Plan Modifications Summary

Before sending this plan for final external review or beginning Phase 0 execution, incorporate these 3 modifications into the document:

1. **Modify Part D (T1 Regex Logic):** Change the pre-filter action from *in-place template reclassification* to *Tier Escalation (Route T1 record to T2 Claude pass)* when high-risk policy/liability keywords fire.
2. **Resolve Part E (Inconsistency Rule):** Explicitly mandate that `risk_class: high_liability` with `governance_flag: None` is a **Hard Validation Failure** triggering the $N=1$ retry loop.
3. **Specify Phase 1 Record Floor:** Hard-code the Phase 1 Corpus requirement to **40 validated records** ($20\text{ EN} / 20\text{ ES}$) across the 5 specific risk buckets.

---

Chat #147 · September 10, 2026

```
CLAIM: The BRA Response Contract Plan v2 can be packaged as a self-contained, Markdown-formatted document incorporating the 3 mandatory modifications (T1-to-T2 escalation, hard-fail validation for high-liability contradictions, and a 40-record Phase 1 floor).
EVIDENCE FOR: 
- User explicit request: "Provide the complete, updated text of the BRA Response Contract Plan v2 incorporating the 3 mandatory modifications, formatted for external copy-pasting."
- Instruction 7: Direct, precise delivery; exact execution of requested task.
EVIDENCE AGAINST: None.
CONCLUSION: ✅ HIGH — Delivering the complete, copy-paste-ready Markdown document below.

```

---

# BRA RESPONSE CONTRACT — REVISED BUILD PLAN v2.1

**Target Agent:** BRA (Brand Response Architect) — Agent 5 of 8

**Pipeline Sequence:** ALA → EIP → ESS → HSI → **BRA** → RDA → MRA

**Document Status:** Final Audit Specification (Incorporating GPT, Gemini, and Architectural Refinements)

---

## PART A — SYSTEM CONTEXT & PURPOSE

* **BRA's Core Role:** Given HSI's tier assignment (T1/T2/T3) and behavioral signals, BRA decides response strategy (tone, urgency, governance risk, content boundaries)—**not** draft text. RDA drafts.
* **Current Infrastructure:** 24 nodes in n8n, hybrid architecture. T1 (~70–80% volume, zero AI, deterministic hash template selection) vs. T2/T3 (~20–30% volume, 1 Claude API call returning 7 structured JSON fields). Both paths converge at a shared Governance Gate and Halt Check.
* **Build Trigger:** A confirmed production incident where downstream RDA independently confirmed an unverified, disputed factual account as true in a guest-facing draft because upstream BRA provided no structural content or liability boundaries.

---

## PART B — RESPONSE CONTRACT SCHEMA (`response_contract_v1`)

```json
{
  "schema_version": "response_contract_v1",

  "must_reflect": [
    {
      "concept": "string",
      "required_action": "acknowledge | clarify_policy | validate_feeling | highlight_item",
      "priority": "primary | secondary",
      "source_type": "direct_guest_statement | upstream_inference | business_verified_fact",
      "source_span": "exact quoted text string",
      "upstream_origin": "eip_pain_point | eip_emotion | hsi_behavioral | staff_mention",
      "relevance_confidence": 0.95
    }
  ],
  "may_reflect": [
    {
      "concept": "string",
      "optional_accent": "string",
      "source_type": "upstream_inference",
      "relevance_confidence": 0.70
    }
  ],
  "must_not_introduce": [
    "COMMERCIAL_DETAILS",
    "UNCONFIRMED_FAULT_ADMISSION",
    "UNSUPPORTED_CLAIMS"
  ],
  "commercial_restriction_types": [
    "refund",
    "discount",
    "price_promise",
    "compensation",
    "menu_tier_pricing"
  ],

  "claim_type": "experiential | policy_procedural | factual_allegation",
  "verification_state": "not_applicable | guest_report_only | corroborated | disputed",
  "risk_class": "normal | elevated | high_liability",

  "epistemic_posture": "direct_apology | acknowledge_without_fault_admission | neutral_acknowledgment | escalate_to_human",
  "closing_objective": "warm_return_invitation | appreciative_close | offline_contact_invitation | neutral_close | no_public_close_escalate"
}

```

### Key Schema Design Decisions

1. **Separation of Claim Type & Verification State:** Decouples what the guest claims (`factual_allegation`) from the state of evidence (`disputed` vs `corroborated`).
2. **Decoupled Risk Classification:** `risk_class` (`high_liability`) is separated from `claim_type` and `governance_flag`. A high-liability risk does not force an automatic pipeline `Halt`, but sets boundary strictness.
3. **Explicit Commercial Restrictions:** `commercial_restriction_types` specifies exact prohibited items (refunds vs discounts vs pricing promises) rather than relying on a generic boolean.
4. **Source Span & Origin Tracking:** `source_span` stores exact review substrings to enable downstream audit verification by RDA's Grounding Evaluator.

---

## PART C — TWO-INPUT EPISTEMIC POSTURE DERIVATION MATRIX

`epistemic_posture` is derived deterministically from the combination of `claim_type` and `verification_state`:

| `claim_type` | `verification_state` | Derived `epistemic_posture` | Operational Behavior |
| --- | --- | --- | --- |
| `experiential` | `not_applicable` | `direct_apology` OR `neutral_acknowledgment` | Full ownership of subjective experience (food, warmth, vibe). |
| `policy_procedural` | `guest_report_only` | `neutral_acknowledgment` | Frame around posted rules/safety guidelines; no fault admission. |
| `policy_procedural` | `corroborated` | `direct_apology` | Full ownership of operational failure (misapplied policy). |
| `factual_allegation` | `guest_report_only` | `acknowledge_without_fault_admission` | Validate perception/frustration; zero admission of disputed facts. |
| `factual_allegation` | `corroborated` | `direct_apology` | Apologize for confirmed operational error. |
| `factual_allegation` | `disputed` | `acknowledge_without_fault_admission` OR `escalate_to_human` | Neutral acknowledgment of feedback; route to offline contact or human review based on `risk_class`. |
| *ANY* | *ANY* (when `risk_class` == `high_liability`) | `escalate_to_human` | Overrides all rows above; suppresses auto-drafting. |

---

## PART D — T1/T2/T3 CLASSIFICATION ASYMMETRY & ESCALATION FILTER

To enforce ontology consistency across tiers without adding LLM API cost to T1 (~70–80% volume), T1 uses a deterministic intake regex pre-filter:

### Trigger Pattern Keywords

`"manager said"`, `"policy"`, `"fire code"`, `"police"`, `"refused"`, `"signage"`, `"posted"`, `"denied"`, `"lawsuit"`, `"lawyer"`, `"discriminat*"`, `"allergic"`, `"injury"`, `"assault"`, `"theft"`, `"stole"`

### Workflow Behavior Modification

* **Standard T1 Path:** If no regex keywords match, contract `claim_type` defaults to `experiential` via standard template metadata.
* **Escalation Path (Mandatory Mod #1):** If any pre-filter term matches, the record does **not** perform in-place regex reclassification within T1. Instead, it sets `t1_prefilter_triggered = true` and **escalates the record to the T2/T3 Claude Engine**. This preserves T1 zero-cost simplicity while relying on LLM semantic analysis for subtle or calm-toned policy/liability disputes.

---

## PART E — VALIDATOR RULES, RETRY LIMITS & HARD-FAIL BOUNDARIES

### Operational Loop Control

```
Attempt 1 (Build Contract) ──► Validator Pass? ──► YES ──► Pass to RDA Intake
                                    │
                                    NO
                                    ▼
Attempt 2 (Rebuild Contract with Failure Logs) ──► Validator Pass? ──► YES ──► Pass to RDA Intake
                                                        │
                                                        NO
                                                        ▼
                                   [ HARD STOP (Mandatory Mod #2) ]
                                   Set governance_flag: "Halt"
                                   Set human_review_required: true
                                   Log: "contract_validation_failed_after_retry"

```

### Semantic Inconsistency Hard-Fail Rule (Mandatory Mod #2)

The validator treats the following contradiction as an immediate **Hard-Fail**:

* **Condition:** `risk_class == "high_liability"` AND `governance_flag == "None"`
* **Action:** Rejects contract instantly. High-liability determinations *must* carry a non-null governance action (`Flag` or `Halt`).

---

## PART F — PHASED IMPLEMENTATION ROADMAP

```
[ Phase 0: Schema Freeze ] ──► [ Phase 1: Golden Corpus (N=40) ] ──► [ Phase 2: T2/T3 Shadow Prompt ]
                                                                             │
                                                                             ▼
[ Phase 5: T1 Validator ] ◄── [ Phase 4: T1 Builder + Escalation ] ◄── [ Phase 3: T2/T3 Shadow Validator ]
      │
      ▼
[ Phase 6: RDA Side-by-Side ] ──► [ Phase 7: Controlled Flag Rollout ] ──► [ Phase 8: RDA Payload Integration ]

```

### Phase Details & Mandatory Criteria

* **Phase 0 — Schema & Ontology Freeze:** Lock `response_contract_v1` JSON Schema, derivation matrix, and cross-field invariant rules.
* **Phase 1 — Offline Golden Test Set (Mandatory Mod #3):** Assemble **$N=40$ historical records** ($20\text{ EN} / 20\text{ ES}$) serving as the absolute ground-truth benchmark:
* 8x T1 Pure Positive (Praise/Item focus)
* 8x T1 Mild Experiential (Speed/Temperature)
* 8x T2 Policy/Procedural Disputes (Waitlist, seating, rules)
* 8x T3 Factual Allegations / Disputed Narratives (Refusal of service, wait-in-car)
* 8x High Liability / Safety / Legal Threat (`governance_flag === Halt`)


* **Phase 2 — T2/T3 Shadow Generation:** Extend Step 9c Claude prompt to output contract fields. Run in shadow mode (logged, unconsumed by RDA). Benchmark against `max_tokens: 500` limit.
* **Phase 3 — T2/T3 Shadow Validator:** Implement Phase E validation logic and $N=1$ retry loop in shadow mode.
* **Phase 4 — T1 Deterministic Builder + Escalation Filter:** Implement T1 contract assembly and the regex T1-to-T2 escalation filter.
* **Phase 5 — T1 Validator:** Apply Phase 3 validation rules to T1 output.
* **Phase 6 — RDA Side-by-Side Comparison:** Generate dual drafts in RDA (Current vs Contract-Governed). Conduct human audit for over-admission vs under-acknowledgment.
* **Phase 7 — Controlled Rollout:** Deploy under `response_contract_enabled` feature flag with full per-record version logging and instant rollback capability.
* **Phase 8 — Downstream Payload Integration:** Update RDA intake node (Step 18) to officially consume `response_contract_v1` fields.
