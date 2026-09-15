**Chat #149 · September 11, 2026**

This checklist is much tighter than the earlier plan. I would **not redesign the architecture again**, but I see several checklist-level issues worth correcting before Phase 0 starts.

## Overall verdict

**Approved with checklist corrections.**

The main architecture is coherent:

**BRA strategy → Response Contract → validation → RDA wording → human approval**

The remaining issues are mostly implementation precision, but two are significant: **the T1 retry logic** and **how the new fields are actually persisted/verified across the BRA→RDA handoff.**

### 1. Phase 1 contains an internal contradiction

You currently have exact-looking bucket counts:

- 10 experiential
- 10 policy/procedural
- 15 factual allegations
- 10 corroborated/disputed pairs
- 10 high-liability
- 5–10 commercial

but then:

> ⚠️ Unresolved: exact per-bucket minimum count

Those two statements conflict.

I would lock the first corpus at **72 records**, using overlapping coverage rather than mutually exclusive buckets:

| Coverage requirement | Minimum |
|---|---:|
| Experiential | 10 |
| Policy/procedural | 10 |
| Factual allegation | 15 |
| Guest-report-only factual allegation | 8 |
| Corroborated factual allegation | 6 |
| Disputed factual allegation | 6 |
| High-liability | 10 |
| Commercial-boundary | 6 |
| Mixed-epistemic / multi-claim | 8 |
| Spanish | **≥30 of 72** |

These are **coverage minima**, not additive buckets. One record can satisfy several.

That resolves the open item without forcing an artificial taxonomy.

---

# 2. Phase 2: “6 new fields” and Phase 7: “7 contract fields” need reconciliation

This immediately stands out.

Phase 2 says:

> Step 9c prompt extended with **6 new required fields**

Phase 7 says:

> **7 contract fields** added to payload

Your contract also appears to contain structured objects/arrays beyond those headline fields.

Before Phase 0 ends, create a single authoritative table:

| Contract field | Produced by | Validated by | Sent to RDA? |
|---|---|---|---|
| `must_reflect` | BRA-C1 | C2 | Yes |
| `may_reflect` | BRA-C1 | C2 | Yes |
| `must_not_introduce` | BRA-C1 | C2 | Yes |
| `commercial_restriction_types` | BRA-C1 | C2 | Yes |
| `primary_claim_type` | BRA-C1 | C2 | Yes |
| `verification_state` | BRA-C1 | C2 | Yes |
| `highest_risk_class` | BRA-C1 | C2 | Yes |
| `strictest_epistemic_posture` | BRA-C1 | C2 | Yes |
| `closing_objective` | BRA-C1 | C2 | Yes |

Whatever the final v2.2 schema contains should appear **exactly once in that manifest**.

Do not let “6 fields,” “7 fields,” and the actual schema drift independently.

---

# 3. Important: T1 deterministic retry currently makes little sense

Phase 5 says:

> N=1 retry and hard-stop confirmed working

But T1 is deterministic.

If:

**same inputs + same rules → same contract**

then:

> Build → fail validation → rebuild

will simply generate the exact same invalid contract again.

That retry is useful for the LLM branch because a constrained repair may succeed.

It is not useful for a purely deterministic T1 builder unless the retry follows a **different repair path**.

I would define:

### T2/T3
Build → validator fails → one targeted LLM rebuild using failure reason → validate again → Halt.

### T1
Build → validator fails → **do not blindly rebuild identically**.

Instead either:

- run a deterministic repair rule if the defect is mechanically repairable, or
- escalate to the Claude path / human review.

This is a real checklist defect.

**Confidence: very high.**

---

# 4. Phase 4 determinism test needs version awareness

Current:

> same ID twice → identical contract

Change this to:

> **same input + same ruleset version + same template-library version + same contract-builder version → identical contract**

An ID alone is insufficient.

If the template library changes next month, the same record ID legitimately may produce a different contract.

Your version fields already exist, so use them in the reproducibility definition.

---

# 5. Phase 4 says T1 derives fields from “template metadata + Step 7 axes only”

I would inspect this carefully.

Your Response Contract contains `must_reflect`.

If T1 only sees generic template metadata and decision axes, you risk losing review-specific anchors such as:

- staff member mentioned
- specific dish
- occasion
- explicit complaint
- loyalty
- concrete praise target

That would produce a structurally valid but generic contract.

If those details are already available in Step 7/template metadata, fine.

If not, Phase 4 needs:

> **Deterministically extract available review-specific anchors from existing upstream/raw-review fields.**

No AI is required for obvious structured entities already present upstream.

The contract should govern **this review**, not merely its template category.

---

# 6. Observability starts too late

Phase 8 currently says:

> Failure observability confirmed...

You need those logs from **Phase 2 onward**.

During shadow testing, record:

- builder version
- validator version
- contract output
- validation result
- failure class
- failure reason
- retry attempted
- retry result
- escalation reason
- latency/token usage for Claude branch

Otherwise Phase 2–5 testing will produce failures without a clean diagnostic trail.

Move observability instrumentation into the first shadow implementation.

---

# 7. Phase 7 needs a real transport test, not a chat confirmation

This line is too weak:

> RDA's own intake confirmed (from RDA's chat window) to extract new fields

Given your documented history of fields disappearing at handoffs, this should be an **executed integration test**.

I would require:

1. BRA creates a known synthetic contract.
2. Step 18 serializes it.
3. Actual webhook payload is captured.
4. RDA receives it.
5. RDA parses it.
6. Values are compared field-by-field against the source.
7. Missing/changed field = test failure.

Use a sentinel payload with unmistakable values.

For example:

```text
primary_claim_type = factual_allegation
verification_state = disputed
highest_risk_class = elevated
```

and distinctive array contents.

The rule should be:

> **Never verify an inter-agent handoff by reading code alone when the failure history is silent field loss.**

Test the live boundary.

---

# 8. “No NocoDB schema change” conflicts potentially with Phase 8 logging

You lock:

> No new NocoDB columns

but Phase 8 requires all four versions logged **per record**.

Where are these stored?

If existing fields or another execution-log table already supports them, document that.

Otherwise:

> “no NocoDB schema change”

and

> “all four versions stored per record”

may be mutually incompatible.

Resolve the storage location in Phase 0.

Possible places:

- existing JSON/audit field
- n8n execution log
- dedicated existing metadata structure
- external run log

But do not leave “logged” undefined.

---

# 9. `decision_trace` also needs an explicit persistence location

You correctly added it in Phase 0, but it disappears from later phases.

Make sure the checklist states:

- created by builder,
- validated,
- available during shadow tests,
- persisted for debugging,
- **not sent to guest-facing generation unless RDA genuinely needs it.**

RDA should generally consume the governed decision, not internal explanation of how BRA reached it.

The trace is for auditability.

---

# 10. Phase 3 validator should validate provenance as well

You list:

- structural
- semantic hard failure
- semantic ambiguity

Add explicit provenance checks:

- `direct_guest_statement` requires valid `source_span`
- `upstream_inference` requires valid `upstream_origin`
- `business_verified_fact` requires verification source/reference
- source span must actually occur in the source review when it claims to be an exact quote

This is particularly important because epistemic posture depends partly on evidence source.

---

# 11. Define what “Halt” means for technical contract failure

The checklist currently says after repeated validation failure:

> `Halt + human_review_required`

Be careful not to make it look as though the guest review itself triggered a governance Halt when the actual problem was:

> BRA failed to construct valid JSON twice.

I would preserve a separate reason dimension:

```text
governance_flag = Halt
halt_reason = contract_validation_failure
```

versus:

```text
governance_flag = Halt
halt_reason = high_liability_guest_signal
```

Same workflow action, very different meaning.

This matters downstream in reporting.

---

# 12. Add one invariant around `escalate_to_human`

If:

`strictest_epistemic_posture = escalate_to_human`

then RDA should not simply receive that contract and generate an ordinary public draft.

Specify the behavior.

For example:

> `escalate_to_human` → public drafting suppressed OR clearly marked non-publishable internal draft, depending on locked product behavior.

Since VRYOH already requires human approval for everything, “escalate to human” needs to mean something **stronger than normal human approval**, otherwise the enum has no operational distinction.

This should be frozen in Phase 0.

---

# 13. Regex term completeness: don't solve this as a word-list problem

Your open item should become:

> **T1 exception-detection ruleset completeness**

not “regex term-list completeness.”

Use concept families:

- policy/procedure
- factual staff action
- discrimination/dignity
- health/allergy
- physical safety
- misconduct/violence
- financial/fraud
- legal/regulatory
- privacy

Each group should contain:

- positive patterns,
- exclusions,
- context requirements,
- false-positive examples.

That will scale much better than continually appending words.

A reasonable Phase 1 goal is **high recall**, because a false positive only promotes a cheap T1 record into more scrutiny; a false negative can allow a sensitive record through the fast path.

I would initially optimize the exception detector toward recall rather than precision.

---

# 14. Add bilingual regex coverage

This is absent from the checklist.

VRYOH is bilingual.

If the T1 exception filter runs on raw guest text and only contains English patterns, Spanish T1 reviews will have a structurally weaker safety net.

You need native Spanish concept rules too, including variants such as:

- `política`
- `me negaron`
- `no me dejaron`
- `discriminación`
- `racista`
- `alergia`
- `me cobraron dos veces`
- `abogado`
- `demanda`
- `policía`
- `agresión`
- `robo`

Do **not** merely translate the English regex mechanically. Validate Spanish expressions against actual review language.

This is a missing checklist requirement.

---

# 15. Phase 6 needs blinded comparison

When reviewing current vs contract-governed drafts, if the reviewer knows which is “new,” confirmation bias becomes likely.

Randomize A/B ordering where practical.

Evaluate first without identifying:

- legacy
- contract-governed

Then reveal after scoring.

For each draft score independently:

- factual grounding
- acknowledgment adequacy
- naturalness
- brand fit
- governance correctness

This makes Phase 6 evidence much stronger.

---

# 16. Phase 8 needs explicit stop conditions

“Subset reviewed before expanding” is too subjective.

Before rollout, define canary stop criteria.

At minimum, immediate stop/rollback if:

- any confirmed unsupported factual admission occurs in a disputed/high-risk case;
- required contract field disappears at the BRA→RDA boundary;
- contract validator failure reaches RDA as a normal pass;
- `escalate_to_human` is ignored;
- malformed contract passes validator.

And define quantitative review triggers for things like:

- large increase in human rejection rate;
- excessive false Halt rate;
- T1 escalation rate unexpectedly spikes.

You don't need perfect numbers before baseline, but catastrophic stop conditions should be explicit now.

---

## Updated status of the open items

I would now close the first open item:

### Golden corpus
**72 initial records**, with overlapping minima rather than additive buckets.

I would replace the second open item:

### Instead of:
“Regex term list completeness”

### Use:
**“EN/ES T1 exception-detector coverage and false-negative rate.”**

That is the actual engineering problem.

The third item remains valid:

> T1 cost-complexity ceiling — future review trigger.

A sensible trigger to revisit the rules engine would be when one or more of these happens:

- exception rules become difficult to reason about;
- false-negative rate stops improving without many special cases;
- changes repeatedly break unrelated patterns;
- maintenance effort exceeds that of a small semantic classifier;
- a meaningful portion of T1 gets escalated anyway.

---

# Final pre-build checklist verdict

### **Ready to begin after targeted corrections — no further architecture round needed.**

I would modify the checklist before Phase 0 in these specific places:

1. Lock the golden corpus at **72** and remove the contradictory unresolved-count item.
2. Create one authoritative **Contract Field Manifest** to resolve 6-vs-7-vs-schema field counts.
3. Change deterministic T1's failed-validation behavior; **don't retry the identical computation**.
4. Define determinism using inputs + versions, not ID alone.
5. Confirm T1 contracts retain review-specific anchors.
6. Start observability in Phase 2.
7. Replace “confirmed from RDA chat” with an actual end-to-end payload integrity test.
8. Specify where version metadata and `decision_trace` are persisted despite the no-new-columns decision.
9. Add provenance validation.
10. Define the operational meaning of `escalate_to_human`.
11. Make the T1 exception detector bilingual EN/ES.
12. Blind the Phase 6 A/B comparison.
13. Define Phase 8 hard rollback conditions.

After those edits, I would start building rather than continue abstract auditing. The next useful information will come from the **golden contracts, shadow outputs, and measured failure cases**, not from another conceptual redesign.
