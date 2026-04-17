# EIP Change Log

## Chat #113 · April 17, 2026
- GitHub repository setup complete
- EIP HOW v4 documentation migrated from Google Drive
- Full dictionary injection strategy documented

## Chat #74 · April 4, 2026
- Full pipeline verification complete
- All 26 nodes operational
- OpenAI prompt caching confirmed working (90% discount on dictionary content)

## Chat #72 · March 28, 2026
- Confirmed EIP as single knowledge injection point
- Validated downstream agents (ESS, HSI, SIA) inherit classifications without re-querying dictionaries
- Token efficiency: 15,350 tokens → 1,535 effective with caching

---

## Design Decisions (Permanent)

### Full Dictionary Injection
**Decision:** Inject complete Emotion Dictionary (161 entries) + Pain Point Master (336 entries) on every record

**Rationale:**
- Quality priority: zero risk of missing correct classification
- Downstream efficiency: all agents inherit resolved output
- Economic viability: OpenAI 90% cache discount makes this cost-effective

**Alternative rejected:** Pre-filter to ~10 entries (quality risk, marginal token savings)

### GPT vs Claude
**Decision:** Use GPT gpt-5.2 for EIP (not Claude)

**Rationale:**
- OpenAI cache TTL longer and more reliable for batch processing
- Anthropic cache expires in 5 minutes (risky if BCA drip interval >5min)
- Both models perform equally well at structured JSON classification

---

**Instructions for future updates:**

When EIP changes occur, add new entries at the top in this format:

## Chat #[NUMBER] · [DATE]
- [Description of change]
- [Description of change]

Keep entries concise (1-2 lines per change).
