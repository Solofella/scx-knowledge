# SubtextCX Main Business Plan

**Version 2.0 | March 2026 | CONFIDENTIAL**

**Solofella LLC**

---

## Executive Summary

Subtext CX is a specialized B2B review intelligence platform developed under Solofella LLC, specifically engineered for the hospitality and food & beverage industries. The platform differentiates itself through a proprietary 8-stage AI pipeline designed to detect explicit, implicit, and masked emotional signals — nuances consistently missed by standard sentiment analysis tools. 

By transforming raw guest reviews into governed response drafts and structured intelligence reports delivered on a weekly and monthly cadence, Subtext CX bridges the gap between automated efficiency and the essential human touch required for brand reputation management.

### Core Promise

> **Read what your guests meant. Protect every response. Know where you're losing them.**

Subtext CX is a review intelligence platform that detects the masked emotions behind polite hospitality reviews, delivers human-approved response drafts that protect brand voice and dignity-risk situations, and produces weekly and monthly intelligence reports that show exactly where guest friction is growing — before it becomes revenue loss. 

Built for boutique hotels and restaurant groups in the US, Peru, and other English- and Spanish-speaking regions. **No AI response reaches the public without human sign-off.** 

CX leaders who act on this intelligence grow 4–8% above their industry average.

---

## Value Proposition

**"We identify the operational failures behind your worst reviews — and tell your managers exactly what to fix."**

---

## Product Architecture — 8-Stage AI Pipeline

The technological core of Subtext CX is its 8-stage pipeline, which moves beyond keyword detection to understand what guests actually meant. This is critical in hospitality, where polite phrasing often masks significant operational failures.

### Pipeline Agents

| Agent | Full Name | Model | Function |
|-------|-----------|-------|----------|
| **ALA** | Audience Listener Agent | GPT | Ingests reviews from Google, TripAdvisor, Booking.com and other sources. Dedup hash, keyword extraction, batch verification. Language-agnostic — processes EN and ES. |
| **EIP** | Emotional Intelligence Processor | GPT | 25-field emotional analysis. Pain Point classification across 10 hospitality domains (Pain Point Master v2.1). Signal Type detection (6 values). ~4,100–4,300 tokens/record. |
| **ESS** | Emotional Signal Scanner | Claude | Expression mode detection: Explicit / Implicit / Masked / Performative / Conflicted / Absent. Emotional Clarity scoring. Masked signal detection — the pipeline's core differentiator. |
| **HSI** | Human Signal Interpreter | Claude | Behavioral intent interpretation. Masked emotion hypothesis. Narrative alignment scoring. Identifies gap between surface language and underlying guest experience. |
| **SIA** | Signal Intelligence Aggregator | GPT | Parallel batch — does NOT gate BRA. Cluster pattern detection across configurable 7/15/30-day windows. Feeds Weekly Brief data. Reads directly from EIP output. |
| **BRA** | Brand Response Architect | Hybrid | T1 (~75%): deterministic template engine — no LLM. Seed = hash(HSI_Record_ID). T2/T3 (~25%): Claude with governance-constrained prompt. T3 = dignity-risk reviews. |
| **RDA** | Response Drafting Agent | Claude | Produces: (1) public response draft + (2) internal operational note. Both pass Human Approval Gate before any client delivery. 48-hour SLA. |
| **MRA** | Metrics & Ratio Agent | n8n JS | Scheduled n8n workflow — NOT an LLM. Computes 6 behavioral ratios on schedule. Feeds Weekly Brief and Monthly Report. Language-agnostic arithmetic output. |

---

## Three Managed Outputs

### Output 1: Individual Governed Response Drafts

For every positive or negative guest or customer review, the pipeline produces a response draft calibrated to the specific emotional signal, pain point category, and risk level of that review — including elevated governance for dignity-risk situations such as discrimination allegations or cultural insensitivity complaints. 

Every draft is reviewed and approved by a human before publishing. The client's brand voice is protected. **No AI-generated response reaches the public without human sign-off.**

### Output 2: Weekly Review Intelligence Brief (Every Monday Morning)

The Brief summarizes what the prior week's reviews revealed: the top pain point domains by frequency, emotional signal distribution across the week, trend comparison versus the prior week, and the three review signals most deserving of management attention. 

The manager receives structured guest signal intelligence in a format he can act on. What he does with that intelligence is his decision, made with his operational expertise and his knowledge of his team.

### Output 3: Monthly Review Pattern Report (1st of Each Month)

A 30-day structured summary of review patterns: which service categories generated the most guest friction, how emotional signal distribution shifted across the month, dignity-risk events flagged during the period, and a month-over-month comparison with ratios and trend lines. 

The report provides the guest language evidence a manager needs to have specific, credible conversations with his team — grounded in what guests actually experienced and expressed, not in assumptions about service quality.

#### Note on Evolution

As the client data asset deepens over time, Subtext CX is designed to evolve toward operational management intelligence — cross-branch pattern comparison, brand-standard gap analysis, and staff coaching evidence packages. These are Phase 2 and Phase 3 capabilities, built on the review management foundation established in Phase 1.

---

## Implementation Roadmap — Phases of Growth

### Phase 1 — Technical Stabilization (Q1–Q2 2026)

**Timeline:** Q1–Q2 2026

**Description:** The full 8-agent pipeline completes a minimum of 50 real hospitality reviews from one branch of one brand — processed from start to finish with zero errors and passing internal quality standards. If the run fails, it is repeated until the standard is met. This gate must pass before any client receives pipeline output. It is the minimum evidence the system is stable enough to be trusted with a real client's public reputation. EDO Restaurants' Pilot 0 review data serves as the stabilization run.

---

### Phase 2 — Three Pilot Clients (Live System) (Q3 2026)

**Timeline:** Q3 2026

**Description:** Three clients actively receiving all three managed outputs through the live pipeline. Real delivery, real feedback, real case study generation. This phase produces the documented client outcomes required to activate standard 2027 pricing and support investment or loan documentation.

---

### Phase 3 — Self-Serve Product Layer (2027)

**Timeline:** 2027

**Description:** Clients access outputs directly without managed delivery. The pipeline runs on a client-configured schedule. The human approval gate remains but is managed by the client's team rather than the Subtext CX team. This phase enables scale without proportional increase in founder time.

---

## Strategic Decisions — LOCKED (March 2026)

### Decision 1: Commercial Priority 2026 ✓

**SubtextCX is the SOLE commercial priority for Solofella LLC in 2026.**

Solofella Solo Lifestyle is paused indefinitely until SubtextCX reaches 3 paying clients and stable MRR. No development resources, marketing spend, or founder time is allocated to Solofella Solo Lifestyle in 2026.

---

### Decision 2: Company Structure ✓

**Solofella LLC is the holding company. SubtextCX is the operating product 2026.**

---

### Decision 3: Vertical Focus ✓

**Hospitality & F&B only — Phase 1. No healthcare, retail, or government.**

---

### Decision 4: Language Strategy ✓

**English + Spanish by design from launch. More languages Phase 3+.**

---

### Decision 5: 2026 Timeline ✓

**Q1–Q2 architecture. Q3 pilot launch. Q4 scale + investment docs.**

---

### Decision 6: Landing Page ✓

**subtextcx.com live EN+ES by June 30, 2026. Hard gate on outbound contact.**

---

### Decision 7: Product Name ✓

**Subtext CX (under Solofella LLC). SENTILO name rejected — active trademark conflict.**

---

### Decision 8: Value Proposition ✓

**Read what your guests meant. Protect every response. Know where you're losing them.**

---

### Decision 9: Primary Segment ✓

**Boutique hotels (3–15 props) + multi-location restaurant groups. US + Peru.**

---

### Decision 10: Team Structure 2026 ✓

**Solo founder. Outsource: landing page build, Spanish calibration, legal review, accounting.**

---

### Decision 11: Product Positioning ✓

**Review management tool with intelligent brief delivery. NOT operational management tool (Phase 2+).**

---

### Decision 12: One-Sentence VP (Landing Page) ⏳

**Pending — capability framing vs. protection framing. Not a document blocker.**

---

## Market Context & Validation

### TAM, SAM, SOM

| Metric | Scope | Market Size | Growth | Source |
|--------|-------|-------------|--------|--------|
| **TAM** | AI in Hospitality (Global) | $90M (2023) → $8B+ (2033) | ~55% CAGR | Independent research |
| **SAM** | US hospitality AI CX intelligence (boutique + mid-market) | ~$800M by 2028 est. | Submarket of TAM | US market share (~10% global) |
| **SOM** | Phase 1: 50 boutique/restaurant group clients by Y3 | $900K–$2.4M ARR (Y3 target) | Conservative-to-moderate | $1,500–$4,000/mo × 50 clients |

### Market Validation Status

| Validation | Status | Evidence |
|-----------|--------|----------|
| **V1 — Market Size** | ✓ **CONFIRMED** | AI in hospitality $90M→$8B+ by 2033 (~55% CAGR). Market is real and accelerating. |
| **V2 — Governed Response Demand** | ✓ **CONFIRMED** | Medallia Smart Response, MARA AI, Revinate Ivy, Customer Alliance all compete on governed response. Buyer expectation confirmed. |
| **V3 — Masked Emotion = Unclaimed Frontier** | ✓ **CONFIRMED** | No current B2B SaaS tool has shipped masked sentiment detection at review-analysis level. Academic/neuromarketing only. |
| **V4 — Boutique/Independent Segment Underserved** | ✓ **CONFIRMED** | Enterprise tools require consulting. Plug-and-play tools are tactical. No managed white-glove option for 3–15 property operators. |
| **V5 — Pain Point Classification Commoditized** | ✓ **CONFIRMED** | TrustYou (700+ cats), Customer Alliance, GuestRevu all do this. SubtextCX value must sit above taxonomy layer. |
| **V6 — Pricing Range Viable** | ⚠️ **ESTIMATED** | $1,500–$4,000/mo sits in accessible range for target segment. Requires pilot validation. |
| **V7 — Pipeline Performance** | ⚠️ **PENDING** | Zero production records processed. Stabilization gate: 3 consecutive 25-record clean runs required before pilot. |

---

## Confirmed Cost Baseline (March 2026 Actuals)

| Item | Cost | Frequency | Status |
|------|------|-----------|--------|
| DigitalOcean droplet (n8n + NocoDB self-hosted) | $12.00/month | Monthly | ACTIVE — CONFIRMED |
| OpenAI ChatGPT Plus + token usage | $30.00/month | Monthly | ACTIVE — CONFIRMED |
| Bluehost Email | $97.95/year ($8.16/mo) | Annual | ACTIVE — CONFIRMED |
| Bluehost Domain (subtextcx.com) | $49.18/year ($4.10/mo) | Annual | PENDING — register immediately |
| Accounting + Taxes — Nazca INC | $200/year ($16.67/mo) | Annual | ACTIVE — CONFIRMED |
| LangSmith Plus (observability) | $39/month | Monthly | PLANNED — activate post-stabilization |
| Anthropic Claude Pro plan | $100.00/month | Monthly | UPGRADE REQUIRED before Pilot 0 |
| **TOTAL CONFIRMED MONTHLY FIXED COST** | **$170.93/month** | | |

**Break-even:** 1 client at $750/mo introductory rate covers all fixed costs

### 2026 One-Time Outsourcing Budget

| Item | Cost | Timeline |
|------|------|----------|
| Landing page build — Webflow developer (EN+ES) | ~$400 | Q2 May–Jun |
| Spanish linguistic calibration — bilingual contractor | ~$750 | Q2–Q3 Jun–Jul |
| Pilot contract legal review — attorney ~1hr | $300 | Q2 April |
| Pitch deck / one-pager design — Canva DIY or Fiverr | ~$150 | Q2 April |
| **TOTAL 2026 ONE-TIME OUTSOURCING** | **~$1,600** | |

---

## Milestone Score Gates

Current score: **6.5/10**

The score reflects construction progress, not proven product. The pipeline is architecturally sound and the market position is real. The score moves when specific, verifiable milestones are passed — not when documents are updated.

| Milestone ID | Milestone | Target | Success Criteria | Score Gate |
|--------------|-----------|--------|------------------|------------|
| **M0** | Pipeline stabilization gate | Q2 Apr–May 2026 | 50 real reviews, 1 branch, 1 brand — zero errors, all 3 runs pass | 6.5 → 7.0 |
| **M1** | EDO Pilot 0 delivered | Q2 May–Jun 2026 | Results delivered to EDO Ops Manager. Feedback call completed. Peru pricing anchor obtained. | 7.0 → 7.2 |
| **M2** | Pilot 1 signed (US) | Q3 Jul 2026 | Letter of intent or contract executed with first US paying client | 7.2 → 7.5 |
| **M3** | Pilot 1 delivered | Q3 Aug 2026 | 4× Weekly Briefs + response drafts + Monthly Report delivered. Case study drafted. | 7.5 → 8.0 |
| **M4** | 3 clients active (MRR $4,500+) | Q3–Q4 Sep 2026 | Three clients receiving live pipeline outputs. Monthly recurring revenue ≥ $4,500. | 8.0 → 8.5 |
| **M5** | Investment documentation ready | Q4 Oct–Nov 2026 | Financial model with actuals + pilot case studies + pitch deck (investment version) | 8.5 → 9.0 |

---

## Document Authority

This document is the rule base for **Subtext CX Action Plan v5 (2026–2027).** 

All downstream sheets — Business Model, Market Analysis, Marketing Plan, GANTT, Budget, Costs, Revenue Plan, Operations, Bilingual Strategy — derive from the decisions locked here. 

**Any change to this sheet requires explicit approval and version increment.**

---

**End of Main Business Plan**
