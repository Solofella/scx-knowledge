**Chat #147 · September 10, 2026**

Overall, this v2 plan is **substantially better structured and close to implementation-ready**, but I would still make a few important changes before calling Phase 0 frozen.

The main strengths are clear: the ontology is cleaner, the high-risk issue is moved upstream where it belongs, T2/T3 are shadow-tested first, fail-closed behavior is explicit, and rollback/observability are now part of the rollout rather than afterthoughts.

## What I would change before implementation

### 1. Fix the Phase 7 / Phase 8 order

Right now you have:

**Phase 7 — Controlled Rollout**  
**Phase 8 — Payload Integration**

That sequence is backwards.

RDA cannot actually use the new contract in controlled rollout until the BRA→RDA payload already carries it.

I would make it:

**Phase 7 — Payload Integration Behind Feature Flag**  
Add the fields to the BRA→RDA payload, validate that RDA receives them correctly, but keep `response_contract_enabled = false`.

**Phase 8 — Controlled Rollout**  
Turn the flag on for a limited environment/client set, observe, then expand.

That is a required change.

---

### 2. Resolve the multi-claim review problem before schema freeze

A single review can contain more than one epistemic situation.

Example:

> “The pasta was cold, the manager refused to replace it, and I was charged twice.”

That contains:

- an experiential/service issue
- a factual allegation
- a possible financial issue

One record-level `claim_type` may be too coarse.

I would either:

**Option A — Better long-term design**  
Put `claim_type`, `verification_state`, and possibly `risk_class` on each `must_reflect` item.

or:

**Option B — Simpler v1**  
Keep record-level fields but explicitly define them as:

- `primary_claim_type`
- `highest_risk_class`
- `strictest_epistemic_posture`

If you keep one record-level value without defining how multi-claim reviews collapse into it, you will eventually get inconsistent behavior.

---

### 3. Change T1 regex from “classifier” to “exception detector”

The current wording still gives the regex too much authority:

> a trigger term upgrades the record's `claim_type`

I would instead have the regex emit something like:

- `exception_trigger_detected = true`
- `candidate_claim_type = policy_procedural`
- `candidate_reason = policy_term`

Then deterministic contextual rules can confirm the classification when obvious.

If not obvious, the record should leave the pure fast path.

That gives you a safer principle:

> **T1 is zero-AI by default, not zero-AI under all circumstances.**

This is an important distinction.

---

### 4. Make `high_liability + governance_flag: None` invalid

I would resolve the open question now.

If a record is explicitly classified:

`risk_class = high_liability`

then:

`governance_flag = None`

should **hard-fail validation**.

That does not mean `risk_class` and `governance_flag` are the same field. They remain different concepts.

It means a workflow action of “None” is incompatible with an explicitly high-liability classification.

I would use:

- `normal` → `None` allowed
- `elevated` → `None` or `Flag`
- `high_liability` → minimum `Flag`
- defined subtypes → mandatory `Halt`

---

### 5. Clarify `source_span`

This should not always be mandatory.

For:

`source_type = direct_guest_statement`

yes, exact `source_span` should be required.

For:

`source_type = upstream_inference`

an exact quote may not exist.

For:

`source_type = business_verified_fact`

you may need a `verification_source` rather than a guest quote.

I would define conditional provenance requirements:

- direct guest statement → `source_span` required
- upstream inference → `upstream_origin` required, `source_span` optional
- business verified fact → `verification_source` required

That prevents the system from fabricating exact spans just to satisfy the schema.

---

### 6. Simplify `relevance_confidence` unless you can calibrate it

A number such as:

`0.87`

looks statistically meaningful.

Unless you actually know that the confidence values are calibrated, I would initially use:

`high | medium | low`

or make the field optional.

You can move to numeric probabilities later if you have a labeled corpus showing those values mean something.

---

## Part C — Epistemic posture table

The revised two-input logic is much stronger.

But this row:

> experiential + not_applicable → direct_apology OR neutral_acknowledgment, keyed off severity

means the table is still effectively using a third variable.

That is fine, but formalize it.

The actual rule is closer to:

**claim_type + verification_state + severity/risk → epistemic_posture**

You probably already have the severity input from HSI/BRA, so you may not need a new field.

Just make the rule explicit before freeze.

---

## Part D — Regex list

The proposed list is fine as a starting set, but I would organize it into **concept groups**, not one flat regex list.

For example:

**Policy/procedure**
- policy
- posted
- signage
- fire code
- refused
- denied

**Legal/regulatory**
- lawyer
- lawsuit
- police
- attorney

**Safety/health**
- allergic
- injury
- assault

**Trust/misconduct**
- discriminate
- stole
- theft

That matters because the matched group should drive the candidate classification and risk handling.

Also test obvious false positives:

> “The manager said the food was amazing.”

> “I loved that your policy allows outdoor seating.”

A regex engine cannot safely infer semantic meaning from a keyword alone.

---

## Part E — Validator

The validator is now well designed.

I would explicitly create three failure classes:

### Structural failure
Malformed or incomplete contract.

Examples:
- missing field
- invalid enum
- missing required provenance

### Semantic hard failure
Internally incompatible state.

Examples:
- disputed allegation + direct fault admission
- high liability + governance None

### Semantic ambiguity
Technically valid but uncertain.

Examples:
- policy complaint where available evidence is incomplete

Then route:

**Structural/hard failure → rebuild once → Halt if still failing**

**Ambiguity → narrow semantic check or human review**

That keeps your deterministic-first approach disciplined.

---

## Golden-set size

I would set the Phase 1 minimum now.

For the contract-specific golden set:

**60–80 records** is a good starting target.

You don't need equal buckets. Use overlapping cases.

Suggested coverage:

- 10 straightforward experiential
- 10 policy/procedural
- 15 factual allegations
- 10 corroborated/disputed paired cases
- 10 elevated/high-liability cases
- 5–10 commercial-boundary cases
- at least 40% Spanish

Later, merge this into the larger 100–150+ RDA regression corpus.

---

## Add one more test category: mixed-epistemic reviews

I would explicitly add this to Phase 1.

Example:

> “Food was great, but they charged me twice and the manager told me there was nothing they could do.”

This forces the contract logic to separate:

- positive experiential signal
- factual financial allegation
- possible staff conduct/policy issue

This will expose whether the record-level schema is too coarse very quickly.

---

## T2/T3 single-call extension

You already added the right benchmark, but also measure whether the **existing 7 BRA fields degrade** after adding the new contract requirements.

Don't only ask:

> Did the new contract come out correctly?

Also ask:

> Did tone, urgency, governance flag, and strategic interpretation get worse because the model now has more work to do?

That comparison should be explicit in Phase 2.

---

## Add `ruleset_version`

For T1 reproducibility, this:

> same `hsi_record_id` → identical contract

is not quite enough.

The more accurate requirement is:

> same input + same template version + same deterministic ruleset version → identical contract

Store at least:

- `contract_builder_version`
- `validator_version`
- `ruleset_version`
- `template_library_version`

That gives you real reproducibility.

---

## One addition I strongly recommend: contract decision trace

For debugging, store a compact internal trace such as:

```text
claim_type_reason = "policy_term_detected + template_category"
risk_class_reason = "safety_keyword"
epistemic_posture_reason = "guest_report_only + elevated_risk"
```

Not chain-of-thought, just **structured decision provenance**.

This will be extremely useful when a contract looks wrong later.

---

# Revised sequence I recommend

I would make the final implementation sequence:

1. **Schema + ontology freeze**
2. **Golden contract corpus**
3. **T2/T3 contract generation — shadow**
4. **T2/T3 validator — shadow**
5. **T1 exception detector + deterministic contract builder**
6. **T1 validator**
7. **RDA side-by-side shadow generation**
8. **BRA→RDA payload integration behind feature flag**
9. **Controlled rollout**
10. **Expand only after measured sign-off**

That sequence is cleaner than the current 7/8 order.

---

# Final judgment

**Approved with modifications before Phase 0 freeze.**

The architecture is now sound enough that I would not propose another major redesign.

The remaining issues are mostly about making the contract precise enough to survive real-world ambiguity:

- multi-claim reviews
- provenance rules
- T1 exception handling
- high-liability invariants
- posture derivation
- reproducibility/versioning
- rollout sequencing

The single most important addition I would make is this:

> **Decide explicitly whether epistemic classification lives at record level or per response-driving claim.**

That decision will affect almost everything downstream. If you resolve that well, the rest of the plan is in strong shape.
