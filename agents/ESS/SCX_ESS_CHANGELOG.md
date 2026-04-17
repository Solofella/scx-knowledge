# ESS Change Log

## Chat #117 · April 17, 2026
- GitHub repository setup complete
- ESS HOW v4 documentation migrated from Google Drive

## Chat #74 · April 4, 2026
- Full pipeline verification complete
- All 23 nodes operational
- Confirmed ESS inherits EIP emotion classifications (zero dictionary re-query cost)

## Chat #72 · March 28, 2026
- Validated ESS does NOT access Emotion Dictionary directly
- EIP provides all emotion data via webhook payload
- Token budget confirmed: ~1,200 per record (no dict injection overhead)

---

## Design Decisions (Permanent)

### Expression Mode Categories (6 Types)
- **Explicit:** Direct emotional language
- **Implicit:** Emotion conveyed through context
- **Masked:** Positive/neutral language hiding negative emotion (SubtextCX differentiator)
- **Performative:** Emotion expressed for social effect
- **Conflicted:** Contradictory emotions simultaneously
- **Absent:** No detectable emotional content

### Emotional Clarity Categories (4 Types)
- **Clear:** Unambiguous, well-articulated
- **Diffuse:** Spread across multiple unfocused themes
- **Fragmented:** Broken or inconsistent narrative
- **Ambiguous:** Deliberately obscured or unclear

### Why Claude (Not GPT)
**Decision:** Use Claude claude-sonnet-4-6 for ESS

**Rationale:**
- Nuanced interpretation required (masked emotion detection)
- Subtle linguistic analysis (performative vs genuine)
- Context-aware tone assessment

**GPT better for:** Structured classification (EIP's task)  
**Claude better for:** Interpretive analysis (ESS's task)

---

**Instructions for future updates:**

When ESS changes occur, add new entries at the top in this format:

## Chat #[NUMBER] · [DATE]
- [Description of change]
- [Description of change]

Keep entries concise (1-2 lines per change).
