# SCX_EIP_HOW_v4.0

**Agent Name:** EIP (Emotion & Intelligence Preprocessor)  
**Version:** 4.0  
**Last Updated:** Chat #74 · April 4, 2026  
**Model:** GPT gpt-5.2  
**Status:** Verified operational - 26 nodes complete

---

## Purpose

EIP is the **knowledge injection layer** of the SubtextCX pipeline. It receives normalized review text from ALA and performs the critical classification work: injecting the complete Emotion Dictionary (161 entries) and Pain Point Master (336 entries) into every GPT call, then resolving all emotion tags, pain points, need states, and cognitive drivers.

**All downstream agents (ESS, HSI, SIA, BRA, RDA) inherit EIP's classifications.** This architecture pays the dictionary cost once and benefits the entire pipeline.

---

## Input Source

**Upstream Agent:** ALA (Acquisition & Language Agent)

**Receives:**
- ALA Record ID (traceability through entire pipeline)
- Client ID
- Platform (Google, Yelp, OpenTable, etc.)
- Date Posted
- Star Rating (1-5)
- Review Text (original)
- Normalized Text (ALA-processed, clean)
- Language Detected

---

## Processing Logic

### Node Flow (26 Nodes Total)

**INIT Nodes (1-4):** Webhook receive, payload parse, dictionary fetch from NocoDB

**Dictionary Injection (5-8):**
- Query NocoDB Emotion Dictionary table (161 entries, 7 fields each)
- Query NocoDB Pain Point Master table (336 entries, 13 fields each)
- Serialize both into text format for GPT prompt injection
- **Total injected: ~15,350 tokens** (13K emotion dict + 20K pain point master + prompt structure)

**GPT Classification (9-18):**
- Inject full dictionaries into system prompt
- GPT analyzes normalized review text
- Resolves: Enriched Emotion Tag, Core Emotion, Need State, Cognitive Driver
- Resolves: Pain Point Sub-Category, Pain Point Domain, Signal Weight
- Outputs structured JSON with all classifications

**Validation & Output (19-26):**
- Parse GPT JSON response
- Validate canonical mappings (deterministic checks)
- Write to EIP NocoDB table
- Pass to ESS webhook with all resolved fields

---

## NocoDB Schema

**Table ID:** `mhicpnrahaesxmy`

**Fields:**
- EIP Record ID (auto-increment, primary key)
- ALA Record ID (traceability)
- Client ID
- **Enriched Emotion Tag** (resolved from Emotion Dictionary)
- **Core Emotion** (canonical mapping)
- **Need State** (Safety/Belonging/Autonomy/Competence/Fairness/Recognition)
- **Cognitive Driver** (negative driver behind emotion)
- **Enriched Emotion Breakdown** (JSON array of all emotions detected)
- **Pain Point Sub-Category** (resolved from Pain Point Master)
- **Pain Point Domain Confirmed** (1 of 13 domains)
- **Signal Weight** (Low/Medium/High based on Pain Point Master lookup)
- **Enriched Pain Point** (full description from master)
- **Enriched Pain Point Breakdown** (JSON array of all pain points detected)
- **Signal Type** (deterministic classification: T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE)
- Created At (timestamp)

---

## Dictionary Injection Strategy

### Full Injection (Not Filtered)

**EIP injects ALL 161 emotion entries and ALL 336 pain point entries on every record.**

**Why no pre-filter?**
- **Quality priority:** Guarantees complete taxonomy coverage, zero risk of missing correct classification
- **Downstream efficiency:** ESS, HSI, SIA, BRA, RDA all inherit resolved output (no re-query cost)
- **Economic viability:** OpenAI prompt caching provides 90% discount on repeated dictionary content

### OpenAI Prompt Caching

**How it works:**
- **First record in batch:** Dictionary content written to cache (10% cost on 14,750 tokens)
- **Subsequent records:** Dictionary content read from cache (10% cost on 14,750 tokens)
- **Unique content per record:** ~600 tokens (review text + context)

**Effective per-record cost:** ~1,535 tokens (vs 15,350 without caching)

**Cache dependency:** BCA (future) must process batches within OpenAI cache TTL window to maintain savings

---

## Token Budget

**Per record with caching:**
- Dictionary injection: 14,750 tokens (cached at 10% cost = ~1,475 effective)
- Review text + context: ~600 tokens
- **Total: ~2,075 tokens input, ~500 tokens output**
- **Effective cost: ~1,535 tokens with 90% cache hit rate**

**Per record without caching:**
- **15,350 tokens input** (catastrophic at scale)

**At pilot scale (300 reviews/month with caching):**
- EIP layer: 460K tokens/month
- Without caching: 4.6M tokens/month (10× higher)

---

## Downstream Inheritance

**What ESS receives from EIP:**
- Enriched Emotion Tag ✓
- Core Emotion ✓
- Need State ✓
- Cognitive Driver ✓
- **ESS does NOT re-query Emotion Dictionary** (zero incremental cost)

**What HSI receives from EIP:**
- Pain Point Sub-Category ✓
- Pain Point Domain Confirmed ✓
- Signal Weight ✓
- **HSI does NOT re-query Pain Point Master** (zero incremental cost)

**What SIA receives from EIP:**
- Domain ✓
- Signal Type (T-NEG/T-AMB/T-POS) ✓
- Enriched Pain Point Breakdown JSON ✓
- Enriched Emotion Breakdown JSON ✓
- **SIA reads directly from EIP NocoDB table** (zero AI calls, pure JavaScript aggregation)

**Pipeline efficiency:** EIP pays 15,350 token cost once. Downstream agents inherit classifications for free.

---

## Signal Type Classification (Deterministic)

**After GPT resolves classifications, EIP applies deterministic logic:**

**T-NEGATIVE:**
- Star Rating ≤ 3 AND Pain Point Domain in high-severity list
- OR Core Emotion in negative set (Anger, Frustration, Disappointment)

**T-POSITIVE:**
- Star Rating ≥ 4 AND Core Emotion in positive set (Joy, Gratitude, Delight)
- AND Pain Point Domain = "None" OR Signal Weight = "Low"

**T-AMBIGUOUS:**
- Mixed signals (positive emotion + medium pain point)
- OR Star Rating = 3 with neutral emotional tone
- OR conflicting signals (high rating + frustration mentioned)

**This classification travels through entire pipeline** and determines BRA template tier (T1/T2/T3).

---

## Key Design Decisions

### Why Full Dictionary Injection vs Pre-Filter?

**Alternative rejected:** Pre-filter to ~10 entries before GPT call

**Reasons:**
1. **Quality risk:** Pre-filter requires second classification step to select relevant entries
   - If pre-filter wrong, correct taxonomy entry never reaches GPT
   - "Masked emotion" detection would fail (guest says "fine" but means "frustrated")

2. **Marginal token savings:** Pre-filter saves ~13K tokens but requires additional GPT call
   - Pre-filter call: ~1,000 tokens
   - Main call with 10 entries: ~800 tokens
   - **Total: ~1,800 tokens** vs full injection with caching: ~1,535 tokens
   - **Savings: 265 tokens per record** (not worth quality risk)

3. **No downstream benefit:** Pre-filter approach still requires ESS/HSI to either:
   - Trust EIP selection (same as current)
   - OR re-query dictionaries themselves (duplicate cost)

### Why GPT Instead of Claude for EIP?

**OpenAI caching > Anthropic caching for this use case:**
- OpenAI cache TTL: longer session-based cache
- Anthropic cache TTL: 5 minutes (risky for batch processing)
- BCA batch drip interval might exceed 5 min → Anthropic cache expires → full cost repeats

**Model performance:** GPT gpt-5.2 and Claude claude-sonnet-4-6 both excel at structured JSON classification. Cost efficiency drove the decision.

---

## Data Flow
ALA normalized review
↓
EIP Webhook (JSON payload with ALA Record ID)
↓
Query NocoDB: Emotion Dict + Pain Point Master
↓
Serialize dictionaries to text (15,350 tokens)
↓
GPT Prompt: System (dictionaries) + User (review text)
↓
GPT Response: JSON with all classifications
↓
Parse JSON, validate canonical mappings
↓
Deterministic Signal Type classification (T-NEG/T-AMB/T-POS)
↓
Write to EIP NocoDB table
↓
Pass to ESS webhook (all resolved fields in payload)

---

## Related Documents

- **Changelog:** [SCX_EIP_CHANGELOG.md](SCX_EIP_CHANGELOG.md)
- **Schema:** [EIP_Schema.md](EIP_Schema.md)
- **Design Rationale:** [EIP_Design_Rationale.md](EIP_Design_Rationale.md)
- **Upstream Agent:** [../ALA/SCX_ALA_HOW_v4.md](../ALA/SCX_ALA_HOW_v4.md)
- **Downstream Agent:** [../ESS/SCX_ESS_HOW_v4.md](../ESS/SCX_ESS_HOW_v4.md)
- **Emotion Dictionary:** ../../dictionaries/Emotion_Dict_v5.0.md (to be added)
- **Pain Point Master:** ../../dictionaries/Pain_Point_Master_v4.1.md (to be added)

---

## n8n Workflow Details

**Workflow Name:** SCX-EIP  
**Trigger:** Webhook POST from ALA  
**Credentials:** NocoDB xc-token, OpenAI API key

**Critical Rules:**
- NocoDB queries for dictionaries happen in INIT nodes (before GPT call)
- Dictionary serialization: key-by-key (no spread operator)
- GPT response: strip ```json fences before parsing
- Canonical validation: deterministic Code Node after GPT (catches hallucinations)

---

**End of EIP HOW v4**
