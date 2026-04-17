# SIA Change Log

## Chat #123 · April 17, 2026
- GitHub repository setup complete
- SIA HOW v3 documentation migrated from Google Drive
- Zero-cost aggregation model documented

## Chat #74 · April 4, 2026
- Full pipeline verification complete
- All 11 nodes operational
- Confirmed SIA uses pure JavaScript (zero AI calls, zero token cost)

## Chat #72 · March 28, 2026
- Validated SIA does NOT access Emotion Dictionary or Pain Point Master
- Confirmed SIA reads EIP NocoDB table directly (no webhook)
- Architecture: SIA = pure aggregation, reads pre-classified EIP output
- Token budget confirmed: 0 per run (no AI APIs called)

---

## Design Decisions (Permanent)

### Zero-Cost Aggregation Model
**Decision:** SIA uses pure JavaScript for all computation (no AI calls)

**Rationale:**
- EIP already classified all emotions and pain points
- SIA only groups, counts, and calculates percentages (deterministic tasks)
- No new insights generated, just statistical rollup
- Zero incremental token cost per review (scales infinitely at zero marginal AI cost)

### Read from EIP Table (Not Webhook)
**Decision:** SIA queries EIP NocoDB table on schedule (daily 6am), not triggered per-review

**Rationale:**
- Aggregation requires entire dataset to calculate percentages
- Trend direction needs current vs prior period comparison
- Batch processing more efficient than per-record webhook

### Aggregation Clusters
**Two-dimensional grouping:**
- **Dimension 1:** Domain (Food Quality, Service Quality, Ambiance, etc.)
- **Dimension 2:** Signal Tier (T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE)

**Output:** Count + percentage per cluster (e.g., "Food Quality | T-NEGATIVE: 12 reviews, 18%")

### Trend Direction Logic
**Comparison to prior period (same date range length):**
- **UP:** Current count >10% higher than prior
- **DOWN:** Current count >10% lower than prior
- **STABLE:** Within ±10% of prior count
- **NEW:** No prior period data (first appearance)

### Why Pure JavaScript (Not Claude/GPT)
**Decision:** Zero AI API usage in SIA

**Rationale:**
- Aggregation is deterministic (same input = same output)
- No interpretation required (just counting and grouping)
- Token cost would be wasted (AI not needed for math)
- Execution speed: <1 second vs ~30 seconds with Claude
- Scalability: Zero marginal cost per additional review

---

**Instructions for future updates:**

When SIA changes occur, add new entries at the top in this format:

## Chat #[NUMBER] · [DATE]
- [Description of change]
- [Description of change]

Keep entries concise (1-2 lines per change).
