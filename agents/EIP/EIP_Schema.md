# EIP Schema

**NocoDB Table ID:** `mhicpnrahaesxmy`  
**Base ID:** `pq249fix22t3ofv`

---

## Table Fields

| Field Name | Type | Source | Purpose |
|------------|------|--------|---------|
| EIP Record ID | Auto-increment | System | Primary key |
| ALA Record ID | Number | ALA pass-through | Traceability through entire pipeline |
| Client ID | SingleLineText | ALA pass-through | PAK-001, EDO-001, etc. |
| **Enriched Emotion Tag** | SingleLineText | **Emotion Dictionary lookup** | Primary emotion detected (e.g., "Gratitude", "Frustration") |
| **Core Emotion** | SingleLineText | **Canonical mapping** | Normalized emotion category |
| **Need State** | SingleSelect | **Emotion Dictionary lookup** | Safety/Belonging/Autonomy/Competence/Fairness/Recognition |
| **Cognitive Driver** | LongText | **Emotion Dictionary lookup** | Negative driver behind emotion |
| **Enriched Emotion Breakdown** | JSON | **Emotion Dictionary lookups** | Array of all emotions detected with intensity scores |
| **Pain Point Sub-Category** | SingleLineText | **Pain Point Master lookup** | Specific pain point identified |
| **Pain Point Domain Confirmed** | SingleSelect | **Pain Point Master lookup** | 1 of 13 domains (Food Quality, Service Quality, etc.) |
| **Signal Weight** | SingleSelect | **Pain Point Master lookup** | Low/Medium/High severity |
| **Enriched Pain Point** | LongText | **Pain Point Master lookup** | Full description from master |
| **Enriched Pain Point Breakdown** | JSON | **Pain Point Master lookups** | Array of all pain points detected |
| **Signal Type** | SingleSelect | **Deterministic classification** | T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE |
| Created At | DateTime | System | Record creation timestamp |

---

## Relationships

**Upstream:** ALA provides normalized review text + metadata

**Downstream:**
- **ESS** inherits: Enriched Emotion Tag, Core Emotion, Need State, Cognitive Driver
- **HSI** inherits: Pain Point Sub-Category, Domain, Signal Weight, Enriched Pain Point
- **SIA** reads EIP NocoDB table directly (no webhook, pure aggregation)

---

## Knowledge Injection

### Emotion Dictionary (161 entries)

**NocoDB Source Table ID:** `mrrscb955j1d2i7`

**Fields injected per entry:**
- Primary_Emotion
- Need_State
- Intensity_Range
- Hospitality_Context
- Common_Expressions
- Masked_Forms
- Negative_Cognitive_Driver

**Token cost:** ~13,000 tokens (all 161 entries × 7 fields)

**Caching:** OpenAI 90% discount after first record in batch

---

### Pain Point Master (336 entries)

**NocoDB Source Table ID:** `meavqh37mdqgl4d`

**Fields injected per entry:**
- Pain_Point_ID (PP-001 through PP-336)
- Domain (13 total: Food Quality, Service Quality, Ambiance, etc.)
- Sub_Category
- Description
- Operational_Signal
- Emotional_Signal
- Signal_Weight (Low/Medium/High)
- Sample_Expression
- [5 additional fields]

**Token cost:** ~20,000 tokens (all 336 entries × 13 fields)

**Caching:** OpenAI 90% discount after first record in batch

---

## Signal Type Classification (Deterministic)

**Runs in Code Node after GPT classification:**

### T-NEGATIVE
```javascript
if (starRating <= 3 && highSeverityDomains.includes(domain)) {
  return 'T-NEGATIVE';
}
if (['Anger', 'Frustration', 'Disappointment'].includes(coreEmotion)) {
  return 'T-NEGATIVE';
}
```

### T-POSITIVE
```javascript
if (starRating >= 4 && positiveEmotions.includes(coreEmotion)) {
  if (domain === 'None' || signalWeight === 'Low') {
    return 'T-POSITIVE';
  }
}
```

### T-AMBIGUOUS
```javascript
// Mixed signals: positive emotion + medium pain point
// OR star rating 3 with neutral tone
// OR conflicting signals
return 'T-AMBIGUOUS';
```

**This classification determines BRA template tier** (T1/T2/T3 response draft strategy)

---

## Token Economics

### Per-Record Cost (With Caching)

**First record in batch:**
- Dictionary injection: 15,350 tokens (write to cache at 10% cost = ~1,535 tokens)
- Review text + context: ~600 tokens
- GPT output: ~500 tokens
- **Total first record: ~2,635 tokens**

**Subsequent records in batch:**
- Dictionary injection: 15,350 tokens (read from cache at 10% cost = ~1,535 tokens)
- Review text + context: ~600 tokens
- GPT output: ~500 tokens
- **Total subsequent records: ~2,635 tokens**

**Effective average:** ~1,535 tokens input cost per record (with 90% cache hit rate on dictionary content)

---

### Per-Record Cost (Without Caching)

**Every record:**
- Dictionary injection: 15,350 tokens (full cost)
- Review text + context: ~600 tokens
- GPT output: ~500 tokens
- **Total: ~16,450 tokens per record**

**At 300 reviews/month:**
- With caching: 460K tokens/month
- Without caching: 4.6M tokens/month
- **Cache saves 4.14M tokens/month (90% reduction)**

---

## Downstream Zero-Cost Inheritance

**ESS receives from EIP payload:**
- Enriched Emotion Tag ✓
- Core Emotion ✓
- Need State ✓
- Cognitive Driver ✓
- **ESS token budget:** ~1,200 (Expression Mode analysis only, NO dictionary re-query)

**HSI receives from EIP payload:**
- Pain Point Sub-Category ✓
- Pain Point Domain Confirmed ✓
- Signal Weight ✓
- Enriched Pain Point ✓
- **HSI token budget:** ~1,800 (Behavioral narrative only, NO master re-query)

**SIA reads from EIP NocoDB table:**
- Domain ✓
- Signal Type ✓
- Enriched breakdowns (JSON) ✓
- **SIA token budget:** 0 (pure JavaScript aggregation, no AI calls)

**Total downstream benefit:** 3 agents inherit EIP's ~15,350 token investment with zero incremental dictionary cost

---

## Data Flow
ALA normalized review → EIP Webhook
↓
INIT-2: Query Emotion Dictionary (161 entries from NocoDB)
↓
INIT-3: Query Pain Point Master (336 entries from NocoDB)
↓
INIT-4: Serialize to text (15,350 tokens total)
↓
Node 9: GPT API call
System Prompt: Full dictionaries injected
User Prompt: Normalized review text
↓
Node 10-15: Parse GPT JSON response
Extract: emotions, pain points, need states, drivers
↓
Node 16: Deterministic Signal Type classification
Apply rules → T-NEG / T-AMB / T-POS
↓
Node 17-24: Validation & canonical mapping checks
↓
Node 25: Write to EIP NocoDB table
↓
Node 26: Pass to ESS webhook
Payload: All resolved fields (emotions, pain points, signal type)

---

## Related Documents

- **HOW Document:** [SCX_EIP_HOW_v4.md](SCX_EIP_HOW_v4.md)
- **Changelog:** [SCX_EIP_CHANGELOG.md](SCX_EIP_CHANGELOG.md)
- **Design Rationale:** [EIP_Design_Rationale.md](EIP_Design_Rationale.md)
- **Emotion Dictionary:** [../../dictionaries/Emotion_Dict_v5.0.md](../../dictionaries/Emotion_Dict_v5.0.md) (to be added)
- **Pain Point Master:** [../../dictionaries/Pain_Point_Master_v4.1.md](../../dictionaries/Pain_Point_Master_v4.1.md) (to be added)

---

**End of EIP Schema**
