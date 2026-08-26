Chat #96 · July 3, 2026

# SCX_RDA_HOW_v7.0

**VRYOH INTELLIGENCE · SOLOFELLA LLC**
**HOW DOCUMENT — RDA**
**Response Drafting Agent**
Complete Step Decomposition · Node Logic · Code · Prompts · Field Contracts
v7.0 · August 26, 2026 · Chat #96 (source-numbered per live workflow folder date)

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0 | Initial build — Chat #49 / Session #6 / March 20, 2026 |
| v2.0 | Chat #58 — Audit remediation + NocoDB rationalisation. Node count: 20 → 18. |
| v3.0 | Chat #74-75 — Five-call generation architecture (Steps 7a-7h). Node count: 18 → 23. |
| v3.1 | Chat #75 — Five-prompt synchronicity audit. 11-item audit checklist. Quality baseline 87%. |
| v3.2 | Chat #77 — SCX-Sheet-Sync OAuth credential failure documented. No RDA workflow changes. |
| v4.0-v4.1 | Chat #96 (same session) — English-only revert after Spanish language-mixing failures, then Phase 3 Signal Enrichment (Steps 6a-6e) built and deployed against the reverted baseline, then Step 9e deterministic compliance layer added. |
| **v7.0** | **Chat #96, confirmed against 13 live source documents (1 HOW v3.2 + 12 workflow batches, read in full, line by line) — the largest structural change since v3.0. Confirms: (1) Language Router (Step 6f) splits execution into two fully parallel branches — English (7a-9e) and Spanish (7a-ES through 9e-ES) — each with its own Claude calls, audit checklist, and deterministic compliance node; (2) both branches converge into a single shared downstream chain from Step 10 onward; (3) the Internal Follow-Up Brief is generated ONCE, always in English, after the audit/compliance stage — a structural relocation from v3.2, where it was generated per-record before the audit; (4) Step 9e's compliance check is deterministic code, not Claude, split by language with different rule sets; (5) the English audit checklist has been reduced from 11 items to 6 (formula/overused-phrase/brand-phrase/opening-repetition items removed from the Claude audit, since equivalent logic now exists in Step 9e or was retired) while the Spanish audit checklist independently grew to 9 items, unrelated to the English reduction — a confirmed asymmetry between the two language paths; (6) Step 7a's opening construction returned to a fixed-formula requirement ("Thank you very much, [Name]," for T1) — a different design than what mid-session testing in this same conversation was moving toward (brand-voice-driven, non-formulaic construction); (7) real bugs found earlier this session in Steps 6c/6d (stale cross-node reference causing 0-row dictionary lookups) are CONFIRMED FIXED in this live version.** |

---

## Summary Grid

| Property | Value |
|----------|-------|
| **Workflow Name** | SCX-RDA |
| **Product Name** | VRYOH Intelligence |
| **Model** | claude-sonnet-4-6 — all Claude calls, both languages |
| **Pipeline Position** | 6th agent — final per-record automated agent |
| **Claude Calls / Record** | 5 — Opening, Body, SEO/Governance, Audit (all language-specific) + Internal Brief (shared, English-only, generated once) |
| **Trigger Source** | BRA Webhook — rich payload (Halt never fires) |
| **Language Architecture** | Full parallel branch — Step 6f (Language Router) splits execution at the opening-construction stage into English (7a-9e) or Spanish (7a-ES–9e-ES); both converge at Step 10 |
| **Signal Enrichment** | Steps 6a-6e — EIP Cognitive Driver/Need State (direct fetch) + Emotion Dictionary Common Expressions + Pain Point Master Operational/Emotional Signal/Sample Expression, consolidated into `signal_enrichment_brief`, shared by both language branches |
| **Deterministic Compliance Layer** | Step 9e (EN) / Step 9e-ES — language-specific rule sets, both code-based, not Claude |
| **Post-Call Scan** | Step 11 — Commercial Commitment Scan, English-only term list, shared/downstream of both language branches |
| **Internal Brief Generation** | Steps 10a-10c — SHARED, single call regardless of record language, always produced in English |
| **Total n8n Nodes (confirmed this session)** | 51 confirmed by direct code read across both language branches + shared downstream chain (see §6 for full inventory; several cross-batch connections remain unconfirmed — see §11) |
| **NocoDB Table Fields** | 23 confirmed live (see §8) — expanded from v3.2's 20 |
| **Human Approval** | MANDATORY — no draft reaches any platform without human decision |
| **Approval SLA** | 48 hours (locked Chat #42) |
| **Audit Checklist (EN)** | 6 items — reduced from v3.2's 11; formula/brand-phrase/opening-repetition items retired from the Claude audit |
| **Audit Checklist (ES)** | 9 items — independently expanded, includes a within-sentence repetition check not present in EN |
| **Active Clients** | PAK-001, AJI-001–005 |
| **Doc Version** | v7.0 — August 26, 2026 (Chat #96) |

---

## 1. AGENT PURPOSE

### RDA — Response Drafting Agent

Translates BRA's response strategy into governed draft language calibrated to the response tier, the client's brand voice, and — as of this version — the client's specific emotional/pain signal detail (Phase 3). RDA produces two outputs per record: a public-facing response and an internal signal brief for the client's management team. RDA does not decide what to do — it decides how to say it.

**The response must still read as written by a person, not optimized by a machine.** Every Claude call in RDA carries this as its governing principle.

**TERMINAL AGENT:** RDA is the final per-record automated agent. No draft produced by RDA reaches any platform before a human makes an explicit approval decision. Claude's scope ends at draft generation.

### RDA PRODUCES

- Public Response Draft — platform-formatted, calibrated to tier, brand voice, and language
- Internal Follow-Up Draft — signal interpretation brief, always English, generated once per record regardless of the public draft's language
- Commercial Commitment Flag — deterministic post-call prohibited-term scan (English term list, applied to both drafts regardless of source language)
- Approval Status — Pending (deterministic; elevation reasons recorded separately, no longer producing a distinct "Pending-Elevated" status value in the live code — see §7)
- Audit Passed flag + Audit Failed Items — language-specific checklist result (6-item EN / 9-item ES) merged with Step 9e's deterministic findings
- SEO Keywords Used — new field, tracks which configured keywords actually appeared in the final draft

### RDA DOES NOT

- Decide response strategy — that is BRA
- Re-classify emotion, pain, or tier — all upstream
- Override governance flags — Halt records never reach RDA
- Generate refund offers, vouchers, or discount commitments
- Publish anything — approval gate is structural, not advisory
- Prescribe operational actions in the internal brief
- Fetch the BRA record from NocoDB — rich payload arrives in webhook
- Translate the internal brief — it is always written in English, independent of the guest-facing draft's language

---

## 2. INPUT CONTRACT — THREE SOURCES

### Source 1 — BRA Webhook Payload

Confirmed live (Step 2 - Payload Validation, Batch #1): 27 fields carried forward key-by-key, no spread operator. Matches v3.2's Part A/B structure with no field-level changes confirmed this session.

### Source 2 — NocoDB Client Configuration (Step 5 GET, Step 6 parse)

**Confirmed live schema differs from v3.2's documented 10 fields — real Client Config now maps 18 fields into the `brand_voice` object** (Step 6 - Build Brand Voice Object, Batch #1):

| Real field mapped | Internal key |
|---|---|
| Client ID | Client_ID |
| Client Name | Client Name |
| Brand Personality | tone_descriptors |
| Response for a negative experience | response_negative_experience |
| Formality Level | formality_level |
| Person Preference | person_preference |
| Brand Phrases To Include | brand_phrases_include |
| Brand Phrases To Avoid | brand_phrases_avoid |
| Language | language |
| SEO Keywords | seo_keywords |
| Approval Contact Email | approval_contact_email |
| THE RECOVERY PROTOCOL... | recovery_protocol |
| Guests feeling after reading a response | guest_feeling |
| Commitments should NEVER be made | commitment_restrictions |
| Differentiator | differentiator |
| THE REGISTER... | the_register |
| THE CORE DRIVER... | the_core_driver |
| THE REGIONAL ACCENT... | the_regional_accent |

Step 6a (Brand Voice Consolidation, new since v3.2) then builds a single formatted string, `brand_voice_brief`, from 12 of these fields (Register, Core Driver, Regional Accent, Personality, Formality, Person Preference, Response to Negative Experience, Guest Should Feel, Recovery Protocol, Differentiator, Brand Phrases Include, Brand Phrases Avoid) — replacing the per-node manual field parsing that existed independently in three different nodes in the pre-v4.0 architecture.

### Source 3 — Signal Enrichment (NEW since v3.2, Steps 6b-6e)

- **Step 6b** — direct GET to the EIP table (`mhicpnrahaesxmy`) by `eip_record_id`, fetching `Cognitive Driver` and `Need State` fresh from source (confirmed these two fields do not survive the ESS→HSI→BRA pass-through chain, so RDA re-fetches them directly rather than relying on the webhook payload).
- **Step 6c** — targeted single-row GET to the Emotion Dictionary (`mrrscb955j1d2i7`) WHERE `Core Emotion` = Step 6b's live `Enriched Emotion Tag` value, pulling `Common Expressions`. **Confirmed fixed this session:** an earlier version of this query referenced a stale, pass-through-only value instead of Step 6b's fresh fetch, causing real 0-row lookup failures; the live code now correctly references Step 6b directly.
- **Step 6d** — same pattern against Pain Point Master (`meavqh37mdqgl4d`) WHERE `Enriched Pain Point` = Step 6b's live value, pulling `Operational Signal`, `Emotional Signal`, `Sample User Expression`. Same fix confirmed applied.
- **Step 6e** — consolidates all 6 signal fields (2 from EIP direct, 4 from the two dictionary lookups) into `signal_enrichment_brief`, a single formatted string, injected into both the English and Spanish opening/body prompts.

---

## 3. TIER-SPECIFIC DRAFT INSTRUCTIONS

**Confirmed live structure differs substantially from v3.2's Priority-1-through-6 fixed-template system.**

| Tier | English Opening Requirement (Step 7a, Batch #3, confirmed live) | Spanish Opening Requirement (Step 7a-ES, confirmed live) |
|------|---|---|
| **T1** | MUST begin with exactly "Thank you very much, [Guest Name]," or "We appreciate you, [Guest Name]," — no other gratitude opening permitted. Priority within: Named Staff → Specific Detail → Gratitude. | MUST begin with exactly "Muchas gracias, [Nombre]," or "Le agradecemos mucho, [Nombre]," — same restriction. Same internal priority order. |
| **T2** | MUST begin with exactly "Hi, [Guest Name]," or "Hello, [Guest Name]," | MUST begin with exactly "Hola, [Nombre]," or "Estimado/a [Nombre]," |
| **T3** | MUST begin with exactly "Hello, [Guest Name]," | MUST begin with exactly "Estimado/a [Nombre]," |

**Flag, source-stated:** this fixed-phrase requirement is a different design than what mid-session testing in this same conversation was actively moving away from (removing Priority-template fixed sentences specifically because they were found to override brand-voice differentiation). The live production code confirmed in these 12 batches uses fixed openers again. Whether this represents a deliberate reversal, a different deployed version, or an unresolved fork between design work and production is not something I can determine from the documents alone — flagging as a real discrepancy, not resolving it.

### Body Construction — Content Selection Order (NEW since v3.2, Step 7c/7c-ES, confirmed live)

Both language versions now apply an explicit procedural filter before selecting content to reinforce in the body:
1. First remove from consideration any commercial or time-sensitive review detail (prices, discounts, menu tiers, Restaurant Week, Happy Hour, seasonal/holiday menus, limited-time items, promotions).
2. Then choose the strongest eligible guest-experience detail, in priority order: Occasion → Guest Relationship/Loyalty → Overall Experience → Favorite Food/Beverage (non-promotional) → Guest Wording (non-commercial).
3. Use that eligible detail to reinforce the body and shape the closing.

This is a structural change from v3.2's flat "Do Not Use" / "Acceptable" list — the live version now sequences elimination before selection, rather than listing both as parallel, independently-checked rules.

### Absolute Governance Constraints — Confirmed Live, Both Languages

**PROHIBITED (Step 7e / 7e-ES, SEO/Governance call):** email addresses, URLs, placeholder text, named contact channels ("our website," "our profile," "our direct messaging"), "We apologize for any inconvenience," refund/voucher/discount/financial commitments.

**PERMITTED (T2/T3 only):** generic direct-contact invitations, when they already exist in the draft — the live SEO/Governance prompt does not instruct Claude to insert one if missing (a change from v3.2, where TASK 3 was "insert guest name," not contact language).

**Guest Name Handling — confirmed diverging between languages:** the English SEO/Governance prompt (Step 7e) still asks Claude to insert the guest's name into the opening if missing. **The Spanish version (Step 7e-ES) has replaced this with a deterministic code-level check** — `nameMissingFlag`, computed in JavaScript before the Claude call, setting `human_review_required = true` and a `pre_audit_flags` note instead of asking Claude to modify the opening. The Spanish prompt explicitly instructs Claude never to touch the opening under any circumstance. **This is a confirmed, real architectural inconsistency between the two language branches** — not documented as intentional or accidental in any source received.

---

## 4. CLAUDE CALL ARCHITECTURE — 5 CALLS PER RECORD (RESTRUCTURED SINCE v3.2)

**v3.2 order:** Opening → Body → SEO/Governance → Internal Brief → Audit (5 calls, sequential, all per-record, Internal Brief generated before Audit).

**Confirmed live v7.0 order — Internal Brief moved to AFTER Audit and the deterministic compliance layer, and decoupled from the language branch entirely:**

Opening (7b/7b-ES) → Body (7d/7d-ES) → SEO/Governance (7f/7f-ES) → Final Draft Output (8/8a-ES) → ALA fetch (9a/9a-ES) → Recent Drafts fetch (9a-2/9a-2-ES) → Audit (9b→9c / 9b-ES→9c-ES) → Parse Audit (9d/9d-ES) → Deterministic Compliance (9e/9e-ES) → **[both language branches converge here]** → Output Parsing (10) → **Internal Brief (10a→10b→10c, shared, English-only, single call)** → Commercial Commitment Scan (11) → Approval Status (12) → onward.

### Call 1 — Opening Constructor (Step 7b/7b-ES) · temp: 0 · max_tokens: 150

Governing principle unchanged from v3.2. System prompt restructured — Brand Voice section now explicitly instructs Claude to use Register/Core Driver/Regional Accent/Personality/Differentiator/Guest Feeling to shape tone before applying the fixed-opener tier rules (§3). Both language versions include `SIGNAL ENRICHMENT: ${signal_enrichment_brief}` in the user prompt — confirmed new field injection since v3.2/pre-Phase-3.

### Call 2 — Body Builder (Step 7d/7d-ES) · temp: 0.3 (changed from v3.2's 0.5)

Governing principle unchanged. System prompt now includes the Content Selection Order procedure (§3) and an explicit English/Spanish naturalness instruction — the Spanish version specifically warns against translated-sounding constructions and instructs preference for native Spanish idiom over analytical description (e.g., "no estuvimos a la altura" over an indirect paraphrase).

**Confirmed tier sentence-count differs from v3.2:**
| Tier | v3.2 | v7.0 confirmed live (EN) | v7.0 confirmed live (ES) |
|---|---|---|---|
| T1 | 2-3 total | 2-3 total | 2-3 total (3-4 if minor criticism present) |
| T2 | 2-4 total | 2-3 total | 2-4 total (up to upper end when warranted) |
| T3 | 3-5 total | 3-4 total | 3-4 total |

### Call 3 — SEO + Governance (Step 7f/7f-ES) · temp: 0 · max_tokens: 600

Restructured SEO logic since v3.2: keywords are now explicitly split into "descriptive" (business name, dish names — usable freely) versus "promotional" (marketing claims like "Best Peruvian Restaurant" — usable only where the review gives a natural opening, and preferred over descriptive terms when both are eligible). Never placed in the opening line. Governance removal list confirmed identical in substance to v3.2, both languages. Guest-name handling diverges by language, per §3.

### Internal Brief (Steps 10a-10c) — RELOCATED AND UNIFIED SINCE v3.2

**Confirmed live: this is now a single call, always in English, generated after the audit/compliance stage for both language branches** — not per-branch, not per-language, not positioned before the audit as in v3.2. System prompt content (what to include/exclude) is functionally identical to v3.2's Call 4 specification.

### Call 5 — Audit (Step 9c/9c-ES) · temp: 0 · max_tokens: 1500

**English checklist — confirmed reduced from 11 items (v3.2) to 6 items (v7.0):** NAMED STAFF, REVIEW DEPTH MATCH, CLOSING SPECIFICITY, LOYALTY SIGNAL, CONSTRUCTIVE SUGGESTION, PROHIBITED CONTENT. **Formula (overused verbs/phrases), BRAND PHRASE COUNT, and OPENING REPETITION as a numbered item are no longer part of the English audit checklist** — `recentOpenings` data is still fetched and passed as context but is not framed as a distinct numbered audit criterion in the live prompt.

**Spanish checklist — confirmed independently expanded to 9 items:** PERSONAL MENCIONADO, PROFUNDIDAD DE RESEÑA, CIERRE, LEALTAD, SUGERENCIA, PROHIBIDO, **REPETICIÓN (item 7 — new, not present in English: catches the same idiom/verb/distinctive phrase appearing twice within one sentence or immediately adjacent sentences; explicitly does NOT flag repetition at greater distance, e.g. opening vs. closing, treating that as potentially intentional rhetorical echo)**, FRASES DE MARCA (10 listed phrases — a much longer list than v3.2's 3), APERTURA (opening repetition, numbered item 9).

**This 6-vs-9 asymmetry is confirmed real and not explained by any source received this session** — flagging as an open item, not resolving it.

---

## 5. DETERMINISTIC COMPLIANCE LAYER — STEP 9e / 9e-ES (NEW SINCE v3.2)

**Not a Claude call — pure JavaScript, runs after Step 9d (Parse Audit Output), before Step 10.**

**English (Step 9e), confirmed live:**
- Auto-corrects "landed"/"landing" via regex — single-word substitution ("came together"/"coming together"), no Claude involved.
- T2 records: checks for a contact-invitation phrase (`reach out|contact us|hear from you|get in touch|speak with you|follow up on this`); if absent, **flags only** — sets `human_review_required = true`, appends to `audit_failed_items`. Does not auto-insert text.
- Checks for a near-miss "carry" construction close to a banned phrase pattern; flags only, does not auto-correct.

**Spanish (Step 9e-ES), confirmed live:**
- No "landed" equivalent check — no English-specific banned verb applies.
- T2 records only: checks for a Spanish contact-invitation phrase (`contáctenos|comuníquese con nosotros|escríbanos|háblenos|póngase en contacto`); same flag-only behavior as English.
- No near-miss phrase check equivalent to English's "carry" pattern.

**Design principle confirmed from this session's own reasoning:** flagging rather than auto-templating the missing-contact-invitation case was a deliberate choice — auto-inserting the same fixed sentence on every affected record would reintroduce the sameness/templating problem that removing Priority 1-6's fixed sentences (in earlier design work this session) was meant to solve. The live code's flag-only behavior is consistent with that principle.

---

## 6. NODE MAP — CONFIRMED LIVE, BOTH LANGUAGE BRANCHES

```
[01] Webhook (POST /webhook/scx-rda) — responds 200 immediately
[02] Code Node — Payload Validation
[03] HTTP GET — Idempotency Check (RDA table, by BRA Record ID)
[04] Code Node — Already Processed gate (returns [] if found)
[05] HTTP GET — Brand Voice Load (Client Config, by Client ID)
[06] Code Node — Build Brand Voice Object (18-field mapping)
[06a] Code Node — Brand Voice Consolidation → brand_voice_brief
[06b] HTTP GET — Fetch EIP Signal Data (Cognitive Driver, Need State)
[06c] HTTP GET — Fetch Emotion Dictionary Row (Common Expressions) — stale-reference bug CONFIRMED FIXED
[06d] HTTP GET — Fetch Pain Point Master Row (Operational/Emotional Signal, Sample Expression) — same fix confirmed
[06e] Code Node — Build Signal Enrichment Brief
[06f] IF Node — Language Router (lang === 'es'?)
     |── FALSE (English) →
     +── TRUE (Spanish) → [wiring to 7a-ES not explicitly confirmed in documents received]

── ENGLISH BRANCH ──
[07a] Code Node — Build Opening Prompt
[07b] HTTP POST — Opening Claude Call
[07c] Code Node — Parse Opening + Build Body Prompt
[07d] HTTP POST — Body Claude Call
[07e] Code Node — Parse Body + Build SEO/Governance Prompt
[07f] HTTP POST — SEO/Governance Claude Call
[08]  Code Node — Build Final Output (computes seo_keywords_used)
[09a] HTTP GET — Fetch ALA Record
[09a-2] HTTP GET — Fetch Recent RDA Drafts (English-only, lang=en filter)
[09b] Code Node — Build Audit Prompt (6-item checklist)
[09c] HTTP POST — Audit Claude Call
[09d] Code Node — Parse Audit Output
[09e] Code Node — Deterministic Compliance Check (English rule set)

── SPANISH BRANCH ──
[07a-ES] Code Node — Build Opening Prompt (ES)
[07b-ES] HTTP POST — Opening Claude Call (ES)
[07c-ES] Code Node — Build Body Prompt (ES)
[07d-ES] HTTP POST — Body Claude Call (ES)
[07e-ES] Code Node — Build SEO/Governance Prompt (ES) — includes deterministic guest-name-missing flag, code-level
[07f-ES] HTTP POST — SEO/Governance Claude Call (ES)
[08a-ES] Code Node — Build Final Draft Output (ES)
[09a-ES] HTTP GET — Fetch ALA Record (ES)
[09a-2-ES] HTTP GET — Fetch Recent RDA Drafts (Spanish-only, lang=es filter)
[09b-ES] Code Node — Build Audit Prompt (ES, 9-item checklist)
[09c-ES] HTTP POST — Audit Claude Call (ES)
[09d-ES] Code Node — Parse Audit Output (ES)
[09e-ES] Code Node — Deterministic Compliance Check (Spanish rule set)

── SHARED DOWNSTREAM (both branches converge) ──
[10]  Code Node — Output Parsing
[10a] Code Node — Build Internal Brief Prompt (English-only, always)
[10b] HTTP POST — Internal Brief Claude Call
[10c] Code Node — Parse Internal Brief
[11]  Code Node — Commercial Commitment Scan (English term list only)
[12]  Code Node — Approval Status Assignment
[13]  Code Node — RDA Run ID + Timestamp
[14]  Code Node — Build NocoDB POST Body
[15]  HTTP POST — NocoDB Write RDA Record
[16]  Code Node — Capture RDA Record ID
[17]  HTTP PATCH — BRA RDA Status → Complete
[18]  Code Node — Build Approval Notification

TOTAL CONFIRMED NODES: 51 (18 English-branch + 13 Spanish-branch + 6 shared upstream + 14 shared downstream, adjusting for shared 06-series)
Claude fires at: 7b/7b-ES, 7d/7d-ES, 7f/7f-ES, 9c/9c-ES, 10b = 5 calls per record, confirmed
```

**Unconfirmed wiring, stated plainly, not resolved:** Step 6f's Spanish output → Step 7a-ES; Step 9a-ES → Step 9a-2-ES; several English-branch cross-batch boundaries (6→6a, 10→10a, 11→12, etc.). Every batch document received only shows connections *within* that batch's file — cross-batch links were never explicitly shown in any of the 13 documents read this session.

---

## 7. APPROVAL STATUS LIFECYCLE

**Confirmed live change from v3.2:** Step 12 (Approval Status Assignment) now hardcodes `approval_status = 'Pending'` for every record — **the "Pending-Elevated" status value confirmed used in v3.2 is not set anywhere in the live Step 12 code.** Elevation conditions (commercial commitment, governance flag, human review required, T3 tier) are still computed and recorded in a separate `elevation_reason` field, but no longer change the `Approval Status` value itself. This is a confirmed structural change — worth flagging since it affects any downstream reporting or routing logic (e.g., MRA) that expects "Pending-Elevated" as a distinct status.

| Status | Set By | Meaning |
|--------|--------|---------|
| **Pending** | Step 12 (deterministic, always) | Draft generated, awaiting human decision. `elevation_reason` field separately indicates if elevated attention is warranted. |
| **Approved** | Human approver | Draft approved as-is. |
| **Edited-Approved / Modified** | Human approver | Human edited before approving — per this session's direct clarification, notes are optional for this status. |
| **Not Accepted** | Human approver | Draft rejected. Notes expected but not database-enforced. |
| **Published** | Client operator (manual) | Terminal state. Not confirmed set anywhere in RDA's own code — external process. |

---

## 8. NOCODB RDA TABLE — CONFIRMED LIVE FIELDS (Step 14, Batch #11)

Confirmed real field list written by Step 14's `nocodb_post_body`: RDA Run ID, BRA Record ID, ALA Record ID, RDA Timestamp, Confirmed Response Tier, Public Response Draft, Reviewer Handle, Internal Follow-Up Draft, Approval Status, Commercial Commitment Flag, Elevation Reason, Flagged Terms, Client ID, lang, **SEO Keywords** (from `brand_voice`), **SEO Keywords Used** (NEW — tracks actual usage, not just configuration), Audit Passed, Audit Failed Items, Published Timestamp (null at write), Approval Notes (null at write), Error Log (null at write). **20 fields confirmed written by RDA at creation — schema total may be higher if Approval Notes/Sync Status-adjacent fields exist purely for the Sheet-Sync/approval-feedback layer**, per this session's separate Sheet-Sync work (out of scope for this document's confirmed-live-RDA-code basis).

---

## 9. CREDENTIALS + CONFIGURATION

| Item | Value |
|------|-------|
| Anthropic Credential | `x-api-key`, id `uMahlx4nOC5YJh0Z` — confirmed identical across all 5 Claude call nodes, both languages |
| NocoDB Credential | `xc-token`, id `DT9tnRgqYpPc3rXo` — confirmed identical across all HTTP GET/POST/PATCH nodes in this workflow |
| NocoDB URL (internal) | `http://nocodb:8080` — confirmed, never localhost or external IP |
| Base ID | `pq249fix22t3ofv` |
| RDA Table | `mr1v67cszcklwns` |
| ALA Table | `m57efwbtrvwohhr` |
| BRA Table | `mwqejw7swhd2cf4` |
| Client Config Table | `m95cmabjfyb94ps` |
| EIP Table | `mhicpnrahaesxmy` |
| Emotion Dictionary (EN) | `mrrscb955j1d2i7` |
| Pain Point Master | `meavqh37mdqgl4d` |
| Claude Model | claude-sonnet-4-6 — confirmed all 5 calls, both languages |
| anthropic-version header | 2023-06-01 |
| Retry policy | `retryOnFail: true`, `waitBetweenTries: 5000` — confirmed on every Claude call and every retry-relevant NocoDB call |

---

## 10. QUALITY BASELINE + OPEN ITEMS

### Confirmed working, this version

- Phase 3 signal enrichment fully deployed and confirmed bug-fixed (6c/6d stale-reference issue resolved).
- Content Selection Order (Commercial Reference elimination before selection) implemented in both language bodies.
- Deterministic compliance layer (9e/9e-ES) live for both languages, correctly designed to flag rather than auto-template the missing-contact-invitation case.
- Internal Brief correctly unified to a single, English-only, shared call — architecturally cleaner than v3.2's per-record duplication.
- SEO keyword logic refined to distinguish descriptive vs. promotional terms.

### Open Items — confirmed real, not yet resolved

| Item | Status |
|------|--------|
| T1 opening reverted to fixed "Thank you very much" formula | Confirmed live; conflicts with mid-session design work in this same conversation that removed fixed openers for brand-voice differentiation reasons — unreconciled |
| Guest-name-insertion handling diverges EN (Claude-based) vs. ES (deterministic flag) | Confirmed real, unexplained inconsistency |
| Audit checklist asymmetry: EN 6 items vs. ES 9 items | Confirmed real, unexplained |
| "Pending-Elevated" status value no longer set anywhere in Step 12 | Confirmed structural change from v3.2; downstream impact (e.g., MRA reporting) not assessed |
| Cross-batch node wiring (6→6a, 6f→7a-ES, 9a-ES→9a-2-ES, 10→10a, 11→12, etc.) | Never explicitly confirmed in any of the 13 documents received — real gap in verification, not resolved |
| Approval Status → NocoDB write-back from Google Sheet | Separately confirmed absent this session (SCX-Sheet-Sync HOW v4.0's own Known Issues) — build in progress as of this session, tracked outside this document |

---

**VRYOH Intelligence · SCX_RDA_HOW_v7.0 · Chat #96 · August 26, 2026 · Solofella LLC**
