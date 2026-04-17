# BRA Change Log

## Chat #127 · April 17, 2026
- GitHub repository setup complete
- BRA HOW v3.2 documentation migrated from Google Drive
- Hybrid model (template + Claude) documented

## Chat #74 · April 4, 2026
- Full pipeline verification complete
- All 19 nodes operational
- Confirmed T1 template engine working (zero AI cost for 70-80% of reviews)

## Chat #18 · March 11, 2026
- **HYBRID MODEL LOCKED:** T1 deterministic templates, T2/T3 Claude generation
- Template Library NocoDB table created with 50+ initial templates
- Tier assignment logic finalized: severity + star rating → T1/T2/T3
- Token savings confirmed: 75% reduction vs all-Claude approach

---

## Design Decisions (Permanent)

### Hybrid Model (Template + Claude)
**Decision:** Use deterministic templates for T1 (70-80% of reviews), Claude for T2/T3 (20-30%)

**Rationale:**
- **T1 reviews:** Low-severity, positive experiences → templates work well, guest already satisfied
- **T2/T3 reviews:** Moderate-to-high severity → nuanced situations require Claude customization
- **Token savings:** 75% reduction in BRA cost (templates = 0 tokens, Claude = 2,500 tokens)
- **Quality acceptable:** RDA refines ALL drafts downstream with brand voice, so templates get personalized

### Tier Assignment Logic
**Deterministic based on severity + star rating:**
- **T1:** Severity ≤3 AND Star Rating ≥4 (positive, low-severity)
- **T3:** Severity ≥8 OR Star Rating ≤2 (negative, high-severity)
- **T2:** Everything else (ambiguous, moderate-severity)

**Observed distribution:** T1 ~75%, T2 ~20%, T3 ~5%

### Template Selection (Deterministic Seed)
**Decision:** Use hash(HSI Record ID) as seed for template selection

**Rationale:**
- Same review always gets same template (reproducibility for debugging)
- Distributes templates across reviews (prevents "everyone gets Template #1")
- No randomness = clear audit trail

### Why BRA ≠ Final Draft
**Decision:** BRA generates initial draft, RDA refines with brand voice

**Rationale:**
- Separation of concerns: BRA = tier-appropriate content, RDA = brand voice personalization
- Prevents template library explosion (50 templates vs 200+ if brand-specific)
- RDA handles conditional guest name logic (avoids duplicating in BRA)

### Template Library Size
**Current:** 50+ templates across 13 domains × T1 tier × 4 brand voice modes

**Future expansion:** T2 templates (optional, if Claude cost becomes issue at scale)

**T3 templates:** Not planned (high-severity situations too nuanced for templates)

---

**Instructions for future updates:**

When BRA changes occur, add new entries at the top in this format:

## Chat #[NUMBER] · [DATE]
- [Description of change]
- [Description of change]

Keep entries concise (1-2 lines per change).
