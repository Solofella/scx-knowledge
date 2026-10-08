# Master Continuity Document v7.6

**Last Updated:** Chat #386 · October 8, 2026
**Supersedes:** MCD v7.5 (Chat #105, August 20, 2026)
**Owner:** Solofella LLC · Product: VRYOH Intelligence
**Purpose:** A shareable, current-state summary of the VRYOH system for other agents, sessions, projects, or partners.

**Reading notes**
- Components are shown by neutral codes, not names. A private key mapping is held by the owner.
- Internal identifiers (server address, credential IDs, table and field IDs, contact emails) are deliberately excluded. A separate private reference holds them.
- **Source basis:** (a) MCD v7.5, read from a raw repository file (⚠️ MEDIUM confidence); (b) the owner's stored project records from earlier chats; (c) decisions stated in Chat #386. **Nothing here was re-verified against the live workflows in Chat #386.** Per the source hierarchy (§11), live code and the owner's direct pastes outrank this document.
- **Diagnose-Only:** this document records problems and open items. It does not propose or apply fixes. Where it says "fixed," the owner or a build session applied the change and recorded it.
- Where sources conflict, the conflict is flagged in §9, not silently resolved.

---

## 0. What Changed Since v7.5 (Aug 20 → Oct 8, 2026)

**Resolved since v7.5**
1. **P5 naming:** confirmed in the live system prompt. The v7.5 naming conflict is closed.
2. **P5 internal step connection (9b → 10):** confirmed connected by the owner's direct canvas observation, overriding an earlier export gap. The wrong-field governance override is live, not inert, but fires rarely or never correctly.
3. **B1 schedule:** live code is daily 7am, weekly 8am, monthly 9am UTC. Older documentation (6am/6am) was wrong.
4. **B2 daily signal-tier counting:** now covers all six real signal types. Fix applied and confirmed by the owner (Aug 21).
5. **P5 wrong-field Trust/Dignity check:** corrected in P5 (Sep 9). The check now uses the confirmed-domain field with an exact match. See §9 for P4's status.
6. **Dashboard status:** a client portal plan now exists (§7).

**Materially new since v7.5**
1. **Pilot complete.** The three-month pilot ended. One client is live (§2).
2. **Price changed.** Final price is **$25 per location per month** (§2).
3. **New governance layer in P5:** a "Response Contract" (§4), in build in a duplicate workflow only. Triggered by a production incident in which a draft stated fault as fact on a disputed, one-sided account.
4. **B2 changed state:** weekly sending is paused, monthly is disabled, and the daily report is now one consolidated email per account (§3).
5. **P6 expanded:** signal enrichment, compliance checker, and approval-notes feedback were added (§3).
6. **New workstreams:** Google Business Profile integration, reseller model, client portal and landing page, brand identity (§7).
7. **Isolation warning:** the duplicate P5 workflow shares the real data tables. See §4.

---

## 1. Pipeline Overview

**Per-record flow (webhook-chained):** P1 → P2 → P3 → P4 → P5 → P6

| Code | Role |
|---|---|
| P1 | Intake: ingests reviews, normalizes, detects language, deduplicates |
| P2 | Classification: emotion, pain point, signal type |
| P3 | Expression stabilizer: how the guest expresses the emotion |
| P4 | Behavioral interpretation and response-tier assignment |
| P5 | Response strategy (and, in build, the Response Contract) |
| P6 | Draft writer, English and Spanish, human approval required |

**Scheduled branches (not per-record)**
- **B1:** signal aggregator. No webhook in, no trigger out. Writes trend clusters to its own table.
- **B2:** reporting. Reads B1 (weekly, monthly) and P2 directly (daily).

**Adjacent surface**
- **S1:** approval-sheet sync. Runs daily at 5am UTC and populates each client's Google Sheet with pending P6 drafts for human approval. It is not an approval agent.

**Governance principle (locked):** VRYOH detects and interprets guest signal only. It never prescribes operational actions. This appears as an explicit prohibition in the P4 and P5 prompts.

**Code-level enforcement**
- **P4 human-review gate:** required when the tier is the highest, interpretation confidence is Low, structural confidence is below 0.50, or reliability is Weak.
- **P5 Halt Check:** stops processing on discrimination, legal, or escalation content.
- **P6 is terminal:** the published timestamp stays permanently null at approval. No code path exists anywhere that can auto-publish a draft.

**Node counts (⚠️ claims, not re-verified this chat)**

| Code | Nodes | Model | Status |
|---|---|---|---|
| P1 | 23 | GPT gpt-5.2 | Operational |
| P2 | 29 per v7.5; 26 per project records (conflict, §9) | GPT gpt-5.2 | Operational, EN+ES |
| P3 | 23 | claude-sonnet-4-6 | Operational |
| P4 | 32 | claude-sonnet-4-6 | Operational |
| P5 (live) | 24 | Hybrid: deterministic tier 1, claude-sonnet-4-6 for tiers 2 and 3 | Operational |
| P5 (duplicate copy) | 29 | Same | In build, not live (§4) |
| P6 | ~46 | claude-sonnet-4-6, 5 calls per record | Operational, EN+ES independent branches |
| B1 | 18 | None (pure JavaScript) | Operational |
| B2 | 30 or 38 (unreconciled) | None (pure JavaScript) | Partly paused (§3) |
| S1 | ~33 per project records | None | Operational |

**Total:** roughly 225–235 nodes across P1–P6, B1, B2, plus S1 (~33).

---

## 2. Commercial State

**Client roster (project records, Sep 7 and Oct 7, 2026)**
- **One live client:** client ID AJI-001 (one location, Florida). The pilot is complete and this is VRYOH's first client.
- Other previously listed client IDs are inactive and must not be treated as clients. Status of one earlier-listed international prospect is unconfirmed. Not in the pipeline.
- **Goal:** 10 locations live by December 31, 2026 (unchanged).
- One Client ID per physical location, always.

**Pricing (supersedes all earlier tiers)**
- **$25 per location per month. Confirmed final and per location (Oct 7).**
- Includes: reply drafting and a client portal with daily and weekly results.
- Monthly results: available as a paid add-on.
- This replaces the earlier $100 introductory / $150 regular tiers.
- **Open:** total cost per client at this price is unknown. Only one cost is documented: about $0.0072 per processed review for the P2 classification call.

**Commercial reasoning (owner's stance, Oct 7):** operators treat review response, signals, and SEO as low priority, which pushes the price down. The product must therefore be basic, visual, and self-serve, with no manual onboarding.

**Reseller model (planned, stated Oct 1)**
- A marketing company resells VRYOH white-label to its restaurant clients, invisible to the end client.
- Google account consent is needed per end-client business profile.
- VRYOH invoices the marketing company in bulk.
- Data isolation must be selectable: one combined dashboard for the reseller, or fully separate clients.

**Competitive positioning:** maintained separately.

---

## 3. Component Specifications

### P1: Intake
- v5.0 per v7.5. Model GPT gpt-5.2 (locked). 23 nodes. Batch size locked at 1.
- Ingests reviews by CSV upload (Date, Platform, Stars rating, Raw Text, Reviewer Name, Lang, Client ID), normalizes text, detects language, runs a preliminary classification, deduplicates by hash, writes to the database, triggers P2 per record.
- **Locked fixes:** the star column name is `Stars rating`; the star-rating mapping in the normalization step; `lang`/`Lang` case fix for Spanish detection; blocked console logging removed; the reviewer handle was added as a fifth component of the dedup hash, which resolved same-day, same-location star-only review collisions.
- **Open:** the batch-integrity step was reverted to a prior working version. Whether it is still a tautology that always passes is unconfirmed (v7.5 said it was). A failure-branch loop risk was deferred to a later version.
- **Open:** silent drop of duplicates with no audit trail (intent unconfirmed).
- **Open:** record fetch capped at 100 with no pagination.
- **Input method:** CSV upload by hand is the current phase. Google Business Profile ingestion is in build (§7).

### P2: Classification
- v5.1 per v7.5. GPT gpt-5.2. EN+ES. Node count conflict (§9).
- Re-fetches the P1 record, injects full emotion and pain-point dictionaries into one call, resolves emotion, pain, and signal. Cost about $0.0072 per record (caching eligible).
- **Dictionaries:** emotion dictionary 161 rows (EN and ES); pain-point master 336 rows EN, 253 rows ES (Spanish tables cleaned June 2026).
- **Signal type (6 values, deterministic):** Positive / Masked Negative / Ambiguous Negative / Dignity-Risk / Negative / Mixed. **Intensity (4):** Low / Moderate / High / Critical.
- **Certainty-score target:** under 20% of records below the 0.65 threshold (was about 48%).
- **Open issues:**
  1. The stored "emotion tag" column holds the canonical emotion, not the true nuanced phrase. P6's lookup depends on this mislabeling, so a fix needs coordination across two components.
  2. Dignity-Risk detection matches English keywords only, so Spanish records cannot receive that classification.
  3. The semantic-collapse step is a non-functional stub.
  4. No retry on the classification call.
  5. No error fallback on the P3 trigger.
  6. No pain-domain validation for Spanish records.
  7. One computed category field is silently dropped before persistence.

### P3: Expression stabilizer
- v5.0. claude-sonnet-4-6. 23 nodes.
- Analyzes how the emotion is expressed. Does not re-classify. One combined model call returns expression mode (6 types), emotional clarity (4 types), and narrative alignment (0.0–1.0). The structural confidence score is fully deterministic.
- A hard integrity check blocks failing records from reaching the model or P4. Masked-emotion detection (cross-referencing text against star rating) is the stated competitive differentiator.
- **Open issues:**
  1. A string-versus-boolean inconsistency across the P3→P4 handoff works only by coincidence.
  2. No fault tolerance on the P4 trigger.
  3. Core emotion and need state travel in memory only, not stored (intent unconfirmed).

### P4: Behavioral interpretation
- v4.0. claude-sonnet-4-6. 32 nodes live.
- Interprets behavioral meaning and assigns the preliminary tier (T1 standard, T2 calibrated, T3 dignity-restoration). Deterministic synthesis steps cost nothing. One model call produces four fields. Sets the human-review gate (§1).
- **Open issues:**
  1. A suspected case-sensitivity bug in the reviewer-history routing (not live-tested). If real, repeat-guest detection is non-functional.
  2. The tier-3 check for the Trust/Dignity domain uses the wrong field. Its current status is unconfirmed (§9).
  3. Dead patch-body pattern.
  4. No retry or error handling on the P5 trigger.
  5. Known bugs listed in project records for steps 6, 16, 22, and 25.

### P5: Response strategy
- **Live version:** v4.0 per v7.5, 24 nodes, audited node by node (Aug 22). Its document needs regeneration to reflect the confirmed internal connection.
- Receives the P4 tier and produces strategy: framing, urgency, governance assessment. It does not write the public draft.
- **Hybrid design:** tier 1 (about 70–80% of volume, an estimate not re-measured) uses deterministic template selection through a seeded hash for auditability, at zero AI cost. Tiers 2 and 3 use a model call.
- **Template library:** 21 templates (10 negative, 6 positive, 5 mixed). The "13 domains" claim is unverified.
- **Tone ownership moved to P6** (§4): P5 no longer sets tone.
- **Governance:** the pre-flag, the Halt Check, and a wrong-field override (now corrected, see §0).
- **Open issues:**
  1. The Halt Check ends in a bare error with no database write. The pipeline's most important stop has the weakest observability.
  2. The dead patch-body pattern (fourth occurrence).
  3. No retry or error handling on the P6 trigger (tracked as a separate item).
  4. The payload to P6 does not send two fields: dominant pole and masked-emotion hypothesis.

### P6: Draft writer
- Model claude-sonnet-4-6. About 46 nodes, five calls per record. English and Spanish are fully independent chains (Spanish live since Jul 15), merging only at the internal follow-up step. The internal follow-up brief is English-only by design.
- **Approval status (4 values):** Pending / Approved / Not Accepted / Modified (spelled "Modify" in v7.5, §9). "Published" was removed.
- **Added since v7.5:**
  - Signal enrichment steps pulling cognitive driver, need state, and dictionary fields.
  - A deterministic compliance checker for both languages.
  - A rejected-and-modified-patterns brief built from approval notes, via a small sync workflow (3 nodes) and a Google Sheets script using an installable trigger (a platform requirement).
- **Spanish terminology fix:** guest-facing wording uses "opinión" or "comentario" instead of "reseña."
- **Working:** zero language mixing since the Spanish rebuild; duplicate-greeting bug resolved; positive client feedback confirmed.
- **Open issues:**
  1. The commercial-commitment scan (refund, discount, compensation) is English-only. Live Spanish exposure exists today.
  2. The enrichment lookups have no language branching. The zero-row cross-reference bug was fixed (v7.0/7.1), but English-only dictionaries remain.
  3. The elevation-signal check looks for a status value that no longer exists, so every approval email has the same subject.
  4. No confirmed email-send node after the approval notification is built.
  5. A suspected missing `+` operator in one English-chain step, flagged since Chat #96 and still unchecked.
  6. A dead guest-name field.
  7. A governance gap: the draft cannot distinguish "the guest experienced something wrong" from "the guest disagrees with a valid, explained policy." The Response Contract (§4) addresses this upstream. A separate classification option (Option 3) is recorded as pending.
- **Handoff pending (§4):** three items from the Response Contract build.

### B1: Signal aggregator
- v5.0 per v7.5. 18 nodes. No AI. Schedules: daily 7am, weekly 8am, monthly 9am UTC.
- Groups by client, domain, and signal tier. Trend threshold 20%, minimum cluster size 2. Trend values: Stable / Growing / Declining / New. Maps all six signal types correctly. The client ID was added to the grouping key to prevent cross-client contamination.
- **Open issues:**
  1. Two fetch steps use a hardcoded limit of 1,000 with no date filter. This fails silently once the tables exceed 1,000 rows, which matters for the growth target.
  2. The final bulk write has no post-write verification.
  3. Daily clustering produced all-singleton clusters. Whether daily clustering is meaningful is a design question.
  4. The daily window is 2 days. Intent unconfirmed.
  5. One step is a confirmed no-op (removal candidate).
  6. The B1 document revision is not yet written. Eight open items await the owner's decisions.

### B2: Reporting
- No AI. Reads B1 and P2. **Current state (project records):**
  - **Daily:** one consolidated email per account, with a two-band header, an HTML-table KPI strip, and per-location rows. A flexbox layout was ruled out because Gmail mobile strips it.
  - **Weekly:** per-location design. **Sending is paused** through a gate that routes around the send step. Database writes continue, and delivery status still shows "sent" (a known, accepted inaccuracy).
  - **Monthly:** trigger disabled.
- A nested account-to-location loop is built and live-tested.
- **Output surface:** email and database writes only. No spreadsheet export.
- **Confirmed defects:**
  1. A dead dependency on the removed "Published" status zeroes all performance metrics (published count, SLA compliance, velocity, SEO) in every report.
  2. The weekly email hardcodes "100% Response Coverage."
  3. Trend arrows check values that don't exist, so they always show a static arrow.
  4. The dashboard access token uses a non-secure random function. **This must be fixed before any link-based access is launched.**
  5. Delivery is marked "sent" without checking the email provider's response.
  6. The sheet-link fix applies in one template only. Two others still use a dead link.
  7. The calendar-month window versus B1's rolling 30-day window may mismatch.
  8. The same limit=1,000 fetch risk, at three more steps.
- **Housekeeping:** one node is wired but purposeless (removal candidate). The parent-account ID does not pass through one step (currently harmless). Sender name casing varies between templates.

### S1: Approval-sheet sync
- Daily at 5am UTC. Service account authentication. Fan-out per client. A migration to the company's own cloud organization is complete; the old credential is kept as a fallback.
- Scope must be `https://`. A plain `http://` scope silently yields no real access.
- **Open:** the bulk fetch is capped at 100 records and needs pagination.
- **Open:** the version and node count conflict with v7.5, and so does the status of the sheet-edit write-back loop (§9).
- Three real-data findings are attributed to upstream components: corrupted rating markers (P1 encoding), a doubled punctuation at a draft's close (P6), and an opening-line pattern deviation (P6).

---

## 4. Response Contract (P5 Build In Progress)

**Why:** a production draft stated fault as fact on a disputed, one-sided account. Two independent external model audits and the owner's own reasoning agreed that the fix belongs upstream, inside P5. No new component is added.

**What it is:** a structured contract built and validated before P6 drafts anything. It defines what P6 must reflect, may reflect, and must never say, and how much fault or certainty to express.
- Tier 1 builds it deterministically at zero model cost.
- Tiers 2 and 3 add the fields to the existing model call.

**Locked rules**
- **No apology, ever.** Any rating, any claim type. VRYOH has no operational visibility into the client's business and cannot verify any account or assume a cause.
- **Three usable postures:** acknowledge without fault admission, neutral acknowledgment, escalate to human.
- **Escalation still produces a draft**, marked non-publishable until elevated review.
- **1–3 star strategy:** one public reply that specifically acknowledges the actual complaint, then invites contact privately.
- **Boundary:** acknowledging how a guest felt is fine. Judging whether the business's decision was right or wrong is not.
- **Staff names** are allowed in public drafts. **Physical descriptions** are not.
- **Tone moved to P6**, driven by each client's voice settings.
- **Storage:** no new database columns. Versions and a decision trace live inside the existing error-log field as structured data. The trace is never sent to P6.
- **Mixed reviews:** record-level fields take the worst-case component. The contract still requires addressing positive content.
- **Confidence cap:** each concept's confidence is capped by P4's interpretation confidence.
- **Schema:** locked at the field level (schema version `response_contract_v1`).

**Build state (as of Oct 6, project records)**
- Built in a **duplicate** of P5 (29 nodes). Complete: tiers 2 and 3 prompts, tier 1 builder, validator with retry loop, tier 1 escalation filter, storage packing, pass-through steps.
- **First test (Oct 4):** a tier 1 record ran end to end. A tier 2/3 Spanish record failed with invalid JSON. A run was cancelled at exactly 10 minutes (suspected retry-loop gap). Cause: several steps rebuild the record from fixed field lists and dropped the new fields.
- Fixes were specified Oct 4. **Not confirmed applied.**
- P5 document v6.0 (full rewrite, duplicate copy) written Oct 6.

**⚠️ Isolation warning:** the duplicate is **not data-isolated**. It shares the real tables. The Oct 4 test wrote a row to the real P5 table and marked a real P4 record Complete. Before any retest, confirm that the live P5 is off and that its handoff step points at the P6 copy.

**Known weaknesses**
- Tier 1 claim-type detection is one crude keyword check.
- The bilingual keyword filter exists as an escalation filter. Its false-positive golden test is required.

**Planned sequence:**
1. Finish P5 in the duplicate.
2. Hand three items to P6's chat window:
   - a per-client escalation contact (all 1–3 star drafts, not only high-liability);
   - removal of P6's own fault and claim-verification judgment, so the two don't contradict;
   - pick up tone from client settings.
3. Build a duplicate P6.
4. Run the P5 copy → P6 copy test, fully isolated.
5. Build the 72-record golden set, with a blinded side-by-side comparison. The plan makes bad drafts less likely. Only the comparison measures whether good drafts improve.

---

## 5. Cross-Component Bug Patterns

| Pattern | Where | Count |
|---|---|---|
| Dead patch body computed but never used | P2, P3, P4, P5 | 4 |
| Missing error handling or retry on handoffs | P2→P3, P3→P4, P4→P5, P5→P6 (only P1→P2 has it) | 4–5 |
| Wrong-field Trust/Dignity check | P4 step 16, P5 step 9b | 2. Same origin: one design session, hand-copied into two workflows. P5 side fixed (Sep 9) |
| Unfiltered fetch with limit=1,000 | B1 (2), B2 (3) | 5 nodes |
| Dependency on the removed "Published" status | B2 (consequential), S1 (harmless) | 2 |

---

## 6. Governance Architecture

**Strong (code-confirmed)**
- Universal human approval at P6; no auto-publish path exists.
- P4's computed human-review gate.
- P5's Halt Check (with the observability gap above).
- Masked-emotion detection at P3.
- The no-apology and no-assumed-cause rules (§4), once the Response Contract ships.

**Gaps, currently live**
- The P5 Halt Check has no queryable record when it fires.
- The P6 commercial-commitment scan is English-only.
- The P4 Trust/Dignity check may still use the wrong field.
- The P4 reviewer-history bug could disable repeat-guest detection.
- B2's reporting formerly undercounted safety-critical signal types (fixed Aug 21; confirm in live output).
- The Response Contract is not yet in production.

---

## 7. Infrastructure and Front-End

**Hosting:** one self-hosted Linux cloud server running n8n 2.4.6 and NocoDB with Docker Compose (v1 syntax). Email delivery through Brevo. Google Sheets is the client approval interface.
- Internal database address inside n8n is always the container name, never localhost.
- The external database address is currently plain HTTP (see portal item below).
- A secure subdomain for OAuth callbacks runs behind a reverse proxy with auto-renewing certificates (current certificate expires Dec 7, 2026).
- Two separate Google Cloud projects: one for sheet sync, one for Business Profile access.

**Google Business Profile integration (in progress)**
- Callback workflow (5 nodes) built. OAuth flow tested end to end on VRYOH's own account. A manager role on the client's listing is sufficient; no owner consent is needed for the pilot client.
- Business profile verified Aug 11, 2026. **Eligible to request API access Oct 10, 2026.**
- Outstanding: the account-ID and location-ID lookup steps.
- **Reseller implication:** consent will be needed per end-client profile.

**Landing page and brand (Oct 7–8)**
- The earlier hosted page was text-heavy, had no price or sign-up action, and told search engines not to index it.
- A new single-page site has been built in Lovable (Pro plan, $25 per month), with an uploaded 15-second video. The domain is owned and is to be pointed at it.
- **Brand style:** page background #FCF9F7; headings and body #50352B; primary and buttons #835E54; accents #C9907C, #B3BDB5, #F5EAD8, #C67139, #C7DBB0, #7A8A5E. Headings in Crimson Pro Bold, body in Open Sans Regular. All client-facing material uses VRYOH Intelligence. The earlier product name is retired from client-facing use.
- A logo brief was drafted. A designer package is under consideration.

**Client portal (decision stage, Oct 7)**
- **Goal:** clients onboard themselves by following in-page instructions and view daily and weekly results after login. Seeing that reviews are answered daily matters most.
- **Chosen tool:** Lovable. Confirmed in its docs: a built-in managed Postgres database with row-level security, custom connectors for any REST API (https base URL required), server-side credentials, custom domain on Pro.
- **Cost:** hosting and backend use draw from the same credit balance as building. The credit cost of the portal is unknown.
- **Not ready-made, must be built:** n8n-to-portal data intake, client-to-location link table, onboarding checklist, user invites, location switcher.
- **Open design choice:**
  - **Path A:** n8n pushes results into the portal's own database through a custom server-side function. Security rules then apply inside one system.
  - **Path B:** the portal queries the main database live through a server-side function.
- **Blocker for any direct connection:** the main database address must be https.
- **Open questions:** where client logins live; how a Client ID is attached tamper-proof; whether one login covers several locations (multi-location clients and resellers).
- **Next step:** a free-tier feasibility test of login, link table, one results table, and the n8n webhook.
- **Security prerequisite:** the non-secure token generator (B2) must be replaced before any link-based or login access.
- The earlier full authenticated platform is deferred to after the pilot.

**Future architecture (parallel, 12-month window):** orchestration plus a Postgres data layer with built-in authentication. n8n and the current database remain the production system through about the end of 2027, up to about 20 locations. Cutover only when output quality at the live client matches or exceeds today's.

---

## 8. Locked Build Rules and Lessons

**n8n 2.4.6**
- No spread operators; build objects key by key.
- IF node fires both branches. Gate the false branch with `return []`.
- Console logging and unused variables are blocked by the task runner.
- Workflow static data is unreliable. Pass state through the payload chain.
- Webhook payloads arrive under `.body`.
- PATCH uses JSON body mode, not RAW.
- Bulk POST bodies need a Code node with `JSON.stringify`.
- Count records with `pageInfo.totalRows`, never list length.
- Boolean IF logic belongs in a Code node.
- `require('http')` works; `$http`, `$helpers`, and `fetch` are blocked.
- After a restart, reconnect the database container to the n8n network.
- `for...of` and `const` inside loop bodies cause errors in Code nodes. Use index loops.
- Nested Split-In-Batches nodes do not auto-reset. Tag the first item and wire it into the inner reset.
- `notEmpty` treats null as non-empty.
- `$(node).all()` returns the full execution history. Filter with an allow-list.
- `alwaysOutputData:true` with a legitimate empty return can silently halt downstream. Add an IF after it.
- The Google Sheets node reads document and sheet expressions once from the first item. Wrap multi-client runs in a batch of one.
- Cross-execution state bleed between scheduled runs is not possible.

**Google**
- New organizations block service-account key creation by default (an organization policy).
- OAuth scope must be `https://`.
- Apps Script simple triggers cannot make external calls. Use installable triggers.

**Prompting**
- Case-specific rule lists perform worse than fewer broad principles.
- Rule stacking degrades quality, confirmed in English and Spanish.
- Keep the generation prompt minimal. Let the audit step enforce rules afterward.

**Architecture**
- Each component stores only what it computed. Upstream fields travel in the webhook payload.
- The P1 record ID is the traceability key across all tables.
- Client ID originates at the CSV source. A record without one is skipped.
- A Field Traceability Map is required before any new build, with fields declared at their source before node one.

---

## 9. Unreconciled Conflicts (Flagged, Not Resolved)

1. **P2 node count:** 29 (v7.5) versus 26 (project records).
2. **B2 node count:** 30 versus 38.
3. **P6 table state:** "frozen, no new columns" (project records) versus "not frozen, 21 fields" (v7.5).
4. **P6 document version and date:** v6.0, Jul 25 (v7.5) versus v7.1, Sep 8 (project records). The chat-number-to-date mapping is itself inconsistent across sources.
5. **P6 status wording:** "Modified" versus "Modify."
6. **S1 version and node count:** v3.0 and "not separately counted" (v7.5) versus v2.0 and about 33 (project records).
7. **Approval write-back loop:** project records describe a built sheet-to-workflow-to-P6 path (3-node workflow plus two P6 steps) and, elsewhere, say the write-back is not built. Both cannot be current.
8. **B2 document version:** v5.0 (v7.5) versus v7.0 (project records). The v7.0 file labeled itself "Chat #108," was not adopted, and stays ⚠️ MEDIUM until cross-checked live.
9. **P1 batch-integrity step:** tautology (v7.5) versus "reverted to prior working version" (project records). The meaning of "working" is not stated.
10. **P4 step 16:** project records confirm the P5 fix only. Whether P4's own check is fixed is not stated.
11. **P6 enrichment lookups:** the zero-row bug is recorded as fixed, but English-only language handling is still recorded as open. Treated as partially fixed.
12. **Chat-number dates:** several earlier documents carry chat numbers whose dates disagree with each other.

---

*End of MCD v7.6 · Chat #386 · October 8, 2026 · Solofella LLC*
*No fixes proposed or applied. Diagnosis and consolidation only, per the standing Diagnose-Only Rule.*
