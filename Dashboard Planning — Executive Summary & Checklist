Chat #109 · September 1, 2026

# Dashboard Planning — Executive Summary & Checklist
*(For handoff to dedicated Dashboard build chat window)*

## Business rationale
- ☐ Origin: pilot feedback that daily per-location emails were burdensome → led to daily-email consolidation (separate, completed workstream).
- ☐ Separately, Miguel identified that weekly/monthly reporting has a distinct weakness email can't solve: "email is a snapshot — no historical comparison, no filtering, no drill-down."
- ☐ Miguel's stated position: the trend/signal data VRYOH surfaces **is the product's core value**, not a secondary reporting feature — this justifies prioritizing dashboard build even at pilot stage, rather than treating it as a later nice-to-have.

## Options evaluated, and why each was ruled out or kept
- ☐ **Google Workspace internal portal apps** (Chat/Spaces/Drive/Groups) — ruled out. Wrong category of product: built for internal org users with Workspace/Google SSO accounts, not external clients with no Google Workspace tie to VRYOH.
- ☐ **Looker Studio** — evaluated in depth twice (once independently in an earlier session "VRYOH platform evolution with reporting dashboards," once again in this session, both landing on the same conclusion). Ruled out for three confirmed, specific reasons:
  - No native NocoDB connector; would require direct Postgres exposure (firewall/SSL/read-replica risk) or an intermediate Sheets/BigQuery sync.
  - URL-parameter filtering is confirmed **insecure at the data layer** (client-editable, not enforced server-side) — a client could edit the URL to see another client's data.
  - True row-level security requires each client to have a Google account login — breaks the required "no-login, magic-link" interaction model.
  - "One-report-per-client" is the only secure workaround, but has no programmatic/API way to clone+scope+link a report per new client — manual UI work per client, doesn't scale toward the 10-location 2026 goal.
- ☐ **Self-hosted Metabase** (or similar BI tool, e.g. Superset) — identified as a real, viable alternative. Pros: fast to stand up (Docker container), connects directly to NocoDB's Postgres, has native signed-JWT tenant-scoped embedding (solves the security gap Looker Studio couldn't). Cons: generic BI-tool UI, not VRYOH-branded.
- ☐ **Custom frontend** — selected as the final direction. Reasoning: Miguel confirmed brand look-and-feel matters as part of VRYOH's product identity, which a generic BI tool (Metabase) can't provide. This is the only option that lets the dashboard visually match VRYOH's existing design system (warm parchment/terracotta palette, established in the daily email rebuild).

## Confirmed technical facts relevant to the dashboard build
- ☐ Backend is entirely self-hosted: n8n + NocoDB (Postgres/SQLite-backed) on a single DigitalOcean droplet.
- ☐ A magic-token access pattern already exists and works in production today: `Step 12 - Build MRA Record` generates a token via `Math.random()`-based hex assembly (not a CSPRNG), used today to open Google Sheets directly from the daily email's approval buttons, no login required.
- ☐ This same token mechanism was proposed and agreed as the reusable access-control pattern for the dashboard: a link opens directly into a client's own scoped view, no password/login screen.
- ☐ Known limitation of the current token: `Math.random()` is not cryptographically secure — flagged previously (§16 open items in MRA HOW documentation) as a decision point for whether to switch to a real CSPRNG. This becomes more important for a dashboard (longer-lived, broader access) than for a one-time Sheet-open link.
- ☐ Primary data source for trend content: **SIA**'s cluster table — stores `Signal Tier` (T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE), `N`, **`Prior N`** (only one prior period stored, not a full history), `Trend Direction` (Stable/Growing/Declining/New), `Is Singleton`, `Domain`, keyed with `Run Timestamp`.
- ☐ Secondary data source: **MRA's `Step 9`**, recently updated this session — now computes `approval_rate`, `approved_count`, `not_accepted_count`, `modified_count`, `decided_count`, tier breakdown (T1/T2/T3 totals/approved/pending), review volume/rating, platform counts. (Note: this update was daily-report-scoped; weekly/monthly's own `Step 9`/`Step 11` consumption still needs the same fix applied when those reports are rebuilt — separate, not-yet-done item.)
- ☐ **Historical-depth constraint, unresolved:** SIA's table only stores current + one prior period per cluster. A dashboard showing more than "this period vs. last period" (e.g., an actual multi-week line chart) would require querying multiple past `Run Timestamp` entries directly — technically possible (the field exists), but not yet designed or decided.

## Client/account structure relevant to scoping
- ☐ Two active accounts today: PAK-001 (single location, Park Avenue Kitchen) and AJI-001–005 (five locations, Aji Ceviche Bar) — grouped under `parent_account_id` in Client Config (this grouping mechanism, built for the daily-email consolidation, is directly reusable for scoping a dashboard per brand/account).
- ☐ Stated 2026 goal: 10+ locations live — relevant to how many client-facing dashboard instances/links need to be issued and maintained over time.

## Open decisions, NOT yet resolved, needed before the dashboard build begins
- ☐ Data scope for v1 — full set (trend signals + approval rate + review/rating trends + tier breakdown) vs. a smaller starting subset.
- ☐ Historical depth for v1 — simple "this period vs. last" comparison, or real multi-week charts requiring a new NocoDB query pattern.
- ☐ Hosting location — same DigitalOcean droplet as n8n/NocoDB (simplest, but adds load/surface to production infrastructure), or separate.
- ☐ Token mechanism — reuse existing `Math.random()`-based token as-is, or upgrade to a CSPRNG specifically for this build.
- ☐ Not yet decided: whether this is a fully separate build effort, or bundled with the still-pending weekly/monthly one-email-per-account rebuild (which itself is blocked/deferred pending this dashboard direction, per earlier scope conversations in this same chat).

## Explicitly out of scope / ruled out
- ☐ Google Workspace internal-portal-style apps — wrong product category, confirmed ruled out.
- ☐ Looker Studio — confirmed ruled out for specific, documented security/scalability reasons (not just preference).
- ☐ Full platform rebuild with persistent password/PIN auth, standalone Node.js API service architecture — this was the *original* full-platform scope from an earlier session, explicitly reversed/descoped by Miguel mid-session as too complex for pilot stage. The current magic-token, no-login approach is the deliberate, simpler replacement for that original plan.

---
*This is a factual record of what has been discussed and decided/ruled out — no new recommendations or next steps proposed here, per the instruction to clarify without concluding.*
