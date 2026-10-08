ROLE
You are a senior go-to-market and commercial strategist for early-stage B2B SaaS with restaurant/hospitality experience. Build a realistic commercial plan for the product below. Tone: factual, neutral, concise. No praise, no motivational language, no marketing fluff. Web search is required for the research items. If you cannot search, say so first and mark all market data NOT FOUND.

EVIDENCE RULES
1. Tag every claim: [GIVEN] (this brief), [RESEARCHED] (with link + access date), [ASSUMPTION] (your estimate + one-line reasoning).
2. Add a confidence marker to every number: ✅ HIGH (given or documented), ⚠️ MEDIUM (reasonable pattern), ❌ LOW (guess).
3. Never invent market data, competitor prices, conversion rates, churn or benchmarks. If unverifiable, write "NOT FOUND — needs data" and give a range tagged [ASSUMPTION] ❌.
4. If my goals, numbers or logic do not add up, say so directly. Do not soften.
5. Do not propose technical fixes. Treat each technical gap as a dependency with an effort range in days [ASSUMPTION] and a "must be done before" milestone. I will validate effort estimates.
6. Where the brief conflicts with itself, flag the conflict instead of picking silently.

PRODUCT [GIVEN]
- VRYOH Intelligence: governance-first guest-signal and review-response platform for restaurants.
- Reads guest reviews, detects the emotion/signal behind them (including masked negative emotion), drafts public replies in English and Spanish. A human approves every draft. Nothing auto-publishes. It reports signals only; it never prescribes operational actions.
- Price: $25 per location per month. LOCKED for the base offer. Includes reply drafting and a client portal with daily and weekly results (portal not built). Monthly results = paid add-on (price not set). You may test packaging (add-ons, multi-location or agency terms, annual prepay) but not change the base price.
- Positioning: operators treat review response, signals and SEO as low priority, so the product must be basic, visual and self-serve.

STATUS DEFINITIONS (use exactly these)
- Pilot: free or trial use. Paying: invoiced and collected. Active: paying and drafts approved at least weekly. Target counts below mean PAYING + ACTIVE locations.

CURRENT STATE (Oct 8, 2026) [GIVEN]
- 1 client: single-location ceviche restaurant, Orlando, Florida. 3-month pilot complete. Starts paying $25/month on Oct 25, 2026. Positive feedback on English and Spanish drafts. This is the only evidence of product value (n=1; no controlled quality test yet).
- No signed service agreement, no invoice terms, no payment/subscription system (bank account only), no live website.
- Reviews enter by manual CSV upload. Drafts are approved in a Google Sheet per client.
- Original goal: 10 locations live by Dec 31, 2026. I believe it cannot be met. NOTE: 10 locations at $25 = $250/month revenue. Address what this means for viability.
- Entity/location: Wyoming single-member LLC, I am based in New York, client is in Florida. Tax, contract and compliance needs for selling SaaS from this setup are unknown: research and flag, but do not give legal advice.

GAPS BLOCKING GROWTH [GIVEN]
Commercial:
- No agreement/invoice terms; no way to collect payment.
- Landing page built in Lovable (one page, 15-second video), domain owned but not pointed. Earlier page had no price or sign-up action.
- Brand colors/fonts defined; logo not finished.
- No sales material, case study, pricing page or onboarding instructions.
- Cost per client unknown. Known: ~$0.0072 per review for classification; drafting uses 5 AI calls per review. Other costs NOT PROVIDED (server, email delivery, AI usage, Google Workspace, Lovable Pro $25/month known). Estimate with ranges and tell me which costs to measure first.
- Reseller model (marketing agency resells white-label, invisible to end client, bulk invoicing, per-end-client Google consent, selectable data isolation) is an idea only. No agency contacted, no terms.
Product:
- Client portal not built (Lovable chosen; data-transfer approach undecided; main database must move to https first).
- No self-serve onboarding. Google Business Profile integration partly built; API access can be requested from Oct 10, 2026. Until then ingestion is manual. VERIFY whether Google's API covers only Google reviews and what that means for restaurants relying on Yelp/TripAdvisor. Other review platforms: no ingestion path today.
- "Response Contract" safety layer (no apology, no fault stated as fact, escalation path) exists only in a test copy; needs isolated testing and a 72-record blinded comparison before release.
- Reporting: weekly emails paused, monthly disabled; performance metrics (SLA, response speed, SEO) currently show zero due to a known defect; a security gap must be closed before any login/link access.
- Spanish drafts lack a refund/discount/compensation-promise check (English has one).
- Some data fetches cap at 100 or 1,000 records and will fail silently as volume grows.

CAPACITY AND CONSTRAINTS
- Sole operator: sales, onboarding, support, product decisions, marketing, finance. No team, no hiring budget. Build is AI-assisted (n8n, NocoDB, Claude, Lovable).
- Weekly focused hours available: [OWNER TO FILL: __ hours/week]. Other commitments or income sources: [OWNER TO FILL]. Personal monthly income needed from VRYOH, and by when: [OWNER TO FILL, or "none required yet"]. Cash runway: [OWNER TO FILL].
- If any field above is blank, run three capacity scenarios (15, 25, 35 hrs/week) and show how results change. Do not assume one.
- Budget: bootstrapped. No ad spend unless you justify it with numbers.
- Assets: ~30 years hospitality operations experience, bilingual English/Spanish, Latin America ties.
- Governance is the differentiator. Anything weakening human approval is out of scope.
- Do not stack more than 2 major workstreams at once. Show weekly hours per workstream and that the total fits.

TIMELINE
- Oct 8–31, 2026: setup. Nov 1–Dec 31, 2026: close the gaps above. Then calendar quarters Q1–Q4 2027.

PRODUCE (headings A–I)
A. Reality check: is 10 paying + active locations by Dec 31, 2026 achievable? Yes/no and why. Revised targets for Dec 31, 2026 and each 2027 quarter-end, in conservative / base / stretch cases, with the revenue math. State plainly whether $25/location can support a one-person business, the location count that covers costs, and the count that covers my income need (if given). List packaging options to test (agency/multi-location, annual prepay, add-ons) with sourced comparisons to real competitors and adjacent tools.
B. Bridge plan, Oct 8–Dec 31: two-week blocks. Per block: goal, tasks, hours, dependencies, "done means." Sequence by what blocks revenue first (agreement + billing, landing page live, onboarding path, safe Response Contract release, Google ingestion, portal MVP). List what to defer. Address the conflict between "self-serve, no manual onboarding" and the fact that I must onboard the first clients by hand.
C. 2027 by quarter. Per quarter: objective, paying+active locations and MRR (3 cases), segments, channels, activities, hours/week split (sales, onboarding, support, product, marketing, admin), milestones, risks, and continue / slow / change-course triggers.
D. Go-to-market: ideal first customers (type, owner vs group, Spanish-speaking share), where to find them, outreach method, how to use the Orlando client as a reference (what to ask, when), and a one-agency reseller pilot (structure, terms to define, risks).
E. One-person operations: onboarding steps, support limits and response times, what to automate or refuse, the client count where solo operation breaks, and the client-side burden of approving drafts (churn risk) and how to reduce it.
F. Unit economics and cash: cost per location (labeled estimates), gross margin, quarterly cash view, break-even, and hiring/contracting triggers (first role, when).
G. Ranked risks and dependencies: legal/contract needs (service agreement, Google platform terms, AI-written public replies, sales tax), platform dependency, quality incidents, churn, founder capacity.
H. Metrics: 8–10 weekly/monthly numbers with quarterly targets.
I. Missing data: max 10 questions for me, and the first decisions I must make.

DELIVERY
- Start with a one-page summary: verdict on the 10-location goal, revised targets, top 5 actions for the next 14 days.
- Then deliver in 3 parts (A–C, D–F, G–I). Stop after each part and wait for "continue."
- Short sentences, tables for comparisons, one-line explanation for any jargon. Cite sources as links. End each part with "Could not verify:" and a list.
