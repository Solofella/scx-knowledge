# SubtextCX Business Plan

**Company:** Solofella LLC  
**Version:** 1.0  
**Date:** March 2026  
**Classification:** CONFIDENTIAL

---

## 1. Executive Summary

SubtextCX is a B2B Operational CX Intelligence platform built for the hospitality and food & beverage industries. Developed under Solofella LLC, it processes guest and customer reviews through an 8-stage AI pipeline to detect explicit, implicit, and masked emotional signals — capabilities that no competing B2B SaaS product has commercially shipped at this level.

**The platform delivers three managed outputs to clients:**

1. **Individual governed response drafts** (with human approval)
2. **Weekly Operational Intelligence Brief**
3. **Monthly Manager Action Summary**

### Target Market

**Primary clients:** Independent boutique hotels (3–15 properties) and multi-location restaurant groups in the US English-language market.

**Pricing:** $1,500–$4,000/month managed service retainer.

**Phase 1 objective:** 3 pilot clients, 3 consecutive clean 25-record pipeline runs.

### Market Opportunity

AI in hospitality projected to grow from **$90M (2023) to $8B+ (2033)** at ~55% CAGR.

The masked emotion / governed intelligence segment is **the market's unclaimed frontier** — confirmed by independent competitive intelligence research.

---

## 2. Company & Product Identity

### Legal Structure

| Element | Detail |
|---------|--------|
| **Legal Entity** | Solofella LLC |
| **Incorporation** | Wyoming LLC |
| **Company Address** | Wyoming (virtual) / Operations: New York metro |
| **Founded** | 2024 |

### Product Profile

| Element | Detail |
|---------|--------|
| **Product Name** | SubtextCX (provisional) |
| **Product Stage** | Prototype / Pre-Pilot |
| **Category** | Operational CX Intelligence |
| **Target Industry** | Hospitality & F&B |

### Differentiation

**Primary Differentiator:**  
Masked and implicit emotion detection (ESS + HSI agents) with governed AI response drafting — the market's commercially unclaimed frontier as of 2026.

**Secondary Differentiator:**  
Dignity-layer governance (T3 BRA) for discrimination, cultural insensitivity, and high-stakes reputation events — no competitor has named or marketed this.

### Intellectual Property Assets

**Emotion Dictionary v4.0:**  
161 entries, hospitality context, 6 locked Need_State values (Safety/Belonging/Autonomy/Competence/Fairness/Recognition).

**Pain Point Master v2.1:**  
336 entries, 13 domains, 13 fields (PP-001→PP-336, zero gaps/duplicates).

### Revenue Model

**Primary:** Managed Service Retainer ($1,500–$4,000/mo)  
**Secondary:** Data licensing (Phase 3)

### Value Statement (LOCKED)

> "We identify the operational failures behind your worst reviews — and tell your managers exactly what to fix."

---

## 3. Product Description — The 8-Agent Pipeline

### Pipeline Architecture
ALA → EIP → ESS → HSI → SIA (parallel) → BRA → RDA → MRA (scheduled)
↓
Client Deliverables

### Agent Details

| Agent | Full Name | Model | Function | Token Cost |
|-------|-----------|-------|----------|------------|
| **ALA** | Audience Listener Agent | GPT gpt-5.2 | Ingests reviews from multiple sources; deduplication hash; keyword extraction | ~400 |
| **EIP** | Emotional Intelligence Processor | GPT gpt-5.2 | 25-field emotional analysis; pain point classification; signal type detection (6 values) | ~1,535 (cached) |
| **ESS** | Emotional Signal Scanner | Claude claude-sonnet-4-6 | Expression mode (6 types); emotional clarity scoring; masked signal detection | ~1,200 |
| **HSI** | Human Signal Interpreter | Claude claude-sonnet-4-6 | Behavioral intent interpretation; masked emotion hypothesis; narrative alignment | ~1,800 |
| **SIA** | Signal Intelligence Aggregator | Pure JavaScript | Parallel batch; cluster patterns; configurable 7/15/30-day windows | 0 |
| **BRA** | Brand Response Architect | Hybrid | T1 (~75%): deterministic template engine; T2/T3: Claude with governance constraint | ~625 avg |
| **RDA** | Response Drafting Agent | Claude claude-sonnet-4-6 | Public response draft + internal operational note; human approval gate | ~3,000 |
| **MRA** | Metrics & Ratio Agent | n8n JavaScript | Scheduled; computes 6 behavioral ratios; feeds Weekly Brief | 0 |

**Total Pipeline Cost:** ~9,560 tokens per record (with caching)

### Key Pipeline Features

**Masked Emotion Detection (ESS + HSI):**  
Identifies emotional signals that guests express indirectly or implicitly — the gap between what they wrote and what they meant.

**Governed Response (BRA T3):**  
Dignity-layer governance for discrimination, cultural insensitivity, and high-stakes reputation events requiring human intervention.

**Human Approval Gate (RDA):**  
100% of response drafts reviewed by human before client delivery — non-negotiable architectural constraint.

**Zero-Cost Aggregation (SIA + MRA):**  
Pure JavaScript processing — scales infinitely at zero marginal token cost.

---

## 4. Business Sections Overview

The complete business plan is organized into 10 detailed sections:

### [Business Model](02_Business_Model.md)
Value creation, delivery, capture — how SubtextCX generates and monetizes value.

### [Market Analysis](03_Market_Analysis.md)
TAM/SAM/SOM, competitive landscape, buyer personas, market validation.

### [Marketing Plan](04_Marketing_Plan.md)
GTM strategy, positioning, outreach sequence, channel mix, content approach.

### [GANTT](05_GANTT.md)
6-month execution roadmap (Mar–Aug 2026): milestones, owners, dependencies.

### [Budget](06_Budget.md)
Monthly investment plan by category for 2026.

### [Costs](07_Costs.md)
Itemized infrastructure, tools, and SGA expense register.

### [Revenue Plan](08_Revenue_Plan.md)
Scenario-based revenue projections (conservative / moderate / aggressive).

### [Operations](09_Operations.md)
Pipeline workflow, client delivery process, QA, and SLA framework.

### [Bilingual Strategy](10_Bilingual_Strategy.md)
English + Spanish market strategy — core asset, not add-on.

---

## 5. Key Milestones & Score Gates

### Milestone Tracking System

SubtextCX uses a score-based progression system to track product-market fit maturity:

| Score | Stage | Criteria |
|-------|-------|----------|
| 6.5 | Pre-pilot | Pipeline architecture complete |
| 7.0 | Stabilized | 3 consecutive 25-record clean runs |
| 7.5 | Pilot launch | First pilot client signed |
| 8.0 | Validated | First deliverable delivered to client |
| 8.5 | Commercial | 3 clients active, MRR ≥ $4,500 |
| 9.0+ | Scale-ready | Investor-grade documentation complete |

### 2026 Milestones

| ID | Milestone | Success Criteria | Target Date | Score Gate |
|----|-----------|------------------|-------------|------------|
| **M1** | Pipeline stabilization | 3 consecutive 25-record clean runs | Apr 2026 | 6.5 → 7.0 |
| **M2** | First pilot client signed | Letter of Intent or contract executed | May 2026 | 7.0 → 7.5 |
| **M3** | First deliverable delivered | Weekly Brief + response drafts to client | Jun 2026 | 7.5 → 8.0 |
| **M4** | 3 clients active | Monthly recurring revenue ≥ $4,500 | Sep 2026 | 8.0 → 8.5 |
| **M5** | Investment documentation ready | Full financial model + pilot evidence | Oct 2026 | Investor-grade |

**Current Status (April 2026):** Score 7.4 — stabilization complete, pilot launch authorized.

---

## 6. Strategic Decisions — LOCKED (March 16, 2026)

The following strategic decisions are **permanent constraints** for 2026 execution:

### DECISION 1 — Commercial Priority 2026 ✓

**SubtextCX is the SOLE commercial priority for Solofella LLC in 2026.**

Solofella Solo Lifestyle is paused indefinitely until SubtextCX reaches 3 paying clients and stable MRR. No development resources, marketing spend, or founder time is allocated to Solofella Solo Lifestyle in 2026.

**Rationale:** Focus wins. Diluted attention across two products guarantees neither succeeds.

---

### DECISION 2 — Company Structure ✓

**Solofella LLC is the holding company. SubtextCX is the operating product.**

Future products (Solofella Solo Lifestyle, additional verticals) are Phase 3+ and do not affect 2026 planning or resource allocation.

**Rationale:** Clean separation between legal entity and product brands. SubtextCX can be sold, licensed, or spun out without unwinding the parent company.

---

### DECISION 3 — Vertical Focus ✓

**SubtextCX serves the Hospitality & F&B vertical exclusively in Phase 1.**

No outreach, no demos, no product development for healthcare, retail, government, or other CX verticals until Phase 2 (2027+). This is a strategic constraint, not a limitation — **depth wins before breadth.**

**Rationale:**
- Miguel's 30 years hospitality experience = authentic buyer credibility
- Emotion Dictionary and Pain Point Master are hospitality-contextualized
- Market validation is faster with vertical focus
- Competing tools are horizontal generalists — vertical depth is the moat

---

### DECISION 4 — Language Strategy ✓

**SubtextCX is designed as a bilingual product (English + Spanish) from the ground up.**

**Phase 1:** English pipeline processing + Spanish-language landing page and marketing.

**Phase 2 (Q3-Q4 2026):** Spanish review processing capability added to pipeline (ESS/HSI Spanish vocabulary extension).

**Phase 3+:** Additional languages added via dedicated language department.

**Rationale:**
- US Hispanic hospitality market (hotel owners, restaurant operators, bilingual guest interactions) is a **primary addressable segment**, not an afterthought
- ~44% of independent restaurant operators in major US cities are Hispanic-owned
- Competing tools are English-only by default — bilingual is a market access advantage
- Spanish capability is NOT a translation feature — it's a market entry strategy

---

### DECISION 5 — 2026 Execution Timeline ✓

**Q1 2026 (Jan-Mar):** Pipeline architecture — n8n rebuild, agent audits, HOW docs.

**Q2 2026 (Apr-Jun):** Pipeline stabilization + landing page + pre-pilot commercial assets.

**Q3 2026 (Jul-Sep):** Pilot client 1 delivery + outreach for Pilots 2 and 3.

**Q4 2026 (Oct-Dec):** 3 clients active, MRR established, investment documentation.

**Revenue starts Q3, not Q2.** The GANTT reflects this sequencing.

**Rationale:** Architecture first, commercialization second. No client conversations before the pipeline is stable.

---

### DECISION 6 — Landing Page ✓

**A SubtextCX landing page is a required 2026 deliverable.**

Must be live **before any outbound prospect contact.** Bilingual (EN/ES).

**Content:**
- Value proposition
- 3 client deliverables
- Pricing tier summary
- 'Request a Demo' CTA
- Founder credibility statement

**Platform:** TBD (Webflow/WordPress/custom)

**Target:** Live by **June 30, 2026** (hard deadline).

**Rationale:** Every prospect will search for "SubtextCX" before taking a call. The landing page is a credibility asset, not a marketing nice-to-have.

---

## Related Documents

- **[Business Model Canvas](02_Business_Model.md)** - Value streams and revenue model
- **[Market Analysis](03_Market_Analysis.md)** - TAM/SAM/SOM and competitive landscape
- **[Marketing Plan](04_Marketing_Plan.md)** - Go-to-market strategy
- **[GANTT Timeline](05_GANTT.md)** - Week-by-week execution roadmap
- **Master Continuity Document** - Pipeline technical documentation

---

**End of Business Plan Section**
