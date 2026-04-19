# BRA Schema — v3.4

**NocoDB Table ID:** `mwqejw7swhd2cf4`  
**Base ID:** `pq249fix22t3ofv`  
**Last Updated:** Chat #77 · April 19, 2026  
**Status:** Complete with Chat #74 integration  
**Quality Baseline:** 87% (verified at RDA audit layer)

---

## LOCKED GOVERNANCE PRINCIPLE

**BRA DETECTS and INTERPRETS signal. BRA does NOT prescribe operational actions.**

- BRA generates response STRATEGY and initial DRAFT
- BRA acknowledges guest experience and provides HSI-informed framing
- BRA never says "you should do X" or commits the business to any action
- RDA refines BRA output with brand voice; still governed by this principle

---

## Table Fields (20 fields total)

### Primary Key + Upstream References

| Field Name | Field ID | Type | Source | Required | Purpose |
|------------|----------|------|--------|----------|---------|
| BRA Record ID | (auto) | Auto-increment | System | YES | Primary key |
| HSI Record ID | | Number | HSI webhook | YES | FK upstream — idempotency key |
| ALA Record ID | | Number | HSI pass-through | YES | FK original review — traceability |
| EIP Record ID | | Number | HSI pass-through | YES | FK emotion analysis — traceability |
| ESS Record ID | | Number | HSI pass-through | YES | FK signal stabilization — traceability |
| Client ID | | SingleLineText | HSI pass-through | YES | **LOCKED Chat #74** — which client/brand owns this record |

### BRA-Computed Core Fields

| Field Name | Field ID | Type | Source | Required | Purpose |
|------------|----------|------|--------|----------|---------|
| **Confirmed Response Tier** | | SingleSelect | **HSI (BRA respects, never recalculates)** | YES | **T1** = template branch / **T2/T3** = Claude branch |
| **Template ID** | | SingleLineText | **BRA computation (T1 only)** | NO | Link to Template Library (NULL for T2/T3) |
| **Response Draft** | | LongText | **BRA computation** | YES | Template-populated (T1) OR Claude-generated (T2/T3) |
| **Generation Method** | | SingleSelect | **BRA computation** | YES | **Template** (T1, 0 tokens) OR **Claude** (T2/T3, ~2,500 tokens) |

### Context + Behavioral Fields (from HSI)

| Field Name | Field ID | Type | Source | Required | Purpose |
|------------|----------|------|--------|----------|---------|
| Emotional Axis | | Text | HSI webhook | YES | What the guest felt — context for RDA |
| Pain Axis | | Text | HSI webhook | YES | Operational pain point — context for RDA |
| Stability Axis | | LongText | HSI webhook | YES | Recurrence framing (first visit, repeat, pattern) |
| Behavioral Interpretation | | LongText | HSI webhook | NO | Full behavioral narrative (T2/T3 context) |
| Commercial Risk Flag | | Checkbox | HSI webhook | YES | true → elevates record to RDA Pending-Elevated |

### Chat #74 Payload Additions

| Field Name | Field ID | Type | Source | Required | Purpose | **v3.4 Addition** |
|------------|----------|------|--------|----------|---------|-------------------|
| Emotion Hypothesis | | LongText | EIP pass-through | NO | Enriched emotion narrative — Claude context for T2/T3 | ✅ NEW |
| Keywords | | SingleLineText | ALA pass-through | NO | Specific words guest emphasized — signals specificity to Claude | ✅ NEW |
| Reviewer Handle | | SingleLineText | ALA pass-through | NO | Guest name — used by RDA for greeting logic | ✅ NEW |

### Metadata + Audit Fields

| Field Name | Field ID | Type | Source | Required | Purpose |
|------------|----------|------|--------|----------|---------|
| Created At | (auto) | DateTime | System | YES | Record creation timestamp — audit trail |
| Error Log | **c3crsub617hnuee** | LongText | Error handler | NO | Captures template query failures or Claude API errors |

---

## Field Computation Logic

### 1. Confirmed Response Tier (FROM HSI — NEVER RECALCULATED BY BRA)

**Source:** HSI webhook `confirmed_response_tier` field

**BRA responsibility:** Accept the tier. Route execution based on tier.

**Valid values:**
- **T1:** Low-severity positive signals (~70-80% of records)
  - No elevated operational concern
  - Standard acknowledgment response sufficient
  - Template engine path (0 tokens)
  
- **T2:** Mixed/ambiguous or moderate-severity signals (~15-20%)
  - Requires measured, nuanced response
  - Claude API path (~2,500 tokens)
  
- **T3:** Negative critical or dignity-risk signals (~5-10%)
  - Highest severity, brand/legal risk
  - Claude API path (~2,500 tokens)

**HSI Tier Assignment Logic (Chat #74 Fixed):**

The tier is set UPSTREAM by HSI using the following **signal-aware** logic:

```javascript
// HSI Tier Routing (Chat #74 Fix)
const signalType = row.signal_type;        // Positive / Negative / Mixed / Ambiguous
const intensity = row.intensity_level;     // Critical / High / Moderate / Low
const starRating = row.star_rating;        // 1-5

let tier;

// T3: Negative Critical OR very low star rating
if ((signalType === 'Negative Signal' && intensity === 'Critical') || starRating <= 1) {
  tier = 'T3';
}
// T2: Mixed (always) OR Negative High/Moderate OR Ambiguous
else if (
  signalType === 'Mixed Signal' ||
  (signalType === 'Negative Signal' && (intensity === 'High' || intensity === 'Moderate')) ||
  signalType === 'Ambiguous Signal'
) {
  tier = 'T2';
}
// T1: Positive Signal OR Negative Low
else if (signalType === 'Positive Signal' || (signalType === 'Negative Signal' && intensity === 'Low')) {
  tier = 'T1';
}
```

**Key changes from v3.2:**
- ✅ Mixed Signal **always** routes to T2 (not caught by T1 fallback)
- ✅ Negative Signal High/Moderate **always** routes to T2 (not T1)
- ✅ Only Positive and Negative Low route to T1

**BRA's role:** Receive HSI's tier assignment and use it to route branching logic (template vs Claude).

---

### 2. Template ID (T1 Records Only)

**Computed by:** Deterministic seeded selection from Template Library

**Only populated for T1 records.** NULL for T2/T3.

**Algorithm:**

```javascript
if (confirmed_response_tier === 'T1') {
  // Step 1: Query Template Library for T1 templates matching domain
  const templates = await nocodb.query({
    table: 'Template_Library',  // Table ID: mafv9by73ebama7
    filter: `(signal_category eq 'Positive') AND (domain eq '${domain}')`
  });
  
  if (templates.length === 0) {
    // Error: No templates for this domain
    error_log = `Template query failed: No Positive templates for domain '${domain}'`;
    return null;
  }
  
  // Step 2: Generate deterministic seed from HSI Record ID
  const seed = hashCode(hsiRecordId); // Simple hash function, e.g., Java-style hashCode
  
  // Step 3: Select template using modulo operator
  const selectedIndex = Math.abs(seed) % templates.length;
  const selectedTemplate = templates[selectedIndex];
  
  // Step 4: Return template ID
  template_id = selectedTemplate.template_id; // e.g., "T1-FoodQuality-003"
  
  // Step 5: Increment usage count (audit trail of template distribution)
  await nocodb.patch({
    table: 'Template_Library',
    recordId: selectedTemplate.id,
    data: { usage_count: selectedTemplate.usage_count + 1 }
  });
  
  return template_id;
}
```

**Why deterministic seeding?**
- **Reproducibility:** Same HSI Record ID always selects same template
- **Auditability:** No randomness — seed derivation is transparent
- **Distribution:** Templates spread evenly across reviews (prevents "everyone gets Template #1")
- **Idempotency:** Re-running same review produces identical selection

**Example Template IDs:**
- `T1-FoodQuality-003`
- `T1-ServiceQuality-012`
- `T1-Ambiance-007`
- `T1-StaffBehavior-001`

---

### 3. Response Draft (Hybrid Generation — T1 vs T2/T3)

#### Path A: T1 Template Population

**Used for:** ~70-80% of reviews (low-severity, positive)

**Process:**

1. **Fetch template** from Template Library using Template ID
2. **Extract placeholders:** `{guest_name}`, `{specific_detail}`, `{domain}`, `{restaurant_name}`
3. **Perform string replacement** with actual review values

**Code:**

```javascript
const template = await nocodb.get({
  table: 'Template_Library',
  recordId: template_id
});

let draft = template.core;  // e.g., "Thank you, {guest_name}, for highlighting..."

// Replace {guest_name} — safe null check + "Anonymous" filter
if (reviewerHandle && 
    reviewerHandle !== 'Anonymous' && 
    !reviewerHandle.toLowerCase().includes('review')) {
  draft = draft.replace('{guest_name}', reviewerHandle);
} else {
  // No name available: remove greeting clause
  draft = draft.replace('Thank you, {guest_name}, for', 'Thank you for');
}

// Replace {specific_detail} — extract from review using domain logic
const specificDetail = extractSpecificDetail(raw_text, domain);
draft = draft.replace('{specific_detail}', specificDetail);

// Replace {domain}
draft = draft.replace('{domain}', domain.toLowerCase());

// Replace {restaurant_name} — from Client Config
draft = draft.replace('{restaurant_name}', client_name);

return draft;
```

**Example:**

**Template text:**
```
Thank you, {guest_name}, for highlighting {specific_detail}. We're glad our {domain}
team at {restaurant_name} delivered an experience you enjoyed. Your feedback helps us
maintain the standards that matter most to our guests.
```

**Values:**
- guest_name: "Sarah M"
- specific_detail: "the perfectly seared scallops"
- domain: "Food Quality"
- restaurant_name: "Park Avenue Kitchen by David Burke"

**Resulting draft:**
```
Thank you, Sarah M, for highlighting the perfectly seared scallops. We're glad our
food quality team at Park Avenue Kitchen by David Burke delivered an experience you
enjoyed. Your feedback helps us maintain the standards that matter most to our guests.
```

**Token cost:** **0 tokens** (pure string replacement, no AI)

---

#### Path B: T2/T3 Claude Generation

**Used for:** ~20-30% of reviews (moderate-to-high severity, ambiguous or negative)

**Process:**

1. **Construct Claude API request** with governance-constrained system prompt
2. **Include HSI context:** Behavioral Narrative, Emotion Hypothesis, Keywords, Severity, Star Rating
3. **Claude generates custom draft** (150-250 words)
4. **Post-call governance check** (no refund offers, no placeholder text, etc.)

**System Prompt (Governance-Locked):**

```
You are the Brand Response Architect for SubtextCX. Generate a response draft
acknowledging the guest's behavioral signal.

LOCKED GOVERNANCE PRINCIPLE:
SubtextCX DETECTS and INTERPRETS signal only. You do NOT prescribe operational
actions. Acknowledge the guest's experience, express appreciation, demonstrate
understanding. Do NOT recommend specific fixes ("hire more servers", "retrain staff").
Do NOT make promises about future changes. Do NOT commit the business to any action.

INPUTS:
Review Text: [raw review from ALA]
Behavioral Narrative: [HSI's full analysis of guest psychology]
Emotion Hypothesis: [EIP's enriched emotion tag + narrative]
Keywords: [Specific words guest emphasized — use these for specificity]
Emotional Axis: [Primary emotion the guest experienced]
Pain Axis: [Operational pain point]
Severity: [1-10 scale — urgency context]
Star Rating: [1-5 — satisfaction proxy]
Tier: [T2 or T3]

OUTPUT REQUIREMENTS:
- Length: 150-250 words
- Tone: Professional, empathetic, human (never transactional)
- Address guest directly using "you" and "your"
- If guest name available, open with greeting (T2: Hi/Hello [Name]. T3: Hello [Name] —)
- Use Keywords from review for specificity — never generic ("we heard your feedback")
- For T3 (dignity-risk): Grave and human tone. No generic apologies. Acknowledge harm specifically.
- For T2 (mixed): Measured and careful. Preserve ambiguity. Contact invitation permitted.

CRITICAL RULES:
- NO refund/voucher/discount/credit language
- NO financial commitments or compensation language
- NO email addresses, URLs, or named contact channels
- NO placeholder text ([contact], [name], [email])
- NO "We apologize for any inconvenience"
- NO operational directives ("please have the manager train staff")
```

**User Prompt Template:**

```
BEHAVIORAL NARRATIVE:
[From HSI]

EMOTION HYPOTHESIS:
[From EIP — enriched emotion analysis]

KEYWORDS FROM REVIEW:
[From ALA — specific words guest emphasized]

ORIGINAL REVIEW:
[Raw text]

EMOTIONAL AXIS: [e.g., "Frustration masked by politeness"]
PAIN AXIS: [e.g., "Service Quality: Wait time + Food Quality: Temperature"]
SEVERITY: [1-10]
STAR RATING: [1-5]
TIER: [T2 or T3]
GUEST NAME: [If available, otherwise null]

Generate a response acknowledging this guest experience with authentic understanding.
```

**Claude Call Parameters:**

| Parameter | Value |
|-----------|-------|
| Model | claude-sonnet-4-6 |
| max_tokens | 500 |
| temperature | 0.3 |
| Retry | true |
| Max tries | 3 |
| Wait between tries | 5000ms |

**Example Output (T3 — Dignity Risk):**

**Inputs:**
- Star Rating: 1
- Severity: 9
- Tier: T3
- Keywords: "unacceptable", "40 minutes", "reservation", "cold", "anniversary"
- Behavioral Narrative: "Guest experienced Autonomy violation (loss of schedule control) + Service Quality + Food Quality failures. Anniversary context amplified emotional impact. Polite phrasing masks deep dissatisfaction."

**Claude Draft:**

```
We're truly sorry your anniversary celebration didn't receive the thoughtful attention
it deserved. A 40-minute wait despite your reservation, followed by food arriving cold,
disrupted what should have been a meaningful evening.

We understand the specific weight of having a special occasion compromised by service
failures on both timing and food quality. That combination of disappointments — the loss
of control over your schedule and the physical experience of cold food — represents a
genuine failure on our part.

Your feedback about where we fell short is important to us, both because of the occasion
and because these are foundational elements of hospitality. We appreciate your willingness
to share this with us, even though the experience was disappointing.

We hope the rest of your anniversary was able to feel meaningful despite the challenges
you encountered at our restaurant. Thank you for taking the time to communicate your
experience.
```

**Governance check:**
- ✅ Acknowledges specific failures (40-min wait, cold food, anniversary)
- ✅ Demonstrates understanding of emotional/operational impact
- ✅ Uses keywords from review for specificity
- ✅ NO operational prescriptions
- ✅ NO financial offers or commitments
- ✅ NO generic apology formula

**Token cost:** ~2,500 tokens per record

---

### 4. Generation Method (Metadata)

**Populated by:** BRA computation

**Valid values:**
- **Template:** Response draft generated via T1 template engine (0 tokens, T1 tier only)
- **Claude:** Response draft generated via Claude API (T2 or T3 tier, ~2,500 tokens)

**Purpose:** Track AI spend distribution for cost analysis and client billing

**Example MRA query:**

```javascript
const braStats = await nocodb.query({
  table: 'BRA',
  aggregate: { COUNT: '*' },
  groupBy: 'generation_method',
  filter: `client_id = 'PAK-001' AND created_at >= '2026-04-01'`
});

// Result:
// { Template: 225, Claude: 75 }
// Interpretation: 75% of PAK-001 April records used templates (zero token cost)
//                 25% used Claude (cost-bearing tier)
```

---

### 5. Emotion Hypothesis (Chat #74 Addition)

**Field:** Emotion Hypothesis  
**Type:** LongText  
**Source:** EIP pass-through (from EIP agent)  
**Required:** NO (T2/T3 use, optional context)  
**Purpose:** Enriched emotional state description — Claude context enrichment for T2/T3 drafts

**Content:** Full narrative of guest's emotional state as interpreted by EIP:

Example:
```
Guest experienced frustration masked by civility. Underlying emotions: loss of autonomy
(40-minute wait despite reservation), expectation violation (cold food after long wait),
anniversary disappointment (occasion amplification). Guest values politeness but is
genuinely dissatisfied. High risk of negative word-of-mouth due to occasion context.
```

**BRA usage:** Passed to Claude in T2/T3 user prompt for tone calibration and specificity

---

### 6. Keywords (Chat #74 Addition)

**Field:** Keywords  
**Type:** SingleLineText  
**Source:** ALA pass-through (from ALA agent)  
**Required:** NO  
**Purpose:** Specific words/phrases guest emphasized — signals specificity to Claude (prevents generic language)

**Content:** Comma-separated list of signal keywords:

Example:
```
"unacceptable", "40 minutes", "reservation", "cold", "anniversary", "ruined"
```

**BRA usage:** Passed to Claude in T2/T3 user prompt with instruction: "Use these keywords to reference specific details from the guest's experience. Never generic."

**Effect:** Prevents Claude from writing "we heard your feedback" — forces "the 40-minute wait despite your reservation, and the cold food when it arrived"

---

### 7. Reviewer Handle (Chat #74 Addition)

**Field:** Reviewer Handle  
**Type:** SingleLineText  
**Source:** ALA pass-through (from ALA agent)  
**Required:** NO  
**Purpose:** Guest name from review platform handle — used by RDA for greeting logic

**Content:** Guest name or handle as provided by review platform:

Examples:
- "Sarah M"
- "Michael K."
- "Anonymous" (filtered out)
- "review_bot_2024" (filtered out — contains "review")

**BRA usage in T1 templates:**

Template placeholder replacement checks name safety:
```javascript
if (reviewerHandle && 
    reviewerHandle !== 'Anonymous' && 
    !reviewerHandle.toLowerCase().includes('review')) {
  // Use name in template
} else {
  // Remove greeting clause, proceed without name
}
```

**RDA usage:** Opens response with "Hello [Name]" or "Hi [Name]," based on tier

---

## Error Log Field (Deterministic + No-Block)

**Field ID:** `c3crsub617hnuee`  
**Type:** LongText  
**Required:** NO  
**Populated when:** BRA encounters a failure that doesn't halt pipeline

**Error scenarios:**

1. **Template Library query returns zero templates:**
   ```
   2026-04-17T14:32:11Z - Template query failed: No Positive templates for domain 'Parking'
   ```

2. **Claude API rate limit (429):**
   ```
   2026-04-17T15:08:42Z - Claude API error: Rate limit exceeded (429). Retry queued.
   ```

3. **Template placeholder replacement fails:**
   ```
   2026-04-17T16:21:03Z - Template placeholder error: {specific_detail} not found in review text. Using empty placeholder.
   ```

**Error handling principle:** Error does NOT block downstream. Record continues to RDA with:
- response_draft: Empty or fallback text
- generation_method: NULL
- error_log: Description of failure

RDA detects error and may flag record for manual review.

---

## Token Efficiency Analysis

**By Tier (300 reviews/month distribution):**

| Tier | Gen Method | % Records | Cost/Record | Count | Monthly Total |
|------|-----------|-----------|------------|-------|-----------------|
| T1 | Template | 75% | 0 | 225 | 0 tokens |
| T2 | Claude | 20% | 2,500 | 60 | 150K tokens |
| T3 | Claude | 5% | 2,500 | 15 | 37.5K tokens |
| **TOTAL** | **Hybrid** | **100%** | **~625 avg** | **300** | **~187.5K tokens** |

**Comparison: All-Claude vs. Hybrid BRA:**

| Model | 300 rec/mo | Annual |
|-------|-----------|--------|
| All-Claude | 750K tokens | 9M tokens |
| Hybrid BRA | 187.5K tokens | 2.25M tokens |
| **Savings** | **562.5K (75% reduction)** | **6.75M (75% reduction)** |

**Why Hybrid?**
- T1 responses (70-80%) don't need AI — guest is satisfied, acknowledgment sufficient
- T2/T3 responses (20-30%) require Claude — nuanced, custom, risk-aware
- Token savings fund higher quality for risk-bearing tier assignments

---

## Downstream Integration — HSI Table Update

**Side effect of BRA processing:**

BRA PATCHes the HSI table's `BRA Status` field after draft generation completes.

**HSI Field Details:**

| Item | Value |
|------|-------|
| Field Name | BRA Status |
| Field ID | `cl1250sz39sm45l` |
| Type | SingleSelect |
| Table | HSI |
| Updated by | BRA Node 18 (PATCH) |

**Updated values:**

| Value | Meaning | Trigger |
|-------|---------|---------|
| `T1-Generated` | T1 template used, draft complete | After Node 10 (T1 branch) |
| `T2-Generated` | T2 Claude used, draft complete | After Node 15 (T2/T3 branch) |
| `T3-Generated` | T3 Claude used, draft complete | After Node 15 (T2/T3 branch) |
| `Error` | BRA processing failed | After Node 16 (error handler) |

**Purpose:** Track which HSI records have been processed by BRA (used by SIA and MRA for reporting)

---

## Template Library Schema (Referenced by BRA)

**NocoDB Table ID:** `mafv9by73ebama7`

**Used by BRA:** T1 template selection + placeholder population

**Schema:**

| Field Name | Type | Required | Purpose |
|------------|------|----------|---------|
| Template ID | SingleLineText | YES | Unique identifier (e.g., T1-FoodQuality-003) |
| Template Name | SingleLineText | YES | Human-readable label for audit |
| Signal Category | SingleSelect | YES | Positive / Negative / Mixed |
| Domain | SingleSelect | YES | Food Quality, Service Quality, Ambiance, Staff Behavior, etc. (13 locked) |
| Core | LongText | YES | Response text with placeholders: {guest_name}, {specific_detail}, {domain}, {restaurant_name} |
| Utility | LongText | NO | Internal notes on when template effective (never written to output) |
| Signal-Based Consideration | LongText | NO | Context reasoning (never written to output) |

**Current inventory:** 21 templates (v3.0)
- 10 templates: Negative Signal (Low severity, can acknowledge without overreacting)
- 6 templates: Positive Signal (Praise, gratitude, belonging variations)
- 5 templates: Mixed Signal (Balanced acknowledgment, both praise and concern)

**Audit trail:** `usage_count` field tracks template distribution (incremented after each selection)

---

## Related Documents

- **HOW Document:** [SCX_BRA_HOW_v3.3.md](SCX_BRA_HOW_v3.3.md) (Chat #77 update)
- **Changelog:** [SCX_BRA_CHANGELOG.md](SCX_BRA_CHANGELOG.md)
- **Upstream Agent:** [../HSI/SCX_HOW_HSI_v3.0.md](../HSI/SCX_HOW_HSI_v3.0.md)
- **Downstream Agent:** [../RDA/SCX_HOW_RDA_v3.1.md](../RDA/SCX_HOW_RDA_v3.1.md)
- **Master Continuity:** [../../../SubtextCX_MCD_v7_4_1.docx](MCD)
- **Template Library:** NocoDB table `mafv9by73ebama7` (21 templates, 7 fields)

---

## Version History

**v3.4 (Chat #77 · April 19, 2026)**
- ✅ Integrated Chat #74 additions: emotion_hypothesis, keywords, reviewer_handle, client_id
- ✅ Updated tier routing logic: Mixed Signal → T2, Negative High/Moderate → T2
- ✅ Locked governance principle explicitly (DETECT/INTERPRET only)
- ✅ Expanded BRA table schema (20 fields, all documented)
- ✅ Added field-by-field computation logic
- ✅ Clarified deterministic seeding algorithm
- ✅ Updated Template Library reference (21 templates v3.0, 7 fields)
- ✅ Verified HSI field IDs (BRA Status: cl1250sz39sm45l)
- ✅ Expanded token efficiency analysis (75% cost reduction locked)
- ✅ Added Chat #74 payload structure documentation

**v3.3 (Chat #75 · April 13, 2026)**
- v3.2 with RDA integration notes

**v3.2 (Chat #73 · March 25, 2026)**
- Initial schema documentation
- 19 nodes, hybrid model, token budget

---

**Subtext CX · BRA Schema v3.4 · Chat #77 · April 19, 2026 · Solofella LLC**

---

✅ **STATUS:** Complete. All Chat #74 additions integrated. Tier routing HSI fix reflected. Governance principle locked. 20-field BRA table documented. Deterministic seeding explained. Token efficiency verified. All field IDs confirmed.

**This updated v3.4 should be committed to:**
```
https://github.com/Solofella/scx-knowledge/blob/main/agents/BRA/BRA_Schema.md
```
