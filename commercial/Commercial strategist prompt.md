ROLE
You are a senior go-to-market and commercial strategist for early-stage B2B SaaS, with restaurant/hospitality experience. Build a realistic commercial plan. Do not write marketing fluff or motivational language. Be factual, concise, and specific.

RULES FOR YOUR ANSWER
1. Label every claim: [GIVEN] (from this brief), [RESEARCHED] (with source/link), [ASSUMPTION] (your estimate, with reasoning).
2. Do not invent market data, competitor prices, conversion rates, or benchmarks. If you cannot verify one, write "NOT FOUND — needs data" and give a range marked [ASSUMPTION].
3. Compare against real competitors and adjacent tools (review-response and reputation tools for restaurants, marketing agencies that resell them). Cite sources.
4. If my goal or numbers do not add up, say so directly. Do not soften it.
5. Be realistic about capacity: ONE person does everything (sales, onboarding, support, product decisions, marketing, finance). There is no team and no budget for hires yet. Technical build is done with AI-assisted tools (n8n, NocoDB, Claude, Lovable), and my time is the scarcest resource. Estimate weekly hours per workstream and show the plan fits in a realistic week. Do not stack more than 2 major workstreams at once.

PRODUCT (all [GIVEN])
- VRYOH Intelligence: governance-first guest-signal and review-response platform for restaurants.
- Reads guest reviews, detects the emotion and signal behind them (including masked negative emotion), drafts public replies in English and Spanish. A human must approve every draft. Nothing auto-publishes.
- It reports signals only. It never prescribes operational actions.
- Includes a client portal with daily and weekly results (portal not built yet).
- Monthly results are a paid add-on.
- Price: $25 per location per month (final, per location).
- Positioning from me: operators treat review response, signals and SEO as low priority, so the product must be basic, visual, self-serve, with no manual onboarding.

CURRENT STATE (as of Oct 8, 2026) [GIVEN]
- One live client: a single-location ceviche restaurant in Orlando, Florida. Pilot (3 months) is complete. It starts paying $25/month on Oct 25, 2026.
- No signed service agreement, no invoice terms, no payment system (only a bank account), no live website.
- Reviews are currently ingested by manual CSV upload.
- Draft approval happens in a Google Sheet per client.
- Positive client feedback received on Spanish and English drafts.
- Original goal: 10 locations live by Dec 31, 2026. I believe this cannot be met because of the gaps below.

GAPS BLOCKING GROWTH (all [GIVEN])
Commercial / business:
- No signed agreement or invoice terms; no way to collect payment (billing/subscription).
- No live landing page: new single-page site is built in Lovable, domain owned but not yet pointed; earlier page had no price or sign-up action.
- No logo/brand package finished (brand colors and fonts defined; logo brief drafted).
- No sales material, case study, pricing page, or onboarding instructions.
- Cost per client is unknown (only one cost known: about $0.0072 per processed review for the classification step; drafting uses 5 AI calls per review). Unit economics at $25/location are unverified.
- Reseller model (marketing agency resells white-label, bulk invoicing, per-end-client Google consent, selectable data isolation) is an idea only; no terms, no agency conversations.
Product:
- Client portal not built (tool chosen: Lovable; open decision on how results reach it; main database address must be https first).
- Self-serve onboarding does not exist; Google Business Profile integration is partly built (API access can be requested starting Oct 10, 2026; account/location lookups outstanding). Until then, ingestion is manual.
- A new "Response Contract" safety layer (stops drafts from stating fault as fact; no-apology rule; escalation path) is built only in a test copy, not in production, and must go through an isolated test and a 72-record blinded comparison before release.
- Reporting emails: weekly sending paused, monthly disabled; performance metrics (SLA, response speed, SEO) currently compute as zero due to a known defect; a security gap in the dashboard access token must be resolved before any link or login access.
- Spanish drafts lack a check for refund/discount/compensation promises (English has it).
- Scale limits: some data fetches cap at 100 or 1,000 records and will fail silently as volume grows.
Do not propose technical fixes. Treat each as a dependency with an estimated effort in days and a "must be done before" milestone.

CONSTRAINTS
- Sole operator. Realistic working capacity: assume I can give roughly [30] focused hours/week to VRYOH unless you recommend otherwise; state your assumption. Account for support time that grows with each client.
- Budget: bootstrapped. Known tools cost: Lovable Pro $25/month. No ad budget assumed unless you justify one.
- Base: New York; client in Florida; bilingual English/Spanish; ~30 years of hospitality operations experience (credibility asset); Latin American ties.
- Governance is the differentiator and must not be compromised for speed. Anything that weakens human approval is out of scope.

WHAT I NEED YOU TO PRODUCE
A. Reality check (short): is 10 locations by Dec 31, 2026 achievable? State yes/no and why. Propose a revised, defensible target for Dec 31, 2026 and for each quarter-end of 2027, with a conservative, base, and stretch case. Show the math for revenue at $25/location/month. Tell me plainly if this price can support a one-person business, what number of locations covers costs, and what pricing or packaging options (add-ons, agency/multi-location pricing, annual prepay) should be tested. Back with sources.
B. Bridge plan: Oct 8 – Dec 31, 2026 (use November and December specifically to close the gaps above). Week-by-week or two-week blocks. For each block: goal, tasks, hours needed, dependencies, and "done means" criteria. Sequence gaps by what blocks revenue first (agreement + billing, landing page live, onboarding path, safe release of the Response Contract, GBP ingestion, portal MVP). Mark what to defer.
C. 2027 plan by quarter (Q1, Q2, Q3, Q4). For each quarter: objective, target locations and MRR (three cases), customer segments and channels, key activities, hours/week split (sales, onboarding, support, product, marketing, admin), milestones, risks, and the trigger to continue/slow down/change course.
D. Go-to-market: ideal first customers (type of restaurant, owner vs group, English/Spanish-speaking), where to find them, outreach approach, how to use the live Orlando client as a reference (what to ask for, how), and a test of the agency/reseller channel (what a pilot with one agency would look like, terms to define, risks).
E. Operations and support model for one person: onboarding steps, support limits and response times, what to automate, what to refuse, and the client count at which solo operation breaks (state your reasoning).
F. Unit economics and cash: cost per location (estimate, labeled), gross margin, monthly cash view per quarter, break-even point, and what would trigger hiring or contracting help (first role, and when).
G. Risks and dependencies (ranked): include legal/contract needs (service agreement, data and platform terms for Google Business Profile access, AI-generated public replies), platform risk, quality incidents, churn, and founder capacity/burnout.
H. Metrics: a short dashboard of 8–10 numbers to track weekly and monthly, with targets per quarter.
I. Open questions and missing data: list what you need from me to make the plan more accurate (max 10), and which decisions I must make first.

FORMAT
Use clear headings A–I, tables where comparing, short sentences. No jargon without a one-line explanation. End with a "Top 5 actions for the next 14 days." Cite sources as links. State what you could not verify.
