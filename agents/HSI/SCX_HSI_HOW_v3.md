# SCX_HSI_HOW_v3.0

**Agent Name:** HSI (Hospitality Signal Intelligence)  
**Version:** 3.0  
**Last Updated:** Chat #74 · April 4, 2026  
**Model:** Claude claude-sonnet-4-6  
**Status:** Verified operational - 30 nodes complete

---

## Purpose

HSI synthesizes pain point classifications from EIP and emotional expression analysis from ESS into actionable behavioral narratives. It does not re-classify pain points (EIP already resolved those). Instead, HSI interprets what the combined emotion + pain point pattern means for guest behavior and operator understanding.

**HSI inherits all pain point data from EIP and all expression analysis from ESS.** Zero dictionary re-query cost.

---

## Input Sources

**Upstream Agent 1:** EIP (Emotion & Intelligence Preprocessor)

**Receives from EIP:**
- **Pain Point Sub-Category** ✓ (resolved by EIP)
- **Pain Point Domain Confirmed** ✓ (resolved by EIP, 1 of 13 domains)
- **Signal Weight** ✓ (resolved by EIP: Low/Medium/High)
- **Enriched Pain Point** ✓ (full description from Pain Point Master)
- **Enriched Pain Point Breakdown JSON** ✓ (all pain points detected)

**Upstream Agent 2:** ESS (Emotional Signal Synthesis)

**Receives from ESS:**
- **Expression Mode** ✓ (Explicit/Implicit/Masked/Performative/Conflicted/Absent)
- **Emotional Clarity** ✓ (Clear/Diffuse/Fragmented/Ambiguous)
- **Narrative Alignment Score** ✓ (1-10 scale)
- Core Emotion (pass-through from EIP)
- Need State (pass-through from EIP)

**HSI does NOT query Pain Point Master.** All pain point data inherited from EIP.

---

## Processing Logic

### Node Flow (30 Nodes Total)

**INIT Nodes (1-4):** Webhook receive from ESS, payload parse, validate EIP+ESS fields present

**Signal Severity Assessment (S4A1-S4A6):** Deterministic Code Nodes
- S4A1: Base severity from Signal Weight (EIP inheritance)
- S4A2: Expression Mode modifier (Masked emotion increases severity)
- S4A3: Emotional Clarity modifier (Ambiguous increases severity)
- S4A4: Star Rating cross-check (2-star + positive words = severity increase)
- S4A5: Domain-specific severity rules (Food Quality pain points weigh higher)
- S4A6: Final severity score (1-10 scale)

**Behavioral Narrative Synthesis (7-22):** Claude API call
- System prompt: Synthesize emotion + pain point into behavioral interpretation
- Input: All EIP/ESS data + severity score
- Output: Hospitality Signal narrative (what this pattern means for guest experience)

**Hospitality Signal ID Generation (23-26):** Deterministic
- Format: `HSI-[Domain]-[Severity]-[YYYYMMDD]-[Sequence]`
- Example: `HSI-ServiceQuality-High-20260417-003`

**BRA Status Field Update (27-28):**
- Set initial status: "Pending Template Selection"
- BRA will update this field when draft is generated

**Output (29-30):** Write to HSI NocoDB table, pass to SIA and BRA webhooks

---

## NocoDB Schema

**Table ID:** `mb8nv8t3nk6xzed`

**Fields:**
- HSI Record ID (auto-increment, primary key)
- ALA Record ID (traceability)
- EIP Record ID (upstream reference)
- ESS Record ID (upstream reference)
- Client ID
- **Hospitality Signal ID** (HSI computation: unique identifier)
- **Signal Severity Score** (HSI computation: 1-10, deterministic from S4A1-S4A6)
- **Behavioral Narrative** (HSI computation: Claude synthesis)
- **Operator Insight** (HSI computation: what operator should understand, not what to do)
- Pain Point Domain (inherited from EIP, stored for reference)
- Signal Weight (inherited from EIP, stored for reference)
- Expression Mode (inherited from ESS, stored for reference)
- Emotional Clarity (inherited from ESS, stored for reference)
- **BRA Status** (field ID: `cl1250sz39sm45l`) - Updated by BRA when draft generated
- Created At (timestamp)

---

## Signal Severity Assessment (S4A1-S4A6)

**Deterministic calculation across 6 steps:**

### S4A1: Base Severity (From Signal Weight)

```javascript
const baseSeverity = {
  'Low': 3,
  'Medium': 6,
  'High': 9
};
let severity = baseSeverity[signalWeight]; // From EIP
```

### S4A2: Expression Mode Modifier

```javascript
if (expressionMode === 'Masked') {
  severity += 1; // Guest hiding dissatisfaction = more severe
}
if (expressionMode === 'Conflicted') {
  severity += 0.5; // Mixed signals = moderately more severe
}
if (expressionMode === 'Performative') {
  severity -= 0.5; // Social amplification = less operationally severe
}
```

### S4A3: Emotional Clarity Modifier

```javascript
if (emotionalClarity === 'Ambiguous' || emotionalClarity === 'Fragmented') {
  severity += 1; // Unclear communication = harder to address
}
if (emotionalClarity === 'Clear') {
  severity += 0; // No penalty or bonus for clarity
}
```

### S4A4: Star Rating Cross-Check

```javascript
if (starRating <= 2 && expressionMode === 'Masked') {
  severity += 1; // Low rating + masked emotion = critical signal
}
if (starRating >= 4 && signalWeight === 'High') {
  severity -= 1; // Positive rating softens high pain point
}
```

### S4A5: Domain-Specific Rules

```javascript
const highPriorityDomains = ['Food Quality', 'Service Quality', 'Cleanliness'];
if (highPriorityDomains.includes(domain)) {
  severity += 0.5; // Core hospitality domains weigh higher
}
```

### S4A6: Final Score (Capped at 1-10)

```javascript
severity = Math.min(10, Math.max(1, Math.round(severity)));
return severity; // Integer 1-10
```

**Use case:** SIA uses severity score for trend analysis. BRA uses severity for template tier selection.

---

## Behavioral Narrative Synthesis (Claude)

**Computed by:** Claude claude-sonnet-4-6

**Input:**
- Review text (original)
- EIP pain point classifications (Domain, Sub-Category, Signal Weight)
- ESS expression analysis (Mode, Clarity, Alignment Score)
- HSI severity score (from S4A1-S4A6)
- Core Emotion, Need State (from EIP)

**Prompt focus:** "Synthesize emotion + pain point into behavioral interpretation. Describe what this pattern means for guest experience. Do NOT prescribe operational actions."

**Output:** Two fields

### Field 1: Behavioral Narrative (Guest-Facing Context)

**What it describes:**
- How emotion + pain point impacted guest experience
- What unmet need drove the signal
- Behavioral pattern (e.g., "Guest felt loss of control during service delay")

**Example:**
Guest expressed frustration (Masked expression: said "fine" despite 2-star rating)
regarding wait time during weekend dinner service. The delay violated their Need State
for Autonomy — they lost control over their schedule for the evening. This pattern
suggests the guest values predictability and may not return without confidence in
consistent service timing.

**Governance constraint:** Describe behavior and impact, never prescribe fix.

---

### Field 2: Operator Insight (Internal Understanding)

**What it describes:**
- What the signal means for operator understanding
- Context that helps interpret guest behavior
- Pattern recognition (e.g., "Weekend capacity signal")

**Example:**
This signal indicates a capacity management pattern during weekend dinner service.
The guest's masked expression (saying "fine" rather than directly stating frustration)
suggests they prioritize politeness over directness. Operator should recognize that
positive-sounding language from 2-star reviews may hide deeper dissatisfaction.

**Governance constraint:** Interpret signal meaning, never recommend action.

**Bad example (violates governance):**
Operator should add more servers during weekend dinner rush. ← PRESCRIPTIVE, NOT ALLOWED

---

## Hospitality Signal ID Format

**Pattern:** `HSI-[Domain]-[Severity]-[YYYYMMDD]-[Sequence]`

**Components:**
- **HSI:** Prefix (Hospitality Signal Intelligence)
- **Domain:** Pain Point Domain (e.g., ServiceQuality, FoodQuality, Ambiance)
- **Severity:** Low/Medium/High (based on severity score 1-3=Low, 4-7=Medium, 8-10=High)
- **YYYYMMDD:** Date (e.g., 20260417)
- **Sequence:** Incremental counter per day (001, 002, 003...)

**Examples:**
- `HSI-ServiceQuality-High-20260417-003`
- `HSI-FoodQuality-Medium-20260417-012`
- `HSI-WaitTime-Low-20260417-001`

**Use case:** Unique identifier for cross-referencing in SIA aggregation, BRA template selection, RDA draft generation, MRA reporting.

---

## Token Budget

**~1,800 tokens per record**

**Breakdown:**
- Behavioral Narrative synthesis: ~900 tokens (Claude API call)
- Operator Insight synthesis: ~900 tokens (same Claude call, two outputs)
- Signal Severity Assessment: 0 tokens (deterministic S4A1-S4A6)
- Signal ID generation: 0 tokens (deterministic Code Node)

**Zero dictionary cost** (all pain point data inherited from EIP)

---

## Key Design Decisions

### Why HSI Doesn't Re-Query Pain Point Master

**EIP already resolved:**
- Which pain points are present (Enriched Pain Point)
- What domain they belong to (1 of 13)
- How severe they are (Signal Weight: Low/Medium/High)

**HSI only interprets:**
- What the pain point + emotion combination means behaviorally
- How expression mode affects severity
- What the operator should understand (not what to do)

**Re-querying Pain Point Master would be redundant** (20K tokens wasted per record).

---

### Why Claude Instead of GPT for HSI

**HSI requires narrative synthesis and behavioral interpretation:**
- Combining emotion + pain point into coherent story
- Detecting nuanced patterns (masked frustration + wait time = capacity signal)
- Writing operator-facing insights (governance-constrained, interpretive)

**Claude claude-sonnet-4-6 excels at:**
- Long-form narrative generation
- Context-aware synthesis
- Governance-compliant prose (detect/interpret, never prescribe)

**GPT works well for structured classification (EIP). Claude works better for narrative synthesis (HSI).**

---

## Downstream Handoff

**HSI → SIA:**
- Hospitality Signal ID ✓
- Signal Severity Score ✓
- Domain ✓
- **SIA reads from HSI NocoDB table** (no webhook, pure JS aggregation)

**HSI → BRA:**
- Hospitality Signal ID ✓
- Signal Severity Score ✓
- Behavioral Narrative ✓
- Domain ✓
- **BRA uses this to select template tier** (T1/T2/T3) and populate draft context

---

## Related Documents

- **Changelog:** [SCX_HSI_CHANGELOG.md](SCX_HSI_CHANGELOG.md)
- **Schema:** [HSI_Schema.md](HSI_Schema.md)
- **Upstream Agent (EIP):** [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md)
- **Upstream Agent (ESS):** [../ESS/SCX_ESS_HOW_v4.md](../ESS/SCX_ESS_HOW_v4.md)
- **Downstream Agent (SIA):** [../SIA/SCX_SIA_HOW_v3.md](../SIA/SCX_SIA_HOW_v3.md)
- **Downstream Agent (BRA):** [../BRA/SCX_BRA_HOW_v3.2.md](../BRA/SCX_BRA_HOW_v3.2.md)

---

## n8n Workflow Details

**Workflow Name:** SCX-HSI  
**Trigger:** Webhook POST from ESS  
**Credentials:** NocoDB xc-token, Anthropic API key

**Critical Rules:**
- S4A1-S4A6 severity assessment runs BEFORE Claude call (deterministic)
- Signal ID generation uses date + sequence counter (unique per day)
- BRA Status field update happens in HSI (BRA will PATCH this field later)
- Claude API manual headers: `anthropic-version:2023-06-01` + `Content-Type:application/json`
- Operator Insight must never include prescriptive language (governance check in prompt)

---

**End of HSI HOW v3**
