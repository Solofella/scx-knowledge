Chat #113 · September 10, 2026

```
CLAIM: Achieving high-quality RDA outputs requires transforming quality measurement from a subjective human check into an automated, multi-axis evaluation pipeline while optimizing generation-time field presentation and resolving inherent criteria tensions.
EVIDENCE FOR: Industry-standard LLM-as-a-judge frameworks (G-Eval, DeepEval, MT-Bench) prove that structured, multi-axis rubrics, reference-based pair comparisons, and probability/CoT-driven scoring achieve strong alignment with human preference.
EVIDENCE AGAINST: Over-constraining generation prompts with explicit rule lists creates robotic text (failing Naturalness), while pure free-form generation underuses Client Config or upstream signals.
CONCLUSION: ✅ HIGH — Proceeding with a dedicated Quality Optimization Strategy covering Evaluation, Generation-Time Techniques, Conflict Resolution, and Actionable Plan.

```

---

### 1. Concrete Evaluation & Verification Mechanisms (Scoring Strategy)

To evaluate the 3 criteria automatically without relying solely on manual human checks, implement a dedicated **Offline Evaluation & CI/CD Gate Framework** using an LLM-as-a-Judge pattern running on a strict 1–5 scoring scale or binary verification matrices.

```
                 [ Candidate RDA Draft (EN/ES) ]
                               │
       ┌───────────────────────┼───────────────────────┐
       ▼                       ▼                       ▼
[ Evaluator 1:          [ Evaluator 2:          [ Evaluator 3:
  Naturalness ]           Brand Fidelity ]        Signal Coverage ]
       │                       │                       │
 1-5 Scale +             Distinguishability      Recall / Precision
 CoT Reasoning           Pairwise Test           Claim Extraction
       │                       │                       │
       └───────────────────────┼───────────────────────┘
                               ▼
               [ Composite Quality Score (Q-Score) ]

```

#### Criterion 1 — Naturalness (Native English & Spanish)

* **Evaluation Method:** **G-Eval Chain-of-Thought (CoT) Fluency & Persona Assessment**.
* **Mechanism:** Run candidate drafts through a dedicated judge prompt (using Claude 3.5 Sonnet or GPT-4o at `temperature=0`) that evaluates native sentence flow, cadence, idiomatic accuracy, and absence of "AI Markers" (e.g., *testament, delve, tapestry, ensuring, landed* in English; *estimado, en resumen, no dudes en, constante compromiso* in Spanish).
* **Scoring:** 1–5 continuous scale derived via weighted probability or explicit multi-point rubric.
* **Target Gate:** Score $\ge 4.5/5.0$. Any draft flagged with an explicit "AI Marker" receives an automatic failure tag.

#### Criterion 2 — Client Configuration (100% Recognizable & Distinguishable)

* **Evaluation Method:** **Adversarial Blind Pairwise Distinguishability Test**.
* **Mechanism:**
1. Take the generated draft for Client A (with Client A's name redacted).
2. Provide the Judge LLM with **two** full Client Config profiles: Client A and a randomly selected decoy, Client B.
3. Ask the Judge: *"Based strictly on Register, Core Driver, Personality, and Brand Phrases present in this text, which Client Config produced this response: Client A or Client B? Provide your confidence score (0-100%) and cite specific phrase evidence."*


* **Metric:** **Brand Attribution Accuracy Rate**.
* **Target Gate:** The Judge must correctly identify Client A with $>85\%$ confidence. If the judge cannot distinguish Client A from Client B, the response is too generic and fails the deployment gate.

#### Criterion 3 — Logical Coherence with Upstream Inputs (100% Relevant & Grounded)

* **Evaluation Method:** **Bidirectional Claim & Signal Recall Matrix (RAG Triad adaptation)**.
* **Mechanism:**
* **Forward Recall (No Omissions):** Extract all identified upstream signals (e.g., `Staff_Mention: "Maria"`, `Pain_Point: "Long wait times"`). Verify if each signal is addressed in the draft.

$$\text{Signal Recall} = \frac{\text{Addressed Upstream Signals}}{\text{Total Upstream Signals}}$$


* **Reverse Precision (No Hallucinations/Over-extensions):** Extract all entity and event claims from the public draft. Verify if every claim traces back to either the Raw Review or the Upstream Signal Payload.

$$\text{Claim Precision} = \frac{\text{Grounded Claims in Draft}}{\text{Total Claims in Draft}}$$




* **Target Gate:** $\text{Signal Recall} = 100\%$ and $\text{Claim Precision} = 100\%$.

---

### 2. Generation-Time Optimization Techniques

#### Optimizing Criterion 2: Client Config Presentation

* **Current Flaw:** Flattening 12–18 configuration fields into a wall of context text leads to instruction attenuation (the model treats them as background knowledge rather than stylistic imperatives).
* **Recommended Method:** **XML Structural Isolation + Stylistic Contrast Framing**.
Instead of listing raw database fields, transform the Client Config into an active persona block inside XML tags, utilizing **Negative Contrast ("What we are NOT")**:

```xml
<brand_identity client="TacoBar_01">
  <voice_persona>
    Your tone is Casual Coastal (Formality: 2/5). You speak like a warm local bartender, NOT a corporate hospitality manager.
  </voice_persona>
  
  <stylistic_rules>
    <core_driver>We champion craft freshness and fast, friendly energy.</core_driver>
    <person_perspective>Use "we" and "our team". Never use "I".</person_perspective>
    <signature_phrases>Use occasionally when natural: "See you at the bar", "Fresh pressed"</signature_phrases>
  </stylistic_rules>

  <anti_patterns>
    DO NOT sound formal. NEVER use phrases like: "We strive for excellence", "We appreciate your feedback", "Your satisfaction is our priority".
  </anti_patterns>
</brand_identity>

```

#### Optimizing Criterion 3: Upstream Signal Structure

* **Current Flaw:** Passing signals as loose key-value pairs (`Cognitive Driver: Disappointment`, `Need State: Validation`) allows the LLM to treat them as optional context.
* **Recommended Method:** **Signal-to-Action Mapping (Mandatory Inclusion Directives)**.
Structure upstream signals not as passive labels, but as explicit **Drafting Tasks** that the prompt engine must complete:

```xml
<upstream_signals_mandatory_checklist>
  <signal id="1" type="STAFF_MENTION" priority="HIGH">
    <value>Server "Carlos"</value>
    <required_action>Acknowledge Carlos specifically in the body. Do NOT use a generic staff thank you.</required_action>
  </signal>
  <signal id="2" type="PAIN_POINT" priority="CRITICAL">
    <value>Cold food delivery</value>
    <required_action>Address the kitchen temperature drop directly without issuing an unqualified legal admission of fault.</required_action>
  </signal>
</upstream_signals_mandatory_checklist>

```

---

### 3. Managing Criteria Tensions

| Tension | Conflict Description | Resolution Strategy |
| --- | --- | --- |
| **Brand Specificity vs. Naturalness** | Over-injecting forced Brand Phrases or strict register constraints makes the prose sound contrived or robotic. | **Soft Injection Thresholds:** Instruct the model that Brand Phrases are *optional stylistic spices* (max 1 per response), whereas Register and Personality are *structural constraints*. |
| **Signal Recall vs. Naturalness** | Trying to address every single upstream signal tag in a short review causes "checklist writing" and bloats sentence count. | **Signal Tiering & Deduplication:** Upstream BRA/EIP must pass a max of **2 core anchors** (Primary Anchor + Secondary Accent) to RDA. RDA is forbidden from addressing more than 2 distinct signals per response. |
| **Grounding (Precision) vs. Warm Empathy** | Strict adherence to factual bounds can make the response sound cold or defensive. | **Separate Fact from Empathy:** Allow high emotional warmth in the *Opening*, but restrict factual claims in the *Body* strictly to confirmed upstream payload elements. |

---

### 4. Quality Optimization Plan (Roadmap)

#### Phase 1: Evaluation Infrastructure & Golden Baseline (Week 1)

1. **Build Golden Benchmark Corpus:** Create a static test suite of 30 review records (15 EN / 15 ES) representing all 3 tiers, staff mentions, policy disputes, and edge cases.
2. **Deploy Automated Evaluators:** Implement the 3 LLM-as-a-Judge evaluators (Naturalness CoT, Brand Pairwise Distinguishability, Signal Recall Matrix) as an offline test script.

#### Phase 2: Generation-Time Prompt Transformation (Week 2)

1. **Refactor Client Config Injection:** Reformat the 18 NocoDB fields into XML-isolated `<brand_identity>` blocks featuring explicit `<anti_patterns>`.
2. **Refactor Signal Injection:** Reformat EIP/BRA signals into `<upstream_signals_mandatory_checklist>` with explicit `<required_action>` tags.

#### Phase 3: Automated Quality Gating & CI/CD (Week 3)

1. **Composite Q-Score Gate:** Integrate the evaluation scripts into the deployment pipeline. Any prompt change must achieve:
* Naturalness Score $\ge 4.5/5.0$
* Brand Distinguishability $\ge 85\%$
* Signal Recall / Precision $= 100\%$


2. **Continuous Calibration:** Validate LLM-as-a-Judge outputs against human reviewer ratings on the benchmark suite to ensure alignment.

---
