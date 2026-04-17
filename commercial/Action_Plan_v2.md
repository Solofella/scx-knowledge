# SubtextCX Action Plan v2

**Last Updated:** March 2026  
**Status:** Pilot phase active (PAK-001, EDO-001, AJI-001)  
**Next Milestone:** 5-10 external prospect conversations in parallel with pilots

---

## Current State (April 2026)

### Pipeline Status
- **All agents ALA→RDA:** ✓ Complete and verified (151 nodes operational)
- **MRA:** On hold pending pilot data
- **BCA:** Deferred pending volume analysis
- **Pilot scale:** 5-10 reviews/day across 3 clients

### Active Pilots
1. **PAK-001** (Park Avenue Kitchen by David Burke) - Active, brand voice questionnaire sent
2. **EDO-001** (EDO Restaurants) - Confirmed Q2 start
3. **AJI-001** (AJI Ceviche, Florida) - Exploratory

### Infrastructure
- DigitalOcean droplet operational (~$63/mo)
- n8n 2.4.6 + NocoDB self-hosted
- GitHub knowledge base established (token cost reduction)

---

## 2026 Goals

### Primary Objective
**Malta Nomad Residence Permit qualification:** €3,500/mo gross minimum from outside Malta

**Breakdown:**
- SubtextCX target: €2,500/mo minimum
- Solofella (on standby): €1,000/mo supplemental
- **Combined threshold:** €3,500/mo by Q4 2026 or Q1 2027

### Commercial Validation Framework

**No conclusion about SubtextCX viability until:**
1. ✓ At least 5 external prospect conversations completed
2. ⏳ At least 3 pilot feedback cycles received
3. ⏳ At least 1 explicit rejection documented

**Current status:** 3 pilots active, 0 external prospect conversations, 0 feedback cycles complete

**Pricing:** Unvalidated at zero paying clients. Pilots are test opportunities, not traction.

---

## Q2 2026 Priorities (April-June)

### 1. Pilot Execution & Data Collection
- **PAK-001:** Weekly check-ins with Christine (GM), response draft approval workflow active
- **EDO-001:** Onboarding Q2, brand voice questionnaire delivery
- **AJI-001:** Convert from exploratory to committed pilot or close

**Success metrics:**
- Response draft approval rate by tier (T1/T2/T3)
- Time to approval (48hr target)
- Client satisfaction with internal briefs
- Manual workflow friction points

### 2. Dashboard Delivery
- **Freelancer handoff:** Brief delivered (SubtextCX_Dashboard_Freelancer_Brief.pdf)
- **Implementation:** DigitalOcean port 3000, Express.js, Tailwind CSS, Chart.js
- **Target:** Functional dashboard by end Q2 for pilot client visibility

### 3. External Prospect Conversations (5-10 minimum)
**Blocker:** subtextcx.com landing page must be live first

**Target prospects:**
- Independent restaurants (10-25 locations)
- Boutique hotel groups (5-15 properties)
- F&B operators with high review volume (100+ reviews/month)

**Goal:** 5-10 conversations, expect 1-2 explicit rejections (learning opportunity)

### 4. Website Launch
**subtextcx.com landing page:**
- Entry point line: "You're losing repeat customers for reasons your team never sees. We show you exactly where that happens."
- Product positioning: Review management tool with intelligent brief delivery
- Clear differentiation: Dual-instrument masked emotion detection (no B2B SaaS competitor has shipped this at review-analysis level as of March 2026)

**Priority:** Must be live before outbound prospect contact begins

---

## Q3 2026 Priorities (July-September)

### 1. Pilot Feedback Integration
- Implement fixes from pilot feedback cycles
- Refine RDA internal brief quality (governance constraint: detect/interpret only)
- Adjust brand voice variation (reduce to 1 phrase/draft, add variation)

### 2. Auto-Ingestion Build (Phase 1b)
- **Google Business Profile API** integration
- **Yelp Fusion API** integration
- OpenTable partner application submit
- TripAdvisor developer program apply

**Goal:** Eliminate manual CSV upload for pilot clients

### 3. Pricing Validation
- Test pricing tiers with external prospects
- Adjust based on objection patterns
- Validate 2026 Starter Rate ($299-$499/mo) vs standard tiers

### 4. First Paying Client (Stretch Goal)
- Convert 1 pilot to paying client OR
- Close 1 external prospect to paid contract

**Revenue target:** $299-$499/mo minimum (Starter Rate)

---

## Q4 2026 Priorities (October-December)

### 1. Scale to 5 Paying Clients
**Target revenue:** $1,500-$2,500/mo (5 clients × $299-$499 avg)

**Client mix:**
- 2-3 converted from pilots
- 2-3 closed from external prospect pipeline

### 2. MRA Build & Activation
**Trigger:** Sufficient pilot data exists (minimum 30 days of reviews per client)

**Deliverables:**
- 48-hour response draft summary (automated email via Brevo)
- Weekly Monday 7am intelligence brief
- Monthly 1st-day comprehensive report

**Model:** Pure JavaScript, zero LLM calls (cost efficiency)

### 3. Geographic Expansion
**Peru market entry:**
- Google Business Profile API covers Peru
- Spanish language support confirmed (Phase 1)
- Target: 1-2 Peru pilot conversations by end Q4

### 4. Parent Brand Trading Name Decision
**Candidates:** Covero, Hostiq, Serviq, Venuiq, Pelliq, Lumari, Arcta, Vendiq

**Decision factors:**
- .com availability
- Trademark clearance
- Long-term portfolio fit (SubtextCX is Product 1 of planned hospitality tech suite)

**Timing:** Deferred to separate naming session, finalize Q4 2026

---

## 2027 Roadmap (Preliminary)

### Q1 2027: Malta NRP Qualification
**Target:** €3,500/mo gross minimum achieved

**SubtextCX contribution:** 10-12 clients × €250-€350 avg = €2,500-€4,200/mo

**Solofella contribution (if needed):** €1,000/mo from solo living platform resumption OR deferred if SubtextCX exceeds €3,500 alone

### Q2-Q4 2027: Scale & Productization
- Expand to 20-30 paying clients
- Stabilize MRR at €5,000-€7,000/mo
- Evaluate Product 2 build (hospitality tech portfolio expansion)
- Consider freelancer/agency partnerships for client onboarding scale

---

## Strategic Constraints (Permanent)

### 1. Quality Over Speed
**Quality is the primary objective.** Token cost priced into client tiers, not used to justify reduced prompt quality.

### 2. Governance Lock
**SubtextCX detects and interprets only — never prescribes operational actions.**

This is a competitive differentiator and non-negotiable product principle.

### 3. Model-Agnostic Architecture
**Value must come from architecture, knowledge structures, and accumulated data** — not the AI model itself.

Reduces long-term LLM dependency risk.

### 4. Commercial Validation Discipline
**No premature scaling.** 5 external conversations + 3 pilot feedback cycles + 1 rejection required before conclusions about product-market fit.

---

## Key Metrics to Track

### Pilot Phase
- Response draft approval rate (by tier)
- Time to approval (target: <48hr)
- Manual workflow friction points
- Client satisfaction with internal briefs

### External Conversations
- Prospect-to-pilot conversion rate
- Objection patterns (pricing, feature gaps, competitor comparisons)
- Explicit rejections (learning opportunities)

### Revenue
- MRR growth (monthly recurring revenue)
- Client acquisition cost (time + paid marketing if any)
- Churn rate (target: <10% monthly during pilot phase)

### Product
- Pipeline token cost per review (target: <12K with caching)
- API reliability (uptime, error rates)
- Dashboard load time (target: <2s)

---

## Risk Mitigation

### Technical Risks
- **API rate limits:** BCA batch controller design ready, build when volume justifies
- **Token cost blowout:** OpenAI caching + model-agnostic architecture limits exposure
- **Infrastructure failure:** DigitalOcean backup strategy (snapshots weekly)

### Commercial Risks
- **No paying clients by Q4:** Solofella resumption OR Malta NRP timeline extension to Q2 2027
- **Pricing too high:** 2026 Starter Rate ($299-$499) as floor, adjust upward based on value perception
- **Competitor launch:** Dual-instrument emotion detection + governance differentiation defensible

### Operational Risks
- **Founder capacity:** Miguel works at Park Avenue Kitchen while operating SubtextCX — time allocation critical
- **Single point of failure:** All build/ops knowledge centralized — GitHub documentation reduces this risk

---

## Open Questions

1. **Parent brand trading name:** Finalize in separate naming session
2. **subtextcx.com launch timing:** Q2 2026 target, blocks outbound
3. **Freelancer dashboard delivery:** End Q2 target, critical for pilot visibility
4. **Peru market entry timing:** Q4 2026 or defer to 2027?
5. **Product 2 scope:** What is second product in hospitality tech portfolio?

---

**End of Action Plan v2**
