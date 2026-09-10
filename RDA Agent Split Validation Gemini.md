Chat #122 · September 10, 2026

```
CLAIM: RDA as currently scoped is over-extended, but should be split along functional responsibility boundaries into a 2-agent model (Upstream Strategy + Downstream Drafting & Audit) rather than a 3-agent model, mitigating handoff failure via a Single Enriched Payload contract.
EVIDENCE FOR: 
1. System Principle: BRA = strategy ("what to say"), RDA = execution ("how to say it"). Q0/Q1 perform strategy work.
2. Call-level separation in LLM-as-a-Judge architectures provides equal statistical/attention isolation to agent-level separation without network-level overhead.
3. Instruction 6 Source Hierarchy & historical regression data confirm 3 separate instances of payload truncation across n8n agent boundaries.
EVIDENCE AGAINST: 
1. Leaving Q0/Q1 inside RDA avoids 1 n8n webhook boundary.
2. Merging Q0/Q1 into BRA increases BRA's node complexity.
CONCLUSION: ✅ HIGH — Recommend a 2-Agent Split (Absorb Q0/Q1 into BRA; keep Generation + Q2 Evaluation in RDA with Call-Level Separation).

```

---

### 1. Architectural Strategy Split: Q0/Q1 Location

```
CLAIM: Q0 (Contract Validator) and Q1 (Response Contract Builder) belong inside BRA, not RDA or a new third agent.
EVIDENCE FOR: BRA's explicit system role is deciding response strategy. Q0/Q1 define epistemic posture, signal priority, and closing objective.
EVIDENCE AGAINST: Placing Q0/Q1 in BRA increases BRA payload size sent to RDA.
CONCLUSION: ✅ HIGH — Move Q0/Q1 upstream into BRA.

```

**Reasoning:**

* **Role Alignment:** BRA exists to decide *what* the response must accomplish. Defining `must_reflect` signals, `epistemic_posture` (fault admission level), and `closing_objective` is pure strategic decision-making. Placing Q0/Q1 in RDA forces RDA to negotiate strategy before writing, violating single-responsibility design.
* **Why NOT a 3rd Agent:** Creating a new "Contract Agent" between BRA and RDA adds an unnecessary network boundary, increases execution latency by $1.2 - 2.0\text{ seconds}$, and introduces a new point of failure for payload truncation without adding functional isolation.

---

### 2. Evaluator Isolation: Agent-Level vs. Call-Level Separation for Q2

```
CLAIM: Call-level separation within the same n8n workflow is architecturally sufficient for Q2 Independent Evaluation.
EVIDENCE FOR: Transformers evaluate tokens based strictly on the context window of the individual API call. Cross-call memory does not exist in stateless API calls.
EVIDENCE AGAINST: Agent-level separation allows independent scaling/hosting of evaluation logic.
CONCLUSION: ✅ HIGH — Retain Q2 inside RDA using Call-Level Separation.

```

**Reasoning:**

* **Attention Isolation is Absolute at the Call Level:** Claude API calls are completely stateless. An API call running the `Naturalness Evaluator` prompt at `temperature=0` has zero attention or memory leakage from the previous API call that generated the draft, even if both calls are executed sequentially by the same n8n workflow node.
* **What Agent-Level Separation Would Buy:** The *only* advantage of agent-level separation for Q2 is independent infrastructure scaling (e.g., running evaluations on an asynchronous worker queue). In an n8n environment processing discrete review records, this provides zero quality benefit while doubling execution setup complexity.

---

### 3. Weighing Handoff Fragility vs. Single-Responsibility

```
CLAIM: Historical field-dropping during inter-agent handoffs is solved by an Upstream Canonical Payload Contract, not by over-consolidating agent roles.
EVIDENCE FOR: Direct history shows 3 instances (guest_name, brand_voice_summary, signal_enrichment) dropping during ESS->HSI->BRA passes due to unvalidated JSON mutators.
EVIDENCE AGAINST: Consolidating all logic into 1 mega-agent eliminates handoffs entirely.
CONCLUSION: ✅ HIGH — Implement a Canonical Pass-Through Payload schema at the BRA->RDA boundary.

```

**Reasoning:**

* **The Root Cause:** Fields dropped in past builds because upstream nodes created *new* JSON objects at each handoff rather than appending to an immutable, expanding context wrapper.
* **The Tradeoff:** Over-consolidating strategy and drafting into a single "Mega-RDA" agent to avoid webhooks creates a monolithic, unmaintainable prompt/node structure where prompt instructions compete for attention (e.g., the confirmed positive-beats-negative rule failure).
* **The Solution:** Fix handoff fragility structurally by enforcing a **Canonical Pass-Through Payload schema** (`payload_v2`), validated by JSON Schema at the entry point of RDA.

---

### 4. Direct Recommendation & Target Architecture Topology

**Recommendation:** **Execute a Clean 2-Agent Split.**

```
[ UPSTREAM AGENTS ]
  ALA ──► EIP ──► ESS ──► HSI
                           │
                           ▼
[ AGENT 6: BRA (Strategy & Governance) ]
  ├── 1. Determine Strategic Strategy
  ├── 2. Execute Q1 (Response Contract Builder)
  └── 3. Execute Q0 (Contract Validator Gate)
                           │
                           │  <-- Canonical Payload Pass (Validated via JSON Schema)
                           ▼
[ AGENT 7: RDA (Drafting & Evaluation) ]
  ├── 1. Native Generation Pass (Consolidated EN/ES Call using Q1 Contract)
  ├── 2. Q2 Evaluation Suite (Call-Level Separation: 3 Independent Judge Calls)
  └── 3. Deterministic Compliance & NocoDB/Sheet Write

```

#### Summary of System Owner Tradeoffs

| Option | Architectural Clarity | Latency & Cost | Handoff Risk | Recommendation |
| --- | --- | --- | --- | --- |
| **Status Quo (All in RDA)** | Low (RDA decides strategy AND drafts) | Medium | Zero (Internal) | ❌ Reject (Over-scoped) |
| **Recommended 2-Agent Split** | High (BRA = Strategy, RDA = Drafting/Audit) | Optimal | Low (Mitigated by JSON Schema) | ✅ **APPROVE** |
| **3-Agent Split (New Contract Agent)** | Medium (Excessive fragmentation) | High (+2s Latency) | High (2 Handoff Boundaries) | ❌ Reject (Over-engineered) |
