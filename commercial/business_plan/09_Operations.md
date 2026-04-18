# SubtextCX Operations Plan

**Client Delivery Workflow, QA Framework, SLA Structure | Phase 1 Managed Service**

---

## 1. Service Delivery Overview

SubtextCX operates as a **fully managed B2B service**. The client provides review access. SubtextCX runs the 8-agent pipeline, produces 3 client-facing deliverables, and delivers them on a defined schedule.

**No client-side technical setup is required** beyond granting review platform access.

### Pipeline Flow
ALA (ingestion) → EIP (emotional processing) → ESS (signal scan) → HSI (interpretation)
↓
SIA (pattern aggregation, parallel) → BRA (response architecture) → RDA (draft + operational note)
↓
Human Approval Gate → Client Delivery → MRA (scheduled ratios, Monday morning)

### Human Approval Gate (Non-Negotiable)

**Every RDA output — public response draft + internal operational note — passes through human review before any client delivery.**

This is a **non-negotiable architectural constraint** and a core product feature.

---

## 2. Client Deliverables (3 Outputs)

### Deliverable 1: Individual Review Response + Operational Note

**Cadence:** Per review batch (typically 2–3× per week)

**Agent:** RDA (Response Drafting Agent)

**Format:**
- Public-facing response draft (human-approved before sending)
- Internal operational note (staff/process insight)
- Response on top, op note below

**Delivery:** Email to GM/Owner

**Example format:**
FROM: SubtextCX hello@subtextcx.com
TO: gm@parkavenuekitchen.com
SUBJECT: Response Ready for Review: 5-Star Google Review from Michael K.

RESPONSE DRAFT (Ready to Publish):
"Michael, thank you for taking the time to share your experience with the scallops.
We're delighted you enjoyed them. Our team at Park Avenue Kitchen takes pride in
sourcing the finest ingredients and preparing each dish with care. Your feedback
about the sear reminds us why we do what we do — creating moments that matter for
our guests. We hope to welcome you back soon."

INTERNAL OPERATIONAL NOTE (For Your Team):
This guest experienced high satisfaction with Food Quality (specific praise for
searing technique). No operational signals detected. This is a retention anchor —
positive word-of-mouth likely. Consider sharing this feedback with your kitchen team
as recognition of technique excellence.

APPROVE / MODIFY / DECLINE buttons for your response

---

### Deliverable 2: Weekly Operational Intelligence Brief

**Cadence:** Every Monday morning (client timezone)

**Agent:** SIA + MRA aggregation

**Content:**
- Top emotional failure signals this week
- Operational risk areas (patterns across reviews)
- 3 recommended manager actions
- Trend vs. prior week

**Format:** 1-page PDF or formatted email

**Status:** ⚠️ **FORMAT NOT YET DESIGNED** — BLOCKER before pilot

**Example outline (placeholder):**
PARK AVENUE KITCHEN — WEEKLY INTELLIGENCE BRIEF
Week of April 15–21, 2026
TOP SIGNALS THIS WEEK:

Service Quality (Negative) — 3 reviews mention wait times despite reservations
Food Quality (Positive) — 5 reviews praise specific dishes
Ambiance (Mixed) — Noise level complaints in 2 evening service reviews

OPERATIONAL RISK:
Pattern: Weekend service scaling issue. Friday/Saturday waits exceed guest expectations
despite reservation system. Staff may be understaffed or routing delays at door.
WHAT YOUR TEAM CAN DO:

Review reservation-to-table time metrics for Fri/Sat
Brief host stand on managing expectation setting during peak
Consider capacity adjustment or pre-arrival communication protocol

TREND:
↑ UP: Service consistency improving (Mon–Thu response times stable)
↓ DOWN: Weekend peaks becoming reputation risk

---

### Deliverable 3: Monthly Manager Action Summary

**Cadence:** 1st of each month

**Agent:** BRA action aggregation

**Content:**
- Operational patterns observed last month
- Suggested protocol changes
- Staff touchpoints (coaching opportunities)
- Correlation with rating movement

**Format:** 2-page report

**Status:** ⚠️ **FORMAT NOT YET DESIGNED** — complete before Month 2 of pilot

**Example outline (placeholder):**
PARK AVENUE KITCHEN — MONTHLY ACTION SUMMARY
March 2026
PATTERNS OBSERVED:

Service consistency (83% positive sentiment, ↑ from 78% in Feb)
Food Quality (91% positive, stable)
Staff responsiveness (86% positive, new high)

RECOMMENDED PROTOCOL CHANGES:

Maintain Fri/Sat staffing at current levels (working)
Extend host-stand briefing (seeing results in reservation management)
Consider food pairing suggestions (guest requests increasing)

STAFF COACHING OPPORTUNITIES:

Kitchen: Technique feedback from guests (continue current practice)
Front-of-house: Upselling opportunities (guests mentioning budget availability)
Management: Reservation system performing well (no changes needed)

RATING CORRELATION:

Overall property rating: 4.6 (Mar) vs 4.4 (Feb)
Review velocity: 24 reviews (Mar) vs 19 reviews (Feb)
Repeat guest mentions: 8 in Mar vs 4 in Feb

This improvement trajectory suggests operational changes are working.

---

## 3. Client Onboarding Process

### Day-by-Day Onboarding Timeline

| Day | Activity | Detail |
|-----|----------|--------|
| **Day 0** | Contract signed | Pilot agreement executed. Data rights and exit clause confirmed. |
| **Day 1–2** | Access setup | Client provides: Google Business Profile access, TripAdvisor login or export, Booking.com review feed URL. ALA configured for client's review sources. |
| **Day 3–5** | Historical batch run | ALA ingests 30–90 days of historical reviews. First pipeline run. Internal QA check. No client delivery yet. |
| **Day 5–7** | Calibration call | 30-min call with GM. Review sample output. Confirm brand voice preferences for RDA. Flag any dignity-sensitive history. |
| **Day 7** | First Weekly Brief delivered | First Deliverable 2 sent to client. Introduce the format. Collect immediate feedback. |
| **Day 7+** | Ongoing cadence | Weekly Brief every Monday. Individual response drafts within 48h of batch. Manager Action Summary at Day 30. |
| **Day 28–30** | Pilot close | Feedback call. Case study interview. Renewal or contract conversion offer. |

---

## 4. SLA Framework

### Service Level Agreements

| SLA Item | Standard | Commitment Level |
|----------|----------|------------------|
| **Response Draft Delivery** | Within 48 hours of review batch ingestion | **COMMITTED** |
| **Weekly Brief Delivery** | Every Monday 8:00 AM (client timezone) | **COMMITTED** |
| **Human Approval Gate** | 100% — no response published without human review | **NON-NEGOTIABLE** |
| **Pipeline Uptime** | ≥95% weekly availability (Phase 1 target) | **TARGET** |
| **Emergency / Dignity-Risk Response** | T3 reviews flagged for same-day human review | **COMMITTED** |
| **Monthly Action Summary** | Delivered by 1st of month for prior month | **COMMITTED** |
| **Client Feedback Response** | Within 24 hours during business days | **COMMITTED** |

---

## 5. QA & Error Handling

### Pipeline QA Gate

Each pipeline run checks for:
- EIP signal certainty <0.65 (Fix 2 flag) — escalate
- ESS/HSI null outputs — log error
- BRA tier misclassification — review manually
- RDA empty draft — mark for review

**No output reaches client without passing QA gate.**

---

### Stabilization Gate (Hard Stop)

**Requirement:** 3 consecutive 25-record clean runs before any client receives pipeline output.

**This is a hard constraint** — no exceptions.

**Status (Apr 2026):** Gate passed. Pilot launch authorized.

---

### Human Review Layer

**Every RDA output reviewed by Miguel before delivery.**

Dignity-risk T3 flags require same-day review regardless of schedule.

**Rationale:** Human approval is structural. The product is the human + AI collaboration, not the AI alone.

---

### Error Log

**All pipeline errors logged in NocoDB Error Log field.**

**LangSmith Plus (post-stabilization)** adds trace-level observability.

**Monthly error report** to identify systemic issues:
- Which agents have highest failure rate?
- What are the failure patterns?
- Do they require prompt adjustment or architectural fix?

---

### Client Issue Escalation

**Any client-reported quality issue investigated within 24h.**

Root cause documented. Pipeline fix prioritized if systemic.

**Example:**
- Client reports: "Response draft for Review #47 is blank"
- Investigation: RDA agent failed to generate output (Claude API timeout)
- Action: Regenerate draft, add retry logic to RDA pipeline
- Follow-up: Test retry logic with 5-record batch

---

### Data Retention

**All NocoDB records retained indefinitely.**

Processed records are the core data asset — never deleted.

**Rationale:** Every record is a labeled emotional signal. At 10,000+ records, this becomes an independent commercial asset (dataset licensing opportunity).

---

## 6. Critical Pre-Pilot Blockers

### Blocker Status (As of April 2026)

| ID | Blocker | Status | Required Action | Impact |
|----|---------|--------|-----------------|--------|
| **B1** | Pipeline Stabilization | ✅ **RESOLVED** | 3 consecutive 25-record clean runs passed (Apr 2026). | Gate removed, pilot launch authorized. |
| **B2** | Value Proposition Lock | ⚠️ **PENDING** | One sentence value prop must be confirmed and appear consistently in all materials. | Messaging inconsistency if not locked. |
| **B3** | Category Identity Lock | ⚠️ **PENDING** | "Operational CX Intelligence" must be locked and used uniformly. | Brand confusion without clear category. |
| **B4** | RDA Operational Note Format | ⚠️ **PENDING** | Delivery format (email template, PDF, Notion page?) not yet specified. | Client confusion on how to consume operational intelligence. |
| **B5** | Weekly Brief Format (D2) | ⚠️ **PENDING** | Deliverable 2 format and template must be designed before pilot. | **CRITICAL — Cannot deliver to client without this.** |
| **B6** | Manager Action Summary (D3) | ⚠️ **PENDING** | Deliverable 3 format — can complete in Pilot Month 1 but must design before Month 2. | Month 1 deliverable can proceed; Month 2 requires format lock. |
| **B7** | MRA Rendering Layer | ⚠️ **PENDING** | MRA produces data. No UI layer exists. Client demo impossible without visualization. | Cannot show metrics to prospects without dashboard. |
| **B8** | Pitch Deck Pricing Fix | ⚠️ **PENDING** | Known pricing inconsistency in slides 9/10. Must fix before any prospect meeting. | Credibility damage if prospects see conflicting prices. |
| **B9** | GPT Model Name Verification | ⚠️ **PENDING** | gpt-5.2 in deployed code unconfirmed. Verify before n8n rebuild. | Model name mismatch could break ALA/EIP workflows. |
| **B10** | Pilot Contract Template | ⚠️ **PENDING** | Terms, deliverables, exit clause, IP/data rights. Required before signing. | Legal risk if Pilot 1 signed without clear terms. |

### Unblock Priority

**Must resolve before Pilot 1 signed:**
- B2 (Value Prop)
- B5 (Weekly Brief Format)
- B10 (Pilot Contract)

**Must resolve before Pilot 1 delivery (Day 7):**
- B4 (RDA Operational Note Format)
- B6 (Manager Action Summary format can wait until Month 2)

**Can resolve post-pilot:**
- B7 (MRA Rendering — dashboard for Pilots 2+3)
- B8 (Pitch Deck)
- B9 (GPT Model — verify in test batch)

---

## 7. Operations Governance

### Approval Authority

| Decision | Authority | Input Required |
|----------|-----------|-----------------|
| **Pilot Contract Terms** | Miguel (founder) | Legal review |
| **Client SLA Adjustment** | Miguel | Business case |
| **Pipeline Error Escalation** | Miguel | Error log + impact analysis |
| **Data Deletion Request** | Miguel | Legal review (ensure GDPR compliant if applicable) |
| **New Client Onboarding** | Miguel | Qualification criteria check |

---

### Monthly Operations Review

**Every month, review:**

1. **Client satisfaction:** Feedback from all active pilots
2. **Pipeline performance:** Error rate, uptime, processing time
3. **SLA compliance:** % of drafts delivered within 48h, etc.
4. **Data quality:** Signal certainty scores, ESS/HSI variance
5. **Cost per client:** Actual AI spend vs. budget

**Action items from review flow to next month's planning.**

---

## 8. Scaling Operations (Phase 2 — 2027+)

### Hiring Trigger: Operational Bottleneck

**Current:** Miguel handles all operations (pipeline management, client communication, QA)

**Hiring trigger:** 5+ concurrent clients with <2 hour per day QA time per client

**First hire:** Operations Manager or QA Specialist

**Responsibility:** Client communication, SLA monitoring, error investigation

---

### Automation Trigger: Process Optimization

**Current:** Manual weekly brief delivery, status updates

**Automation opportunity:** Build MRA rendering layer (dashboard) to auto-publish metrics

**Expected savings:** 3–5 hours/week per client

---

### Client Success Framework (Post-Pilot)

**Once 3+ clients active, implement:**

1. **Weekly check-in call** (30 min, async option available)
2. **Monthly business review** (60 min, discuss trends + recommendations)
3. **Quarterly planning** (90 min, next quarter strategy + budget planning)

**Investment:** 2–3 hours per client per month

---

## Related Documents

- **[Business Model](02_Business_Model.md)** - Service delivery channels and customer relationships
- **[Marketing Plan](04_Marketing_Plan.md)** - Pilot offer design and onboarding
- **[GANTT](05_GANTT.md)** - Timeline for deliverable design
- **[Budget](06_Budget.md)** - Operational cost structure
- **[Master Continuity Document](../../MCD_v7.4.md)** - Full pipeline technical documentation

---

**End of Operations Section**
