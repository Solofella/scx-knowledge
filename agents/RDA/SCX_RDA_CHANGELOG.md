# RDA Change Log

## Chat #108 · April 17, 2026
- GitHub repository setup complete
- RDA HOW v3.1 documentation migrated from Google Drive

## Chat #73 · March 2026
- Implemented conditional guest name logic
- 6 structural variations for T1 warm acknowledgment
- T2/T3 simple "Hi [name]" or "Hello [name]"
- When no name: open with specific signal detail (never generic "thank you for feedback")

## Chat #68 · March 2026
- Brand voice integration complete
- Client Config table fetch added before Claude call
- 4 brand voice modes: Standard/Warm/Professional/Casual

---

## Open Issues

### Brand Voice Variation (Not Yet Implemented)
**Current:** Allows up to 3 brand phrases per draft  
**Fix needed:** Reduce to 1 phrase max, add variation constraint to system prompt  
**Priority:** Medium (prevents over-repetition in client drafts)

### Internal Brief Quality (Not Yet Implemented)
**Current:** `internal_followup_draft` sometimes includes operational prescriptions  
**Fix needed:** Strengthen system prompt - describe signal meaning only, no "add servers" or "change menu" language  
**Priority:** High (governance violation)

---

**Instructions for future updates:**

When RDA changes occur, add new entries at the top in this format:

## Chat #[NUMBER] · [DATE]
- [Description of change]
- [Description of change]

Keep entries concise (1-2 lines per change).

Move resolved issues from "Open Issues" section to dated entries when fixed.
