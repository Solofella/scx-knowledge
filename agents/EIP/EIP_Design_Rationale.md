# EIP Design Rationale — Full Dictionary Injection

**Decision Date:** Chat #18 · March 11, 2026  
**Status:** Locked (permanent architecture decision)

---

## Decision

EIP injects the complete Emotion Dictionary (161 entries) and Pain Point Master (336 entries) into the GPT prompt on every record. Total injection: ~15,350 tokens per call.

**No pre-filtering. No sampling. Complete taxonomy every time.**

---

## Rationale

### 1. Single Knowledge Injection Point

**EIP is the ONLY agent that accesses dictionaries directly.**

All downstream agents (ESS, HSI, SIA, BRA, RDA) inherit EIP's resolved classifications:
- ESS receives: Enriched Emotion Tag, Core Emotion, Need State, Cognitive Driver
- HSI receives: Pain Point Sub-Category, Domain, Signal Weight
- SIA reads: EIP NocoDB output directly (zero AI calls)
- BRA/RDA: Operate on HSI output (second-order inheritance)

**Pipeline efficiency:** Dictionary cost paid once at EIP. Zero incremental cost for 5 downstream agents.

---

### 2. Quality Priority (Non-Negotiable)

**SubtextCX's competitive claim:** Dual-instrument masked emotion detection at review-analysis level.

**"Masked emotion" detection requires complete taxonomy coverage:**
- Guest says: "Everything was fine"
- Masked meaning: Frustration (see Emotion Dictionary entry EMO-089: "Fine" as performative dismissal)
- Pre-filter risk: If "fine" triggers filter to exclude "Frustration" entry, detection fails

**Full injection guarantees:** Every emotion + pain point combination visible to GPT on every review.

**Quality is the primary objective.** Token cost should be priced into client tiers, not used to justify reduced taxonomy coverage.

---

### 3. Economic Viability (OpenAI Prompt Caching)

**OpenAI automatic prompt caching** makes full injection cost-effective at scale.

#### How Caching Works

**Dictionary content is identical across all records in a batch:**
- First record: Cache write (10% cost on 14,750 dictionary tokens)
- Subsequent records: Cache read (10% cost on 14,750 dictionary tokens)
- Unique per record: ~600 tokens (review text + context)

**Effective per-record cost:**
- Without caching: 15,350 tokens input
- With caching (90% hit rate): ~1,535 tokens effective input
- **Savings: 90% on dictionary portion**

#### Scale Economics

**At pilot scale (300 reviews/month):**
- With caching: 300 × 1,535 = 460K tokens/month
- Without caching: 300 × 15,350 = 4.6M tokens/month
- **Savings: 4.14M tokens/month**

**At production scale (1,500 reviews/month across 5 clients):**
- With caching: 1,500 × 1,535 = 2.3M tokens/month
- Without caching: 1,500 × 15,350 = 23M tokens/month
- **Savings: 20.7M tokens/month**

---

### 4. Downstream Zero-Cost Inheritance

**Token budget comparison:**

| Agent | Dict Access | AI Tokens | Notes |
|-------|-------------|-----------|-------|
| **EIP** | **Full (15,350)** | **1,535 effective** | **Pays dict cost once** |
| ESS | Inherits | 1,200 | Expression Mode analysis only |
| HSI | Inherits | 1,800 | Behavioral narrative only |
| SIA | Inherits | 0 | Pure JS aggregation |
| BRA | Inherits | 2,500 avg | Template selection |
| RDA | Inherits | 3,000 | Final draft generation |

**Total pipeline:** ~10,435 tokens/record effective (with EIP caching)

**If each agent queried dictionaries independently:**
- EIP: 15,350
- ESS: 13,000 (Emotion Dict)
- HSI: 20,000 (Pain Point Master)
- **Total: 48,350+ tokens** (not counting SIA, BRA, RDA re-queries)

**EIP single-injection saves:** ~38,000 tokens per record across pipeline

---

## Alternative Rejected: Pre-Filter to ~10 Entries

### Proposed Approach (Not Implemented)

**Step 1:** Pre-classification GPT call analyzes review, selects ~10 relevant emotion/pain point entries

**Step 2:** Second GPT call receives only filtered subset, performs final classification

### Why Rejected

#### 1. Quality Risk (Unacceptable)

**Pre-filter requires second classification step:**
- If pre-filter misclassifies, correct taxonomy entry never reaches final GPT call
- Example: Review mentions "fine" (masked frustration) → pre-filter selects "Satisfaction" entries → "Frustration" entry excluded → masked emotion undetected
- **Failure mode:** Defeats SubtextCX's core value proposition

#### 2. Marginal Token Savings

**Pre-filter approach:**
- Pre-filter call: ~1,000 tokens (analyze review, select entries)
- Final call with 10 entries: ~800 tokens (reduced dict + review text)
- **Total: ~1,800 tokens per record**

**Full injection with caching:**
- Single call: ~1,535 tokens effective (with 90% cache discount)

**Savings with pre-filter: 265 tokens per record**

**Not worth quality risk** for 15% token reduction.

#### 3. No Downstream Benefit

**Pre-filter approach still requires ESS/HSI to either:**
- Trust EIP's entry selection (same inheritance model as current)
- OR query dictionaries themselves for validation (duplicate cost)

**Full injection approach:**
- EIP sees complete taxonomy → resolves all fields definitively
- Downstream agents trust resolved output (no re-query needed)

---

## Trade-Offs Accepted

### High Token Count at EIP Layer

**15,350 tokens per record is expensive** compared to typical AI classification tasks (~500-1,000 tokens).

**Accepted because:**
- OpenAI caching reduces effective cost to ~1,535 tokens (comparable to typical tasks)
- Downstream agents benefit from resolved output (pipeline-wide efficiency)
- Quality guarantee worth the investment

---

### Cache Dependency

**Risk:** If OpenAI cache expires between records, full cost repeats.

**Mitigation:**
- BCA (future build) will control batch drip interval to stay within cache TTL window
- Current direct pass-through (ALA → EIP) processes records immediately (natural batching)
- Monitor cache hit rate in production (target: >90%)

---

### Zero Tolerance for Cache Misses at Scale

**At 1,500 reviews/month:**
- 90% cache hit rate: 2.3M tokens/month (acceptable)
- 50% cache hit rate: 12M tokens/month (cost blowout)
- 0% cache hit rate: 23M tokens/month (catastrophic)

**Operational constraint:** BCA batch controller must ensure cache-friendly processing intervals.

---

## Why GPT Instead of Claude for EIP?

### OpenAI Cache > Anthropic Cache for This Use Case

**OpenAI prompt caching:**
- TTL: Session-based, longer cache window
- Reliability: Stable across batch processing
- Cost: 90% discount on cached content

**Anthropic prompt caching:**
- TTL: 5 minutes (short window)
- Risk: BCA drip interval might exceed 5 min → cache expires → full cost repeats
- Cost: 90% discount when cache active (same as OpenAI)

**Decision driver:** Cache reliability, not model performance.

**Model performance parity:** Both GPT gpt-5.2 and Claude claude-sonnet-4-6 excel at structured JSON classification. Cost efficiency and cache stability drove the choice.

---

## Governance Constraint (Applies to All Agents)

**EIP detects and classifies. It does NOT prescribe.**

**Internal followup drafts** (used by RDA) must describe signal meaning, not operational actions.

**Good classification:**
- "Guest expressed frustration with wait time during weekend dinner service (Need State: Autonomy violated, Cognitive Driver: Loss of control over schedule)"

**Bad classification (violates governance):**
- "Add more servers during weekend dinner" ← Prescriptive, not allowed

**EIP classifications are descriptive only.** Downstream agents maintain this principle.

---

## Long-Term Model-Agnostic Strategy

**Current state:** EIP uses GPT gpt-5.2 (commercial LLM dependency)

**Future vision:** Value must come from architecture, knowledge structures, and accumulated data — not the AI model itself.

**EIP as potential open-source substitution candidate:**
- Structured JSON classification task (well-defined input/output)
- Complete taxonomy provided in prompt (no model training required)
- Deterministic validation layer catches hallucinations

**Roadmap (2027+):** Evaluate fine-tuned open-source models (Llama, Mistral) for EIP to reduce commercial LLM exposure.

**Retained commercial LLM agents:** BRA (T2/T3), RDA (governance-critical creative generation)

---

## Related Documents

- **HOW Document:** [SCX_EIP_HOW_v4.md](SCX_EIP_HOW_v4.md)
- **Schema:** [EIP_Schema.md](EIP_Schema.md)
- **Changelog:** [SCX_EIP_CHANGELOG.md](SCX_EIP_CHANGELOG.md)
- **SIA Design Rationale:** [../SIA/SIA_Design_Rationale.md](../SIA/SIA_Design_Rationale.md) (zero-cost aggregation model)

---

**End of EIP Design Rationale**
