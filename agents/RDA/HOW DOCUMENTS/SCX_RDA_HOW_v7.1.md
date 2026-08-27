Fetching the real v7.0 document to use as the base.Chat #96 · July 3, 2026

Confirmed real v7.0 content — identical to what's already in this conversation. Base document unchanged. Appending the v7.1 addendum now, full detail, per your instruction.

---

# SCX_RDA_HOW_v7.1

*(All content of v7.0 above stands unchanged. The following is the v7.1 addendum, appended in full.)*

---

## ADDENDUM — v7.1: Sheet↔NocoDB Approval Status Feedback Loop

**Date: July 3, 2026 (Chat #96, continued session) · Status: Built, tested, deployed across all 6 client Sheets**

### A. Why This Was Built

Prior to this addendum, RDA's `Approval Status` field wrote `Pending` at creation (Step 14/15) and **never changed again** — confirmed directly: no mechanism existed to write a human's real decision (made in the Google Sheet, the actual approval interface) back into NocoDB. Every record was functionally "Pending" forever from RDA's own point of view, regardless of what was decided.

This was independently confirmed as a known, documented gap in a separate agent's own HOW document (SCX-Sheet-Sync HOW v4.0, fetched this session): *"Apps Script `onEdit` → webhook → RDA PATCH feedback loop — Not built — Confirmed absent on all 6 sheets."*

**Two consequences of this gap, both closed by this addendum:**
1. No real quality signal reached RDA — approvals, rejections, and edits were invisible to the system generating future drafts.
2. Downstream reporting (e.g., MRA's approval-rate and time-to-decision metrics) had no real decision data to compute from.

### B. Governance vs. Style Decision, Established Before Build

Confirmed directly by the user: this feedback loop's purpose is strictly **quality control**, not automated regeneration and not a notification-only alert system. When a human changes status to **Approved**, that confirms the draft's quality and message were correct. When changed to **Not Accepted** or **Modified**, that signals the system should consider a different approach for that client's future drafts — implemented as data injected into the next generation cycle, not as a triggered action.

**Four status values, confirmed meaning, direct from the user:**
| Status | Meaning |
|---|---|
| **Pending** | Awaiting a status change by the user. |
| **Approved** | User satisfied with quality; response used exactly as generated. |
| **Not Accepted** | User rejects the response entirely; will not be used. Approval Notes expected to contain a reason. |
| **Modified** | User agrees with part of the response but will make changes before posting; the nature of the changes is entirely the user's discretion — **not obligated to share what was changed.** Approval Notes may be empty for this status. |

### C. Timestamp Field — Explicitly Rejected

A separate approval-decision timestamp field (and a corresponding new `RDA Approval Log` table) was proposed during design but **explicitly rejected by the user**: *"si el tiempo no es indicador esencial para el sistema, lo importante es la calidad"* — time is not an essential indicator; quality is what matters. No timestamp field or log table was built. Status + Approval Notes alone carry the full quality signal this addendum was built to capture.

### D. Architecture — Three Components

#### D.1 — Google Sheet Column Repurposing (Manual)

**Confirmed real column layout, all 6 client Sheets (PAK-001, AJI-001–005):**

| Column | Position | Name |
|---|---|---|
| 1 | SCX Date |
| 2 | Review Date |
| 3 | Platform |
| 4 | Star Rating |
| 5 | Reviewer Handle |
| 6 | Review Text |
| 7 | Proposed Response |
| **8** | **Status** |
| **9** | **Approval Notes** (renamed from "Edited Response" — confirmed real column repurposing, not a new column) |
| 10 | ALA Record ID |
| **11** | **RDA Record ID** |
| **12** | **Sync Status** |

**Finding, confirmed via direct fetch of SCX-Sheet-Sync HOW v4.0:** column 12, "Sync Status," was already present in every Sheet, hardcoded to the literal string `'pending_sync'` by Sheet-Sync's own Step 9 at row-creation time — confirmed to never have been updated by anything prior to this addendum. Safe to repurpose as a live sync-confirmation indicator, since nothing else reads or writes it.

#### D.2 — Google Apps Script (per-Sheet, deployed to all 6)

**Confirmed final, tested, working code:**

```javascript
function onEdit(e) {
  const range = e.range;
  const sheet = range.getSheet();
  const col = range.getColumn();
  const row = range.getRow();

  const STATUS_COL = 8;
  const APPROVAL_NOTES_COL = 9;
  const RDA_RECORD_ID_COL = 11;
  const SYNC_STATUS_COL = 12;

  if (col !== STATUS_COL) return;
  if (row === 1) return; // skip header row

  const rdaRecordId = sheet.getRange(row, RDA_RECORD_ID_COL).getValue();
  const newStatus = range.getValue();
  const approvalNotes = sheet.getRange(row, APPROVAL_NOTES_COL).getValue();

  if (!rdaRecordId) return;

  const payload = {
    rda_record_id: rdaRecordId,
    new_status: newStatus,
    approval_notes: approvalNotes || ''
  };

  const response = UrlFetchApp.fetch('https://n8n.solofella.com/webhook/scx-sheet-approval-sync', {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(payload),
    muteHttpExceptions: true
  });

  if (response.getResponseCode() === 200) {
    sheet.getRange(row, SYNC_STATUS_COL).setValue('synced');
  }
}
```

**CRITICAL SETUP REQUIREMENT, confirmed necessary through live debugging (see Section G, Bug #3):** `onEdit(e)` **must be registered as an installable trigger, not left as a simple trigger.** Google Apps Script's simple-trigger security model silently blocks `UrlFetchApp.fetch` (external network calls) from firing inside auto-detected `onEdit` functions — no error is shown anywhere, the script simply never reaches the webhook. This is a hard platform restriction, not a bug in this code.

**Required manual setup, per Sheet (×6):**
1. Open the Sheet → Extensions → Apps Script.
2. Paste the script above (with the correct, real Production URL — not the Test URL).
3. Save, name the project (e.g., "RDA Approval Sync").
4. Run the function once manually to trigger the first-time authorization prompt (this manual run itself will throw `TypeError: Cannot read properties of undefined (reading 'range')` — expected, since `e` is undefined outside a real edit event; this error does not indicate a code problem).
5. Approve the permission prompt (Review permissions → Advanced → Go to [project] (unsafe) — expected for personal scripts).
6. Go to the clock icon (Triggers) in the left sidebar → "+ Add Trigger."
7. Set: Function = `onEdit`, Deployment = `Head`, Event source = `From spreadsheet`, Event type = `On edit`. Save.
8. Approve the second, full permission prompt that appears at this step (distinct from step 5's manual-run prompt).

Repeat steps 1-8 identically for all 6 Sheets.

#### D.3 — n8n Workflow "RDA Sheet Approval Sync" (new, separate workflow, 3 nodes)

**Confirmed built and tested, all 3 nodes:**

**Node 1 — Step 1 - Webhook:**
- Method: POST
- Path: `scx-sheet-approval-sync`
- Authentication: None
- Confirmed real Production URL: `https://n8n.solofella.com/webhook/scx-sheet-approval-sync`
- **Note confirmed via live debugging:** the node's editor view defaults to displaying the Test URL (`/webhook-test/...`) whenever the node is open — this is normal n8n behavior and does not indicate the Production URL is inactive, provided the workflow is saved and activated.

**Node 2 — Step 2 - Validate Payload (Code Node):**
```javascript
const body = $input.first().json.body;

if (!body || !body.rda_record_id || !body.new_status) {
  throw new Error('Invalid payload — missing rda_record_id or new_status');
}

return [{ json: {
  rda_record_id: body.rda_record_id,
  new_status: body.new_status,
  approval_notes: body.approval_notes || ''
} }];
```

**Node 3 — Step 3 - PATCH NocoDB (HTTP Request Node):**
- Method: PATCH
- URL (Expression mode required — see Section G, Bug #4): `http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns/{{ $json.rda_record_id }}`
- Authentication: `xc-token` credential
- Body (JSON):
```json
{
  "Approval Status": "{{ $json.new_status }}",
  "Approval Notes": "{{ $json.approval_notes }}"
}
```
- **Confirmed live output, direct record PATCH by NocoDB's own record ID** (not a WHERE-filtered search) — simpler and more reliable than the originally-proposed lookup-then-update approach, since `RDA Record ID` (Sheet column 11) is confirmed to be NocoDB's own internal row `Id`.

**Full workflow chain:** Webhook → Validate Payload → PATCH NocoDB. Confirmed activated (Published) and tested successfully end-to-end.

### E. RDA Pipeline Addition — Rejected/Modified Pattern Feedback (Steps 6g-6h)

**Placement, confirmed correct after architecture review:** since the RDA pipeline's Language Router (Step 6f) splits execution into two full parallel branches, the rejected-patterns brief must be built **before** Step 6f, so both language branches can consume it. Originally proposed as "Step 9/10" additions, corrected to the 6-series to avoid collision with real existing Steps 9a-9e/10.

**Step 6g — Fetch Recent Rejected/Modified Drafts (HTTP GET, new node):**
- Position: after Step 6e, before Step 6f.
- URL:
```
=http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns?where=(Client%20ID,eq,{{ $('Step 6a — Brand Voice Consolidation').first().json.client_id }})~and((Approval%20Status,eq,Not%20Accepted)~or(Approval%20Status,eq,Modified))&sort=-RDA%20Timestamp&limit=8
```
- Authentication: `xc-token`.

**Step 6h — Build Rejected Patterns Brief (Code Node, new node):**
```javascript
const inp = $('Step 6e — Build Signal Enrichment Brief').first().json;
const recordsData = $input.first().json;
const recordsList = recordsData.list || [];

const rejectedNotes = [];
const modifiedNotes = [];

for (let i = 0; i < recordsList.length; i++) {
  const rec = recordsList[i];
  const note = rec['Approval Notes'];
  const status = rec['Approval Status'];
  if (note && note.trim().length > 0) {
    if (status === 'Not Accepted') {
      rejectedNotes.push(note);
    } else if (status === 'Modified') {
      modifiedNotes.push(note);
    }
  }
}

const briefParts = [];

if (rejectedNotes.length > 0) {
  briefParts.push('REJECTED — these approaches were fully declined, avoid repeating:\n' +
    rejectedNotes.map(function(n, i) { return (i + 1) + '. ' + n; }).join('\n'));
}

if (modifiedNotes.length > 0) {
  briefParts.push('MODIFIED — these were close but needed adjustment:\n' +
    modifiedNotes.map(function(n, i) { return (i + 1) + '. ' + n; }).join('\n'));
}

const rejectedPatternsBrief = briefParts.length > 0
  ? briefParts.join('\n\n')
  : 'RECENT REJECTED/MODIFIED PATTERNS: None available.';

const out = {};
const inpKeys = Object.keys(inp);
for (let i = 0; i < inpKeys.length; i++) { out[inpKeys[i]] = inp[inpKeys[i]]; }
out.rejected_patterns_brief = rejectedPatternsBrief;

return [{ json: out }];
```

**Design rationale — Not Accepted vs. Modified handled distinctly, not merged:** per the confirmed status definitions (Section B), a `Not Accepted` record's note describes an approach to fully avoid; a `Modified` record's note (when present — not guaranteed, per the "no obligation to share" clarification) describes an approach that was close but needed refinement. Presenting these under separate labels lets Claude distinguish "never do this" from "this direction is right, adjust the execution."

**Rewiring:** Step 6e → Step 6g → Step 6h → Step 6f (replacing the prior direct Step 6e → Step 6f connection).

### F. Injection Into Generation Nodes — Four Nodes, Both Languages

**Confirmed requirement:** since the language split happens at Step 6f, downstream of Steps 6g/6h, the rejected-patterns brief must be injected into **all four** opening/body prompt-builders — not two, as originally scoped before the parallel-branch architecture was confirmed.

| Node | Variable used | Injected line |
|---|---|---|
| Step 7a | `rec` | `REJECTED PATTERNS: ${rec.rejected_patterns_brief \|\| 'None available'}` |
| Step 7c | `inp` | `REJECTED PATTERNS: ${inp.rejected_patterns_brief \|\| 'None available'}` |
| Step 7a-ES | `rec` | `PATRONES RECHAZADOS: ${rec.rejected_patterns_brief \|\| 'No disponible'}` |
| Step 7c-ES | `inp` | ` Patrones rechazados: ${inp.rejected_patterns_brief \|\| 'No disponible'}` |

**Pass-through requirement, confirmed necessary:** `rejected_patterns_brief` must be added to the return objects of 7a, 7c, 7e (and ES equivalents) — same pattern as `signal_enrichment_brief` and `brand_voice_brief` — or it is lost after the node that first receives it.

### G. Bugs Found and Fixed During Build/Test (Direct, Real, Confirmed)

**Bug #1 — Test URL used instead of Production URL (Apps Script).** First draft of the deployed script used `https://n8n.solofella.com/webhook-test/scx-sheet-approval-sync`. Test URLs only respond while the target workflow is open in the n8n editor — this would have caused silent, intermittent failure once editing stopped. Corrected to the Production URL (no `-test` segment).

**Bug #2 — Path misspelling (Apps Script).** A subsequent draft read `scx-sheet-approval-syn` (missing the final "c"). Corrected by direct character comparison against the confirmed-real Production path.

**Bug #3 — Simple trigger cannot make external calls (Google Apps Script platform restriction).** Confirmed via live test: Sheet edits produced no webhook call, no n8n execution, no NocoDB update, and no visible error anywhere — `Sync Status` remained `pending_sync`, NocoDB remained `Pending`, n8n Executions tab showed 0 executions. Root cause: Google restricts simple triggers (auto-detected `onEdit`) from calling `UrlFetchApp.fetch`. Resolved by converting to an installable trigger (Section D.2, steps 6-8).

**Bug #4 — URL field not set to Expression mode (n8n PATCH node).** Error: `Invalid URL: =http://nocodb:8080/... URL must start with "http" or "https".` The literal `=` character was being sent as part of the URL string rather than being interpreted as n8n's expression-mode indicator — meaning the field was in Fixed/plain-text mode rather than Expression mode. Resolved by switching the URL field to Expression mode before re-entering the URL.

**Post-fix confirmation:** live test successful — Step 3 - PATCH NocoDB output confirmed showing `Approval Status: Approved` written to the real NocoDB record, matching the Sheet edit exactly.

### H. Deployment Status — Confirmed Complete

All 6 client Sheets (PAK-001, AJI-001, AJI-002, AJI-003, AJI-004, AJI-005) confirmed by the user to have: the repurposed Approval Notes column, the deployed Apps Script, and the installable `onEdit` trigger correctly configured. The "RDA Sheet Approval Sync" n8n workflow confirmed Published and tested successfully end-to-end.

### I. Impact on VRYOH System — What This Closes

1. **RDA's Approval Status field is no longer permanently "Pending."** It now reflects the real, current human decision within seconds of a Sheet edit.
2. **The rejected-patterns brief (Steps 6g-6h) now operates on real data** instead of returning "None available" on every run — RDA's future draft generation is informed by actual approval/rejection/modification history, per client, for the first time.
3. **A separately-identified, independently-documented gap is now closed** — the SCX-Sheet-Sync HOW v4.0's own "Known Issues" entry for this exact feedback loop no longer applies to the live system as of this addendum.
4. **Downstream reporting metrics that depend on real decision data** (e.g., approval rate, previously blocked entirely — not time-to-decision, which was explicitly descoped per Section C) now have real, non-Pending status values to compute from.

### J. Open Items Remaining After This Addendum

- The `Modified` status's Approval Notes are optional by design (user confirmed no obligation to share edit details) — meaning Steps 6g/6h's `MODIFIED` brief section will frequently be empty even when genuine modifications occurred. This is accepted, expected behavior per the confirmed design, not a defect.
- No mechanism exists to alert a human if `Sync Status` fails to update to `synced` after an edit (e.g., if n8n is briefly down) — a silent-failure risk was identified during Bug #3's debugging and is structurally similar, though the installable-trigger fix resolves the specific cause found this session. Not further monitored or alerted on as of this addendum.
- Steps 6g/6h's query limit (8 records) and the split between `Not Accepted` (always shown if noted) vs. `Modified` (shown only if noted) has not been tested against real accumulated approval history at volume — behavior confirmed correct in code review, not yet observed against a large real dataset.

---

**VRYOH Intelligence · SCX_RDA_HOW_v7.1 · Chat #96 · July 3, 2026 · Solofella LLC**
