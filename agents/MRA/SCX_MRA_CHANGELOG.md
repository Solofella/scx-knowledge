# MRA Change Log

## Chat #133 · April 17, 2026
- GitHub repository setup complete
- MRA HOW v1.0 documentation migrated from Google Drive
- Zero-cost reporting model documented

## Chat #75 · April 13, 2026
- MRA build complete - 8 nodes operational
- Three report types implemented: 48hr summary, weekly brief, monthly report
- Brevo email integration configured
- Magic link token security implemented
- Pure JavaScript architecture confirmed (zero AI calls, zero token cost)

---

## Design Decisions (Permanent)

### Zero-Cost Reporting Model
**Decision:** MRA uses pure JavaScript for all report generation (no AI calls)

**Rationale:**
- Reporting task is data aggregation and formatting (deterministic)
- SIA already provides aggregated metrics (MRA just queries and reformats)
- No interpretation needed (just presenting numbers)
- Zero incremental token cost per report (scales infinitely)

**Same architecture as SIA** - proven zero-cost aggregation model

### Three Report Cadences
**Decision:** Daily 48hr summary, weekly Monday brief, monthly 1st-day report

**Rationale:**
- 48hr summary: Keeps clients informed of immediate activity
- Weekly brief: Provides trend context without overwhelming
- Monthly report: Comprehensive analysis with month-over-month comparison

**All three delivered via email** with magic link to dashboard

### Magic Link Security
**Decision:** Time-limited tokens (7-day expiration) for dashboard access

**Rationale:**
- No password management burden on clients
- Secure one-time access from email
- Token tied to specific client ID (can't access other clients' dashboards)

### Email Service: Brevo
**Decision:** Brevo (formerly Sendinblue) for transactional email delivery

**Rationale:**
- Free tier: 300 emails/day (sufficient for 10 clients × 3 reports/day = 30 emails)
- Transactional focus (not marketing spam)
- Simple REST API integration
- Delivery tracking built-in

### SLA Target: 90% Within 48 Hours
**Decision:** Track and report % of response drafts approved within 48hr of review posting

**Rationale:**
- Industry standard response time expectation
- Measurable client performance metric
- Dashboard displays compliance rate

---

**Instructions for future updates:**

When MRA changes occur, add new entries at the top in this format:

## Chat #[NUMBER] · [DATE]
- [Description of change]
- [Description of change]

Keep entries concise (1-2 lines per change).
