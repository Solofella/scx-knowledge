# Master Continuity Document v7.4

**Last Updated:** Chat #74 · April 4, 2026  
**Current Session:** Chat #104 · April 17, 2026  
**Status:** All agents ALA→RDA complete and verified  
**Next Milestone:** MRA build (on hold pending pilot data)

---

## Pipeline Overview

**Active Flow:** ALA → EIP → ESS → HSI → SIA → BRA → RDA  
**Status:** ✓ All 7 agents operational and verified  
**On Hold:** MRA (awaiting pilot data), BCA (deferred pending volume analysis)

**Node Counts:**
- ALA: 24 nodes ✓
- EIP: 26 nodes ✓
- ESS: 23 nodes ✓
- HSI: 30 nodes ✓
- SIA: 11 nodes ✓
- BRA: 19 nodes ✓
- RDA: 18 nodes ✓

**Total Pipeline:** 151 nodes operational

---

## Active Pilots

| Client ID | Name | Status | Notes |
|-----------|------|--------|-------|
| PAK-001 | Park Avenue Kitchen by David Burke | Active | Brand voice questionnaire sent to Christine (GM) |
| EDO-001 | EDO Restaurants | Confirmed Q2 | Pilot start pending |
| AJI-001 | AJI Ceviche (Florida) | Exploratory | Initial conversations only |

---

## Agent Specifications

### ALA (Acquisition & Language Agent)
- **Version:** v4
- **Model:** GPT gpt-5.2
- **Status:** ✓ Verified operational
- **HOW Doc:** [agents/ALA/SCX_ALA_HOW_v4.md](agents/ALA/SCX_ALA_HOW_v4.md)
- **Changelog:** [agents/ALA/SCX_ALA_CHANGELOG.md](agents/ALA/SCX_ALA_CHANGELOG.md)
- **Function:** Review normalization and language detection
- **Token Budget:** ~400 per record

### EIP (Emotion & Intelligence Preprocessor)
- **Version:** v4
- **Model:** GPT gpt-5.2
- **Status:** ✓ Verified operational
- **HOW Doc:** [agents/EIP/SCX_EIP_HOW_v4.md](agents/EIP/SCX_EIP_HOW_v4.md)
- **Function:** **Knowledge injection layer** - injects full Emotion Dictionary (161 entries) + Pain Point Master (336 entries)
- **Token Budget:** ~15,350 tokens per record (with OpenAI 90% cache discount = ~1,535 effective)
- **Design Rationale:** All downstream agents inherit EIP classifications (zero incremental dict cost)

### ESS (Emotional Signal Synthesis)
- **Version:** v4
- **Model:** Claude claude-sonnet-4-6
- **Status:** ✓ Verified operational
- **HOW Doc:** [agents/ESS/SCX_ESS_HOW_v4.md](agents/ESS/SCX_ESS_HOW_v4.md)
- **Function:** Expression Mode + Emotional Clarity analysis (inherits EIP output)
- **Token Budget:** ~1,200 per record

### HSI (Hospitality Signal Intelligence)
- **Version:** v3
- **Model:** Claude claude-sonnet-4-6
- **Status:** ✓ Verified operational
- **HOW Doc:** [agents/HSI/SCX_HSI_HOW_v3.md](agents/HSI/SCX_HSI_HOW_v3.md)
- **Function:** Behavioral narrative synthesis (inherits EIP output)
- **Token Budget:** ~1,800 per record

### SIA (Signal Intensity Assessment)
- **Version:** v3
- **Model:** None (pure JavaScript)
- **Status:** ✓ Verified operational
- **HOW Doc:** [agents/SIA/SCX_SIA_HOW_v3.md](agents/SIA/SCX_SIA_HOW_v3.md)
- **Function:** Zero-cost aggregation - groups EIP records by Domain + Signal Tier
- **Token Budget:** 0 (no AI calls, pure aggregation)

### BRA (Brand Response Agent)
- **Version:** v3.2
- **Model:** Hybrid (T1=deterministic template, T2/T3=Claude claude-sonnet-4-6)
- **Status:** ✓ Verified operational
- **HOW Doc:** [agents/BRA/SCX_BRA_HOW_v3.2.md](agents/BRA/SCX_BRA_HOW_v3.2.md)
- **Function:** Template selection + response drafting
- **Token Budget:** ~2,500 avg (70-80% T1=0 tokens, 20-30% T2/T3=Claude)

### RDA (Response Drafting Agent)
- **Version:** v3.1
- **Model:** Claude claude-sonnet-4-6
- **Status:** ✓ Verified operational
- **HOW Doc:** [agents/RDA/SCX_RDA_HOW_v3.1.md](agents/RDA/SCX_RDA_HOW_v3.1.md)
- **Changelog:** [agents/RDA/SCX_RDA_CHANGELOG.md](agents/RDA/SCX_RDA_CHANGELOG.md)
- **Function:** Final response draft with brand voice integration
- **Token Budget:** ~3,000 per record

### MRA (Management Reporting Agent)
- **Status:** ⏸ On hold (build prompt ready)
- **Build Prompt:** Saved Chat #75 (SCX_MRA_Build_Prompt_Chat75.docx)
- **Function:** Scheduled intelligence briefs (48hr summary, weekly Mon 7am, monthly 1st 7am)
- **Model:** Pure JavaScript (zero LLM calls)
- **Awaiting:** Pilot data to exist before build

### BCA (Batch Controller Agent)
- **Status:** ⏸ Deferred (Option B design documented)
- **Function:** Drip 10 records at a time from ALA to EIP (prevents API rate limit losses)
- **Design:** Reads from ALA NocoDB table, not CSV upload
- **Priority:** Build after pilot volume analysis (currently 5-10 reviews/day, direct pass-through sufficient)

---

## Pipeline Token Economics

**Per-record cost breakdown (with caching):**

| Agent | Tokens | Notes |
|-------|--------|-------|
| ALA | 400 | Normalization only |
| **EIP** | **1,535** | **15,350 with 90% OpenAI cache discount** |
| ESS | 1,200 | Inherits EIP output |
| HSI | 1,800 | Inherits EIP output |
| SIA | 0 | Pure JavaScript |
| BRA | 2,500 avg | Hybrid (mostly deterministic) |
| RDA | 3,000 | Final draft generation |
| **Total** | **~10,435 tokens/record** | |

**At pilot scale (300 reviews/month):** 3.13M tokens/month

**Key efficiency:** EIP dictionary injection (15,350 tokens) serves entire downstream pipeline. Zero incremental dict cost for ESS, HSI, SIA, BRA, RDA.

---

## Knowledge Structures

### Emotion Dictionary v5.0
- **Entries:** 161
- **Fields:** 7 (Primary_Emotion, Need_State, Intensity_Range, Hospitality_Context, etc.)
- **Need States:** 6 (Safety, Belonging, Autonomy, Competence, Fairness, Recognition)
- **Storage:** NocoDB table `mrrscb955j1d2i7`
- **Usage:** EIP injects full dictionary on every record (~13K tokens with caching)

### Pain Point Master v4.1
- **Entries:** 336
- **Domains:** 13
- **ID Range:** PP-001 through PP-336 (zero gaps/duplicates)
- **Storage:** NocoDB table `meavqh37mdqgl4d`
- **Usage:** EIP injects full master on every record (~20K tokens with caching)

### Schema Registry v2
- **Location:** [schemas/Schema_Registry_v2.md](schemas/Schema_Registry_v2.md)
- **Contains:** All NocoDB table IDs, field IDs, data types, relationships

---

## Architecture Principles (Permanent Locks)

### 1. Governance Rule
**SubtextCX detects and interprets only — never prescribes operational actions.**

Internal briefs must describe signal meaning only. No operational recommendations.

### 2. Knowledge Cost Containment
**EIP = single dictionary injection point.**

All downstream agents (ESS, HSI, SIA, BRA, RDA) inherit resolved classifications. Dictionary cost paid once, benefits entire pipeline.

### 3. Pre-Build Protocol
**Field Traceability Map required before Node 1** of any new agent build.

Every output field must trace to source before pipeline construction begins.

[Protocol: protocols/SCX_PreBuild_Protocol_v1.0.md](protocols/SCX_PreBuild_Protocol_v1.0.md)

### 4. Quality Priority
**Quality is the primary objective.** Token cost should be priced into client tiers, not used to justify reduced prompt quality.

### 5. Model-Agnostic Architecture
Value must come from architecture, knowledge structures, and accumulated data — not the AI model itself.

---

## Commercial State

### Pricing (Unvalidated - Zero Paying Clients)
- Score 7.5: $2,800/mo
- Score 8.0: $3,500–$5,500/mo
- Score 9.0: $4,200–$11,500/mo
- **2026 Starter Rate:** $299–$499/mo (introductory pilot pricing)

### 2027 Goal
**Malta Nomad Residence Permit threshold:** €3,500/mo gross minimum from outside Malta (renewable annually).

SubtextCX + Solofella must collectively reach this threshold.

### Entry Point Line (Locked)
"You're losing repeat customers for reasons your team never sees. We show you exactly where that happens."

---

## Dashboard Specification

**Platform:** DigitalOcean port 3000, Express.js + vanilla HTML, Tailwind CSS via CDN, Chart.js via CDN

**Auth:** Magic link + NocoDB token table

**URL Structure:** `subtextcx.[parentbrand].com/[client-id]` via subdomain with Nginx + Let's Encrypt SSL

**Sections:**
1. Header (client name, period toggle)
2. Signal Pulse (T-NEG/T-AMB/T-POS distribution)
3. Signal Distribution (domain breakdown)
4. Response Draft Status (approved/pending counts)
5. Trend Signals (week-over-week changes)
6. SEO Signal (keyword hits, top keyword, coverage rate, response rate, avg velocity)

**Period Toggle:** This Week / Last Month

**What's NOT shown:** Review text, guest names, draft text, operational recommendations

**Freelancer Brief:** [commercial/Dashboard_Freelancer_Brief.md](commercial/Dashboard_Freelancer_Brief.md)

---

## Approval Workflow (Google Sheet)

**Architecture:** One Google Sheet per client (shared with solofellausa@gmail.com)

**Columns:**
- Date / Platform / Stars / Review / Proposed (protected) / Edited / Status / RDA-ID

**Status Values:** Approved / Modify / Not Accepted

**Integration:** Apps Script onEdit webhook → n8n SCX-Sheet-Approval → PATCH RDA NocoDB

**Miguel marks Published:** Daily manual update in dashboard

**6 locked fixes:** Trigger type, column protection, idempotent PATCH, 7am sync, blank edit logic, confirmation email

---

## Ingestion Architecture

**Phase 1 (Current - Pilot):** Manual CSV upload per client

**Phase 1b (Next):** Auto-ingestion via:
- Google Business Profile API ✓
- Yelp Fusion API ✓
- OpenTable (partner application to submit at partners.opentable.com)
- TripAdvisor (developer program application needed)

**Unified flow:**
Platform adapters → Normalize to Google Sheet (7 cols: Date/Platform/Star Rating/Review Text/Reviewer Handle/Client ID/Ingestion Status) → n8n ALA reads from sheet daily → Dedup at sheet level (Reviewer Handle+Date+Platform)

**Peru market:** Covered by Google Business Profile

---

## Infrastructure

**Hosting:** DigitalOcean droplet `n8n-Solofella`
- Region: NYC3
- OS: Ubuntu 24.04
- RAM: 4GB
- Disk: 50GB
- IP: 161.35.133.49
- Cost: ~$63/mo

**Stack:** n8n 2.4.6 + NocoDB self-hosted

**Docker:** v1 hyphen syntax only (`docker-compose`, never `docker compose`)

**After restart:** `docker network connect n8n_default nocodb` ("already exists" error = safe to ignore)

**n8n Credentials:**
1. xc-token (NocoDB Header Auth)
2. Subtext-CX-Anthropic (Header Auth with manual headers per node: `anthropic-version:2023-06-01` + `Content-Type:application/json`)
3. Subtext-CX-OpenAI (Header Auth: `Authorization: Bearer [key]`)

**NocoDB URL inside n8n:** Always `http://nocodb:8080` (never localhost or external IP)

---

## NocoDB Table IDs

| Agent | Table ID | Key Fields |
|-------|----------|------------|
| ALA | `m57efwbtrvwohhr` | ALA Record ID (auto), Client ID, Review Text, Normalized Text |
| EIP | `mhicpnrahaesxmy` | EIP Record ID, ALA Record ID, Enriched Emotion Tag, Domain |
| ESS | `m5yektnbtxf8evk` | ESS Record ID, Expression Mode, Emotional Clarity |
| HSI | `mb8nv8t3nk6xzed` | HSI Record ID, Hospitality Signal ID, BRA Status field `cl1250sz39sm45l` |
| SIA | `mdn68l4lm609fve` | SIA Record ID, Domain, Signal Tier, Percentage |
| BRA | `mwqejw7swhd2cf4` | BRA Record ID, Response Draft, Error Log `c3crsub617hnuee` |
| RDA | `mr1v67cszcklwns` | RDA Record ID, Response Draft Final, Approval Status |
| Template Library | `mafv9by73ebama7` | Template ID, Tier, Brand Voice Mode |
| Client Config | `m95cmabjfyb94ps` | Client ID, Brand Voice Settings |
| Emotion Dict | `mrrscb955j1d2i7` | 161 entries, 7 fields |
| Pain Point Master | `meavqh37mdqgl4d` | 336 entries, 13 domains |
| Schema Registry | `mqv1znpza948pm9` | All table/field metadata |

**Base ID:** `pq249fix22t3ofv` (all tables)

---

## Open Issues (Post-Chat #74)

1. **BCA Batch Controller** - Option B design documented, build deferred pending volume analysis
2. **RDA Internal Brief Quality** - Must describe signal meaning only, no operational prescriptions
3. **RDA Brand Voice Variation** - Reduce from 3 phrases to 1 per draft, add variation constraint
4. **Parent Brand Trading Name** - Naming session deferred (candidates: Covero, Hostiq, Serviq, etc.)
5. **subtextcx.com Landing Page** - Must be live before outbound prospect contact
6. **MRA Build** - On hold until pilot data exists (5 pre-build items: table creation, RDA record count, email credential via Brevo, client display name, prior-month baseline)

---

## GitHub Knowledge Base (This Repository)

**Purpose:** Reduce token cost for Claude context in build/strategy chats

**Token Savings:**
- Old method: 15,000-35,000 tokens per chat (uploading HOW docs)
- New method: 3,000-7,000 tokens per chat (fetching from GitHub)
- **Savings: ~80% reduction**

**Update Pattern:**
- **Small changes:** Add entry to agent CHANGELOG.md (30 seconds)
- **Major rewrites:** Replace HOW document (rare, quarterly)

**Usage in Chats:**
Fetch from GitHub scx-knowledge:

agents/RDA/SCX_RDA_HOW_v3.1.md
agents/RDA/SCX_RDA_CHANGELOG.md
MCD_v7.4.md


---

## Communication Rule (Permanent)

**Every response must begin with:** `Chat #[N] · [Date]`

No exceptions. No motivational language. No praise heuristics. Factual only. Full code blocks only. One node at a time with confirmation before proceeding.

---

## Key Learnings (Build Sessions)

- Never remove working verified code based on build lessons
- Never make assumptions about data structures without verification
- Always cross-reference output schema against source schema before building
- n8n 2.4.6: no Save button, Publish only. "Execute workflow" = test mode
- Code Node rules: no spread operator, no `for...of` in filters, no `const` in sort functions, `typeof` check before JSON.parse

---

**End of MCD v7.4**
