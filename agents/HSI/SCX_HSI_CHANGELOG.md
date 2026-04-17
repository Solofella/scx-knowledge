# HSI Change Log

## Chat #120 · April 17, 2026
- GitHub repository setup complete
- HSI HOW v3 documentation migrated from Google Drive

## Chat #74 · April 4, 2026
- Full pipeline verification complete
- All 30 nodes operational
- Signal Severity Assessment (S4A1-S4A6) validated as deterministic

## Chat #72 · March 28, 2026
- Confirmed HSI does NOT access Pain Point Master directly
- EIP provides all pain point data via webhook payload
- ESS provides all expression analysis via webhook payload
- Token budget confirmed: ~1,800 per record (no master re-query overhead)

---

## Design Decisions (Permanent)

### Signal Severity Assessment (S4A1-S4A6)
**Six-step deterministic calculation:**
- S4A1: Base severity from Signal Weight (EIP)
- S4A2: Expression Mode modifier (Masked +1, Performative -0.5)
- S4A3: Emotional Clarity modifier (Ambiguous/Fragmented +1)
- S4A4: Star Rating cross-check (Low rating + Masked +1)
- S4A5: Domain-specific rules (Core domains +0.5)
- S4A6: Final score capped at 1-10

**Purpose:** Consistent severity scoring without AI variability

### Hospitality Signal ID Format
**Pattern:** `HSI-[Domain]-[Severity]-[YYYYMMDD]-[Sequence]`

**Example:** `HSI-ServiceQuality-High-20260417-003`

**Purpose:** Unique identifier for cross-referencing in SIA aggregation, BRA template selection, MRA reporting

### Dual Output (Behavioral Narrative + Operator Insight)
**Behavioral Narrative:** Guest-facing context (how emotion + pain point impacted experience)

**Operator Insight:** Internal understanding (what signal means, pattern recognition)

**Governance constraint:** Both must describe meaning, never prescribe actions

### Why Claude (Not GPT)
**Decision:** Use Claude claude-sonnet-4-6 for HSI

**Rationale:**
- Narrative synthesis required (combine emotion + pain point into coherent story)
- Long-form prose generation (behavioral narratives, operator insights)
- Governance-constrained writing (detect/interpret only, never prescribe)

**GPT better for:** Structured classification (EIP's task)  
**Claude better for:** Narrative synthesis (HSI's task)

---

**Instructions for future updates:**

When HSI changes occur, add new entries at the top in this format:

## Chat #[NUMBER] · [DATE]
- [Description of change]
- [Description of change]

Keep entries concise (1-2 lines per change).
