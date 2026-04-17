# ESS Schema

**NocoDB Table ID:** `m5yektnbtxf8evk`  
**Base ID:** `pq249fix22t3ofv`

---

## Table Fields

| Field Name | Type | Source | Purpose |
|------------|------|--------|---------|
| ESS Record ID | Auto-increment | System | Primary key |
| ALA Record ID | Number | Pass-through | Traceability through entire pipeline |
| EIP Record ID | Number | Pass-through | Upstream reference |
| Client ID | SingleLineText | Pass-through | PAK-001, EDO-001, etc. |
| **Expression Mode** | SingleSelect | **ESS computation (Claude)** | How emotion is expressed: Explicit/Implicit/Masked/Performative/Conflicted/Absent |
| **Emotional Clarity** | SingleSelect | **ESS computation (Claude)** | Clarity of emotional communication: Clear/Diffuse/Fragmented/Ambiguous |
| **Narrative Alignment Score** | Number | **ESS computation (deterministic)** | 1-10 scale: alignment between expression and star rating |
| Core Emotion | SingleLineText | Inherited from EIP | Pass-through for downstream reference |
| Need State | SingleSelect | Inherited from EIP | Pass-through for downstream reference |
| Created At | DateTime | System | Record creation timestamp |

---

## Relationships

**Upstream:** EIP provides all emotion classifications

**Downstream:** HSI receives Expression Mode, Emotional Clarity, Narrative Alignment Score

---

## Field Computation Details

### Expression Mode (Claude Analysis)

**Computed by:** Claude claude-sonnet-4-6

**Input:**
- Review text (original)
- EIP emotion classifications (Enriched Emotion Tag, Core Emotion, Need State)

**Prompt focus:** "Analyze HOW this emotion is expressed, not WHAT emotion it is (EIP already resolved that)"

**Output:** One of 6 categories

**Categories:**

| Mode | Definition | Example |
|------|------------|---------|
| **Explicit** | Direct emotional statement | "I was frustrated by the wait" |
| **Implicit** | Emotion conveyed through context | "Server never checked on us" (implies neglect) |
| **Masked** | Positive/neutral words hiding negative emotion | "It was fine" (2-star rating = masked dissatisfaction) |
| **Performative** | Social effect, not genuine state | "OMG BEST MEAL EVER!!!" (amplified for social media) |
| **Conflicted** | Contradictory emotions | "Food amazing, service ruined it" |
| **Absent** | No emotional content | "We ordered salmon. It arrived." (purely factual) |

**Token cost:** ~600 tokens (review text + EIP context, no dictionary)

---

### Emotional Clarity (Claude Analysis)

**Computed by:** Claude claude-sonnet-4-6

**Input:**
- Review text (original)
- Expression Mode (from previous computation)
- EIP emotion classifications

**Prompt focus:** "Assess clarity of emotional communication"

**Output:** One of 4 categories

**Categories:**

| Clarity | Definition | Indicators |
|---------|------------|------------|
| **Clear** | Unambiguous, well-articulated | Consistent emotion, specific examples, mode aligns with rating |
| **Diffuse** | Present but unfocused | Multiple emotions without hierarchy, general tone, no precision |
| **Fragmented** | Broken or inconsistent | Contradictory statements, emotion shifts, disjointed examples |
| **Ambiguous** | Unclear or obscured | Masked emotion, irony/sarcasm, performative masking |

**Token cost:** ~600 tokens (review text + Expression Mode context)

---

### Narrative Alignment Score (Deterministic)

**Computed by:** Code Node (no AI)

**Input:**
- Expression Mode
- Emotional Clarity
- Star Rating (from ALA/EIP payload)
- Core Emotion

**Logic:**

```javascript
// High alignment (8-10)
if (expressionMode === 'Explicit' && clarity === 'Clear') {
  if (matchesStarRating(coreEmotion, starRating)) {
    return 9;
  }
}

// Moderate alignment (5-7)
if (expressionMode === 'Implicit' || clarity === 'Diffuse') {
  return 6;
}

if (expressionMode === 'Conflicted') {
  return 5; // Mixed emotions = moderate alignment
}

// Low alignment (1-4)
if (expressionMode === 'Masked') {
  return 3; // Positive words, negative rating = low alignment
}

if (clarity === 'Ambiguous' || clarity === 'Fragmented') {
  return 2;
}

if (expressionMode === 'Performative') {
  return 4; // Social performance = moderate-low alignment
}

if (expressionMode === 'Absent') {
  return 5; // No emotion = neutral alignment
}
```

**Use case:** BRA uses this score to adjust response draft strategy. Low alignment = acknowledge complexity, don't assume stated emotion is true feeling.

---

## Token Budget

**Total per record:** ~1,200 tokens

**Breakdown:**
- Expression Mode analysis: ~600 tokens
- Emotional Clarity analysis: ~600 tokens
- Canonical validation: 0 tokens (deterministic Code Node)
- Narrative Alignment Score: 0 tokens (deterministic Code Node)

**Zero dictionary cost** (all emotion data inherited from EIP via webhook payload)

---

## Inherited Data from EIP

**ESS does NOT query these tables:**
- Emotion Dictionary (NocoDB table `mrrscb955j1d2i7`) ✗
- Pain Point Master (NocoDB table `meavqh37mdqgl4d`) ✗

**ESS receives via webhook payload:**
- Enriched Emotion Tag ✓
- Core Emotion ✓
- Need State ✓
- Cognitive Driver ✓
- Enriched Emotion Breakdown JSON ✓

**Token savings:** 13,000 tokens per record (avoided Emotion Dictionary re-query)

---

## Canonical Integrity Check (Deterministic)

**Runs in Code Node before Claude calls:**

**Validates:**
1. Core Emotion maps correctly to Enriched Emotion Tag
   - Example: Core Emotion "Gratitude" must map to Enriched Tag "Gratitude" or "Appreciation"
   
2. Need State matches Emotion Dictionary entry
   - Example: "Frustration" must have Need State "Autonomy" or "Competence"

3. Cognitive Driver aligns with emotion type
   - Example: Negative emotion must have negative cognitive driver

**If validation fails:**
- Log error to ESS Error Log field
- Flag record for manual review
- Continue pipeline (don't block processing)

**Purpose:** Catch EIP output errors or GPT hallucinations before downstream agents inherit bad data

---

## Data Flow
EIP webhook → ESS
↓
INIT-1: Receive payload (EIP emotion classifications)
↓
INIT-2: Parse payload, validate required fields present
↓
Node 4-8: Canonical Integrity Check (deterministic)
Validate: Core Emotion ↔ Need State ↔ Cognitive Driver
↓
Node 9-16: Expression Mode Analysis (Claude)
Input: Review text + EIP emotion data
Output: Explicit/Implicit/Masked/Performative/Conflicted/Absent
↓
Node 17-21: Emotional Clarity Analysis (Claude)
Input: Review text + Expression Mode
Output: Clear/Diffuse/Fragmented/Ambiguous
↓
Node 22: Narrative Alignment Score (deterministic)
Calculate: 1-10 based on mode + clarity + star rating
↓
Node 23: Write to ESS NocoDB table
Pass to HSI webhook (Expression Mode, Clarity, Alignment Score)

---

## Competitive Differentiation

**Masked Emotion Detection = SubtextCX's core value:**

**Example scenario:**
- Guest leaves 2-star review
- Review text: "Everything was fine"
- Star rating conflicts with language

**ESS detects:**
- Expression Mode: **Masked** (positive words hiding dissatisfaction)
- Emotional Clarity: **Ambiguous** (deliberately obscured)
- Narrative Alignment Score: **3** (low alignment)

**Downstream impact:**
- HSI interprets "fine" as signal of unmet Need State (likely Autonomy or Fairness)
- BRA selects T3 response template (acknowledges complexity, doesn't assume guest is satisfied)
- RDA drafts response that addresses underlying dissatisfaction (not surface-level "glad you enjoyed")

**No B2B SaaS competitor has shipped dual-instrument masked emotion detection at review-analysis level** (as of March 2026).

---

## Related Documents

- **HOW Document:** [SCX_ESS_HOW_v4.md](SCX_ESS_HOW_v4.md)
- **Changelog:** [SCX_ESS_CHANGELOG.md](SCX_ESS_CHANGELOG.md)
- **Upstream Agent:** [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md)
- **Downstream Agent:** [../HSI/SCX_HSI_HOW_v3.md](../HSI/SCX_HSI_HOW_v3.md)

---

**End of ESS Schema**
