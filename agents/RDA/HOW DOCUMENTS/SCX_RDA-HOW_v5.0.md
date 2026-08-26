# SCX_RDA_HOW_v5.0

**VRYOH INTELLIGENCE · SOLOFELLA LLC**
**HOW DOCUMENT — RDA**
**Response Drafting Agent**
Complete Step Decomposition · Node Logic · Code · Prompts · Field Contracts
v5.0 · July 8, 2026 · Chat #97

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0 | Chat #49 — Initial build |
| v2.0 | Chat #58 — Audit remediation + NocoDB rationalisation. 20 → 18 nodes. |
| v3.0 | Chat #74–75 — Five-call generation architecture. 18 → 23 nodes. |
| v3.1 | Chat #75 — Five-prompt synchronicity audit. 11-item audit checklist. Quality baseline 87%. |
| v3.2 | Chat #77 — SCX-Sheet-Sync OAuth credential failure documented. No RDA changes. |
| v4.0 | Chat #88–89 — Spanish deployment complete. Multi-client architecture. Client Config 22 fields. |
| v4.1 | Chat #96 — Spanish deployment reverted to English-only after language-mixing failures. Signal Enrichment layer added (6a–6e). Step 9e deterministic checker added. Governance vs. Style principle established. Step 7c missing `+` operator left unresolved. |
| **v5.0** | **Chat #97 — Full rebuild of this document. Spanish drafting rebuilt from scratch as a fully independent, natively-authored parallel branch — root-cause fix, not a patch. Internal follow-up brief relocated to end-of-chain, English-only regardless of guest language. Approval Status simplified 6→3 values. SEO keyword usage tracking added — RDA schema confirmed NOT frozen (corrects v4.1 Section 13). SEO instruction revised (descriptive vs. promotional keywords). HSI tier-routing gap diagnosed, deferred to V2, contained workaround applied. External GPT audit incorporated. Multiple self-caught bugs fixed. Dev/parallel test workflow agreed, not yet built.** |

---

## Summary Grid

| Property | Value |
|---|---|
| **Workflow Name** | SCX-RDA |
| **Model** | `claude-sonnet-4-6`, all calls, both languages. **Upgrade path to `claude-sonnet-5` identified — not yet tested/switched.** |
| **Claude Calls / Record** | 5 per branch (opening, body, SEO/gov, audit, internal brief) — internal brief is now a single shared call post-merge, not duplicated per language |
| **Languages** | **English + Spanish, both fully live**, independent parallel chains from generation through audit, merging only at Step 10 |
| **Total Nodes** | ~46 (v4.1's 28 + 6f router + 10 new Spanish-branch nodes + 10a/10b/10c shared internal-brief chain, minus 4 nodes reused-in-place: old 7g/7h EN+ES orphaned not deleted) |
| **NocoDB RDA Table** | **21 fields — NOT frozen** (corrects v4.1). New column: `SEO Keywords Used`. |
| **Approval Status** | **3 values: Pending, Approved, Modify, Not Accepted** (was 6: Pending/Pending-Elevated/Approved/Edited-Approved/Rejected/Published) |
| **Published Timestamp** | Confirmed permanently `null` at approval — "Approved" ≠ "published externally," which remains a separate, unbuilt mechanism |
| **Human Approval** | Mandatory, unchanged. 48-hour SLA, unchanged. |
| **Active Clients** | PAK-001, AJI-001–005 — AJI-001 confirmed processing real Spanish-language Google reviews live (record 5352/5353 onward) |
| **Doc Version** | v5.0 — July 8, 2026 |

---

## 1. AGENT PURPOSE (unchanged from v4.0/v4.1)

RDA translates BRA's response strategy into governed draft language, in the guest's own language, calibrated to tier and brand voice. Produces a public response and an internal signal brief. Terminal automated agent — no draft reaches any platform without explicit human approval.

**Governance principle, reaffirmed and extended this session:** mechanical/deterministic rules belong in code; judgment rules belong in the Claude prompt. Chat #97 concretely demonstrated this principle's cost when violated — every guest-name-insertion failure this session traced back to asking Claude to *judge* something (is the name present?) that should have been a deterministic check.

---

## 2. INPUT CONTRACT (unchanged from v4.1)

BRA webhook payload, Client Config (`m95cmabjfyb94ps`, 25 fields — confirmed directly against live schema this session, matches v4.1 exactly), EIP direct fetch, Emotion Dictionary + Pain Point Master targeted lookups. No changes to this contract this session.

---

## 3. SIGNAL ENRICHMENT LAYER — STEPS 6a–6e (unchanged from v4.1)

Structurally and functionally unchanged. Confirmed this session: both English and Spanish branches consume `signal_enrichment_brief` identically — this layer sits upstream of the language split (Step 6f) and requires no per-language logic.

---

## 4. LANGUAGE ROUTING — STEP 6f (NEW, Chat #97)

**Node type:** native n8n IF node (not Code — pure routing, no transformation).

**Condition:** `{{ $json.lang }}` equals `es` (string, strict).

**Verified field custody, full chain, direct code read:** `lang` extracted at Step 2 from the incoming webhook payload → explicitly listed in Step 6's key-by-key reconstruction (`lang: inp.lang`) → survives Steps 6a/6e via generic `Object.keys()` full-object copy (not a selective list) → reaches Step 6f intact. No field-loss risk found anywhere in this path.

**Distinguished from a separate, unrelated field:** `brand_voice.language` (Client Config's `Language` MultiSelect, e.g., `"En,Es"` for AJI-001) represents which languages a client *operates in generally* — not which language *this specific record* is in. Step 6f correctly uses the per-record `lang` field, not this one.

**Wiring:** True → `7a-ES`. False → existing `7a` (EN), unchanged.

---

## 5. SPANISH BRANCH — FULL REBUILD (Chat #97)

### 5.1 — Why rebuilt, not patched

Chat #96's failure mode (confirmed via that session's own root-cause analysis, corroborated again this session): one English-authored prompt with a late conditional `LANGUAGE:` block, causing live within-generation code-switching. The fix adopted: fully separate, natively-authored Spanish prompts, zero shared conditional logic, from generation through audit.

### 5.2 — Node-by-node (Spanish branch)

**`7a-ES` — Build Opening Prompt (ES).** Native Spanish system prompt. Tier is received (`confirmed_response_tier`), never self-classified — Chat #97 fix, prompted by external GPT audit finding that the original phrasing ("identify the first applicable signal") competed with the tier value already provided in the user prompt. T1-no-name case explicitly still leads with gratitude (audit-identified gap, fixed).

**`7b-ES` — Opening Claude Call.** Mechanical mirror of EN `7b` — same credential, same headers, only the body field differs.

**`7c-ES` — Build Body Prompt (ES).** Consolidated system prompt after multiple live-tested iterations this session (see Section 9 for the full iteration history and the key lesson it produced). Current locked content: opening-copy rule, tier rules (T1 extended for minor-criticism case, T2 loosened rigidity — contact invitation now conditional, not automatic), gratitude-for-feedback sentence, content-selection ordering, idiomatic-over-analytical instruction. Deliberately does **not** include: idiom-repetition ban, concrete-verb-preference rule, or closing-repetition ban — all three were added and then reverted this session after confirmed quality regression (Section 9).

**`7d-ES` — Body Claude Call.** Mechanical mirror.

**`7e-ES` — Build SEO/Governance Prompt (ES).** SEO instruction revised this session to distinguish descriptive from promotional keywords (Section 7). **Does not insert into the opening under any circumstance** — if the guest name is genuinely missing, sets `human_review_required = true` and a `pre_audit_flags` note, defers to human review. This is the final, corrected state after two earlier attempts both failed (Section 9.3).

**`7f-ES` — SEO Governance Claude Call.** Mechanical mirror.

**`8a-ES` — Build Final Draft Output (ES).** Parses the finished public draft. Computes `seo_keywords_used` (Section 7). No longer produces or requires `internal_followup_draft` — that generation moved to end-of-chain (Section 6).

**`9a-ES` / `9a-2-ES` — Fetch ALA Record / Fetch Recent RDA Drafts (ES).** Mechanical mirrors of EN `9a`/`9a-2`, with one confirmed, deliberate difference: both filter by `lang` in addition to `Client ID` (`~and(lang,eq,es)`), preventing a mixed-language client's Spanish opening-repetition check from being compared against an English draft. Same fix applied to EN `9a-2` (`~and(lang,eq,en)`).

**`9b-ES` — Build Audit Prompt (ES).** Native Spanish 9-item checklist (compliance items 1–6, repetition items 7–8, opening-repetition item 9). Repetition rule (item 7) narrowed this session to same/adjacent-sentence repetition only, after the original broad version risked flagging legitimate rhetorical echo. Brand-phrase list (item 8): one shared, client-agnostic Spanish list of 10 phrases (confirmed V1 simplification — see Section 11, open items).

**`9c-ES` — Audit Claude API Call.** Mechanical mirror. Confirmed language-agnostic by construction (pure HTTP POST on a field name).

**`9d-ES` — Parse Audit Output (ES).** Native Spanish closing arrays: T1 = `Un saludo afectuoso / ¡Saludos! / ¡Que tengas un día excelente!`; T2/T3 = `Un cordial saludo / Con mucho aprecio`. Selected deterministically via `bra_record_id % closings.length`.

**`9e-ES` — Deterministic Compliance Check (ES).** Minimal — T2 contact-invitation presence check only. Deliberately does not replicate EN `9e`'s "landed"-style word-ban logic, since no equivalent Spanish failure pattern has yet been observed in real data; inventing one preemptively would violate the project's observed-failure-first patching discipline.

### 5.3 — Merge point

`9e` (EN) and `9e-ES` both wire into the same `Step 10` input. No Merge node used or needed — only one branch ever fires per record (guaranteed by Step 6f's IF split), so `Step 10`'s single input safely receives whichever branch ran.

---

## 6. INTERNAL FOLLOW-UP BRIEF — RELOCATED (Chat #97)

**Previously:** generated mid-chain (old `7g`/`7h`, duplicated per language as `7g-ES`/`7h-ES`).

**Now:** generated once, shared, after full EN/ES merge — `Step 10a` (Build Internal Brief Prompt) → `Step 10b` (Internal Brief Claude Call) → `Step 10c` (Parse Internal Brief).

**Content: English-only, regardless of guest language** — confirmed by direct user instruction this session. Rationale: this is an internal ops note for the US-based management team, not guest-facing; Spanish fluency (the actual differentiator) belongs only in the public draft.

**Old `7g`/`7h` (EN) and `7g-ES`/`7h-ES`:** orphaned, disconnected, left on canvas per the session's safety-first build discipline (nothing deleted until the replacement was confirmed working end-to-end). Content reused verbatim in `10a`.

**A real gap found and fixed during this relocation:** `Step 10b`'s raw Claude response was initially wired directly into `Step 11`, which expects a parsed plain-text field, not a raw API response object. `Step 10c` was added specifically to close this gap — same parse-immediately-after-every-Claude-call pattern used everywhere else in this pipeline.

---

## 7. SEO KEYWORD TRACKING (NEW, Chat #97)

**Problem diagnosed:** the existing `SEO Keywords` NocoDB field stores the client's full *configured* keyword list — it never recorded which keywords a given draft actually used. MRA's weekly SEO report (Keywords placed / Top keyword / Coverage) was reading empty/zero because no mechanism anywhere computed actual usage.

**Fix:** new NocoDB column, `SEO Keywords Used` (LongText). Computed at `Step 8` (EN) and `Step 8a-ES` via deterministic string-matching — the finished draft body checked against the split, trimmed configured keyword list, case-insensitive. **Confirmed via live audit of 13 real AJI drafts this session: the mandatory brand-signature line ("Aji Ceviche Bar - Orlando") must be excluded from this match**, since every draft naturally contains the business name there regardless of any real SEO effort — an unexcluded match would show near-100% false coverage on every record.

**SEO instruction revised** (`7e`/`7e-ES`): distinguishes descriptive keywords (business/dish names — usable freely) from promotional keywords (superlative claims like "Best Peruvian Restaurant" — only usable when the review gives a natural opening, and prioritized over descriptive terms when that opening exists). **Diagnosed root cause, confirmed via live 13-draft audit:** prior to this fix, 7/13 real drafts had an in-body keyword match, but every single one was only the brand name or dish name — never a promotional keyword — because the original instruction's own escape hatch ("omit if it disrupts tone") gave Claude permission to treat every promotional-sounding phrase as tone-risk and skip it by default.

**Field custody:** `seo_keywords_used` threaded through every node from `Step 8`/`8a-ES` to `Step 14` (`9b→9d→9e→10→10a→10c→11→12→13→14`), same discipline as `lang`/`lang_resolved`.

**Status: code applied, NocoDB column confirmed created, field custody confirmed complete. Not yet tested against a real batch** (Section 11, open item).

---

## 8. APPROVAL STATUS — SIMPLIFIED (Chat #97)

**Old (v4.1):** Pending / Pending-Elevated / Approved / Edited-Approved / Rejected / Published.

**New:** **Pending / Approved / Modify / Not Accepted.** Confirmed already updated directly in NocoDB's SingleSelect field by the user before `Step 12`'s code was changed to match.

**`Step 12` simplified:** always writes `Pending` at creation (no more elevated/standard branching at the status-value level). The underlying elevation *reasoning* (`commercial_commitment_flag`, `governance_flag`, `human_review_required`, T3 tier) still computes identically and still populates `Elevation Reason` and still drives `Step 18`'s `[ELEVATED — APPROVAL REQUIRED]` email subject line — only the second dropdown value was removed, not the signal itself.

**Meaning, confirmed by direct user clarification:**
- **Approved** = user agrees with the draft 100% as written.
- **Modify** = user will rewrite parts of the draft themselves.
- **Not Accepted** = user disagrees with the draft entirely.
- **Approved and Modify are both treated as a positive signal** — "the logic used to generate this was correct." **Not Accepted is a negative signal** — "the logic was wrong for this case." This is intended as a quality/learning signal for future prompt refinement, not merely a status log — **left explicitly open for the next build stage** (Section 11).

**`Published Timestamp`:** confirmed to stay `null` permanently at Approval. Approval ≠ publishing externally; that remains a separate, not-yet-built mechanism entirely outside VRYOH's visibility. **Consequence, confirmed:** the dashboard's originally-specified "Avg approval time" metric (RDA Timestamp → Published Timestamp) can never be computed as designed; would need redefinition as time-to-decision if that metric still matters.

---

## 9. PROMPT ITERATION HISTORY — `7c-ES` (Chat #97 key lesson)

Documented in full because the pattern is a direct, empirically-confirmed instance of the rule-competition risk v4.1 first named theoretically.

1. **Baseline:** three additions (gratitude sentence, paragraph breaks, idiomatic-over-analytical instruction) — live-tested, confirmed by the client's native-Spanish-speaking owner as genuinely improved output, beating a raw zero-effort GPT comparison on some counts.
2. **Fourth-round addition:** idiom-repetition ban, concrete-verb-preference rule, closing-repetition ban — added in response to specific observed issues. **Confirmed regression:** output became vaguer, longer, less specific — the closing-repetition ban in particular caused Claude to avoid naming the specific complaint detail *anywhere*, not just in the closing, trading away the specificity that made version 1 good.
3. **Reverted to baseline (step 1).** Quality confirmed restored immediately.
4. **Targeted re-additions**, narrower and more precisely scoped than round 2: T1 extended for minor-criticism cases, T2 rigidity loosened (contact invitation now conditional), closing-repetition concern deliberately **not** re-added to generation — left as a live gap (Section 11), not solved by instruction.

**Locked lesson:** `7c-ES` (and by extension `7c`) should carry a minimal, high-value rule set for *generation*; redundancy/repetition checking belongs at the *audit* stage (`9b-ES`), checked against finished text — more reliable than instructing a drafting model to avoid a problem it cannot fully see while still writing.

---

## 10. GUEST-NAME INSERTION — THREE-NODE CONFLICT, RESOLVED (Chat #97)

**Diagnosed via external GPT-based prompt audit, confirmed independently against live test output:** `7a-ES`, `7e-ES`, and `9b-ES` each carried their own guest-name-insertion instruction, in three different formats. This caused a live, reproduced-twice duplicate-greeting bug (e.g., "Estimado/a JL Martines — Hola, JL Martines...").

**Fix sequence, both attempts documented:**
1. First fix: made `7e-ES`'s insertion *decision* deterministic (JS substring check instead of asking Claude to judge). **Insufficient** — the *inserted format* still didn't match `7a-ES`'s actual required patterns.
2. **Final fix:** `7e-ES` no longer inserts anything. If the name is genuinely missing, it sets `human_review_required = true` and a `pre_audit_flags` note, deferring entirely to human review rather than a second Claude rewrite.

**`9b-ES`'s item 11 (the same stale instruction) was removed entirely**, not fixed in place — since `7e-ES` no longer needs a downstream double-check of a decision it no longer makes.

---

## 11. OPEN ITEMS — NOT YET RESOLVED

1. **SEO instruction revision** — applied, untested against real output.
2. **Empty-review-text / rating-only submissions** — confirmed undocumented at ALA, EIP, and RDA (all three HOW documents checked directly this session). Unknown if it occurs in real platform data. No fix designed.
3. **Dev/parallel test workflow** — agreed approach (n8n native workflow duplication, `TEST-DEV` client ID excluded from MRA aggregates; Execute Workflow/sub-workflow explicitly rejected on 4GB-droplet memory and filesystem-v2 grounds, confirmed via independent model cross-check). **Not yet built.**
4. **`9b-ES`'s narrowed repetition rule** and **`9d-ES`'s closing arrays** — both changed this session, untested at volume.
5. **`9b-ES`'s shared, client-agnostic Spanish brand-phrase list** (10 phrases) — confirmed V1 simplification; V2 should add a per-client Spanish field to Client Config, mirroring English's existing per-client list.
6. **Closing-repeats-complaint-detail** — identified, a fix was tried and reverted in `7c-ES` (round 4, Section 9) due to regression; **not re-solved by any other means.** Live example: a draft closed with "brindarle una experiencia extraordinaria" (native speaker's preferred correction) vs. a later version repeating the ceviche detail directly — external GPT audit argued the repetition can sometimes strengthen a closing; **genuine, unresolved tension between two data points, not adjudicated this session.**
7. **HSI tier-routing gap** — confirmed via direct code read of `Step 16 - Preliminary Response Tier`: "Mixed Signal" unconditionally routes to T2, with no distinction between genuinely ambiguous and minor-recoverable-complaint patterns. **Explicitly deferred to V2** (locked, cross-pipeline rule affecting BRA's T1/T2 volume ratio for every client). Contained RDA-side workaround applied instead: T1/T2 rule loosening in `7c-ES`/`7c`.
8. **GPT-vs-Claude standalone comparison workflow** — scoped, not built; superseded in practice by this session's live testing and external audit.
9. **Model upgrade to `claude-sonnet-5`** — confirmed via current search to outperform `4-6` on every relevant benchmark at a lower price than Opus 4.8. Not yet tested; should go through item 3's dev workflow first.
10. **Feedback loop — reason/comment capture.** Confirmed design intent: Approved/Modify = positive signal, Not Accepted = negative signal, meant to inform future prompt refinement — **not** a simple status log. Whether the Google Sheet captures a reason alongside status (enabling *why*-level diagnosis) or status-value-only — **explicitly left open for the next stage**, per direct instruction.
11. **Per-client separate NocoDB tables** — proposed, evaluated, **rejected** this session (conflicts with the locked multi-client architecture; no measured performance problem presented to justify the fragmentation cost). Settled, not open.
12. **MRA/SCX-Sheet-Sync duplicate email reports** — traced to MRA/SCX-Sheet-Sync, confirmed not an RDA artifact. Outside RDA's scope.
13. **`Step 7c`'s (EN) missing `+` operator**, first diagnosed in v4.1 — **not touched or verified this session.** Carried forward as still open; all of this session's work targeted `7c-ES`, not the English node.

---

## 12. GOVERNANCE vs. STYLE (unchanged from v4.1)

Principle stands as locked in Chat #96. No new client-history calibration performed this session to extend it further.

---

## 13. NOCODB RDA TABLE — 21 FIELDS, **NOT FROZEN** (correction, Chat #97)

**v4.1 stated this table was frozen with no new columns permitted. This is corrected: the schema is not frozen** — confirmed directly by the user this session. One new column added: `SEO Keywords Used` (LongText, `cxh0iqavbr9ggyk`). Full current 21-field list confirmed directly against live NocoDB schema this session, including all original 20 fields (`RDA Run ID`, `BRA Record ID`, `ALA Record ID`, `RDA Timestamp`, `Confirmed Response Tier`, `Public Response Draft`, `Internal Follow-Up Draft`, `Approval Status`, `Commercial Commitment Flag`, `Elevation Reason`, `Flagged Terms`, `Client ID`, `lang`, `Published Timestamp`, `Approval Notes`, `Error Log`, `Reviewer Handle`, `SEO Keywords`, `Audit Passed`, `Audit Failed Items`) plus the new one.

---

## 14. ACTIVE CLIENT CONFIGURATION

Unchanged roster (PAK-001, AJI-001–005). **AJI-001 confirmed live-processing genuine Spanish-language Google reviews** as of this session's testing (records referencing JL Martines, Luis) — no longer a deactivated/theoretical capability as v4.1 described.

---

## 15. CREDENTIALS + CONFIGURATION (unchanged from v4.1)

`xc-token` (NocoDB), `Subtext-CX-Anthropic` (Claude, `x-api-key`). `Subtext-CX-OpenAI` credential exists and remains unused by RDA in production; would be needed only if the GPT comparison workflow (Section 11, item 8) is eventually built.

---

## 16. STANDING PROCESS RULES

Carried forward from v4.1, with one addition and one softening:

- **DIAGNOSE-ONLY RULE** — carried forward as the default posture, but this session operated primarily in **active build mode** at direct, explicit user instruction — diagnosis followed immediately by proposed/implemented fixes, confirmed step-by-step, one node at a time, always with exact field names/wiring/expected output stated before the user acted.
- **web_fetch DISTRUST** — reaffirmed. This session additionally established: always re-verify exact code/field names directly from source before reusing them in a new node, rather than relying on an earlier paraphrase in the same conversation — two real bugs this session (Step 8's node-name mismatch, 9b-ES's incomplete paste) trace directly to skipping this re-verification.
- **NocoDB schema is NOT frozen** (corrects v4.1 Section 13/16).
- Every response begins `Chat #[N] · [Date]`.
- **New this session:** before building any new node, confirm the exact current code of any node it will reference by name — do not assume a name matches what was originally specified without direct confirmation.

---

## 17. QUALITY BASELINE + OPEN ITEMS — Chat #97

### Confirmed working:
- Spanish branch fully independent, zero language-mixing observed across all live tests this session ✅
- Duplicate-greeting bug resolved (deterministic detection, then no-insertion/flag-instead) ✅
- Internal brief correctly English-only post-relocation ✅
- SEO keyword usage tracking — code and schema complete, custody verified end-to-end ✅
- Approval Status simplification — deployed, syntax-error-fixed, confirmed working ✅
- `Published Timestamp` semantics correctly resolved (stays null at approval) ✅
- Real, positive client feedback received — Aji Ceviche Orlando's owner confirmed satisfaction with 7/17–7/18 test drafts and will continue using VRYOH ✅

### Unresolved, carried into next session (full list, Section 11):
SEO instruction untested · empty-review-text handling undesigned · dev/parallel workflow unbuilt · `9b-ES`/`9d-ES` untested at volume · shared Spanish brand-phrase list is V1-only · closing-repeats-complaint tension unresolved · HSI tier-routing gap deferred to V2 · GPT comparison workflow unbuilt · Sonnet 5 upgrade untested · feedback-loop reason-capture design left open · EN `7c`'s missing `+` operator still unverified/unfixed.

---

**VRYOH Intelligence · SCX_RDA_HOW_v5.0 · Chat #97 · July 8, 2026 · Solofella LLC**
