HANDOFF PROMPT FOR THE BRA CHAT WINDOW — RESPONSE CONTRACT BUILDER + CONTRACT VALIDATOR (REVISED — integrated with real BRA architecture)

**Paste this at the start of the BRA session.**

---

## 1. WHO YOU ARE AND WHY THIS MATTERS

You are working on BRA — **Brand Response Architect** (confirmed live system-prompt designation; "Brand Response Agent" is a stale naming artifact from an older doc, unresolved separately, not part of this work). BRA is the fifth agent in `ALA → EIP → ESS → HSI → BRA → RDA → MRA`. It is about to take on a genuinely new, more important responsibility, as the direct result of an extensive RDA quality-improvement effort. Read this fully — the history and reasoning matter as much as the checklist.

---

## 2. HOW WE GOT HERE — FULL HISTORY

RDA (Response Drafting Agent) generates the guest-facing response and an internal brief, in English or Spanish, routed through human approval before anything publishes. During an extended hardening effort, a real production failure surfaced: a 1-star Spanish review described a guest denied bar seating and told to wait in his car. RDA's draft issued an unqualified admission of fault — but a staff account subsequently disputed the guest's entire account: the policy is fire-code-driven, posted on-site, and the car-wait suggestion was a heat accommodation, not a dismissal. **RDA had confirmed a disputed, one-sided account as fact — a real governance risk, not just a wording problem.**

Two independent external LLM audits (GPT and Gemini), run separately on the full RDA architecture, both identified this same gap as the single most serious open issue, and both concluded the fix belongs **upstream of RDA** — deciding how much fault to admit is a strategy decision, not a wording decision.

Separately, a quality definition was clarified directly by the system owner: a response is high-quality only if it simultaneously satisfies three criteria — natural writing in both languages, the client's brand voice 100% recognizable and distinguishable, and logical coherence with upstream data, 100% relevant, nothing invented, nothing important omitted. Two further rounds of external audit (GPT/Gemini, each run twice) produced a detailed architecture for achieving this, centered on a "Response Contract" — a specification of what a response must, may, and must never say, built before RDA ever writes anything.

**The final question — does this contract-building logic belong in RDA, in a brand-new agent, or in BRA — was answered three separate ways** (the system owner's own reasoning, and two independent external audits, none seeing the others' answer) and all three converged on the identical conclusion: **BRA**, because "decide response strategy" is already BRA's job description. No new agent should be created.

**This handoff was drafted once already, then revised after BRA's real, current architecture (SCX_BRA_HOW, live-verified) was reviewed in full.** That review surfaced real integration requirements the first draft didn't account for — covered in the next section.

---

## 3. CRITICAL — HOW THIS INTEGRATES WITH BRA'S REAL, EXISTING ARCHITECTURE

**Do not treat this as new logic bolted onto BRA. It must integrate with what BRA already does, reusing existing fields and existing branching wherever they already do the same job.**

**BRA is hybrid, and the new work must respect that split:**
- **T1 (~70-80% of records):** deterministic template selection, **zero Claude calls.** The Response Contract for T1 records must be built **deterministically too** — derived directly from the already-selected template's fields (`Template Name`, `Signal Category`, `Signal-Based Consideration`) plus the existing pre-derived axes (Step 7's `stability_axis`, `pain_axis`, `urgency_level_default`, `commercial_risk_preflag`). **Do not add a Claude call to the T1 path.** This preserves BRA's core cost-control/auditability design principle — the same `hsi_record_id` should still deterministically produce the same contract.
- **T2/T3 (~20-30% of records):** already makes exactly one Claude call (Step 9c-9f, 7 JSON fields, `max_tokens: 500`, `temperature: 0.3`). **The new Response Contract fields should be added as additional output fields in this same existing call — not a second, separate Claude call.** Extend the system/user prompt and the required-fields list; do not create a parallel prompt.

**Reuse existing governance fields — do not create duplicates that do the same job:**
- `governance_flag` (`None`/`Flag`/`Halt`) is BRA's real, working Halt mechanism. **The new `claim_type: high_liability` category should map directly onto `governance_flag === "Halt"`, not be a separate, newly-invented classification running in parallel.**
- `commercial_risk_flag` already exists and already feeds the governance gate at Step 10. **The new `must_not_introduce` restriction on financial commitments should be derived from/aligned with this existing flag, not duplicated as unrelated new logic.**
- `signal_based_consideration` (internal-only, never client-visible) already carries strategic reasoning close to what the new `epistemic_posture` field is meant to capture. **These two should be designed together — `epistemic_posture` becomes a new, more specific field alongside `signal_based_consideration`, informed by it, not a redundant restatement of it.**
- `tone_style`, `urgency_level`, `behavioral_interpretation` already exist and already travel to RDA — the new contract should incorporate these as-is, not re-derive them under new names.

**A live, known bug must be fixed as part of this same work, not deferred separately:** Step 9b's T1 governance override checks `pain_point_sub_category.includes("Trust, Dignity")` — the wrong field. The correct field is `pain_point_domain_confirmed`, which actually contains the canonical domain name `"Trust, Dignity & Belonging"`. This bug is confirmed **live and consequential** — Step 9b's output reaches Step 10, so most T1 records with genuine dignity/trust risk currently pass through with `governance_flag: "None"` uncaught. **This is directly, materially relevant to the new claim-type classification work — a `factual_allegation_disputed` or `high_liability` T1 record is exactly the kind of record this bug is currently letting through incorrectly flagged.** Fix this bug as part of building BRA-C1, not as a separate, later task — the new classification logic would otherwise inherit the same coverage gap it's meant to close.

**Extend the existing 28-field RDA payload (Step 18) — do not replace it.** Add the new contract fields (`must_reflect`, `may_reflect`, `must_not_introduce`, `epistemic_posture`, `claim_type`, `closing_objective`, `schema_version`) to the existing payload build, alongside the 28 fields already sent. Note two fields BRA already computes internally but currently does **not** send to RDA — `dominant_pole` and `masked_emotion_hypothesis` — both are directly relevant to the new claim-type classification and should likely be added to the RDA payload now, since they weren't needed before but are now.

---

## 4. THE RESPONSE CONTRACT — REVISED SPECIFICATION (integrated, not standalone)

```json
{
  "schema_version": "response_contract_v1",
  "review_id": "ala_record_id",
  "client_id": "client_id",
  "language": "lang",
  "tier": "confirmed_response_tier",

  "must_reflect": [
    {
      "concept": "...",
      "required_action": "acknowledge | ...",
      "priority": "primary | secondary",
      "source": "raw_review | EIP | ESS | HSI",
      "source_span": "exact quoted text",
      "upstream_origin": "which field produced this",
      "confidence": 0.0
    }
  ],
  "may_reflect": [ "...same shape, optional..." ],
  "must_not_introduce": [
    "derived from commercial_risk_flag / commitment_restrictions",
    "..."
  ],

  "epistemic_posture": "direct_apology | acknowledge_without_fault_admission | neutral_acknowledgment | escalate_to_human",
  "claim_type": "experiential | policy_procedural | factual_allegation_disputed | high_liability",

  "closing_objective": "...",

  "governance_flag": "None | Flag | Halt",
  "commercial_risk_flag": true,
  "tone_style": "existing enum, unchanged",
  "urgency_level": "existing enum, unchanged",
  "signal_based_consideration": "existing field, unchanged",
  "behavioral_interpretation": "existing field, unchanged (null for T1)"
}
```

**`claim_type: high_liability` is set whenever `governance_flag === "Halt"` — direct mapping, not independent logic.** For T1, `claim_type` defaults based on the selected template's `Signal Category` and whether `commercial_risk_preflag` or the (now-fixed) Step 9b override fired. For T2/T3, `claim_type` is one of the 7 fields Claude now produces in the existing call.

---

## 5. FULL CHECKLIST — BRA-C1: RESPONSE CONTRACT BUILDER

**T1 path (deterministic, no new Claude call):**
- [ ] Insert BRA-C1 (T1) immediately after Step 9a (T1 Template Selection) and Step 9b (Governance Screen, bug now fixed — see below).
- [ ] Derive `must_reflect`/`may_reflect` deterministically from the selected template's fields plus Step 7's pre-derived axes — no invented Claude reasoning for T1.
- [ ] Set `must_not_introduce` from the existing `commercial_risk_flag`/`commercial_risk_preflag` logic already computed at Steps 7/9a/9b.
- [ ] Set `claim_type` from `governance_flag` (Halt → `high_liability`; Flag + dignity/trust template category → `factual_allegation_disputed`; otherwise → `experiential` or `policy_procedural` based on template's `Signal Category`).
- [ ] Set `epistemic_posture` using the default mapping table (Section 6) based on the resulting `claim_type`.
- [ ] Confirm the same `hsi_record_id` always produces the same contract — preserving BRA's core determinism/auditability principle for T1.

**T2/T3 path (extend the existing Claude call, Steps 9c-9f):**
- [ ] Extend Step 9c's system prompt to include the new fields as part of the same required-JSON-output instruction — do not create a second prompt or a second call.
- [ ] Add the new fields to the required-fields list already enforced at Step 9f (currently 7 fields; becomes more): `must_reflect`, `may_reflect`, `must_not_introduce`, `epistemic_posture`, `claim_type`, `closing_objective`.
- [ ] Implement the 4-category `claim_type` classifier inside this same prompt: `experiential`, `policy_procedural`, `factual_allegation_disputed`, `high_liability` — with `high_liability` required to align with `governance_flag === "Halt"` (same enum, same trigger conditions already defined at Step 9c for Halt).
- [ ] Keep `claim_type` and `epistemic_posture` as separate fields, not coupled — an experiential complaint does not automatically require an unqualified apology.
- [ ] Populate `must_reflect` with `required_action`, `priority`, `source`, `source_span`, `upstream_origin`, `confidence` — extending Step 9c's existing user-prompt inputs (`signal_synthesis_summary`, `contextual_linguistic_framing`, `masked_emotion_hypothesis`) as the source material.
- [ ] Set `must_not_introduce` to always include content already prohibited in Step 9c's existing prohibition list (refund amounts, vouchers, discounts, unauthorized promises, operational prescriptions) — extend, don't replace.

**Shared, both paths:**
- [ ] Version the schema (`response_contract_v1`); document what triggers a version bump.
- [ ] Add `must_reflect`/`may_reflect`/`must_not_introduce`/`epistemic_posture`/`claim_type`/`closing_objective`/`schema_version` to the existing Step 18 RDA payload build, alongside the current 28 fields.
- [ ] Add `dominant_pole` and `masked_emotion_hypothesis` to the same payload — currently computed but not sent, now relevant to the new classification work.

---

## 6. FULL CHECKLIST — BRA-C2: CONTRACT VALIDATOR (revised — primarily deterministic, not a third Claude call)

**Design principle, consistent with BRA's own cost philosophy:** since T1 already runs at zero AI cost and T2/T3 runs at exactly one call, **BRA-C2 should be primarily deterministic/rule-based for both tiers**, reserving any LLM-based check only for genuinely ambiguous T2/T3 cases — not a blanket third Claude call added to every record.

- [ ] Implement JSON Schema structural validation first (required fields present, correct types, `must_reflect` array well-formed) — cheap, always runs, both tiers.
- [ ] Implement deterministic semantic checks: does `claim_type: high_liability` actually correspond to `governance_flag === "Halt"` (catches a contradiction directly, no LLM needed); does `must_not_introduce` include the commercial-restriction items whenever `commercial_risk_flag` is true; is `epistemic_posture` one of the four valid enum values.
- [ ] **Apply the fixed Step 9b logic here too** — re-verify the dignity/trust check using the corrected `pain_point_domain_confirmed` field as an independent cross-check, catching any case where T1's deterministic contract-building (Section 5) still missed a dignity/trust signal.
- [ ] For T2/T3 only, reserve a lightweight LLM-based validation pass for cases where the deterministic checks above cannot resolve ambiguity (e.g., verifying `epistemic_posture` is genuinely compatible with the evidence in `signal_synthesis_summary`, not just enum-valid) — this is the one place a second, small Claude call may be justified, applied selectively, not on every record.
- [ ] On any validation failure: route back to contract rebuild — never let a flagged-invalid contract proceed to RDA.
- [ ] Log every correction as a tracked metric — the ongoing justification for this step's existence.

---

## 7. FULL BRA DIAGRAM — INTEGRATED WITH REAL ARCHITECTURE

```
Webhook → Step 2 (Payload Validation) → Step 3 (Idempotency) → Step 4 (Already Processed)
  → Step 5 (Template Load) → Step 6 (Build Template Map) → Step 7 (Pre-derive Axes)
  → Step 8 (Tier Routing: IF T1)

  ├── TRUE (T1, ~70-80%, zero Claude):
  │     Step 9a (Template Selection)
  │       ▼
  │     Step 9b (Governance Screen — ★ BUG FIXED: pain_point_domain_confirmed,
  │              not pain_point_sub_category)
  │       ▼
  │     ★ NEW — BRA-C1 (T1, deterministic): build Response Contract from
  │              template fields + pre-derived axes, no Claude call
  │       ▼
  │     ★ NEW — BRA-C2 (T1, deterministic): schema + rule validation,
  │              re-verify dignity/trust using corrected field
  │       │
  │       ├── FAIL → route back to BRA-C1
  │       └── PASS → Step 10
  │
  └── FALSE (T2/T3, ~20-30%, one extended Claude call):
        Step 9c (Build Claude Prompt — ★ EXTENDED: adds must_reflect, may_reflect,
                 must_not_introduce, epistemic_posture, claim_type, closing_objective
                 to the existing 7-field requirement)
          ▼
        Step 9d (Build Request Body) → Step 9e (Claude API Call)
          ▼
        Step 9f (Output Parsing — ★ EXTENDED: validates the new fields alongside
                 the existing 7, same throw-on-missing pattern)
          ▼
        ★ NEW — BRA-C2 (T2/T3, mostly deterministic + selective LLM check for
                 genuinely ambiguous epistemic_posture cases only)
          │
          ├── FAIL → route back to Step 9c
          └── PASS → Step 10

  → Step 10 (Governance Gate, unchanged, still the shared convergence point)
  → Step 11 (Halt Check, unchanged)
  → Step 12 (BRA Run ID) → Step 13 (Build NocoDB Body — ★ may need new columns
           if contract fields are persisted, or kept payload-only per RDA's own
           "no new NocoDB columns" precedent — decide before building)
  → Step 14 (NocoDB Write) → Step 15 (Capture BRA Record ID)
  → Step 16 (Build Patch Body) → Step 17 (Patch HSI Status)
  → Step 18 (Build RDA Payload — ★ EXTENDED: existing 28 fields + new contract
           fields + dominant_pole + masked_emotion_hypothesis, now included)
  → Step 19 (RDA Trigger — ★ RECOMMEND fixing the confirmed-missing onError/retry
           here too, while this node is being touched anyway, though this is a
           separate known issue from the contract work itself)
```

---

## 8. OPEN DECISIONS FOR THE BRA SESSION TO RESOLVE

1. **Does the new contract data get persisted to BRA's own NocoDB table (new columns), or does it travel payload-only to RDA**, consistent with RDA's own "no new NocoDB columns" precedent set during its own hardening work? Recommend payload-only for consistency, but this is BRA's call.
2. **Should Step 9b's bug fix and Step 19's missing-retry fix be bundled into this same work**, since both are being touched anyway, or kept as separate, independently-tracked fixes? The Step 9b fix is argued above as necessary for this work; Step 19's retry gap is unrelated and could reasonably be deferred.
3. **For T1's deterministic claim_type assignment**, does the existing template library (`mafv9by73ebama7`) need a new column mapping each template to a default claim_type, or can this be derived purely from `Signal Category` + the governance flags already computed? This determines whether the template table itself needs a schema change.

---

## 9. WHY THIS IS CRITICAL FOR RDA'S OUTPUT QUALITY — UNCHANGED FROM THE ORIGINAL RATIONALE

RDA's three quality criteria (naturalness, brand recognizability, upstream grounding) cannot all be achieved while RDA is still deciding *what's relevant* as well as *how to say it*. A validated, correctly-integrated Response Contract — built using BRA's existing hybrid architecture rather than bypassing it — is what finally lets RDA execute a governed plan instead of guessing at one. The original production failure (the disputed bar-seating case) is directly solved by this work, specifically by the `claim_type`/`epistemic_posture` separation now correctly tied to BRA's real, existing `governance_flag` mechanism rather than a redundant new system running alongside it.
