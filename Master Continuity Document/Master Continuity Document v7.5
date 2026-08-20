Master Continuity Document v7.5

Last Updated: Chat #105 · August 20, 2026 Supersedes: MCD v7.4 (Chat #74, April 4, 2026) — v7.4 is stale on node counts, agent names, client roster, and several architectural claims. This version reconciles all conflicts against live-code HOW documents. Status: All 9 pipeline components (7 per-record agents + SIA + MRA) confirmed operational via live-code audit. Sheet-Sync confirmed operational as approval-surface layer. Source basis: Every claim below is sourced from a HOW document explicitly self-described as "as-built from live n8n workflow inspection" or equivalent live-code audit, read in full during Chat #105. Per standing verification protocol, this document does not silently resolve open items — every open item from source docs is carried forward below, unresolved, flagged for owner decision.

0. What Changed Since v7.4 — Read This First

v7.4 (April 2026) predates a full live-code reconciliation pass conducted across Chats #96–104 and consolidated in this document. Every agent HOW doc has since been rewritten "as-built" against the live n8n editor, correcting v7.4/v3.x-era claims that were never actually true in production. Materially wrong or stale items in v7.4:

Node counts were wrong for every agent. v7.4 stated 151 total nodes across 7 agents; live-code audit confirms ~150+ nodes across 7 per-record agents alone, plus SIA (18) and MRA (30–38, unreconciled) as separate branches, plus Sheet-Sync — the pipeline is materially larger than v7.4 described, and SIA/MRA were not separately itemized before.
HSI's acronym was wrong. v7.4 says "Hospitality Signal Intelligence." Live system prompt confirms "Human Signal Intelligence." v7.4's HSI doc reference (v3) also omitted roughly half the real workflow — no idempotency check, no canon gate, no reviewer history lookup, no tier classification, no human review gate documented at all.
BRA's acronym is disputed, unresolved. Live system prompt says "Brand Response Architect"; a prior baseline said "Brand Response Agent." Not resolved as of this document.
Client roster is entirely stale. v7.4 lists PAK-001 "Active," EDO-001 "Confirmed Q2," AJI-001 "Exploratory — initial conversations only." Current confirmed state (see §2) is materially different: AJI is 5 live locations on a 45-day pilot, PAK-001 is stalled, EDO-001 pushed to Q4 2026/Q1 2027.
MRA was "on hold, build prompt ready" in v7.4. MRA is now live, production, multi-client, three-cadence — and has multiple confirmed defects (see §6).
SIA was described as "Signal Intensity Assessment," 11 nodes, "groups EIP records by Domain + Signal Tier." Confirmed name is "Signal Intelligence Aggregator," confirmed node count 18, and the aggregation key was found to have been missing client_id at an earlier point (since fixed — confirmed via inline code comment "FIXED: Include client_id in key").
BCA (Batch Controller Agent) is not mentioned in any document read this session. Status unknown — may be abandoned, built without a HOW doc, or renamed. Flagged as an open item, not assumed resolved.
Pricing section is entirely unvalidated speculation from v7.4 ("Score 7.5: $2,800/mo" etc., explicitly marked "zero paying clients"). Current confirmed pricing (per commercial-side memory, not the pipeline audit) is $100/location pilot, $150/location standard — a completely different model. v7.4's pricing table should be treated as historical, not current.
Dashboard specification in v7.4 describes a planned build. Current state: dashboard build status remains unconfirmed per commercial-side memory; MRA's magic-link token generation (Math.random(), not a CSPRNG) is now confirmed as the live access mechanism for whatever dashboard surface exists.
Approval workflow (Google Sheet) section in v7.4 is largely superseded by Sheet-Sync HOW v3.0, which confirms a materially different, more complete architecture (Service Account auth over OAuth2, multi-client fan-out, dedup, Brevo digest email) and explicitly states the sheet-edit-to-RDA feedback loop is still designed but not built — unchanged since v2.0.

Nothing in this document silently fixes anything. Every bug, gap, and unresolved discrepancy found in the source HOW docs is carried forward below exactly as diagnosed.

1. Pipeline Overview

Active flow (per-record, webhook-chained): ALA → EIP → ESS → HSI → BRA → RDA

Separate branches (scheduled, not per-record): SIA (aggregation) and MRA (reporting) each run on independent Schedule Triggers with no webhook in and no per-record trigger. SIA has no downstream workflow trigger — it is terminal and writes to its own NocoDB table. MRA reads SIA's output directly.

Adjacent surface (not a pipeline agent): Sheet-Sync — scheduled (5am UTC daily), populates per-client Google Sheets with pending RDA drafts for human approval. Not an approval agent itself; surfaces data for human decision. The sheet-edit → RDA write-back feedback loop is designed but not built.

Governance principle (unchanged, now confirmed at code level across multiple agents, not just policy): VRYOH detects and interprets guest signal only. It never prescribes operational actions. This constraint appears literally in HSI's and BRA's Claude system prompts as explicit prohibitions, not just as documentation. The concrete code-level enforcement mechanisms are:

HSI Step 17 — human_review_required computed boolean (tier=T3 OR interpretation_confidence=Low OR structural_confidence_score<0.50 OR reliability_tier=Weak)
BRA Step 11 — Halt Check, hard-stops on governance_flag === "Halt" (discrimination/legal/escalation content)
RDA — terminal agent; Published Timestamp confirmed permanently null at approval; external publishing is a separate, unbuilt mechanism. No code path currently exists anywhere in the pipeline that could auto-publish a draft.

Node counts (confirmed, live-code tier):

Agent	Node Count	Model	Status
ALA	23	GPT gpt-5.2	Verified operational, AJI-001–005 + PAK-001 live
EIP	29	GPT gpt-5.2	Verified operational, EN+ES dual-language
ESS	23	claude-sonnet-4-6	Verified operational
HSI	32	claude-sonnet-4-6	Verified operational, first full technical HOW (v3.0 was largely stale)
BRA	24	Hybrid (T1 deterministic / T2-T3 claude-sonnet-4-6)	Verified operational
RDA	~46	claude-sonnet-4-6 (5 calls/record)	Verified operational, EN+ES fully independent branches
SIA	18	None (pure JS, zero AI cost)	Verified operational, live-audited via direct n8n JSON read
MRA	30–38 (unreconciled between two source docs)	None (pure JS)	Production-live, multi-client, 3 cadences
Sheet-Sync	Not separately counted in confirmed doc	N/A (no AI)	Confirmed working since v2.0 (Chat #85)

Total: ~225–235+ nodes across the full system, materially larger than v7.4's stated 151.

2. Active Pilots (Confirmed, Commercial-Side Memory Cross-Referenced)
Client ID	Name	Locations	Status	Notes
AJI-001–005	Aji Ceviche Bar	5 (Orlando, Casselberry, Sarasota, Tampa, St. Petersburg, FL)	Live since July 16, 2026	45-day pilot, $100/location/mo. AJI-001 confirmed live-processing genuine Spanish-language Google reviews. Real positive client feedback confirmed (Orlando owner).
PAK-001	Park Avenue Kitchen (David Burke), NYC	1	Stalled	Internal management matter, no activation date. Contact: Eddie.
EDO-001	EDO Restaurants, Peru	Unconfirmed count	Pushed to Q4 2026/Q1 2027	Not yet in pipeline.

Total active: 6 locations. Target: 10 by December 31, 2026.

v7.4's client table (PAK-001 "Active," EDO-001 "Confirmed Q2," AJI-001 "Exploratory") is fully superseded.

3. Agent Specifications — Full Detail
ALA — Audience Listener Agent

Version: v5.0 · Last Updated: Chat #88, June 29, 2026 · Model: GPT gpt-5.2 (locked) · 23 nodes

Pipeline entry point. Ingests guest reviews via CSV upload (7-column schema: Date, Platform, Stars rating, Raw Text, Reviewer Name, Lang, Client ID), normalizes text, detects language, runs preliminary GPT classification (pain point domain, emotion tag, keywords — no controlled taxonomy at this stage), deduplicates via SHA256 hash (includes client_id in hash key as of Chat #87), writes to NocoDB, triggers EIP per record.

Batch size locked at 1 (sequential, prevents API rate-limit record loss). Table: m57efwbtrvwohhr.

Confirmed open bugs:

Step 21 Batch Integrity Check is a confirmed tautology — always returns PASS regardless of input. No condition produces FAIL.
Duplicate-record silent drop — no audit trail written. Unconfirmed whether this is intentional (a prior version logged duplicates; current version doesn't).
Spanish routing root cause is NOT here — ALA confirmed correctly sends lang: 'es'; EIP confirmed receiving it correctly. The failure is downstream, in EIP.
EIP — Emotional Intelligence Processor

Version: v5.1 · Last Updated: Chat #103, July 25, 2026 · Model: GPT gpt-5.2 · 29 nodes, EN+ES

Knowledge injection and canonical classification layer. Re-fetches full ALA record, loads full Emotion Dictionary + Pain Point Master for the resolved language, injects both dictionaries into a single GPT call (~14,750 tokens dictionary content, eligible for 90% OpenAI prompt-caching discount; ~$0.0072/record confirmed cost). Resolves all emotion/pain/signal classifications inherited by every downstream agent.

Dictionary sizes (confirmed live-code tier):

Emotion Dictionary: 161 rows, both EN and ES (table IDs mrrscb955j1d2i7 EN / mhot4w62tupht71 ES)
Pain Point Master: 336 rows EN (meavqh37mdqgl4d) / 253 rows ES (mwmiyyoucuhsms8) — Spanish table had 3 corrupted ñ-column names and corrupted cell values, resolved and verified clean June 2026.

Signal Type — 6-value deterministic taxonomy (Step 10): Positive Signal / Masked Negative Signal / Ambiguous Negative Signal / Dignity-Risk Signal / Negative Signal / Mixed Signal.

Intensity Level — 4-value: Low / Moderate / High / Critical.

Confirmed open bugs:

Issue 9.8 (resolved from "unconfirmed" to "confirmed" this cycle, using live production data: ALA record 5447, EIP record 2226, client AJI-004): the NocoDB column "Enriched Emotion Tag" stores core_emotion (canonical dictionary term, e.g. "Anger"), NOT the true nuanced GPT-generated phrase (e.g. "furious over delays"). The true phrase travels correctly in-memory to ESS but is never persisted. RDA's Step 6c currently depends on this mislabeled behavior — fixing it in isolation would break RDA's dictionary lookup. Confirmed as requiring a coordinated two-agent fix, not a single-agent patch.
Dignity-Risk Signal detection (Step 10) matches English keywords only against emotion_category, which GPT outputs in Spanish for lang=es — Spanish dignity/trust-risk records cannot receive Dignity-Risk Signal classification.
Step 11 Semantic Collapse Detection is a non-functional stub — always sets collapse_flag: false, no logic implemented, spec never existed.
No retry-on-fail on the GPT classification call (Step 8) — unlike ALA's equivalent, which has retry.
No error fallback on the ESS trigger (Step 18) — an ESS outage fails the entire EIP execution even after a successful classification and write.
No pain-domain validation for Spanish records — a null domain can write silently.
emotion_category computed by GPT, carried through the pipeline, silently dropped at Step 14 — never persisted or forwarded.
ESS — Emotional Signal Stabilizer

Version: v5.0 · Last Updated: Chat #98, July 24, 2026 · Model: claude-sonnet-4-6 · 23 nodes

Analyzes how the guest expresses the emotion EIP already classified — does not re-classify. Produces Expression Mode (6 types), Emotional Clarity (4 types), Narrative Alignment Score (LLM output, 0.0–1.0), and a fully deterministic Structural Confidence Score. Runs a hard Canonical Integrity Check before any Claude call — records failing hard validation are written as error records and never reach Claude or HSI. This is a real, working data-quality gate: not every ingested review makes it into the downstream dataset.

One combined Claude call (not two, correcting a prior v4.0 "Critical Rule" that was wrong) returns all three outputs in a single 3-key JSON response.

Expression Mode categories: Explicit / Implicit / Masked (explicitly stated in the agent's own documentation: "This is SubtextCX's competitive differentiator" — cross-referenced against star rating, e.g. "Everything was fine" on a 2-star review) / Performative / Conflicted / Absent.

Emotional Clarity categories: Clear / Diffuse / Fragmented / Ambiguous.

Structural Confidence Score formula: (certainty_score × 0.40) + (narrative_alignment_score × 0.35) + (canon_bonus × 0.25), minus penalties for ambiguity/masked flags and soft warnings, clamped [0,1]. canon_bonus is a hardcoded constant 1.0 — only canon-passing records ever reach this node.

Confirmed open issues:

canon_pass/canon_flag string-vs-boolean inconsistency across the ESS→HSI handoff — works today only via two coincidentally-agreeing undocumented conventions on either side. Fragile.
Step 19 (HSI trigger) missing fault tolerance — same pipeline-wide pattern as elsewhere.
Core Emotion / Need State not stored in ESS's own NocoDB table despite a prior version claiming they were kept "for reference" — travel in-memory only. Unconfirmed whether intentional or regression.
HSI — Human Signal Intelligence

Version: v4.0 · Last Updated: July 24, 2026 · Model: claude-sonnet-4-6 · 32 nodes

⚠️ Name correction from v7.4/prior docs: "Human Signal Intelligence," NOT "Hospitality Signal Intelligence." This is the first full technical HOW document for this agent — the prior v3.0 was a short conceptual overview; roughly half the real workflow (idempotency check, canon gate, reviewer history lookup, tier classification, human review gate, NocoDB write, downstream handoff) was entirely absent from it.

Interprets behavioral meaning from ESS's synthesized signal — does not re-classify emotion or pain points. Determines whether a record requires human review and assigns the preliminary response tier (T1/T2/T3) governing how BRA and RDA handle the record.

Governance constraint, stated directly in the agent's own system prompt: "HSI interprets signal meaning only. It never prescribes operational actions, never produces response language, and never re-classifies upstream outputs."

Deterministic synthesis (S4A1–S4A6, Steps 7–12, zero LLM cost): Context Classification → Signal Reliability → Temporal Context → Masked Emotion Prep → Behavioral Context Synthesis → Interpretation Confidence.

Claude call (Step 14, 4 required JSON fields): signal_synthesis_summary, contextual_linguistic_framing, temporal_signal_insight, masked_emotion_hypothesis (conditional).

Preliminary Response Tier (Step 16), first-match-wins:

T3 (Dignity-Restoration): Dignity-Risk Signal, OR Critical intensity, OR (masked+Masked expression+Low confidence), OR pain_point_sub_category contains "Trust, Dignity & Belonging" (⚠️ likely dead code — see below)
T2 (Calibrated): Masked/Ambiguous Negative/Mixed Signal types, OR Negative Signal at High/Moderate intensity, OR ambiguity_flag, OR Medium confidence, OR Conflicted expression
T1 (Standard): everything else

Human Review Gate (Step 17) — the concrete code-level governance mechanism:

human_review_required = true when:
  tier === 'T3'
  OR interpretation_confidence === 'Low'
  OR structural_confidence_score < 0.50
  OR reliability_tier === 'Weak'

Confirmed open bugs — one likely severe:

Step 6 IF (Node 11) — likely case-sensitivity routing bug, not yet live-tested. Compares a genuine JS boolean (is_named_reviewer) against the string "True" (capital T) with case-sensitivity on. A JS boolean stringifies to lowercase "true". If confirmed: every record, named reviewer or not, routes to the anonymous-gate fallback — prior_record_count permanently 0, recurrence_signal permanently "First Occurrence." Repeat-guest and systemic-pattern detection would be non-functional in production despite the code appearing correct. Document proposes a specific test (submit a record from a reviewer with known prior HSI records, confirm whether the real lookup fires) — not yet executed as of this document.
T3 tier's "Trust, Dignity & Belonging" clause checks the wrong field (pain_point_sub_category instead of pain_point_domain_confirmed) — almost certainly never matches. Partial coverage survives via the separate Dignity-Risk Signal check.
Dead patch-body pattern (Step 22/23) — same pipeline-wide issue as elsewhere.
Missing onError/retry on the BRA trigger — fourth confirmed instance of this pattern pipeline-wide.

Architectural corrections vs. the prior v3.0 doc (not incremental drift — v3.0 was substantially wrong): no severity score exists anywhere in the live system (v3.0 described a 1–10 scale that was never built); Claude outputs are 4 fields, not 2; Run ID format is pure timestamp, not domain-encoded; node count is 32, not 30.

BRA — Brand Response Architect

Version: v4.0 · Last Updated: Chat #98, July 24, 2026 · Model: Hybrid — T1 deterministic (zero AI) / T2-T3 claude-sonnet-4-6 · 24 nodes

⚠️ Naming unresolved: live system prompt says "Brand Response Architect"; a prior baseline said "Brand Response Agent." Recommended (not yet executed) to resolve alongside HSI's parallel naming question in one pass.

Receives HSI's tier assignment, produces the strategic response layer (axis framing, tone, urgency, governance assessment) that RDA drafts from. BRA does not draft the public response — that is RDA's role.

Hybrid architecture:

T1 (~70–80% of records, estimate not yet re-measured against real pilot volume): deterministic template selection via seeded hash on hsi_record_id — same record always selects the same template, described explicitly as "a governance feature, not an optimization" for auditability. Zero AI cost. tone_style hardcoded "Empathetic-Warm," commercial_risk_flag hardcoded false, governance_flag starts "None."
T2/T3 (~20–30%): Claude call (max_tokens 500) producing 7 structured fields: emotional_axis, signal_based_consideration, tone_style (4-value enum), urgency_level (3-value enum), commercial_risk_flag (boolean), governance_flag (None/Flag/Halt), behavioral_interpretation.

signal_based_consideration is explicitly never client-visible — internal reasoning artifact only, passed to RDA for strategic context.

Governance enforcement — two layers:

Pre-flag (Step 7): commercial_risk_preflag set deterministically for Dignity-Risk or Masked Negative Signal types.
Halt Check (Step 11) — the pipeline's actual hard-stop for discrimination/legal/escalation content. Triggers on governance_flag === "Halt".

Confirmed open bugs — one is the most consequential governance-observability finding in the entire audit:

Step 11 Halt Check has zero NocoDB error-record write, zero status update, anywhere. It is a bare throw. This is explicitly stated as "the highest-stakes governance control in the pipeline and currently has the weakest failure-path observability of any node audited across ALA through BRA." If it correctly blocks genuinely dangerous content, there is currently no way to confirm that happened except raw n8n execution logs — not queryable, not reportable.
Step 9b's governance override (meant to catch Dignity/Trust-risk signals slipping into the T1 lane) is a second confirmed instance of the same wrong-field bug found in HSI — checks pain_point_sub_category instead of pain_point_domain_confirmed, and separately, one of its two trigger conditions is structurally unreachable in the T1 branch at all (HSI already routes true Dignity-Risk Signal records to T3). Likely fully inert.
Whether Step 9b actually connects to Step 10 (Governance Gate) was never confirmed across any audit batch — could be a partial-export artifact, needs direct live-connections-JSON verification.
Template library's claimed "21 templates, 13 domains" not re-queried this cycle — unverified against live table.
Dead patch-body pattern (4th confirmed occurrence) and missing onError/retry (4th/5th confirmed occurrence) — same pipeline-wide patterns.
Prior claim that BRA passes fields through "without mid-pipeline additions" is corrected — BRA derives 14+ new fields.
RDA — Response Drafting Agent

Version: v6.0 · Last Updated: Chat #100, July 25, 2026 · Model: claude-sonnet-4-6, all calls, both languages · ~46 nodes, 5 Claude calls/record

Terminal automated agent. Translates BRA's strategy into governed draft language in the guest's own language, calibrated to tier and brand voice. No draft reaches any platform without explicit human approval — this is now the most structurally confirmed claim in the entire audit: Published Timestamp is confirmed to stay permanently null at approval; external publishing is a separate, unbuilt mechanism. No code path currently exists that could auto-publish a draft.

Governance principle, reaffirmed in this doc: mechanical/deterministic rules belong in code; judgment rules belong in the Claude prompt.

Spanish branch — fully rebuilt (Chat #97) as a completely independent, natively-authored parallel chain from generation through audit, after a prior single-prompt-with-conditional-language-block design caused live within-generation code-switching. Zero language-mixing observed since the rebuild. Several deliberate asymmetries between EN and ES chains are locked design decisions (e.g., ES excludes an idiom-repetition ban that was tried and reverted due to observed quality regression).

Approval Status — 4 values: Pending / Approved / Modify / Not Accepted. Published was removed entirely from this schema — this single change is the confirmed root cause of a major downstream defect in MRA (see §6).

Confirmed open bugs:

Step 12/Step 18 elevation-signal discrepancy — confirmed this session via direct live-editor verification, correcting a prior version's claim that this worked correctly. Step 12 writes approval_status = 'Pending' unconditionally, no branching. Step 18 checks for 'Pending-Elevated', a value that no longer exists in the 4-value schema. Every approval email uses the identical generic subject line regardless of severity/tier — though elevation_reason text itself is still correct in the email body.
Step 11 Commercial Commitment Scan (checks for refund/discount/compensation/etc. before a draft can proceed) is CONFIRMED ENGLISH-ONLY — no Spanish prohibited-terms equivalent exists. Explicitly characterized as "a governance/safety gap, not a style gap." Real exposure for AJI-001's live Spanish-language reviews today, not hypothetical.
Steps 6c/6d (signal enrichment lookups) query English-only dictionary tables, despite sitting upstream of the language router — very likely produces a silently blank signal enrichment brief for every Spanish-language record (no error thrown, defaults to {}).
Cross-agent dependency: Step 6c's lookup only returns correct matches because of EIP's own Issue 9.8 mislabeling — flagged as needing coordination in both agents' docs before either is fixed in isolation.
Dead guest_name JSON field — always null, carried through ~8 nodes, never written to NocoDB. Distinct from an already-resolved guest-name-in-draft-text bug.
No confirmed email-send node exists anywhere in the workflow after the approval-notification email is constructed (subject/body/recipient all built, but no confirmed dispatch mechanism) — may be a second, separate broken/unconfirmed delivery path distinct from MRA's.
Step 7c's (EN chain) suspected missing + operator — flagged since Chat #96, still not verified or touched as of this document.

Confirmed working: Spanish branch fully independent, zero language-mixing; duplicate-greeting bug resolved; internal follow-up brief correctly English-only regardless of guest language; SEO keyword usage tracking code/schema complete (not yet tested against a real batch); Language Router wiring confirmed correct against the live system this session; real positive client feedback confirmed (Aji Ceviche Orlando owner).

SIA — Signal Intelligence Aggregator

Version: v5.0 · Last Updated: Chat #104, July 25, 2026 · Model: None — pure JavaScript, zero AI calls · 18 nodes

⚠️ Name correction: "Signal Intelligence Aggregator," not v7.4's "Signal Intensity Assessment."

Zero-cost, schedule-triggered batch aggregation agent — architecturally distinct from every other agent (no webhook in, no trigger out; three independent Schedule Triggers: daily 7am UTC, weekly Monday, monthly). This IS the pipeline's trend/pattern intelligence layer: reads ALA records (filtered by guest review posting date, not pipeline processing time), matches to EIP records, groups by Client ID + Domain + Signal Tier, computes trend direction against the prior run of the same window type (Stable/Growing/Declining/New, 20% change threshold), writes clusters to its own NocoDB table (mdn68l4lm609fve).

Confirmed correct: trend-tier mapping exhaustively verified against all 6 of EIP's real Signal Type enum values; a real production row's Declining classification independently verified correct; the multi-client composite key includes client_id (an inline code comment "✅ FIXED: Include client_id in key" suggests this correctness fix — preventing cross-client trend contamination — was itself a relatively recent addition, not part of the original design).

Confirmed open issues:

Weekly/monthly trigger hours disagree between the prior doc (6am/6am) and live code (8am/9am) — neither confirmed as the intended correct value.
Steps 5/7 fetch ALL records unfiltered with hardcoded limit=1000, ambiguous-to-missing sort — explicitly flagged as a "ticking time bomb": inert today, but once total record count across all clients exceeds 1000, will silently serve stale/wrong-window data with no error surfaced anywhere. Directly relevant to the stated 6→10 location growth target.
No post-write verification after the final bulk POST — unlike every other agent, no "Capture Record ID, throw if missing" step. A partial write failure could go unnoticed for an extended period since this is an unmonitored scheduled job.
Real production data showed daily-cadence clustering producing 5 clusters, all singletons (n=1) — open design question whether daily-cadence clustering is analytically meaningful at current per-client review volume, not a code defect.
Step 4 (Set Window Bounds) confirmed complete no-op — architectural symmetry only, recommended for deletion.
Downstream consumer of SIA's output table was NOT independently confirmed as of SIA's own document — resolved by MRA's document (see §6), which confirms MRA is the consumer.
MRA — Metrics & Reporting Agent

Version: v5.0 · Compiled: Chat #103, July 25, 2026 · Model: None — pure JavaScript · Node count: UNRECONCILED (30 per one source doc, 38 per a deeper-audit doc)

Generates scheduled intelligence reports for all active clients across three cadences — daily (8am UTC, 24hr summary), weekly (Monday 9am UTC, intelligence brief), monthly (10am UTC, 1st-of-month pattern report). Zero AI calls. Confirmed consumer of SIA's cluster output for weekly/monthly reports (resolves SIA's own unconfirmed-consumer open item); reads EIP directly for daily reports.

Single scheduled trigger fans out to all Client Config rows via Loop Over Items — one workflow run processes every client.

Confirmed critical bugs (this is the source of the previously-known "hardcoded fake metric" gap, now fully traced):

Dead Published-status dependency. RDA's Approval Status schema removed Published entirely (see RDA §above), and Published Timestamp is permanently null. MRA's Step 9 metric logic still checks status === 'Published' — a value that can never occur again. This permanently zeroes, for every client, every report, forever: published_count, sla_compliance_rate, response velocity, and all SEO metrics (avg velocity, response rate, top keyword, coverage rate, keyword hits). This is broader than "SEO is broken" — the entire performance-metrics section of every report is dead.
Weekly email (17b) hardcodes a fabricated "100% Response Coverage" metric, shown to every client every week regardless of real data — displayed directly beside the genuinely-computed (and permanently zero, per the bug above) "SLA Compliance: 0%," producing a visibly self-contradictory pair of numbers in the same email.
Daily report signal-tier classification (§7 of source doc) only counts 3 of EIP's 6 real Signal Type values, plus a mistyped value matching nothing real. Masked Negative Signal, Dignity-Risk Signal, and Ambiguous Negative Signal are silently uncounted in every daily report — including Dignity-Risk, "the platform's most safety-critical category." SIA's own getSignalTier() function already handles this correctly and could be reused.
Magic Link dashboard access token generated via Math.random(), not a CSPRNG, despite functioning as a real access-control credential (72hr expiry) embedded in live dashboard URLs. Security-relevant, unfixed.
Trend-arrow logic checks for 'UP'/'DOWN' enum values, but SIA's real values are Stable/Growing/Declining/New — the arrow always falls back to a static →, real trend word never displayed.
Delivery status marked "sent" unconditionally, without checking Brevo's actual send response — a soft failure (bad recipient, Brevo-level rejection) would not be caught.
Two previously-fixed but historically severe bugs: (1) daily reports were silently stopping after client #1 on every run due to a loop-back wiring defect — fixed by moving the loop-back to a node that unconditionally emits one item every iteration; (2) a hardcoded literal in two email-template nodes was sending every client (including all 5 AJI locations) PAK-001's own Google Sheet link — fixed.
Sender name casing inconsistent ("VRYOH intelligence" vs. "VRYOH Intelligence") across different email templates.
Same unfiltered limit=1000 fetch risk pattern as SIA, at three additional nodes.

Unresolved design questions (not defects): MRA's monthly window uses a true calendar-month boundary; SIA's monthly window is a rolling 30-day window — these only coincide by coincidence, and since MRA reads SIA's cluster data directly for its monthly report, the report's stated date range may not match what the underlying data actually covers. Not yet reconciled.

Sheet-Sync — Google Sheets Approval Surface

Version: v3.0 · Last Updated: Chat #100, July 25, 2026 · No LLM calls

Populates each client's dedicated Google Sheet with pending RDA drafts, enriched with original ALA review text, routed by Client ID. Not an approval agent — surfaces data for human approval; does not decide approvals itself. The feedback loop (sheet edit → RDA write-back) is designed but not built — unchanged since v2.0.

Schedule: 5am UTC daily, no webhook. Uses a Service Account for Google Sheets auth (deliberately, over OAuth2, which "doesn't survive scheduled/task-runner execution reliably").

Three findings confirmed directly from real pasted production sheet rows this session (not inferred):

Mojibake in review text — [5â˜…]-style corrupted rating markers, visible in real output. Attributed to ALA-side CSV encoding (consistent with ALA's own documented UTF-8-vs-Windows-1252 warning).
Closing-line punctuation collision — a real draft ends "Cheers!," with a comma appended directly after an exclamation point. Triggers only when the closing phrase already ends in punctuation. RDA-side.
T1 opening pattern deviation — both observed real drafts open "[Name], thank you so much..." rather than the documented required pattern, 2-for-2 in the available sample. RDA-side.

All three are explicitly attributed to upstream agents (ALA, RDA), not to Sheet-Sync's own logic.

Also confirms (harmlessly) the same dead ≠ 'Published' pattern found consequentially in MRA — here it's redundant with other filter conditions and does not change actual filtering behavior.

A separate "node-by-node audit" document claiming 30 nodes and 11 additional defects was explicitly not incorporated into this record, because it was never independently confirmed against the live workflow and was written in instruction form rather than as a passive technical record — a direct, documented application of the standing verification protocol.

4. Cross-Agent Bug Patterns — Consolidated

Several defect classes recur across multiple independently-audited agents, suggesting systemic patterns worth a single coordinated pass rather than per-agent fixes (diagnosis only — no fix proposed or scheduled by this document):

Pattern	Confirmed In	Count
Dead "patch body computed but never used"	EIP, ESS, HSI, BRA	4 agents
Missing onError/retry on inter-agent webhook triggers	EIP→ESS, ESS→HSI, HSI→BRA, BRA→RDA (only ALA→EIP has it)	4–5 handoffs
Wrong-field check for "Trust, Dignity & Belonging" (checks pain_point_sub_category instead of pain_point_domain_confirmed)	HSI Step 16, BRA Step 9b	2 agents, identical mistake
Unfiltered fetch with hardcoded limit=1000, ambiguous/no sort ("ticking time bomb")	SIA (2 nodes), MRA (3 nodes)	5 nodes across 2 agents
Dead dependency on RDA's removed Published status value	MRA (consequential), Sheet-Sync (harmless/redundant)	2 agents
Agent acronym disputed between live system prompt and prior documentation	HSI ("Human" vs "Hospitality"), BRA ("Architect" vs "Agent")	2 agents, unresolved
5. Governance Architecture — Where It Is Strong, Where It Has Gaps

Strong, code-confirmed:

Universal human-approval gate at RDA — no code path for auto-publish exists anywhere in the system.
HSI's human_review_required — a computed, deterministic boolean based on four independent failure-mode conditions, not a policy statement.
BRA's Halt Check — a real hard-stop that terminates a record's processing on detection of discrimination/legal/escalation content.
Masked-emotion detection (ESS's Expression Mode) — explicitly documented in the agent's own HOW doc as the platform's stated competitive differentiator, with working cross-referencing against star rating.

Confirmed gaps, all currently live, none fixed (Diagnose-Only Rule applied throughout):

BRA's Halt Check has zero observability — the highest-stakes control in the pipeline currently cannot be confirmed to have fired except via raw execution logs.
RDA's Commercial Commitment Scan (the deterministic backstop against unauthorized refund/discount promises) is English-only, with live Spanish-language exposure at AJI-001 today.
Two independent agents (HSI, BRA) have the same wrong-field bug in their respective Dignity/Trust-risk safety nets — likely both inert, with partial coverage surviving only via a separate Dignity-Risk Signal check.
MRA's daily reporting undercounts exactly the signal categories (Dignity-Risk, Masked Negative, Ambiguous Negative) that the governance architecture is most explicitly built to surface.
A likely (not yet live-tested) case-sensitivity bug in HSI would silently disable repeat-guest/systemic-pattern detection — a capability directly relevant to distinguishing a one-off complaint from a recurring operational problem.
6. Open Items Requiring Owner Decision (Carried Forward, None Resolved)

This section consolidates every unresolved open item across all 9 source documents. Nothing below has been actioned.

Confirm actual live MRA node count (30 vs. 38).
Resolve HSI acronym ("Human" vs. "Hospitality") and BRA acronym ("Architect" vs. "Agent") — recommended as one combined naming pass.
Live-test the suspected HSI Step 6 case-sensitivity bug (reviewer-history routing).
Decide whether to fix the wrong-field Trust/Dignity checks in HSI Step 16 and BRA Step 9b (same fix, two locations).
Decide whether to add a NocoDB error-record write to BRA's Halt Check.
Decide on one pipeline-wide cleanup pass for the dead-patch-body pattern (4 agents) and missing-onError pattern (4–5 handoffs).
Re-query BRA's template library for actual row count and domain breakdown ("21 templates, 13 domains" unverified this cycle).
Re-measure BRA's and HSI's token-efficiency figures against current live prompt structures (prior figures are stale, based on superseded architectures).
Confirm whether BRA Step 9b actually connects to Step 10 in the live workflow (never observed across any audit batch).
Decide fate of MRA's dead-Published-status metrics — redefine or leave as-is.
Decide whether to fix MRA's daily signal-tier classification to cover all 6 real Signal Type values.
Decide whether to fix MRA's weekly email's hardcoded 100% and broken trend-arrow logic.
Decide whether to switch MRA's Magic Link Token to a CSPRNG.
Decide whether to add Brevo-response verification to MRA before marking delivery "sent."
Reconcile MRA's calendar-month window against SIA's rolling-30-day window.
Confirm intent behind MRA's period_end midnight-truncation (excludes final day of each period).
Decide MRA's fallback behavior when Approval Contact Email is blank.
Decide whether to switch MRA's SEO tracking to read RDA's SEO Keywords Used field directly (would also bypass the dead-Published dependency for that specific metric).
Confirm whether SIA/MRA's unfiltered limit=1000 fetches need server-side date filtering or pagination before record volume crosses the threshold.
Confirm whether SIA's daily-cadence clustering is analytically meaningful given real per-client volume, or should be redesigned/deprioritized relative to weekly/monthly.
Confirm whether the approval-notification email in RDA (subject/body/recipient constructed, no confirmed send node) is actually being dispatched by some mechanism outside this n8n workflow, or not being sent at all.
Verify RDA's suspected missing + operator in Step 7c (EN chain) — flagged since Chat #96, still untouched.
Decide the intended fix for RDA's guest-name field mismatch (Step 12/Step 18 elevation-signal dead code).
Add a Spanish-language equivalent for RDA's Commercial Commitment Scan.
Add language branching to RDA's Steps 6c/6d signal-enrichment lookups, coordinated with a fix (or non-fix) of EIP's Issue 9.8.
BCA (Batch Controller Agent) status is entirely unconfirmed — present in v7.4 as "deferred," not mentioned in any document read this session. Needs a direct status check: built, abandoned, or renamed.
Confirm current dashboard build status (v7.4 describes a planned build; MRA's magic-link token confirms some dashboard-adjacent access mechanism exists in production, but the dashboard itself was not independently confirmed built or unbuilt this session).
7. Infrastructure (Unchanged from v7.4 Unless Noted)

Hosting: DigitalOcean droplet, NYC3, Ubuntu 24.04, 4GB RAM, 50GB disk, IP 161.35.133.49.

Stack: n8n 2.4.6 self-hosted + NocoDB self-hosted. Base ID pq249fix22t3ofv for all pipeline tables.

NocoDB internal URL inside n8n: always http://nocodb:8080 — never localhost or external IP (confirmed as a locked build rule across every agent doc read this session).

Credentials confirmed across agents:

NocoDB: xc-token, httpHeaderAuth, credential id DT9tnRgqYpPc3rXo
Anthropic: x-api-key, httpHeaderAuth, credential id uMahlx4nOC5YJh0Z, manual headers anthropic-version: 2023-06-01 + Content-Type: application/json
Google Sheets (Sheet-Sync only): Service Account Subtext-CX-GoogleSheets-ServiceAccount, id Pf4MiR7hQF3eu3ts
Subtext-CX-OpenAI credential exists but is confirmed unused by RDA in production.

Locked build rules (consistent across all 9 documents):

No spread operator in Code Nodes (confirmed safe only in ALA Steps 6–7, specifically flagged as an exception, not a general permission).
pageInfo.totalRows, never list.length, for NocoDB record counts.
PATCH uses JSON body mode, not RAW.
POST bodies: JSON.stringify() in Code Node + RAW in HTTP Request (except BRA→RDA and RDA's own writes, which use specifyBody: "json" — an explicitly noted exception to the pipeline-wide convention).
No console.warn/console.log — blocked by the n8n task runner.
IF node FALSE branches require an explicit return [] gate.
require('http') works in Code Nodes; $http/$helpers/fetch are blocked.
8. NocoDB Table IDs — Corrected and Expanded
Agent/Table	Table ID	Notes
ALA	m57efwbtrvwohhr	17 fields incl. Raw Tex (truncated field name, do not rename until post-pilot)
EIP	mhicpnrahaesxmy	23 fields written; Enriched Emotion Tag mislabeled — see Issue 9.8
ESS	m5yektnbtxf8evk	Does not store Core Emotion/Need State despite a prior doc's claim
HSI	mb8nv8t3nk6xzed	22+ fields incl. Human Review Required (Checkbox), Preliminary Response Tier
BRA	mwqejw7swhd2cf4	24 fields; several v3.3-documented fields confirmed NOT in live table
RDA	mr1v67cszcklwns	21 fields, confirmed NOT frozen; Approval Status = 4 values only
SIA	mdn68l4lm609fve	20 columns total (5 system + 15 data), live-verified 2026-07-21
MRA output	Reads RDA/SIA/EIP/ESS/Client Config directly — no dedicated MRA output table confirmed in this session's documents	
Template Library	mafv9by73ebama7	Row count/domain breakdown unverified this cycle
Client Config	m95cmabjfyb94ps	22–25 fields depending on source doc (unreconciled)
Emotion Dict EN	mrrscb955j1d2i7	161 rows
Emotion Dict ES	mhot4w62tupht71	161 rows, audited clean June 2026
Pain Point Master EN	meavqh37mdqgl4d	336 rows
Pain Point Master ES	mwmiyyoucuhsms8	253 rows, audited clean June 2026
Schema Registry	mqv1znpza948pm9	Referenced by HSI's doc; not independently re-fetched this session
9. Communication & Process Rules (Unchanged)

Every response begins Chat #[N] · [Date]. Diagnose-Only Rule in effect for all technical audit work — problems are identified and characterized precisely; fixes are proposed only when explicitly requested. Source hierarchy: direct paste > live code read > raw.githubusercontent.com fetch (medium confidence) > github.com non-raw (historically unreliable, though functional this session via full page render) > a HOW doc's own claimed counts (claim, not fact, until cross-checked).

10. What This Document Does Not Cover
Commercial/pricing state — maintained separately in commercial-side memory, not duplicated here to avoid drift between two sources of truth. v7.4's pricing table is explicitly stale and should not be referenced.
Competitive positioning (RightResponse AI, Birdeye, etc.) — maintained separately in commercial-side memory.
Dashboard build status — not independently confirmed this session; flagged as Open Item #27.
BCA status — not confirmed this session; flagged as Open Item #26.

End of MCD v7.5 · Chat #105 · August 20, 2026 · Solofella LLC Synthesized from 9 live-code-tier HOW documents (ALA v5.0, EIP v5.1, ESS v5.0, HSI v4.0, BRA v4.0, RDA v6.0, SIA v5.0, MRA v5.0, Sheet-Sync v3.0), all read in full this session. Supersedes MCD v7.4 in all conflicts. No fixes proposed or applied — diagnosis and consolidation only, per standing Diagnose-Only Rule.
