# SIA Design Rationale — Zero-Cost Aggregation

**Decision Date:** Chat #72 · March 28, 2026  
**Status:** Locked (permanent architecture decision)

---

## Decision

SIA is implemented as pure JavaScript aggregation with **zero AI API calls**.

No OpenAI. No Anthropic. No LLM of any kind.

**Total token cost per SIA run: 0**

---

## Architecture

### Input
SIA reads the **EIP NocoDB table** directly (no webhook trigger).

Query parameters:
- Client ID (e.g., `PAK-001`)
- Date range (e.g., last 7 days, last 30 days)

Returns: Array of EIP records with all classifications already resolved.

---

### Process
Pure JavaScript in n8n Code Nodes:

1. **Group records** by Domain + Signal Tier
   - `for` loops over EIP records
   - `Object.keys()` to create cluster map
   - Key format: `"Food Quality|T-NEGATIVE"`

2. **Count frequencies**
   - Increment counter per cluster
   - Track pain points and emotions per cluster

3. **Calculate percentages**
   - `(cluster count / total records) × 100`
   - Round to integer

4. **Build JSON breakdowns**
   - Count pain point occurrences in cluster
   - Sort by frequency (most common first)
   - Serialize to JSON string

5. **Compare to prior period**
   - Query SIA table for previous date range
   - Calculate percentage change
   - Assign trend direction: UP / DOWN / STABLE / NEW

---

### Output
Write to **SIA NocoDB table** (11 fields per cluster):
- Domain, Signal Tier, Count, Percentage
- Enriched Pain Points JSON, Enriched Emotions JSON
- Trend Direction, Date Range Start/End
- Client ID, Created At

---

## Rationale

### 1. EIP Already Classified Everything

**By the time SIA runs, EIP has already:**
- Resolved all emotion classifications (Enriched Emotion Tag, Core Emotion, Need State)
- Resolved all pain point classifications (Domain, Sub-Category, Signal Weight)
- Determined Signal Type (T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE)

**SIA's job is NOT to interpret.**

SIA's job is to **count and group** what EIP already classified.

**Example:**
- EIP classified Review #47 as: `Food Quality | T-NEGATIVE | "Underseasoned protein"`
- SIA reads that record and increments: `Food Quality | T-NEGATIVE` counter
- No AI needed. Just `count += 1`.

---

### 2. Aggregation is Deterministic

**SIA performs only deterministic operations:**

| Operation | AI Needed? | Why/Why Not |
|-----------|------------|-------------|
| Group by domain | ❌ | Simple object key creation |
| Count records | ❌ | Increment counter in loop |
| Calculate percentage | ❌ | Basic arithmetic: `(count / total) × 100` |
| Sort by frequency | ❌ | Array.sort() with comparison function |
| Build JSON array | ❌ | JSON.stringify() |
| Compare to prior period | ❌ | Subtract counts, calculate % change |

**No step requires interpretation, inference, or natural language understanding.**

**Same input = same output.** Every time. No LLM variability.

---

### 3. Zero Incremental Token Cost

**Token economics at scale:**

| Reviews/Month | EIP Cost (with caching) | SIA Cost | Total Pipeline |
|---------------|-------------------------|----------|----------------|
| 300 (pilot) | 460K tokens | **0 tokens** | ~3.1M tokens |
| 1,500 (5 clients) | 2.3M tokens | **0 tokens** | ~15.7M tokens |
| 10,000 (growth) | 15.4M tokens | **0 tokens** | ~104M tokens |

**SIA scales infinitely at zero marginal AI cost.**

Adding 1,000 more reviews to SIA = same 0 token cost.

**Contrast with AI-based aggregation:**
- If SIA used Claude to "interpret signal intensity": ~1,500 tokens per record
- At 300 reviews/month: 450K tokens just for SIA
- At 10,000 reviews/month: 15M tokens just for SIA

**By using pure JavaScript, SIA eliminates 15M tokens/month at scale.**

---

### 4. Execution Speed

**Performance comparison (300 records):**

| Approach | Execution Time | Why |
|----------|----------------|-----|
| **Pure JavaScript (current)** | **<1 second** | Local computation, no API calls |
| Hypothetical Claude aggregation | ~30 seconds | 300 API calls × 100ms avg latency |
| Hypothetical GPT aggregation | ~25 seconds | Similar latency |

**30× faster execution.**

**Dashboard load time:** Users see Signal Pulse chart in <1 second instead of waiting 30 seconds.

---

### 5. No API Rate Limits

**SIA never hits:**
- OpenAI rate limits (no API calls)
- Anthropic rate limits (no API calls)
- Token-per-minute limits (zero tokens)

**High-volume clients (1,000+ reviews/month) process without throttling.**

**Contrast with AI approach:**
- 1,000 records × Claude calls = potential rate limit errors
- Requires retry logic, exponential backoff
- Unpredictable completion time

**Pure JavaScript = predictable, instant execution.**

---

## Alternative Rejected: Claude-Based Aggregation

### Proposed Approach (Not Implemented)

**Use Claude to "interpret signal intensity":**
1. Pass all EIP records for a domain to Claude
2. Ask Claude: "What's the overall sentiment for Food Quality?"
3. Claude generates summary, intensity score, trend interpretation

### Why Rejected

#### 1. Redundant Interpretation

**EIP + ESS + HSI already provide interpretation:**
- EIP: Emotion + pain point classifications
- ESS: Expression Mode, Emotional Clarity
- HSI: Behavioral narratives, operator insights

**SIA adding another layer of interpretation = redundant.**

**Dashboard needs metrics, not more narratives:**
- "18% negative" (SIA metric) → clear, actionable
- "Food Quality shows concerning patterns" (Claude summary) → vague, not quantifiable

---

#### 2. Token Cost Explosion

**At 300 reviews/month:**
- Claude aggregation: ~1,500 tokens per domain per period
- 13 domains × 3 periods (week/month/quarter) = 39 Claude calls
- 39 × 1,500 = 58,500 tokens/month just for SIA

**Current pure JS approach: 0 tokens/month**

**Savings: 58,500 tokens/month at pilot scale**

**At 10,000 reviews/month:**
- Same 39 Claude calls (domains × periods)
- But each call processes 10,000 records instead of 300
- Token count per call increases (more context to synthesize)
- Estimated: ~3,000 tokens per call
- 39 × 3,000 = 117,000 tokens/month

**Savings: 117,000 tokens/month at scale**

---

#### 3. LLM Variability Breaks Trend Tracking

**SIA compares current vs prior period:**
- Week 1: "Food Quality negative: 18%"
- Week 2: "Food Quality negative: 23%"
- **Trend: UP (+5 percentage points)**

**Deterministic math = reliable trend tracking.**

**If Claude-based:**
- Week 1: Claude says "moderate concern" (interprets 18% as "moderate")
- Week 2: Claude says "significant concern" (interprets 23% as "significant")
- **No numeric comparison possible.** "Moderate" vs "significant" is qualitative.

**Dashboard can't display:** "↑ Food Quality concerns up 5 percentage points"

**It would display:** "Food Quality: significant concern" (no trend direction)

**Users lose actionable insight.**

---

#### 4. Execution Time Penalty

**Pure JS aggregation: <1 second**

**Claude-based aggregation:**
- 39 API calls (13 domains × 3 periods)
- ~100ms per call (network latency + inference)
- Sequential execution (can't parallelize easily in n8n)
- **Total: ~4 seconds minimum**

**4× slower.**

**Dashboard "Trend Signals" section takes 4 seconds to load instead of instant.**

**User experience degradation for zero functional benefit.**

---

## Trade-Offs Accepted

### Limited to EIP Classifications

**SIA can only aggregate what EIP classified.**

If EIP misclassifies a review (wrong domain, wrong emotion), SIA inherits that error.

**Accepted because:**
- EIP quality is high (GPT gpt-5.2 with full dictionary injection)
- Canonical validation in ESS catches most EIP errors
- SIA running Claude wouldn't fix EIP errors anyway (it would just re-interpret them)

---

### No Qualitative Insights

**SIA provides metrics, not narratives:**
- "18% of reviews flagged Food Quality concerns" ✓
- "Top pain point: Underseasoned protein (5 mentions)" ✓
- "What this means for the operator..." ✗ (that's HSI's job)

**Accepted because:**
- Dashboard needs quantitative metrics (percentages, counts, trends)
- Qualitative insights already provided by HSI (Behavioral Narrative, Operator Insight)
- Mixing quantitative aggregation with qualitative interpretation = confusing UX

**Clear division of labor:**
- **SIA:** Metrics for dashboard display
- **HSI:** Narratives for operator understanding

---

### Trend Comparison Requires Prior Data

**SIA can't show trends until second run.**

**Week 1:** No prior period → all trend directions = "NEW"

**Week 2:** Can compare to Week 1 → trend directions = "UP" / "DOWN" / "STABLE"

**Accepted because:**
- Inevitable for any trend tracking system
- Dashboard can hide "NEW" trends (display only UP/DOWN/STABLE)
- After 2 weeks of data, problem resolves itself

---

## Long-Term Model-Agnostic Strategy

**SIA is already model-agnostic** (it uses no AI model).

**Future-proof against:**
- OpenAI pricing changes (doesn't use OpenAI)
- Anthropic pricing changes (doesn't use Anthropic)
- Model deprecations (doesn't depend on any model)
- API outages (no API calls)

**If OpenAI raises prices 10×, SIA cost impact: $0 (unchanged).**

**If Anthropic discontinues Claude 4.6, SIA impact: none (doesn't use it).**

**SIA is the most future-proof agent in the pipeline.**

---

## Competitive Differentiation

**Most review analysis SaaS use AI for aggregation:**
- "Our AI summarizes your reviews" = LLM reading all reviews, generating summary
- Token cost scales linearly with review volume
- Variability in summary quality (LLM hallucinations, inconsistent phrasing)

**SubtextCX SIA:**
- Deterministic aggregation (same input = same output)
- Zero token cost (scales infinitely)
- Fast execution (<1 second vs 30+ seconds)
- Reliable trend tracking (numeric comparison, not qualitative interpretation)

**At 10,000 reviews/month:**
- Competitor AI aggregation: ~15M tokens/month just for dashboard metrics
- SubtextCX SIA: 0 tokens/month

**Cost advantage compounds at scale.**

---

## Related Documents

- **HOW Document:** [SCX_SIA_HOW_v3.md](SCX_SIA_HOW_v3.md)
- **Schema:** [SIA_Schema.md](SIA_Schema.md)
- **Changelog:** [SCX_SIA_CHANGELOG.md](SCX_SIA_CHANGELOG.md)
- **EIP Design Rationale:** [../EIP/EIP_Design_Rationale.md](../EIP/EIP_Design_Rationale.md) (full dictionary injection strategy)

---

**End of SIA Design Rationale**
