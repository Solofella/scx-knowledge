# SCX_RDA_HOW_v8.0

**VRYOH INTELLIGENCE · SOLOFELLA LLC**
**HOW DOCUMENT: RDA, Response Drafting Agent**
Complete Step Decomposition · Node Logic · Code · Prompts · Field Contracts
**v8.0 · October 6, 2026 · Chat #385**
Supersedes v7.0 and v7.1. Complete rewrite, not an addendum.

> **Source rule for this document:** the live n8n canvas and code pasted by Miguel outrank this text. Where this document says "user-reported", I have not seen the evidence myself.

---

## Version History

| Version | Changes |
|---|---|
| v1.0 | Initial build, Chat #49, March 20 2026 |
| v2.0 | Chat #58: audit remediation and NocoDB rationalisation, 20 → 18 nodes |
| v3.0 | Chat #74-75: five-call generation architecture, 18 → 23 nodes |
| v3.1 | Chat #75: five-prompt audit, 11-item checklist, quality baseline 87% |
| v3.2 | Chat #77: SCX-Sheet-Sync OAuth failure documented, no RDA changes |
| v4.0-v4.1 | Chat #86-96: Spanish deployed, then reverted to English-only after language mixing. Phase 3 signal enrichment (Steps 6a-6e) and Step 9e added |
| v7.0 | Chat #96 (file header reads Aug 26 2026): Spanish rebuilt as a fully independent native chain (live Jul 15 2026 per project memory), shared downstream chain, internal brief moved after the audit |
| v7.1 | Sheet ↔ NocoDB approval feedback loop (addendum). Project memory dates it Sep 8 2026, Chat #100. The file header reads "Chat #96 · July 3 2026", a stale header |
| **v8.0** | **Oct 4-6 2026, Chats #326-385. RDA becomes contract-driven: it executes BRA's validated Response Contract. No apology, no fault, no verdicts. Voice comes 100% from Client Config. Guest contact email for negative replies. New writing rules. EN/ES audit parity at 12 items. Step 2 rewritten as a fail-closed gate. Steps 2, 6, 6a, 7a, 7c, 7e, 9b, 9e, 12, 18, 10a, 11 and the Spanish twins updated. No new nodes and no new NocoDB columns on the RDA table** |

**Corrections to earlier versions (flagged, not silent):**
- v7.0 said 51 nodes. A recount of the inventory gives **50**.
- v7.0 said the English audit had 6 items. The real code had **11**, now **12**. Spanish is 12 as well.
- v7.0 called RDA the 6th agent. Per the project overview it is the **7th of 8**.

---

## Summary Grid

| Property | Value |
|---|---|
| Workflow name | SCX-RDA |
| Product | VRYOH Intelligence |
| Model | claude-sonnet-4-6, all calls, both languages |
| Pipeline position | 7th of 8 agents, the last per-record automated agent (MRA reports after it) |
| Trigger | BRA webhook (`Step 19 - RDA Trigger` in BRA), immediate 200 response |
| Claude calls per record | **5**: opening, body, SEO/governance, audit, internal brief |
| Languages | English and Spanish, two fully independent native chains that merge at Step 10 |
| Controlling input | **BRA Response Contract** (`response_contract_v1`): what must, may and must never be said, and how much fault to express |
| Tone ownership | **RDA, 100%**, from the Client Config (BRA no longer sends `tone_style`) |
| Signal enrichment | Steps 6a-6e: EIP Cognitive Driver and Need State, plus Emotion Dictionary and Pain Point Master lookups (English dictionaries only) |
| Gate | Step 2 is **fail-closed**: an invalid contract stops the run |
| Audit | 12-item checklist, identical item list in English and Spanish |
| Deterministic layer | Step 9e / 9e-ES (code, not Claude), plus Step 11 commercial scan |
| Node count | **50** (12 shared upstream, 13 English, 13 Spanish, 12 shared downstream) |
| RDA NocoDB table | `mr1v67cszcklwns`, **20 fields, schema frozen, no new columns** |
| Human approval | Mandatory. No draft reaches any platform without a human decision |
| Approval SLA | 48 hours |
| Approval write-back | "RDA Sheet Approval Sync" workflow (Section 12) |
| Live client | AJI-001. PAK-001 is configured but not live. AJI-002 to 005 are inactive |
| Status | Contract-aware build complete and tested end-to-end in duplicate workflows (user-reported). Promotion to live in progress as of Oct 6 2026. **Update this line when confirmed** |

---

## 1. AGENT PURPOSE

### What RDA is
RDA turns BRA's validated strategy into the guest-facing response and an internal signal brief, in English or Spanish, calibrated to the response tier and the client's own brand voice. **RDA decides how to say it. It never decides what to concede.**

### Why RDA exists, and the rationale for v8.0
A real incident drove this version. A Spanish 1-star review described a guest denied bar seating and told to wait in his car. RDA's draft admitted fault outright. A staff account then disputed the whole story: the policy is fire-code driven and posted on site, and the car-wait was a heat accommodation. RDA had treated one side's account as fact, because nothing upstream told it to be careful. Two independent external LLM audits and the owner's own reasoning all concluded that deciding how much fault to admit is a **strategy decision that belongs in BRA**, not in the writer. So BRA now builds and validates a Response Contract, and RDA executes it.

### The standing structural rule
**VRYOH can never apologize, admit fault, or assume a cause, at any star rating.** VRYOH has no operational visibility into a client's business. It cannot verify any guest's account. This is a positional limit, not a capability gap, so it does not improve with better models.

### Expected outcome (the owner's definition of quality)
A response is high quality only if it satisfies all three at once:
1. **Naturalness** in each language, judged against Claude's own writing ability.
2. **Client Config 100% recognizable**: a reader could tell which client wrote it.
3. **Logical coherence with upstream inputs**: nothing invented, nothing relevant omitted.

### RDA produces
- Public Response Draft (EN or ES)
- Internal Follow-Up Draft (always English, one call per record)
- Commercial Commitment Flag and Flagged Terms (code scan, Step 11)
- Approval Status (`Pending`) and Elevation Reason
- Audit Passed and Audit Failed Items (Claude audit plus deterministic notes)
- SEO Keywords Used

### RDA does NOT
- Decide strategy, fault admission or claim verification (BRA)
- Re-classify emotion, pain or tier (upstream)
- Publish anything
- Prescribe operational actions
- Apologize, admit fault, assume causes, judge who was right, explain policies, or promise changes
- Write new NocoDB columns (schema frozen)

### Relationships to other agents
| Agent | Relationship to RDA |
|---|---|
| ALA | Ingests reviews (manual CSV in Phase 1) and detects language. RDA reads the ALA record at Step 9a for the original review text (`Raw Tex`) used by the audit |
| EIP | Classifies emotion and pain. RDA re-fetches `Cognitive Driver` and `Need State` directly from EIP's table, because they do not survive the relay |
| ESS, HSI | Stabilize and tier the signal. HSI's tier and signal fields reach RDA through BRA |
| SIA | Scheduled aggregation (daily, weekly, monthly). RDA does not consume SIA output |
| **BRA (direct upstream)** | Sends the webhook payload: the existing 28 fields plus the Response Contract, `raw_text`, `star_rating`. BRA owns tier handling, governance flag, commercial risk and the contract |
| MRA (downstream) | Reads the RDA table: approval rate, tier breakdown, SEO keywords used. Its `Published Timestamp` metrics stay permanently zero because RDA never sets it |
| SCX-Sheet-Sync (downstream) | Copies Pending RDA rows into each client's Google Sheet (daily, 5am UTC) |
| RDA Sheet Approval Sync | Writes human decisions back into the RDA record (Section 12) |

---

## 2. INPUT CONTRACT: FOUR SOURCES

### Source 1: BRA webhook payload (validated at Step 2)
- The original BRA payload fields (tier, axes, governance flag, commercial risk, `reviewer_handle`, `client_id`, `lang`, record IDs, keywords, emotion hypothesis, etc.)
- **New in v8.0:** the Response Contract (flat or nested under `response_contract`), `raw_text`, `star_rating`
- **Removed:** `tone_style` (RDA owns tone now)

**Step 2 outputs:** the existing fields, plus:

| Field | Meaning |
|---|---|
| `response_contract` | The validated contract object (Section 11) |
| `hold_for_review` | True when risk is high_liability, posture is escalate_to_human, closing is no_public_close_escalate, or BRA set `halt_reason` |
| `raw_text` | The guest's original review |
| `star_rating` | Number or null |
| `needs_contact` | True when the reply must carry the private contact invitation |

**Critical design choice:** these values are not copied through every node. Each node that needs one reads it directly from Step 2 with `$('Step 2 - Payload Validation').first().json`. This removed about 20 pass-through edits and the dropped-field bugs that came with them. **Consequence: Step 2 must be ON for any run, and its node name must never change.**

### Source 2: Client Config (NocoDB `m95cmabjfyb94ps`), parsed at Steps 5, 6 and 6a
Step 6 builds the `brand_voice` object:

| Internal key | Client Config field |
|---|---|
| Client_ID, Client Name | Client ID, Client Name |
| tone_descriptors | Brand Personality |
| formality_level | Formality Level |
| person_preference | Person Preference |
| brand_phrases_include / avoid | Brand Phrases To Include / Avoid |
| language | Language |
| seo_keywords | SEO Keywords |
| approval_contact_email | Approval Contact Email (internal approvers) |
| **escalation_contact** | **Escalation Contact Email (new column, `cw8gh10cqxuwd72`, guest-facing)** |
| guest_feeling | Guests feeling after reading a response |
| commitment_restrictions | Commitments should NEVER be made |
| differentiator | Differentiator |
| the_register / the_core_driver / the_regional_accent | THE REGISTER / THE CORE DRIVER / THE REGIONAL ACCENT |

**Removed from the writer's view:** `response_negative_experience` and `recovery_protocol`. Their options (for example "Apologize directly and offer to make it right") contradict the no-fault rule. The columns still exist in NocoDB. The onboarding form still offers "Apologize directly", which promises something VRYOH will not deliver, so it needs rewording.

**Step 6a** turns 10 of these fields into `brand_voice_brief`: REGISTER, CORE DRIVER, REGIONAL ACCENT, PERSONALITY, FORMALITY, PERSON PREFERENCE, GUEST SHOULD FEEL, DIFFERENTIATOR, BRAND PHRASES TO INCLUDE, BRAND PHRASES TO AVOID. The brief is English text even for Spanish records. The Spanish prompts say to use it for tone only and write in Spanish.

### Source 3: Signal enrichment (Steps 6b-6e)
- **6b:** GET EIP table by `eip_record_id`, pulling Cognitive Driver and Need State.
- **6c:** GET Emotion Dictionary row by the live EIP `Enriched Emotion Tag`, pulling Common Expressions. The earlier stale-reference bug is fixed.
- **6d:** GET Pain Point Master row by the live `Enriched Pain Point`, pulling Operational Signal, Emotional Signal and Sample User Expression.
- **6e:** builds `signal_enrichment_brief`.

**Known gap:** 6c and 6d query the English dictionaries only, so Spanish records get blank dictionary fields.

### Source 4: Reads at audit time
- Step 9a: ALA record (original review).
- Step 9a-2 and 9a-2-ES: recent RDA drafts per client and language, used for the opening-repetition check.

---

## 3. TIER-SPECIFIC DRAFT INSTRUCTIONS

### Fixed openers (kept deliberately; they match the PAK owner's real voice)

| Tier | English opener | Spanish opener |
|---|---|---|
| T1 | "Thank you very much, [Name]," or "We appreciate you, [Name]," | "Muchas gracias, [Nombre]," or "Le agradecemos mucho, [Nombre]," |
| T2 | "Hi, [Name]," or "Hello, [Name]," | "Hola, [Nombre]," or "Estimado/a [Nombre]," |
| T3 | "Hello, [Name]," | "Estimado/a [Nombre]," |

If no guest name exists, the opening begins with the strongest anchor detail.

**Observed drift:** one test draft opened "Hi Carmencita," with no comma after "Hi". It is cosmetic. Consider allowing both forms.

### Length by tier (total sentences, including the opening)

| Tier | English | Spanish |
|---|---|---|
| T1 | 2-3 | 2-3, up to 4 when a concrete minor criticism is named |
| T2 | 2-3, or 4 when three or more must-address items need room | same |
| T3 | 3-4 | 3-4 |

In Spanish negative replies, a paragraph break goes before the contact invitation.

### The contract drives content
- Every **must-address** item is covered, primary items first and with the most room. Mixed reviews cover both the complaint and the praise.
- **May-address** items are optional.
- **Must-not-introduce** items never appear.
- **Posture** sets the framing and **closing objective** sets the ending (Section 11).

### No-fault rules (all tiers, both languages, in 7a, 7c, 7e, 9b)
1. No apology or regret words (EN: sorry, apologize, apologies, regret. ES: lo sentimos, lamentamos, sentimos mucho, disculpe, disculpas, perdón).
2. No admission of fault (our fault, our mistake, we failed, fell short, let you down, should have. ES: nuestra culpa, nuestro error, fallamos, no estuvimos a la altura, les quedamos mal, debimos).
3. No assumed cause.
4. **No verdicts.** Never say the guest's expectation, claim or complaint is reasonable, fair, justified or correct. Never say the business, a policy or staff were wrong. Never explain why a policy exists. Never answer a comparison to another business.
5. What the guest reports is the guest's account, not a fact.
6. No promises of changes, reviews, training or any action.
7. No physical descriptions of staff. Staff names are allowed.
8. **Test for every sentence:** does it say how the guest felt, or who was right? Feelings are allowed. Who was right is not.

**Approved reference (Miguel's "Dani" water-policy draft), pattern only, never copied:** name the topic, show care about how it felt, take no position, invite private follow-up with the contact email.

### 1-3 star pattern (locked)
One public reply that (a) briefly and specifically acknowledges the actual complaint, then (b) invites the guest to continue privately. There is no separate private-message layer.

### Contact rule
- `needs_contact` is true when `star_rating <= 3`, tier is T3, or the closing objective is `offline_contact_invitation` or `no_public_close_escalate`.
- When true and `escalation_contact` exists, the draft contains that exact address **once**, in plain text.
- When true and no address is configured, the draft uses a generic "reach out to us directly" invitation, and the record is flagged.
- When false, no email may appear.

### Voice rules (the Client Config is the primary reference)
Each field maps to a behavior in 7a, 7c and the Spanish twins:
- **Register:** vocabulary and sentence style.
- **Core Driver:** what to champion first.
- **Regional Accent:** idiom and phrasing.
- **Personality:** energy and warmth. **When a guest reports a problem (T2, T3, escalate_to_human), keep the brand's own vocabulary and warmth but remove jokes, wordplay, exclamation marks and emojis.** A playful brand sounds calm, not comic. A serious brand stays serious. The earlier "stay serious whatever the personality says" rule was removed as a contradiction of Client Config ownership.
- **Formality** (1 casual to 5 polished): contractions and structure.
- **Person Preference:** the team is named exactly as written.
- **Guest Should Feel:** expressed through wording, never named.
- **Differentiator:** shown through one specific detail, never as a label, never invented.
- **Brand phrases:** at most one in the whole response (the opening counts). Never a phrase on the avoid list.
- **Swap test:** if another restaurant's name could replace this one without the text sounding wrong, rewrite it.

### Writing-style rules (new in v8.0)
- **No dashes between words** (no em dash, en dash or double hyphen). A comma or a full stop replaces them. Hyphens inside words stay.
- **Mixed sentence lengths.** With three or more sentences, at least one under 10 words.
- **No "it isn't X, it's Y"** construction in any form (Spanish: "no es solo X, es Y").
- No generic filler (for example "honest feedback helps us understand", "valuable feedback", "made an impression", "resonated", "full picture"). Say something specific.
- Use only what the guest wrote. Do not add circumstances the guest did not mention.
- Never imply that things will improve.
- Native Spanish: no anglicisms, no translated constructions, consistent tú or usted. Use "opinión" or "comentario", not "reseña", as the default word.

### Commercial and time-sensitive details
Prices, discounts, menu tiers, prix fixe, Restaurant Week, Happy Hour, seasonal or holiday menus, limited-time items, promotions, marketing events and future menu options are never used as content, never implied to be available, and never used as a closing anchor.

### Closing for positive replies
Strongest eligible detail in this order: guest occasion, guest relationship or loyalty, overall experience, favorite non-promotional food or drink, the guest's own non-commercial wording. Never a generic closing. Never repeat a noun used in the opening.

### SEO
Descriptive terms (business name, dish names) are free to use. Promotional claims only where the review gives a natural opening. **SEO is skipped for 1-3 star, T3 and held records.** It is never placed in the opening line.

---

## 4. CLAUDE CALL ARCHITECTURE: 5 CALLS PER RECORD

| # | Call | Node (EN / ES) | Temp | Max tokens |
|---|---|---|---|---|
| 1 | Opening | 7b / 7b-ES | 0 | 150 |
| 2 | Body | 7d / 7d-ES | 0.3 | 500 |
| 3 | SEO + governance | 7f / 7f-ES | 0 | 600 |
| 4 | Audit | 9c / 9c-ES | 0 | 1500 |
| 5 | Internal brief (shared, English only) | 10b | 0 | 400 |

**Prompt builders feed each call:** 7a/7a-ES → call 1, 7c/7c-ES → call 2, 7e/7e-ES → call 3, 9b/9b-ES → call 4, 10a → call 5.

### Call 1: Opening (7a / 7a-ES)
One sentence. Inputs: brand voice brief (placed first in the user prompt), tier, posture, guest name, signal type, primary contract points with quoted spans, emotion hypothesis, keywords, enriched emotion, signal enrichment, behavioral context. The prompt carries the voice rules, the no-fault rules, the style rules and the fixed openers. Spanish carries the "write in Spanish, brief is English guidance only" guard.

### Call 2: Body (7c / 7c-ES)
Copies the opening exactly, then completes the reply. User prompt order: opening sentence, brand voice brief, tier, star rating, language, then the contract block (posture, claim type, verification, closing objective, must address, may address, must not introduce, contact instruction), then the guest's original review (reference only), then signal detail.

**Dropped from the writer's view in v8.0:** BRA's strategy block (solution type, axes, signal-based consideration) and behavioral context. The contract replaces them, so the writer no longer holds a second set of judgments.

### Call 3: SEO + governance (7e / 7e-ES)
- Cleans the draft with the smallest possible edit.
- Keeps **only** the allowed contact address and removes every other email, URL and placeholder.
- Removes apology, fault, verdict and promise language.
- Swaps dashes for commas or full stops.
- Applies SEO only where allowed.
- **Guest name:** English asks Claude to insert "Hello, [Name]," if missing. **Spanish never lets Claude touch the opening for the name.** A deterministic code check flags the record instead (`pre_audit_flags`, human review). This divergence is real and unresolved.

### Call 4: Audit (9b / 9c, 9b-ES / 9c-ES), 12 items, identical in both languages

| # | EN name | ES name |
|---|---|---|
| 1 | NAMED STAFF | PERSONAL MENCIONADO |
| 2 | REVIEW DEPTH | PROFUNDIDAD DE OPINIÓN |
| 3 | MUST ADDRESS | ATENDER OBLIGATORIAMENTE |
| 4 | MUST NOT INTRODUCE | NO INTRODUCIR |
| 5 | NO FAULT | SIN CULPA |
| 6 | CLOSING | CIERRE |
| 7 | CONTACT | CONTACTO |
| 8 | PROHIBITED | PROHIBIDO |
| 9 | STYLE | ESTILO |
| 10 | BRAND PHRASES | FRASES DE MARCA |
| 11 | OPENING REPETITION | REPETICIÓN DE APERTURA |
| 12 | GUEST NAME | NOMBRE DEL HUÉSPED |

- **CLOSING** accepts the private contact invitation as the ending for `offline_contact_invitation` and `no_public_close_escalate`, and a brief neutral ending for `neutral_close`.
- **NAMED STAFF** only requires a staff name when a staff member is in must-address. It does not force a name into a complaint reply.
- **Brand phrases** read each client's own include and avoid lists. The old hardcoded PAK phrases are gone.
- **Spanish-only sub-checks** sit inside ESTILO: repetition within the same or adjacent sentences (distant echoes are allowed), anglicisms, and tú/usted consistency. The Spanish audit fails GUEST NAME without inserting it.
- **Output:** JSON `{audit_passed, failed_items, final_public_response_draft}`. The audit corrects only failing elements.

### Call 5: Internal brief (10a, 10b, 10c)
Shared and always English. It states the claim type and **verification state** (for example "guest report only, unverified"). It describes what the guest reported, never what happened, and never implies fault. It contains no recommendations or directives.

---

## 5. DETERMINISTIC COMPLIANCE LAYER

Pure JavaScript, no Claude. Runs after the audit parse (9d / 9d-ES), before Step 10.

### Step 9e (English)
- **Auto-fixes:** "landed" and "landing" (word swap), dashes swapped for commas.
- **Flag only (sets human review, never rewrites):**
  - apology or regret words
  - fault admissions
  - verdict phrases ("reasonable / justified expectation", "you're right")
  - promises of change
  - any email other than the configured one
  - contact address appearing other than exactly once
  - missing private invitation when `needs_contact`
  - escalation contact not configured
  - the "we carry" near-miss pattern
  - high-liability hold

### Step 9e-ES
Same checks with Spanish lists and **accent-insensitive** matching for the contact invitation (`contáctenos`, `comuníquese`, `escríbanos` and more). It also flags **possible English leakage** (the, and, thank you, our team, your visit). It has no "landed" equivalent.

### Design principle
Word and punctuation swaps are auto-corrected. Anything that needs new sentence-level text is flagged and never templated. Auto-inserting one fixed sentence everywhere would bring back the sameness problem that removing the fixed sentence templates solved.

### Step 9d closing signature
Tier-aware closing rotation chosen by BRA record id, plus the client name. **Fix applied:**
```javascript
const finalDraft = publicDraft.trim() + '\n\n' + closing + (/[!?.]$/.test(closing) ? '' : ',') + '\n' + brandSignature;
```
It removes the stray comma that produced "Cheers!," and "¡Que tengas un día excelente!,". Apply the same line in 9d-ES.

### Step 11: Commercial Commitment Scan
Scans both drafts. Term lists in English and Spanish (reembolso, descuento, cortesía, compensación, gratis, sin costo and others), accent-insensitive. Writes `commercial_commitment_flag` and `flagged_terms`.

### Step 12: Approval Status Assignment
Always `Pending`. Elevation Reason accumulates:
- `HIGH-LIABILITY: NON-PUBLISHABLE until elevated review (<hold_reason>)` when held
- Commercial commitment detected
- Governance Flag = Flag
- Human Review Required
- T3 record

### Step 18: Approval Notification
Builds the email subject and body. **It sends nothing in the copy, because no send node follows it.**
- Subject prefix: `[HOLD: NON-PUBLISHABLE]` when held, `[ELEVATED: APPROVAL REQUIRED]` when there are elevation reasons, otherwise `[APPROVAL REQUIRED]`.
- Body shows the contract summary (posture, claim type, verification, closing) and instructs the approver to set Status in the client's Google Sheet.
- The old subject never fired, because it tested for a status ("Pending-Elevated") that Step 12 never set.

---

## 6. NODE MAP (50 nodes)

```
── SHARED UPSTREAM (12) ──
[01] Webhook (RDA trigger)
[02] Payload Validation  ★ contract gate, fail-closed, builds response_contract / hold_for_review / needs_contact
[03] Idempotency check (RDA table by BRA Record ID)
[04] Already-processed gate
[05] Client Config GET
[06] Build Brand Voice Object  ★ adds escalation_contact, drops negative-experience fields
[06a] Brand Voice Consolidation  ★ brief without negative-experience lines
[06b] Fetch EIP Signal Data
[06c] Fetch Emotion Dictionary Row (English dictionary)
[06d] Fetch Pain Point Master Row
[06e] Build Signal Enrichment Brief
[06f] Language Router (lang = es?)

── ENGLISH BRANCH (13) ──
[07a] Opening prompt ★   [07b] Opening Claude call
[07c] Body prompt ★      [07d] Body Claude call
[07e] SEO/Governance prompt ★   [07f] SEO/Governance Claude call
[08]  Final output
[09a] Fetch ALA record   [09a-2] Fetch recent drafts (en)
[09b] Audit prompt ★ (12 items)   [09c] Audit Claude call
[09d] Parse audit + closing signature ★   [09e] Deterministic checks ★

── SPANISH BRANCH (13) ──
[07a-ES] ★  [07b-ES]  [07c-ES] ★  [07d-ES]  [07e-ES] ★  [07f-ES]
[08a-ES] Final draft output
[09a-ES] Fetch ALA record   [09a-2-ES] Fetch recent drafts (es)
[09b-ES] ★ (12 items)   [09c-ES]   [09d-ES] ★   [09e-ES] ★

── SHARED DOWNSTREAM (12) ──
[10] Output parsing   [10a] Internal brief prompt ★   [10b] Claude call   [10c] Parse brief
[11] Commercial scan ★   [12] Approval status ★   [13] Run ID + timestamp
[14] NocoDB POST body   [15] NocoDB write   [16] Capture record ID
[17] PATCH BRA status → Complete   [18] Approval notification ★
```
★ = changed in v8.0.

**Cross-file wiring not shown in the exports and confirmed by the owner:** 6→6a, 6a→6b, 6b→6c, 6d→6e, 6f Spanish exit→7a-ES, 7b→7c, 7f→8, 9b→9c, 7d-ES→7e-ES, 9a-ES→9a-2-ES, 9e-ES→10, 10→10a, 11→12, 14→15.

**Not in this workflow:** Steps 6g and 6h (Section 12).

---

## 7. APPROVAL STATUS LIFECYCLE

| Status | Set by | Meaning |
|---|---|---|
| **Pending** | Step 12, always | Awaiting a human decision. Elevation Reason says whether extra attention is needed |
| **Approved** | Human, via Sheet sync | Used exactly as generated |
| **Not Accepted** | Human, via Sheet sync | Rejected entirely. Notes expected |
| **Modified** | Human, via Sheet sync | Partly agreed, will be edited. Notes optional by design |

- "Published" was removed, so `Published Timestamp` is permanently null.
- "Pending-Elevated" is not used. The hold is expressed in Elevation Reason and in the email subject.
- Held (high-liability) records are **drafted and tagged non-publishable**. They are never skipped.

---

## 8. NOCODB RDA TABLE: 20 FIELDS, SCHEMA FROZEN

Table `mr1v67cszcklwns`, base `pq249fix22t3ofv`. Fields written at creation: RDA Run ID, BRA Record ID, ALA Record ID, RDA Timestamp, Confirmed Response Tier, Public Response Draft, Reviewer Handle, Internal Follow-Up Draft, Approval Status, Commercial Commitment Flag, Elevation Reason, Flagged Terms, Client ID, lang, SEO Keywords, SEO Keywords Used, Audit Passed, Audit Failed Items, Published Timestamp (null), Approval Notes (null at creation), Error Log (null).

**Rule locked:** no new columns on this table. The contract, hold state and contact data travel in the pipeline only. The only new column of this version is **Escalation Contact Email** in the **Client Config** table (`cw8gh10cqxuwd72`), which is a different table.

---

## 9. CREDENTIALS + CONFIGURATION

| Item | Value |
|---|---|
| Anthropic credential | `x-api-key`, id `uMahlx4nOC5YJh0Z`, on all 5 Claude calls |
| NocoDB credential | `xc-token`, id `DT9tnRgqYpPc3rXo` |
| NocoDB internal URL | `http://nocodb:8080` |
| Tables | RDA `mr1v67cszcklwns` · Client Config `m95cmabjfyb94ps` · ALA `m57efwbtrvwohhr` · BRA `mwqejw7swhd2cf4` · EIP `mhicpnrahaesxmy` · Emotion Dictionary EN `mrrscb955j1d2i7` · Pain Point Master `meavqh37mdqgl4d` |
| Model | claude-sonnet-4-6, header `anthropic-version: 2023-06-01` |
| Retry | `retryOnFail`, 5000 ms between tries |
| RDA webhook | Old workflow: `.../webhook/scx-rda` (deactivated). **Promoted copy: `https://n8n.solofella.com/webhook/1bc95aa6-516c-4f2e-b7c2-c3b6dc660402`.** BRA's Step 19 already calls this address, and the owner chose to leave the link unchanged |
| BRA webhook | HSI calls `.../webhook/scx-bra`. The BRA copy takes that path after the old BRA is switched off |
| Cost | About $0.04 per fully processed review (single-day sample of 5 records) |

**n8n rule learned:** a webhook path can be active in only one workflow. Switch the old workflow off and save before the copy takes its path. A copied workflow gets a random path. Set it back deliberately, or point the caller at the new one.

---

## 10. QUALITY BASELINE + OPEN ITEMS

### Evidence (test run, user-reported, drafts pasted)
- **Spanish T1, 5 stars (Andrecito):** required opener, both staff named, freshness acknowledged, consistent tú, no dashes, no apology, a short closing sentence. Defect found and fixed: "!," punctuation in the sign-off.
- **English T2, 5 stars with a small-space remark (Carmencita):** safe, but weak on quality:
  - invented context ("when things get lively")
  - an implied promise ("leave feeling just as good")
  - generic filler
  - no sentence under 10 words
  - not recognizably the brand
  
  Prompt lines were added to address these (Section 3).
- A high-liability record (BRA 1561) reached Step 2 with `halt_reason` set. This revealed that BRA uses `halt_reason` as a label, not a stop. Step 2 was corrected.

### Confirmed working
Contract gate and `needs_contact`. Hold handling. Contact insertion. Spanish and English parity of the audit. Dash rule. Elevated email subjects. Signal enrichment with the 6c/6d fix. The approval write-back loop.

### Open items

| Item | Status |
|---|---|
| Steps 6g and 6h (rejected/modified patterns feedback) are **not in the promoted copy**. The sync writes the decisions, but **nothing in RDA reads them yet** | Rebuild after go-live. The brief must be style feedback only and can never override the contract or the no-fault rules |
| Brand distinctiveness is weak (generic wording, no use of the restaurant's own character) | Best lever: each client's last 3 approved drafts as style-only examples |
| Spanish records get blank dictionary enrichment (6c/6d use English tables) | Open |
| Fixed T1 openers versus brand-voice freedom | Kept deliberately. Revisit with the approved-examples work |
| Opener drift ("Hi Carmencita," without the comma) | Cosmetic. Decide whether to allow both forms |
| The sentence-length rule is not enforced by code, and the audit missed it once | Open. Consider a note-only check in 9e |
| English guest-name handling (Claude inserts) versus Spanish (code flags) | Real divergence, undecided |
| Step 2 is fail-closed, so a bad contract produces no draft and no row | **Monitor the failed-executions list daily after go-live** |
| BRA's Step 19 has no retry on failure | Separate known item |
| `tone_style` leftovers in Steps 8, 9d, 10, 13, 14, 16 | Harmless (empty). Clean up when convenient |
| The Client Config "Response for a negative experience" column | Redundant. Delete only after go-live, export the values first, and check that no other agent reads it |
| The onboarding form offers "Apologize directly" | Reword. It promises something VRYOH will not do |
| Hold tag visibility for approvers working only in the Sheet | Confirm whether Elevation Reason reaches the Sheet |
| PAK-001 has no Escalation Contact Email | Fill it before PAK goes live |
| 9d-ES, 10a to 10c and the Claude call nodes were not read as stored in the final audit | Audit them |

---

## 11. RESPONSE CONTRACT INTEGRATION (v8.0)

### What BRA sends and Step 2 enforces
```
schema_version                 response_contract_v1 (required)
must_reflect[]                 concept, required_action, priority (primary|secondary),
                               source_type, source_span / upstream_origin / verification_source,
                               relevance_confidence (high|medium|low)    (non-empty, required)
may_reflect[]                  optional items
must_not_introduce[]           never-introduce items
commercial_restriction_types[] refund | discount | price_promise | compensation ...
primary_claim_type             experiential | policy_procedural | factual_allegation
verification_state             not_applicable | guest_report_only | corroborated | disputed
highest_risk_class             normal | elevated | high_liability
strictest_epistemic_posture    acknowledge_without_fault_admission | neutral_acknowledgment | escalate_to_human
closing_objective              warm_return_invitation | appreciative_close | offline_contact_invitation
                               | neutral_close | no_public_close_escalate
halt_reason                    label only (never a stop signal). Becomes hold_reason
```
`direct_apology` is **retired**, with no exception for any rating or claim type. Internal-only fields (builder and ruleset versions, decision trace) live in BRA's Error Log JSON and are never sent to RDA.

### Step 2: final code (the gate)
```javascript
const input = $input.first().json;
const p = typeof input.body === 'string' ? JSON.parse(input.body) : input.body;

if (!p.bra_record_id) throw new Error('Missing bra_record_id');
const braId = parseInt(p.bra_record_id);
if (isNaN(braId) || braId <= 0) throw new Error('bra_record_id must be positive integer. Got: ' + p.bra_record_id);
if (p.governance_flag === 'Halt') throw new Error('Halt record reached RDA --- BRA suppression failure. Record: ' + braId);

const src = (p.response_contract && typeof p.response_contract === 'object') ? p.response_contract : p;

// asArray / optArray helpers, enum lists (POSTURES, RISKS, CLAIMS, VERIFY, CLOSINGS)
// Validation: schema_version, must_reflect non-empty with concept + priority,
// all five enums. Any problem -> throw 'Invalid Response Contract for BRA record N: ...'

const response_contract = { /* schema_version, arrays, enums, hold_reason: src.halt_reason ? String(src.halt_reason) : null */ };

const hold_for_review = src.highest_risk_class === 'high_liability'
  || src.strictest_epistemic_posture === 'escalate_to_human'
  || src.closing_objective === 'no_public_close_escalate'
  || !!src.halt_reason;

const needs_contact = (star_rating !== null && star_rating <= 3)
  || p.confirmed_response_tier === 'T3'
  || src.closing_objective === 'offline_contact_invitation'
  || src.closing_objective === 'no_public_close_escalate';

// returns the existing fields plus: response_contract, hold_for_review, raw_text, star_rating, needs_contact
```
(The deployed node holds the complete helper and validation code. Keep it as the single source of truth.)

### Who reads what
| Value | Read by |
|---|---|
| `response_contract` | 7a, 7c, 9b, 10a, 18 and Spanish twins |
| `raw_text` | 7c, 7c-ES |
| `star_rating` | 7c, 7c-ES, 7e, 7e-ES |
| `needs_contact` | 7c, 7e, 9b, 9e and Spanish twins |
| `hold_for_review` | 7e, 7e-ES, 9e, 9e-ES, 12, 18 |
| `brand_voice.escalation_contact` | 7c, 7e, 9b, 9e and Spanish twins |

### Process rules that came out of this build
- One node at a time, with a Name, Wires, Output and field map confirmed before code.
- **Stacked patches caused real bugs** (a duplicate declaration, a cut-off return). For anything touched more than twice, resend the whole node.
- After pasting a large node, jump to the last line and confirm it ends with `} }];`.

---

## 12. SHEET ↔ NOCODB APPROVAL SYNC (formerly the v7.1 addendum)

### Why it exists
RDA wrote `Pending` and nothing ever changed it. Human decisions made in the Google Sheet never reached NocoDB. The SCX-Sheet-Sync HOW v4.0 listed this exact gap as "Not built". It is now built, tested and deployed on all 6 client Sheets.

### Purpose and status meaning
The loop is **quality control only**: no regeneration, no alerting. A change to Approved confirms quality. A change to Not Accepted or Modified is meant to inform the next generation cycle for that client (see the open item on Steps 6g/6h). A decision timestamp field and an approval-log table were proposed and **rejected by the owner**: quality matters, time is not an essential indicator.

### Sheet layout (all 6 Sheets)
| Col | Name |
|---|---|
| 1-7 | SCX Date, Review Date, Platform, Star Rating, Reviewer Handle, Review Text, Proposed Response |
| **8** | **Status** |
| **9** | **Approval Notes** (renamed from "Edited Response") |
| 10 | ALA Record ID |
| **11** | **RDA Record ID** |
| **12** | **Sync Status** (was a static `pending_sync`, now set to `synced` after a successful call) |

### Apps Script (one copy per Sheet, installable trigger required)
```javascript
function onEdit(e) {
  const range = e.range;
  const sheet = range.getSheet();
  const col = range.getColumn();
  const row = range.getRow();
  const STATUS_COL = 8, APPROVAL_NOTES_COL = 9, RDA_RECORD_ID_COL = 11, SYNC_STATUS_COL = 12;
  if (col !== STATUS_COL) return;
  if (row === 1) return;
  const rdaRecordId = sheet.getRange(row, RDA_RECORD_ID_COL).getValue();
  const newStatus = range.getValue();
  const approvalNotes = sheet.getRange(row, APPROVAL_NOTES_COL).getValue();
  if (!rdaRecordId) return;
  const payload = { rda_record_id: rdaRecordId, new_status: newStatus, approval_notes: approvalNotes || '' };
  const response = UrlFetchApp.fetch('https://n8n.solofella.com/webhook/scx-sheet-approval-sync', {
    method: 'post', contentType: 'application/json',
    payload: JSON.stringify(payload), muteHttpExceptions: true
  });
  if (response.getResponseCode() === 200) sheet.getRange(row, SYNC_STATUS_COL).setValue('synced');
}
```
**Setup per Sheet:** Extensions → Apps Script, paste and save. Run once to authorize (the manual run throws a harmless `undefined range` error). Then Triggers (clock icon) → Add Trigger: function `onEdit`, deployment `Head`, source `From spreadsheet`, event `On edit`. Approve the second permission prompt. **A simple trigger cannot make external calls, and fails silently.**

### n8n workflow "RDA Sheet Approval Sync" (3 nodes, published)
1. **Webhook:** POST, path `scx-sheet-approval-sync`, no authentication, Production URL `https://n8n.solofella.com/webhook/scx-sheet-approval-sync`. The editor shows the Test URL by default, which is normal.
2. **Validate Payload (Code):** requires `rda_record_id` and `new_status`, passes `approval_notes`.
3. **PATCH NocoDB (HTTP):** `PATCH http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns/{{ $json.rda_record_id }}` with body `{"Approval Status": ..., "Approval Notes": ...}`, `xc-token` credential. The URL field **must be in Expression mode**. `RDA Record ID` in the Sheet is NocoDB's own row id, so no lookup is needed.

### Who uses the result today
MRA uses the statuses for approval-rate reporting. **RDA itself does not yet read them** (Steps 6g/6h, Section 10).

### Original design for Steps 6g/6h (to be rebuilt)
- **6g:** GET the client's recent `Not Accepted` or `Modified` records (limit 8, newest first).
- **6h:** build a `rejected_patterns_brief` from their Approval Notes, with Not Accepted ("avoid") and Modified ("close, adjust") kept separate.
- Position: after 6e, before 6f, so both languages receive it. Inject into 7a, 7c, 7a-ES and 7c-ES.
- **New requirement:** the brief is style feedback only and never overrides the contract or the no-fault rules.

---

## 13. BUGS FOUND AND FIXED (running log)

| # | Bug | Fix |
|---|---|---|
| 1 | Sync script used the Test URL | Production URL |
| 2 | Sync script path misspelled (`...approval-syn`) | Corrected |
| 3 | Simple trigger cannot call external URLs, silent failure | Installable trigger |
| 4 | PATCH URL field not in Expression mode (literal `=`) | Expression mode |
| 5 | 6c/6d used a stale pass-through value, giving zero-row lookups | Read the fresh EIP fetch from 6b |
| 6 | Step 2 treated `halt_reason` as a stop, though BRA uses it as a label | Label only. `governance_flag: Halt` stays the stop |
| 7 | Step 2: `hold_for_review` declared twice after stacked patches | Whole-node replacement |
| 8 | 7a-ES: cut-off return and "unexpected end of input" | Whole-node replacement |
| 9 | Step 18 `[ELEVATED]` subject could never fire | Tests the hold and Elevation Reason |
| 10 | Sign-off punctuation "!," | Conditional comma in 9d and 9d-ES |
| 11 | 7c missing the calm-not-comic patch, and 7a with two conflicting PERSONALITY lines | Corrected after file audit |
| 12 | Spanish prompts taught "no estuvimos a la altura" and "state the expected standard", which caused the original fault-admission draft | Removed, replaced by no-fault rules |

---

**VRYOH Intelligence · SCX_RDA_HOW_v8.0 · Chat #385 · October 6, 2026 · Solofella LLC**

**Verification status of this document:**
- ✅ **Read from your pasted code:** English nodes 7a through 18 (audited Oct 5), the Spanish nodes as pasted, and Step 2's final text.
- ⚠️ **User-reported, not seen by me:** the passing end-to-end test, and the BRA to RDA link.
- 🚫 **Not verified:** go-live status, 9d-ES, 10a-10c as stored, and the Claude call nodes.

Tell me when go-live is done and I'll update the Status line. Save this to GitHub as `SCX_RDA_HOW_v8.0.md`.
