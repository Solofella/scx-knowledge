# SCX_ESS_HOW_v4.0

**Agent Name:** ESS (Emotional Signal Synthesis)  
**Version:** 4.0  
**Last Updated:** Chat #74 · April 4, 2026  
**Model:** Claude claude-sonnet-4-6  
**Status:** Verified operational - 23 nodes complete

---

## Purpose

ESS receives emotion classifications from EIP and analyzes HOW the guest expresses those emotions. It does not re-classify emotions (EIP already resolved those). Instead, ESS determines Expression Mode (Explicit/Implicit/Masked/Performative/Conflicted/Absent) and Emotional Clarity (Clear/Diffuse/Fragmented/Ambiguous).

**ESS inherits all emotion data from EIP.** Zero dictionary re-query cost.

---

## Input Source

**Upstream Agent:** EIP (Emotion & Intelligence Preprocessor)

**Receives from EIP payload:**
- ALA Record ID (traceability)
- Client ID
- Review Text (original)
- **Enriched Emotion Tag** ✓ (resolved by EIP)
- **Core Emotion** ✓ (resolved by EIP)
- **Need State** ✓ (resolved by EIP)
- **Cognitive Driver** ✓ (resolved by EIP)
- **Enriched Emotion Breakdown JSON** ✓ (all emotions detected by EIP)

**ESS does NOT query Emotion Dictionary.** All emotion data inherited from EIP.

---

## Processing Logic

### Node Flow (23 Nodes Total)

**INIT Nodes (1-3):** Webhook receive from EIP, payload parse, validate EIP fields present

**Canonical Integrity Check (4-8):** Deterministic Code Node
- Verify Core Emotion maps correctly to Enriched Emotion Tag
- Validate Need State matches Emotion Dictionary entry
- Confirm Cognitive Driver aligns with emotion type
- **All validation rules are deterministic (no AI calls)**

**Expression Mode Analysis (9-16):** Claude API call
- System prompt: Analyze HOW emotion is expressed, not WHAT emotion (EIP already resolved that)
- Input: Review text + EIP emotion classifications
- Output: Expression Mode (1 of 6 categories)

**Emotional Clarity Analysis (17-21):** Claude API call
- System prompt: Assess clarity of emotional communication
- Input: Review text + Expression Mode
- Output: Emotional Clarity (1 of 4 categories)

**Narrative Alignment Score (22):** Deterministic calculation
- Compare Expression Mode + Clarity against star rating
- Detect misalignment (e.g., 5-star rating but Masked frustration)

**Output (23):** Write to ESS NocoDB table, pass to HSI webhook

---

## NocoDB Schema

**Table ID:** `m5yektnbtxf8evk`

**Fields:**
- ESS Record ID (auto-increment, primary key)
- ALA Record ID (traceability)
- EIP Record ID (upstream reference)
- Client ID
- **Expression Mode** (ESS computation: Explicit/Implicit/Masked/Performative/Conflicted/Absent)
- **Emotional Clarity** (ESS computation: Clear/Diffuse/Fragmented/Ambiguous)
- **Narrative Alignment Score** (ESS computation: 1-10 scale)
- Core Emotion (inherited from EIP, stored for reference)
- Need State (inherited from EIP, stored for reference)
- Created At (timestamp)

---

## Expression Mode Categories (6 Types)

### Explicit
**Definition:** Guest directly states emotion in clear language

**Examples:**
- "I was frustrated with the long wait"
- "We were delighted by the presentation"
- "The noise level made me uncomfortable"

**Signal:** High interpretability, guest is self-aware and articulate

---

### Implicit
**Definition:** Emotion conveyed through context, not directly stated

**Examples:**
- "The server never checked on us" (implies neglect, frustration)
- "Other tables got their food before us" (implies unfairness)
- "We won't be returning" (implies disappointment without naming it)

**Signal:** Moderate interpretability, requires inference from facts

---

### Masked
**Definition:** Guest uses positive/neutral language to hide negative emotion

**Examples:**
- "Everything was fine" (when 2-star rating reveals dissatisfaction)
- "It was okay" (performative politeness masking disappointment)
- "Not bad" (damning with faint praise)

**Signal:** Low interpretability, requires cross-referencing star rating and context. **This is SubtextCX's competitive differentiator.**

---

### Performative
**Definition:** Emotion expressed for social effect, not genuine internal state

**Examples:**
- "OMG BEST MEAL EVER!!!" (excessive enthusiasm, social media performance)
- "Absolutely unacceptable" (formal complaint language, amplified for effect)
- "Five stars across the board" (formulaic praise without detail)

**Signal:** Guest performing emotion for audience (reviewers, restaurant, social circle)

---

### Conflicted
**Definition:** Multiple contradictory emotions expressed simultaneously

**Examples:**
- "The food was amazing but the service ruined it" (joy + frustration)
- "Great atmosphere, just wish the portions were bigger" (satisfaction + disappointment)
- "Loved the concept, execution needs work" (appreciation + critique)

**Signal:** Mixed experience, requires nuanced response strategy

---

### Absent
**Definition:** No detectable emotional content (purely factual/transactional)

**Examples:**
- "We ordered the salmon. It arrived in 20 minutes."
- "Parking available in rear lot."
- "Reservation confirmed for 7pm."

**Signal:** Informational only, low engagement, possibly automated or compliance-driven

---

## Emotional Clarity Categories (4 Types)

### Clear
**Definition:** Guest's emotional state is unambiguous and well-articulated

**Indicators:**
- Consistent emotion across review
- Specific examples support stated feeling
- Expression Mode aligns with star rating

**Example:** 5-star review, Explicit gratitude, detailed praise → Clear

---

### Diffuse
**Definition:** Emotion present but spread across multiple unfocused themes

**Indicators:**
- Multiple emotions mentioned without hierarchy
- Lack of specific examples
- General positive/negative tone without precision

**Example:** "Everything was good, nice vibe, enjoyed it" → Diffuse positive

---

### Fragmented
**Definition:** Emotional narrative broken or inconsistent

**Indicators:**
- Contradictory statements
- Emotion shifts mid-review without explanation
- Disjointed examples

**Example:** "Great service, terrible food, loved the decor, won't return" → Fragmented

---

### Ambiguous
**Definition:** Emotional state unclear or deliberately obscured

**Indicators:**
- Masked emotion (positive words, negative rating)
- Irony or sarcasm
- Performative language masking true feeling

**Example:** 2-star review: "It was fine" → Ambiguous (masked dissatisfaction)

---

## Narrative Alignment Score (1-10)

**Deterministic calculation based on:**

**High alignment (8-10):**
- Expression Mode matches star rating (Explicit joy + 5 stars)
- Emotional Clarity = Clear
- No contradictions between emotion and facts

**Moderate alignment (5-7):**
- Implicit or Diffuse emotion
- Minor contradictions
- Conflicted emotions with balanced treatment

**Low alignment (1-4):**
- Masked emotion (positive words, low rating)
- Performative exaggeration
- Fragmented or Ambiguous clarity
- Expression Mode conflicts with star rating

**Use case:** BRA uses Narrative Alignment Score to adjust response draft tone. Low alignment = acknowledge complexity, don't assume guest's stated emotion is their true feeling.

---

## Token Budget

**~1,200 tokens per record**

**No dictionary access** (inherits all emotion data from EIP)

**Includes:**
- EIP payload processing
- Claude API call for Expression Mode
- Claude API call for Emotional Clarity
- Deterministic calculations (canonical validation, alignment score)

---

## Key Design Decisions

### Why ESS Doesn't Re-Query Emotion Dictionary

**EIP already resolved:**
- Which emotions are present (Enriched Emotion Tag)
- What those emotions mean (Need State, Cognitive Driver)
- How intense they are (Enriched Emotion Breakdown JSON)

**ESS only analyzes:**
- How those emotions are expressed (Expression Mode)
- How clearly they're communicated (Emotional Clarity)

**Re-querying dictionary would be redundant and expensive** (13K tokens wasted per record).

---

### Why Claude Instead of GPT for ESS

**ESS requires nuanced interpretation of expression style:**
- Detecting masked emotions (guest says "fine" but means "frustrated")
- Identifying performative language (social media amplification)
- Assessing emotional clarity (diffuse vs fragmented)

**Claude claude-sonnet-4-6 excels at:**
- Subtle linguistic analysis
- Context-aware interpretation
- Detecting tone mismatches

**GPT works well for structured classification (EIP's task). Claude works better for interpretive analysis (ESS's task).**

---

## Downstream Handoff

**ESS → HSI:**
- Expression Mode ✓
- Emotional Clarity ✓
- Narrative Alignment Score ✓
- Core Emotion (pass-through from EIP) ✓
- Need State (pass-through from EIP) ✓

**HSI uses ESS output to:**
- Weight behavioral narrative emphasis
- Adjust signal interpretation confidence
- Contextualize pain point severity

---

## Related Documents

- **Changelog:** [SCX_ESS_CHANGELOG.md](SCX_ESS_CHANGELOG.md)
- **Schema:** [ESS_Schema.md](ESS_Schema.md)
- **Upstream Agent:** [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md)
- **Downstream Agent:** [../HSI/SCX_HSI_HOW_v3.md](../HSI/SCX_HSI_HOW_v3.md)

---

## n8n Workflow Details

**Workflow Name:** SCX-ESS  
**Trigger:** Webhook POST from EIP  
**Credentials:** NocoDB xc-token, Anthropic API key

**Critical Rules:**
- Canonical validation runs BEFORE Claude calls (catches EIP output errors)
- Expression Mode and Emotional Clarity are separate Claude calls (not combined)
- Narrative Alignment Score is deterministic Code Node (no AI)
- Claude API manual headers: `anthropic-version:2023-06-01` + `Content-Type:application/json`

---

**End of ESS HOW v4**
