# HSI Schema

**NocoDB Table ID:** `mb8nv8t3nk6xzed`  
**Base ID:** `pq249fix22t3ofv`

---

## Table Fields

| Field Name | Field ID | Type | Source | Purpose |
|------------|----------|------|--------|---------|
| HSI Record ID | (auto) | Auto-increment | System | Primary key |
| ALA Record ID | | Number | Pass-through | Traceability through entire pipeline |
| EIP Record ID | | Number | Pass-through | Upstream reference |
| ESS Record ID | | Number | Pass-through | Upstream reference |
| Client ID | | SingleLineText | Pass-through | PAK-001, EDO-001, etc. |
| **Hospitality Signal ID** | | SingleLineText | **HSI computation (deterministic)** | Unique identifier: HSI-[Domain]-[Severity]-[YYYYMMDD]-[Seq] |
| **Signal Severity Score** | | Number | **HSI computation (S4A1-S4A6 deterministic)** | 1-10 scale severity assessment |
| **Behavioral Narrative** | | LongText | **HSI computation (Claude)** | Guest-facing context: how emotion + pain point impacted experience |
| **Operator Insight** | | LongText | **HSI computation (Claude)** | Internal understanding: what signal means, pattern recognition |
| Pain Point Domain | | SingleSelect | Inherited from EIP | 1 of 13 domains (Food Quality, Service Quality, etc.) |
| Signal Weight | | SingleSelect | Inherited from EIP | Low/Medium/High (from Pain Point Master lookup) |
| Expression Mode | | SingleSelect | Inherited from ESS | Explicit/Implicit/Masked/Performative/Conflicted/Absent |
| Emotional Clarity | | SingleSelect | Inherited from ESS | Clear/Diffuse/Fragmented/Ambiguous |
| **BRA Status** | **cl1250sz39sm45l** | SingleSelect | **Updated by BRA** | Pending/T1-Generated/T2-Generated/T3-Generated/Error |
| Created At | (auto) | DateTime | System | Record creation timestamp |

---

## Relationships

**Upstream 1:** EIP provides pain point classifications

**Upstream 2:** ESS provides expression analysis

**Downstream 1:** SIA reads HSI NocoDB table directly (no webhook)

**Downstream 2:** BRA receives HSI webhook, generates response draft, updates BRA Status field

---

## Field Computation Details

### Hospitality Signal ID (Deterministic)

**Format:** `HSI-[Domain]-[Severity]-[YYYYMMDD]-[Sequence]`

**Component rules:**

| Component | Source | Format | Example |
|-----------|--------|--------|---------|
| Prefix | Fixed | HSI | HSI |
| Domain | EIP Pain Point Domain | CamelCase, no spaces | ServiceQuality |
| Severity | HSI Severity Score | Low (1-3), Medium (4-7), High (8-10) | High |
| Date | System | YYYYMMDD | 20260417 |
| Sequence | Daily counter | 001, 002, 003... | 003 |

**Example full IDs:**
- `HSI-ServiceQuality-High-20260417-003`
- `HSI-FoodQuality-Medium-20260417-012`
- `HSI-WaitTime-Low-20260417-001`
- `HSI-Ambiance-Medium-20260418-001` (new day = sequence resets)

**Generation logic (Code Node):**

```javascript
const domain = items[0].json.painPointDomain.replace(/\s/g, ''); // Remove spaces
const severityLabel = severityScore <= 3 ? 'Low' : severityScore <= 7 ? 'Medium' : 'High';
const dateString = new Date().toISOString().slice(0,10).replace(/-/g, ''); // YYYYMMDD

// Query HSI table for today's count (same domain + date)
const existingToday = await nocodb.query(`date = ${dateString} AND domain = ${domain}`);
const sequence = String(existingToday.length + 1).padStart(3, '0'); // 001, 002, etc.

const hospitalitySignalID = `HSI-${domain}-${severityLabel}-${dateString}-${sequence}`;
```

**Use case:** Unique identifier for cross-referencing throughout pipeline and in MRA reports

---

### Signal Severity Score (S4A1-S4A6 Deterministic)

**Computed across 6 steps in Code Nodes before Claude call:**

#### S4A1: Base Severity

```javascript
const baseSeverity = {
  'Low': 3,
  'Medium': 6,
  'High': 9
};
let severity = baseSeverity[signalWeight]; // From EIP
```

#### S4A2: Expression Mode Modifier

```javascript
const expressionModifiers = {
  'Masked': +1,        // Hiding dissatisfaction = more severe
  'Conflicted': +0.5,  // Mixed signals = moderately more severe
  'Performative': -0.5, // Social amplification = less operationally severe
  'Explicit': 0,
  'Implicit': 0,
  'Absent': 0
};
severity += expressionModifiers[expressionMode] || 0;
```

#### S4A3: Emotional Clarity Modifier

```javascript
const clarityModifiers = {
  'Ambiguous': +1,    // Unclear = harder to address
  'Fragmented': +1,   // Broken narrative = harder to address
  'Diffuse': +0.5,    // Unfocused = moderately harder
  'Clear': 0
};
severity += clarityModifiers[emotionalClarity] || 0;
```

#### S4A4: Star Rating Cross-Check

```javascript
if (starRating <= 2 && expressionMode === 'Masked') {
  severity += 1; // Low rating + masked emotion = critical signal
}
if (starRating >= 4 && signalWeight === 'High') {
  severity -= 1; // Positive rating softens high pain point
}
```

#### S4A5: Domain-Specific Rules

```javascript
const highPriorityDomains = ['Food Quality', 'Service Quality', 'Cleanliness'];
if (highPriorityDomains.includes(painPointDomain)) {
  severity += 0.5; // Core hospitality domains weigh higher
}
```

#### S4A6: Final Score (Capped)

```javascript
severity = Math.min(10, Math.max(1, Math.round(severity)));
return severity; // Integer 1-10
```

**Why deterministic?**
- Consistent severity scoring without AI variability
- Reproducible for audit trail
- Fast computation (no API call)

**Use case:**
- SIA uses severity score for trend analysis
- BRA uses severity for template tier selection (T1 vs T2 vs T3)
- MRA includes severity in intelligence briefs

---

### Behavioral Narrative (Claude Synthesis)

**Computed by:** Claude claude-sonnet-4-6

**Input:**
- Review text (original)
- EIP pain point data (Domain, Sub-Category, Signal Weight, Enriched Pain Point)
- ESS expression data (Mode, Clarity, Alignment Score)
- HSI severity score (from S4A1-S4A6)
- Core Emotion, Need State (from EIP pass-through)

**System prompt excerpt:**
Synthesize the emotion and pain point into a behavioral interpretation.
Describe what this pattern means for the guest's experience.
CRITICAL GOVERNANCE RULE:

Describe behavior and impact ONLY
NEVER prescribe operational actions
NEVER recommend fixes or changes
Focus on guest psychology and experience interpretation

Output format:

Behavioral Narrative (150-250 words)


**Example output:**
Guest expressed frustration (Masked expression: said "fine" despite 2-star rating)
regarding wait time during weekend dinner service. The delay violated their Need State
for Autonomy — they lost control over their schedule for the evening.
The masked expression (positive language hiding negative emotion) suggests this guest
values politeness and may avoid direct confrontation. The 2-star rating reveals the
true impact: the wait time was not acceptable, despite the polite phrasing.
This behavioral pattern indicates the guest prioritizes predictability and control
over their dining timeline. The Service Quality pain point (wait time) combined with
Autonomy violation suggests future visit likelihood is low unless confidence in
consistent service timing is restored.

**Token cost:** ~900 tokens (narrative generation)

---

### Operator Insight (Claude Synthesis)

**Computed by:** Same Claude API call as Behavioral Narrative (dual output)

**System prompt excerpt:**
Provide operator insight: what should the operator UNDERSTAND about this signal?
CRITICAL GOVERNANCE RULE:

Describe signal MEANING and pattern recognition ONLY
NEVER recommend specific actions ("add servers", "change menu", etc.)
Focus on helping operator interpret guest behavior

Output format:

Operator Insight (100-150 words)


**Example output:**
This signal indicates a capacity management pattern during weekend dinner service.
The guest's masked expression (saying "fine" rather than directly stating frustration)
suggests they prioritize politeness over directness.
Operator should recognize that positive-sounding language from 2-star reviews may
hide deeper dissatisfaction. The Autonomy Need State violation means the guest felt
loss of control — a critical psychological factor in hospitality.
This pattern (wait time + masked frustration + low rating) is a leading indicator
of guest churn. The guest may not return without evidence that service timing is
reliable during high-traffic periods. Their polite phrasing should not be mistaken
for satisfaction.

**Token cost:** ~900 tokens (insight generation, same API call)

**Total Claude cost per record:** ~1,800 tokens (both narratives in single call)

---

## Token Budget

**Total per record:** ~1,800 tokens

**Breakdown:**
- S4A1-S4A6 severity assessment: 0 tokens (deterministic)
- Signal ID generation: 0 tokens (deterministic)
- Behavioral Narrative + Operator Insight: ~1,800 tokens (single Claude API call, dual output)

**Zero dictionary cost** (all pain point data inherited from EIP)

---

## Inherited Data from EIP

**HSI does NOT query these tables:**
- Pain Point Master (NocoDB table `meavqh37mdqgl4d`) ✗

**HSI receives via webhook payload from EIP:**
- Pain Point Sub-Category ✓
- Pain Point Domain Confirmed ✓
- Signal Weight ✓
- Enriched Pain Point ✓
- Enriched Pain Point Breakdown JSON ✓

**Token savings:** 20,000 tokens per record (avoided Pain Point Master re-query)

---

## Inherited Data from ESS

**HSI receives via webhook payload from ESS:**
- Expression Mode ✓
- Emotional Clarity ✓
- Narrative Alignment Score ✓
- Core Emotion (ESS pass-through from EIP) ✓
- Need State (ESS pass-through from EIP) ✓

**No additional queries needed**

---

## BRA Status Field (Updated by BRA)

**Field ID:** `cl1250sz39sm45l`

**Initial value (set by HSI):** `Pending Template Selection`

**Updated by BRA after template selection:**
- `T1-Generated` (deterministic template used)
- `T2-Generated` (Claude hybrid used)
- `T3-Generated` (Claude hybrid used)
- `Error` (BRA processing failed)

**Purpose:** Track which HSI records have been processed by BRA for response drafts

**Query pattern (used by MRA future build):**
```javascript
// Count records by BRA status
const stats = await nocodb.query(`
  SELECT bra_status, COUNT(*) 
  FROM hsi_table 
  WHERE client_id = 'PAK-001' 
  AND created_at >= '2026-04-01'
  GROUP BY bra_status
`);
```

---

## Data Flow
ESS webhook → HSI
↓
INIT-1 to INIT-4: Receive + parse payload
Validate: EIP pain point data present
Validate: ESS expression data present
↓
S4A1: Base severity from Signal Weight (EIP)
↓
S4A2: Expression Mode modifier
↓
S4A3: Emotional Clarity modifier
↓
S4A4: Star Rating cross-check
↓
S4A5: Domain-specific rules
↓
S4A6: Final severity score (1-10)
↓
Node 23-26: Generate Hospitality Signal ID
Format: HSI-[Domain]-[Severity]-[YYYYMMDD]-[Seq]
↓
Node 7-22: Claude API call
Input: Review + EIP data + ESS data + severity score
Output: Behavioral Narrative + Operator Insight (dual output)
↓
Node 27-28: Update BRA Status field
Set: "Pending Template Selection"
↓
Node 29: Write to HSI NocoDB table
↓
Node 30: Pass to SIA + BRA webhooks
SIA: Aggregates HSI records by domain + tier
BRA: Generates response draft, updates BRA Status field

---

## Related Documents

- **HOW Document:** [SCX_HSI_HOW_v3.md](SCX_HSI_HOW_v3.md)
- **Changelog:** [SCX_HSI_CHANGELOG.md](SCX_HSI_CHANGELOG.md)
- **Upstream Agent (EIP):** [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md)
- **Upstream Agent (ESS):** [../ESS/SCX_ESS_HOW_v4.md](../ESS/SCX_ESS_HOW_v4.md)
- **Downstream Agent (SIA):** [../SIA/SCX_SIA_HOW_v3.md](../SIA/SCX_SIA_HOW_v3.md)
- **Downstream Agent (BRA):** [../BRA/SCX_BRA_HOW_v3.2.md](../BRA/SCX_BRA_HOW_v3.2.md)

---

**End of HSI Schema**
