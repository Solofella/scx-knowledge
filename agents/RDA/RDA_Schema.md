# RDA Schema

**NocoDB Table ID:** `mr1v67cszcklwns`
**Base ID:** `pq249fix22t3ofv`

---

## Table Fields

| Field Name | Type | Source | Purpose |
|---|---|---|---|
| Id | AutoNumber | System | Primary key |
| RDA Run ID | SingleLineText | Step 13 | Format: RDA-YYYYMMDD-HHMMSS-NNN |
| BRA Record ID | Number | BRA webhook | FK → BRA table |
| ALA Record ID | Number | BRA webhook | FK → ALA table — direct traceability |
| RDA Timestamp | DateTime | Step 13 | ISO 8601 UTC |
| Confirmed Response Tier | SingleSelect | BRA webhook | T1 / T2 / T3 — kept for approver UX |
| Public Response Draft | LongText | Step 9d | Audited or fallback draft. Approver may edit. |
| Internal Follow-Up Draft | LongText | Step 9d | Signal interpretation brief. Internal only. Never shown to client. |
| Approval Status | SingleSelect | Step 12 | Pending / Pending-Elevated / Approved / Edited-Approved / Not Accepted / Published |
| Commercial Commitment Flag | Checkbox | Step 11 | Deterministic scan — EN + ES prohibited terms |
| Elevation Reason | LongText | Step 12 | Pipe-delimited elevation triggers. null if Pending. |
| Flagged Terms | SingleLineText | Step 11 | Prohibited terms detected. null if clean. |
| Client ID | SingleLineText | BRA webhook | Which brand voice config was used |
| lang | SingleLineText | Step 7a | en/es — record-level pipeline lang tag |
| Audit Passed | Checkbox | Step 9d | true if all 11 audit items pass |
| Audit Failed Items | SingleLineText | Step 9d | Item names or AUDIT PARSE ERROR or FULL REVIEW REQUIRED |
| Reviewer Handle | SingleLineText | BRA webhook | Guest name from ALA |
| Published Timestamp | DateTime | Human operator | null until published. MRA anchor. Set manually. |
| Approval Notes | LongText | Human approver | Approver comments. Rejection reason codes. |
| Error Log | LongText | System | null on clean write |

**SingleSelect pre-population — no trailing spaces:**
- Confirmed Response Tier: T1 / T2 / T3
- Approval Status: Pending / Pending-Elevated / Approved / Edited-Approved / Not Accepted / Published

---

## Relationships

**Upstream:** BRA Step 18 fires webhook to RDA with rich payload — all 28 fields including BRA strategy outputs and EIP pass-throughs. RDA does NOT fetch the BRA NocoDB record.

**Downstream:**
- **Human approval gate** — approval email sent to approval_contact_email from Client Config. Client operator updates Approval Status directly in NocoDB.
- **MRA** — reads from RDA NocoDB table. Published Timestamp and Approval Status are the MRA anchors.
- **Dashboard** — read-only display of approval counts and signal metrics. Draft text NOT shown in dashboard.

---

## Field Computation Details

### Public Response Draft — Five-Call Claude Architecture

**Computed by:** Five sequential Claude calls — claude-sonnet-4-6

**Governing principle across all five calls:** The response must read as written by a person, not optimized by a machine.

**Call 1 — Opening Constructor (Step 7b)**
- temp: 0 · max_tokens: 150
- Produces one sentence only
- T1: conditional lead selection with priority ordering (NAMED STAFF → GUEST WORD → OCCASION → SPECIFIC DETAIL → BELONGING → GRATITUDE)
- T2: Hi or Hello + guest name, specific signal
- T3: Hello + guest name, grave and human tone
- Guest name MUST appear when available

**Call 2 — Body Builder (Step 7d)**
- temp: 0.5 · max_tokens: 500
- Receives opening sentence from Call 1 — MUST use it exactly as first line
- Completes draft per tier rules (T1: 2-3 sentences, T2: 2-4 sentences, T3: 3-5 sentences)
- PADDING TEST: each sentence after opening evaluated — adds new information or deleted
- T1 MIXED SIGNAL RULE: minor criticism acknowledged directly in one sentence

**Call 3 — SEO + Governance (Step 7f)**
- temp: 0 · max_tokens: 600
- TASK 1: Weave 1-2 SEO keywords naturally — omit if unnatural
- TASK 2: Remove prohibited content (email, URL, named channels, refund/voucher language, "We apologize for any inconvenience")
- TASK 3: Guest name check — insert if absent (T1: "[Name]," at start; T2/T3: "Hello [Name] —" before opening)

**Call 4 — Internal Brief (Step 7h)**
- temp: 0 · max_tokens: 400
- Produces internal_followup_draft independently
- Describes what signal means — never what to do about it
- Stability framing: trajectory of guest's relationship with the brand
- 3-5 sentences, no padding

**Call 5 — Audit (Step 9c)**
- temp: 0 · max_tokens: 1500
- 11-item checklist: NAMED STAFF, REVIEW DEPTH MATCH, CLOSING SPECIFICITY, LOYALTY SIGNAL, CONSTRUCTIVE SUGGESTION, PROHIBITED CONTENT, OVERUSED VERBS, OVERUSED PHRASES, BRAND PHRASE COUNT, OPENING REPETITION, GUEST NAME PRESENT
- Corrects ONLY failing elements — never rewrites passing elements
- Fallback: if parse fails, generation_draft used with AUDIT PARSE ERROR flag
- If 3+ items fail: FULL REVIEW REQUIRED prefix — signals human rewrite needed

---

### Internal Follow-Up Draft — Governance Constraint

**Computed by:** Call 4 — Internal Brief (Step 7h)

**What to include:**
- What emotional state the guest was in and what drove it
- What the signal reveals about the guest's relationship with the brand
- Stability framing: what does the recurrence context reveal about the trajectory of this guest's relationship with the brand?
- Any behavioral risk or loyalty indicator

**What to never include:**
- Recommended actions or operational directives
- Headers like "Next Steps" or "Action Items"
- Prescriptive language telling the team what to do

**Good example:**
> "This guest's signal is structurally bifurcated: broad satisfaction coexists with a precise, price-anchored disappointment tied to the tomahawk steak. Their analytical framing suggests genuine engagement with the brand rather than a complaint posture, giving this feedback diagnostic weight. As a first recorded signal, the mixed-signal structure at first contact is a meaningful indicator of moderate churn risk contingent on whether a return visit resolves the value gap."

**Bad example (governance violation):**
> "Add more servers during weekend dinner rush." ← PRESCRIPTIVE — NOT ALLOWED

---

### Commercial Commitment Flag — Deterministic Scan

**Computed by:** Step 11 — Code Node (not Claude)

**EN prohibited terms:** refund, voucher, complimentary, discount, credit, compensation, reimburse, waive, free of charge, no charge

**ES prohibited terms:** reembolso, descuento, cortesia, gratuito, gratis, sin costo, sin cargo, compensacion

**Both lists applied regardless of lang — bilingual safety net.**

Scan runs on both public_response_draft and internal_followup_draft combined. Sets commercial_commitment_flag boolean and flagged_terms string.

---

### Approval Status — Lifecycle

**Initial value set by Step 12 (deterministic):**
- `Pending` — no elevation triggers
- `Pending-Elevated` — any of: T3 record, governance_flag = Flag, human_review_required = true, commercial_commitment_flag = true

**Updated by human operator directly in NocoDB:**
- `Approved` — draft accepted as-is, cleared for publication
- `Edited-Approved` — human edited draft before approving, original preserved
- `Not Accepted` — draft rejected, reason in Approval Notes
- `Published` — response published to platform, Published Timestamp set

**Phase 1 approval method:** Client operator updates Approval Status directly in NocoDB. No email parsing. No automated write-back.

---

### Audit Passed + Audit Failed Items

**Computed by:** Step 9d — Parse Audit Output

**audit_passed:** true only if all 11 checklist items pass  
**audit_failed_items:** comma-separated item names, or AUDIT PARSE ERROR, or FULL REVIEW REQUIRED — [item names] if 3+ failures

---

## Token Budget

**~3,750 tokens per record (five calls combined)**

| Call | Step | Approx tokens |
|---|---|---|
| Opening Constructor | 7b | ~150 output |
| Body Builder | 7d | ~500 output |
| SEO + Governance | 7f | ~600 output |
| Internal Brief | 7h | ~400 output |
| Audit | 9c | ~1,500 output |

Input tokens vary by review length and brand voice config size. At PAK-001 operational volume of 3-5 records per day, token cost is negligible at any tier.

---

## Data Flow

```
BRA Step 18 webhook → RDA webhook trigger
↓
Step 2: Idempotency check (BRA Record ID exists in RDA?)
↓
Steps 3-5: Brand Voice load from Client Config NocoDB (session-cached)
↓
Steps 7a-7b: Opening Constructor Claude call → one sentence
↓
Steps 7c-7d: Body Builder Claude call → draft completion
↓
Steps 7e-7f: SEO + Governance Claude call → final public draft
↓
Steps 7g-7h: Internal Brief Claude call → internal_followup_draft
↓
Step 8: Parse internal brief + assemble final output
↓
Step 9a: Fetch ALA raw review text
Step 9a-2: Fetch 2 most recent RDA drafts (opening variation)
↓
Steps 9b-9d: Audit Claude call → 11-item correction pass
↓
Step 10: Output validation
Step 11: Commercial commitment scan (EN + ES)
Step 12: Approval status assignment (Pending / Pending-Elevated)
Step 13: RDA Run ID + timestamp
↓
Steps 14-15: Build NocoDB POST body → write RDA record (20 fields)
Step 16: Capture RDA Record ID
Step 17: PATCH BRA RDA Status → Complete
Step 18: Build + send approval notification email (48-hour SLA)
```

---

## Quality Baseline

**87% · Chat #75 · April 2026**

Guest name enforced at three independent checkpoints. Named staff enforced at three checkpoints. T1 mixed-signal records acknowledge minor criticism. Opening variation mechanism prevents repetition across consecutive records. Commercial commitment scan operational in EN + ES.

**Open items:**
- First-name extraction from reviewer_handle — full surnames appearing, fix pending
- Formula checklist requires periodic review after each pilot month

---

## Related Documents

- **HOW Document:** SCX_RDA_HOW_v3.2.md
- **Upstream Agent:** BRA — `agents/BRA/`
- **NocoDB Client Config Table:** `m95cmabjfyb94ps`
- **NocoDB Base ID:** `pq249fix22t3ofv`
- **MCD:** SubtextCX_MCD_v7.4

---

**End of RDA Schema**
*Subtext CX · RDA_Schema · Chat #75 · April 2026 · Solofella LLC*

---

Review and confirm. When approved I commit both files to GitHub.
