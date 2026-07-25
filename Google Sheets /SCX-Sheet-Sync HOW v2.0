# SCX-Sheet-Sync HOW v2.0

**VRYOH INTELLIGENCE · SOLOFELLA LLC**  
**HOW DOCUMENT — SCX-Sheet-Sync**  
**Multi-Client Google Sheets Approval Workflow + Internal Reporting**  
Complete Step Decomposition · Node Logic · Credentials · Data Flow  
v2.0 · July 19, 2026 · Chat #85

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0 | Chat #76 · April 20, 2026. Single-client (PAK-001). OAuth failure diagnosis. Service Account solution. 11 nodes. |
| v1.1 | Chat #80 · April 26, 2026. Email reporting added (platform counts, pending calc, daily summary). Single-client only. 28 nodes. |
| v2.0 | Chat #83–85 · June 26 – July 19, 2026. Full multi-client architecture (unlimited clients via Client Config table). Brand renamed SubtextCX → VRYOH Intelligence. 8 critical production defects found and fixed during Aji Ceviche Bar (5-location) live pilot onboarding. Internal daily email re-scoped to operator-only (client-facing reporting moved to MRA). See Section 15 for full defect log. |

---

**PURPOSE:** Specifies every step, node, code block, credential, and data field for SCX-Sheet-Sync. Sufficient to build or repair the workflow from zero prior context.

---

## Summary Grid

| Property | Value |
|----------|-------|
| **Workflow Name** | SCX-Sheet-Sync |
| **Purpose** | Populate per-client Google Sheet approval interfaces with pending RDA records; send internal (operator-only) daily summary |
| **Trigger Type** | Schedule — 5am UTC daily |
| **Source Data** | RDA NocoDB table — all pending/pending-elevated records, any client |
| **Client Routing Source** | Client Config NocoDB table (`m95cmabjfyb94ps`) |
| **Destination** | Dynamic — one Google Sheet per client, resolved at runtime |
| **Credential Type** | Google Service Account (JSON key, non-expiring) |
| **Credential Name** | `Subtext-CX-GoogleSheets-ServiceAccount` |
| **Active Clients** | PAK-001 (Park Avenue Kitchen), AJI-001–005 (Aji Ceviche Bar: Orlando, Casselberry, Sarasota, Tampa, St. Petersburg) |
| **Internal Report Recipient** | `miguel@solofella.com` only — NOT sent to clients (see Section 10) |
| **Client-Facing Reporting** | Handled entirely by separate MRA workflow ("24-Hour Review Summary" / "Weekly Intelligence Brief") |
| **Scalability** | New client = 1 Client Config row + share sheet with Service Account. Zero code changes. |
| **Status** | ✅ CONFIRMED WORKING 100% — Chat #85, July 19, 2026 |
| **Doc Version** | v2.0 — July 2026 |

---

## 1. AGENT PURPOSE

### SCX-Sheet-Sync — Multi-Client Google Sheets Approval Workflow

Populates each client's dedicated Google Sheet with pending review response drafts from RDA, enriched with original review text from ALA, routed dynamically by Client ID. Deduplicates per-client against each sheet's own existing rows. Sends a daily internal summary email (operator-only) with per-client counts.

**NOT an approval agent.** Surfaces data for human approval; does not make approval decisions. A separate feedback mechanism (Apps Script → new webhook workflow — **planned, not yet built**, see Section 14) will eventually write approval decisions back to RDA.

### PRODUCES

- Google Sheet rows appended to each client's own Response Approvals sheet, correct tab, 12-column format
- Per-client dedup via RDA Record ID
- Daily internal summary email (all 6 clients' counts) → `miguel@solofella.com` only
- Error log + email notification on failure

### DOES NOT

- Make approval decisions
- Send any email to clients (that's MRA's job)
- Modify RDA records directly (planned feedback loop not yet built)
- Apply cell formatting/alignment (relies on pre-formatted sheet template)

---

## 2. DATA ARCHITECTURE

### Source 1 — RDA NocoDB Table
**Table ID:** `mr1v67cszcklwns`  
**Filter (Step 2b, in-code):** `Approval Status ≠ 'Published'` AND `Published Timestamp` empty AND `Client ID` not empty. No client hardcode.

### Source 2 — ALA NocoDB Table
**Table ID:** `m57efwbtrvwohhr` — bulk-fetched (Step 4), mapped by Id (Step 5).

### Source 3 — Client Config NocoDB Table (Routing)
**Table ID:** `m95cmabjfyb94ps`

Key fields: `Client ID`, `Client Name`, `Sheet ID`, `Sheet Tab Name`, `Approval Contact Email`.

**Confirmed 6 active rows:**

| Client ID | Client Name | Sheet ID | Tab Name |
|-----------|-------------|----------|----------|
| PAK-001 | Park Avenue Kitchen by David Burke | 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w | Park Ave |
| AJI-001 | Aji Ceviche Bar - Orlando | 1NuX8RJXQgqXFEdA2_txPiVFc2CJ26bgIdmkivn8Cuqc | Orlando |
| AJI-002 | Aji Ceviche Bar - Casselberry | 1UXNANFlrd0M5NODLIVO_WMZVqW0HMvn-4sjaGjOCy-E | Casselberry |
| AJI-003 | Aji Ceviche Bar - Sarasota | 1E3efDDLw5WEG7AuxTkVDz3Qr0Bdmr3mYkIfJMNWsNYI | Sarasota |
| AJI-004 | Aji Ceviche Bar - Tampa | 1LC6or__luwhRxNDk3kqPQtWmzGBuW2IGCqm3jiaTMHw | Tampa |
| AJI-005 | Aji Ceviche Bar - St. Petersburg | 1Kq0v3TEpeaGlW-GOeyOiSkBYHgeM4w6vU54HSat3Tyw | St. Petersburg |

**Google Drive ownership:** PAK-001 sheet under `solofellausa@gmail.com`; all 5 AJI sheets under `intelligence@vryoh.com`. Both share access to the same Service Account — ownership account is irrelevant to credential function.

**Note (Chat #85):** An extra empty row previously existed in Client Config, causing a phantom 7th loop iteration in Step 6a/Step 6-loop. Deleted — confirmed exactly 6 rows now.

### Destination — Per-Client Google Sheet (12 columns, A:L)

| Col | Header |
|-----|--------|
| A | SCX Date |
| B | Review Date |
| C | Platform |
| D | Star Rating |
| E | Reviewer Handle |
| F | Review Text |
| G | Proposed Response |
| H | Status |
| I | Edited Response |
| J | ALA Record ID |
| K | RDA Record ID |
| L | Sync Status |

**Requirement:** Sheets must have pre-formatted empty rows (centering, borders, cell sizing) before running. Append uses `OVERWRITE` mode — writes into existing formatted rows rather than inserting new unformatted ones (see Section 15, Defect 5).

**Sharing/Permissions (confirmed working, Chat #84):**
- Service Account: Editor on all 6 sheets (required for daily writes)
- Human approvers: Editor + **Protected Ranges** — protect all columns except Status/Edited Response, restricted to `intelligence@vryoh.com` and `scx-sheet-sync@solofella-cmh-project...` only. This blocks human edits to Review Text/Proposed Response while still allowing daily Service Account API writes (protected ranges do not apply to API/service-identity writes, only UI collaborator edits).
- Commenter-only access does NOT allow Status dropdown changes — must be Editor.

---

## 3. FULL WORKFLOW ARCHITECTURE — DATA FLOW

```
Step 1: Schedule Trigger (5am UTC)
         ↓
Step 2: Fetch Pending RDA Records (NocoDB GET, no client filter)
         ↓
Step 2b: Filter — Status≠Published, Published Timestamp empty, Client ID not empty
         ↓
Step 3: IF Records Exist?
         ↓ (TRUE)
Step 3b: Fetch Client Config (all rows)
         ↓
Step 3c: Build Sheet Map — {client_id: {sheet_id, tab_name, client_name, approval_email}}
         ↓
Step 4: Fetch ALL ALA Records (bulk, limit 100)
         ↓
Step 5: Build ALA Lookup Map (by Id)
         ↓
Step 6a: Explode Client Sheet Map → 6 items (one per client)
         ↓
Step 6-loop: SplitInBatches (batch=1) ⚠️ CRITICAL FIX — see Section 15, Defect 2
         ↓ (LOOP branch, 1 item at a time)
Step 6: Fetch Existing Sheet Rows (per client, dynamic documentId/sheetName)
         ↓ (loops back to Step 6-loop)
         ↓ (DONE branch, after all 6 clients processed)
Step 7: Build Combined Dedup Map (existing_rda_ids, across all 6 client sheets)
         ↓
Step 7b: Merge RDA list + ALA map + dedup map + client_sheet_map → per-record items with client_id/sheet_id/tab_name attached
         ↓
Step 8: SplitInBatches (batch=1) — main record loop
         ↓ (LOOP branch)
Step 9: Consolidated Logic — dedup check, ALA lookup, build 15-field row, normalize \n line breaks. alwaysOutputData: ON.
         ↓
Step 9-check: IF Node ⚠️ NEW (Chat #85, see Section 15, Defect 7) — {{ $json.rda_record_id }} is not empty
         ↓ (TRUE — real record)              ↓ (FALSE — duplicate/empty)
Step 9b: Rate Limit Delay (2 sec)             (directly back to Step 8)
         ↓
Step 10a: Build Sheet Payload — guard clause, OVERWRITE mode URL
         ↓
Step 10b: HTTP Append to Sheet (Service Account, alwaysOutputData: ON)
         ↓ (loops back to Step 8)
         ↓ (DONE branch, after all records processed)
Step 11a: Per-Client Platform Counts (client_counts object)
         ↓
Step 11: Build Summary (workflow-level, no client hardcode)
Step 12: Log Completion
         ↓
Step 12a: Pass-through dates + client_counts
         ↓
Step 12b1: Fetch All Pending RDA (fresh, for pending calc)
         ↓
Step 12b-filter: Per-Client Pending Calculation (pending_by_client)
         ↓
Step 12c: Build Per-Client Email Variables → 6 output items. approval_contact_email HARDCODED to miguel@solofella.com (internal-only, see Section 10)
         ↓
Step 12d: Build Email Body (loops all 6 items, VRYOH branding, per-client subject/content)
         ↓
Step 12e: Send via Brevo API (loops per item)
         ↓
Step 12f: Log Email Delivery (loops per item)
         ↓
[End]

[Error Handler]
ERR1: Error Trigger → ERR2: Build Error Record → ERR3: Send Error Email
```

---

## 4. KEY NODE CODE — CONFIRMED FINAL

### STEP 2b — Filter Pending Records (Multi-Client)

```javascript
const all_input = $input.all();
const filtered = [];

for (let i = 0; i < all_input.length; i++) {
  const item = all_input[i];
  const rec = item.json;

  const status = rec['Approval Status'];
  const published = rec['Published Timestamp'];
  const client = rec['Client ID'];

  const status_ok = (status !== 'Published');
  const published_ok = (published === null || published === undefined);
  const client_ok = (client !== null && client !== undefined && client !== '');

  if (status_ok && published_ok && client_ok) {
    filtered.push(rec);
  }
}

return [{ json: {
  list: filtered,
  pageInfo: { totalRows: filtered.length }
} }];
```

---

### STEP 3b — Fetch Client Config

```
Type: HTTP Request, Method: GET
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m95cmabjfyb94ps?limit=100
Auth: xc-token (Header Auth)
```

---

### STEP 3c — Build Sheet Map

```javascript
const configData = $input.first().json;
const configs = configData.list || [];

const clientSheetMap = {};

for (let i = 0; i < configs.length; i++) {
  const row = configs[i];
  const clientId = row['Client ID'];

  if (clientId && row['Sheet ID'] && row['Sheet Tab Name']) {
    clientSheetMap[clientId] = {
      sheet_id: row['Sheet ID'],
      tab_name: row['Sheet Tab Name'],
      client_name: row['Client Name'] || '',
      approval_email: row['Approval Contact Email'] || ''
    };
  }
}

return [{ json: {
  client_sheet_map: clientSheetMap,
  active_clients: Object.keys(clientSheetMap).length
} }];
```

**Note:** Raw pass-through of `Sheet ID` field — assumes NocoDB stores the bare ID string, not a full URL. Confirmed clean in production data (Chat #85). If a future client's Sheet ID is mistakenly pasted as a full URL, this will break — verify format when adding new clients.

---

### STEP 6a — Explode Client Sheet Map

```javascript
const mapData = $('STEP 3c — BUILD SHEET MAP').first().json;
const clientSheetMap = mapData.client_sheet_map || {};

const output = [];
const keys = Object.keys(clientSheetMap);
for (let i = 0; i < keys.length; i++) {
  const clientId = keys[i];
  output.push({
    json: {
      client_id: clientId,
      sheet_id: clientSheetMap[clientId].sheet_id,
      tab_name: clientSheetMap[clientId].tab_name
    }
  });
}

return output;
```

---

### STEP 6-loop — CRITICAL WRAPPER (v2.0)

```
Type: n8n-nodes-base.splitInBatches
Batch Size: 1
```

**Why:** See Section 15, Defect 2. Native Google Sheets node evaluates Document/Sheet expressions ONCE using only the first input item when fed multiple items simultaneously. This node forces Step 6 to receive exactly 1 item per round, making per-client dynamic resolution correct.

**Wiring:** Step 6a → Step 6-loop → (LOOP) → Step 6 → back to Step 6-loop → (DONE) → Step 7.

---

### STEP 6 — Fetch Existing Sheet Rows (Per Client)

```
Type: Google Sheets (native), Operation: Get Row(s)
Authentication: serviceAccount
Credential: Subtext-CX-GoogleSheets-ServiceAccount
Document: By ID → {{ $json.sheet_id }}
Sheet: By Name → {{ $json.tab_name }}
alwaysOutputData: true
retryOnFail: true, waitBetweenTries: 5000
```

⚠️ Enter expressions as `{{ $json.sheet_id }}` — do NOT manually type a leading `=`; n8n adds it automatically. Typing it yourself produces `=={{ ... }}` and breaks resolution.

---

### STEP 7 — Build Combined Dedup Map

```javascript
const sheet_input = $input.all();
const existing_rda_ids = {};

for (let i = 0; i < sheet_input.length; i++) {
  let row = sheet_input[i].json;
  let rda_id = String(row['RDA Record ID'] || row[10] || '').trim();

  if (rda_id !== '' && rda_id !== 'RDA Record ID') {
    existing_rda_ids[rda_id] = true;
  }
}

return [{ json: {
  existing_rda_ids: existing_rda_ids,
  existing_total: Object.keys(existing_rda_ids).length
} }];
```

---

### STEP 7b — Merge All Data

```javascript
const s7_data = $input.first().json;
const s3_data = $('Step 3 - Check Records Exist').first().json;
const s5_data = $('Step 5 - Build ALA Map').first().json;
const s3c_data = $('STEP 3c — BUILD SHEET MAP').first().json;

const rda_list = s3_data.list || [];
const ala_map = s5_data.ala_map || {};
const existing_rda_ids = s7_data.existing_rda_ids || {};
const clientSheetMap = s3c_data.client_sheet_map || {};

const output = [];
for (let i = 0; i < rda_list.length; i++) {
  const rda = rda_list[i];
  const clientId = rda['Client ID'] || '';
  const sheetInfo = clientSheetMap[clientId] || {};

  output.push({
    json: {
      rda_record: rda,
      ala_map: ala_map,
      existing_rda_ids: existing_rda_ids,
      client_id: clientId,
      sheet_id: sheetInfo.sheet_id || '',
      tab_name: sheetInfo.tab_name || ''
    }
  });
}

return output;
```

---

### STEP 9 — Consolidated Logic (Final)

```javascript
const input = $input.first().json;

const rda = input.rda_record;
const ala_map = input.ala_map || {};
const existing_rda_ids = input.existing_rda_ids || {};

const rda_record_id = String(rda['Id'] || '');

// DEDUP CHECK
if (existing_rda_ids[rda_record_id]) {
  return [];
}

// ALA LOOKUP
const ala_id_raw = rda['ALA Record ID'];
const ala = ala_map[ala_id_raw] || ala_map[String(ala_id_raw)] || ala_map[parseInt(ala_id_raw)] || {};

// NORMALIZE LINE BREAKS (fixes literal \n text — critical for Spanish drafts)
const cleanProposedResponse = String(rda['Public Response Draft'] || '').replace(/\\n/g, '\n');

return [{ json: {
  scx_date: (rda['RDA Timestamp'] || '').slice(0, 10),
  review_date: ala.review_date || '',
  platform: ala.platform || '',
  star_rating: ala.star_rating || '',
  reviewer_handle: ala.reviewer_handle || String(rda['Reviewer Handle'] || ''),
  review_text: ala.review_text || '',
  proposed_response: cleanProposedResponse,
  status: rda['Approval Status'] || 'Pending',
  edited_response: '',
  ala_record_id: String(ala_id_raw),
  rda_record_id: rda_record_id,
  sync_status: 'pending_sync',
  client_id: input.client_id || '',
  sheet_id: input.sheet_id || '',
  tab_name: input.tab_name || ''
} }];
```

**Settings:** `alwaysOutputData: ON`. Required so a legitimate `[]` (duplicate) result still propagates downstream as an empty item — without this, the workflow halts silently at this node.

---

### STEP 9-check — Skip Empty Items ⚠️ NEW (Chat #85)

```
Type: IF Node
Condition: {{ $json.rda_record_id }} — is not empty
TRUE branch  → Step 9b
FALSE branch → directly back to Step 8 (loop continues, record skipped)
```

**Why this exists:** See Section 15, Defect 7. Without this node, an empty item from Step 9 (a correctly-identified duplicate) travels forward into Step 9b → 10a → 10b, where it eventually crashes Step 10b (`URL parameter must be a string, got undefined`). Because n8n halts entire execution on an unhandled node error, this crash on the FIRST duplicate record in the loop order (sorted oldest-first) prevented ALL subsequent records — including genuinely new ones — from ever being processed. This IF node catches empty items immediately and routes them back to the loop instead of letting them travel further.

---

### STEP 10a — Build Sheet Payload (Final)

```javascript
const input = $input.first().json;

// SKIP if empty/duplicate item slipped through (safety net alongside Step 9-check)
if (!input || !input.rda_record_id || input.rda_record_id === '') {
  return [];
}

const sheetId = input.sheet_id;
const tabName = encodeURIComponent(input.tab_name);
const range = tabName + '!A:L';

const url = 'https://sheets.googleapis.com/v4/spreadsheets/' + sheetId + '/values/' + range + ':append?valueInputOption=USER_ENTERED&insertDataOption=OVERWRITE';

const rowValues = [
  input.scx_date || '', input.review_date || '', input.platform || '',
  input.star_rating || '', input.reviewer_handle || '', input.review_text || '',
  input.proposed_response || '', input.status || '', input.edited_response || '',
  input.ala_record_id || '', input.rda_record_id || '', input.sync_status || ''
];

const body = JSON.stringify({ values: [rowValues] });

return [{ json: {
  append_url: url,
  append_body: body,
  client_id: input.client_id || '',
  rda_record_id: input.rda_record_id || ''
} }];
```

**Critical:** `insertDataOption=OVERWRITE` (not `INSERT_ROWS`) — writes into the next available pre-formatted row without shifting/destroying existing formatting. See Section 15, Defect 5.

---

### STEP 10b — HTTP Append to Sheet

```
Type: HTTP Request, Method: POST
URL: {{ $json.append_url }}
Authentication: predefinedCredentialType
Credential Type: Google Service Account API
Credential: Subtext-CX-GoogleSheets-ServiceAccount
Send Body: true, Content Type: raw, application/json
Body: {{ $json.append_body }}
retryOnFail: true, Timeout: 30000
alwaysOutputData: ON ⚠️ REQUIRED (Chat #85)
```

**Required credential setting:** "Set up for use in HTTP Request node" must be enabled on the Service Account credential itself, or this node returns 401 `CREDENTIALS_MISSING` even with correct configuration otherwise. See Section 15, Defect 6.

---

### STEP 11a — Per-Client Platform Counts

```javascript
const loopItems = $input.all();
const clientCounts = {};

for (let i = 0; i < loopItems.length; i++) {
  const item = loopItems[i].json;
  if (!item || Object.keys(item).length === 0) continue;

  const clientId = item.client_id || 'UNKNOWN';

  if (!clientCounts[clientId]) {
    clientCounts[clientId] = {
      records_processed: 0, google_count: 0, opentable_count: 0,
      yelp_count: 0, tripadvisor_count: 0, other_count: 0, t3_count: 0
    };
  }

  clientCounts[clientId].records_processed++;

  const platform = (item.platform || '').toLowerCase();
  if (platform === 'google') clientCounts[clientId].google_count++;
  else if (platform === 'opentable') clientCounts[clientId].opentable_count++;
  else if (platform === 'yelp') clientCounts[clientId].yelp_count++;
  else if (platform === 'tripadvisor') clientCounts[clientId].tripadvisor_count++;
  else if (platform !== '') clientCounts[clientId].other_count++;

  const status = item.status || '';
  if (status === 'Pending-Elevated') clientCounts[clientId].t3_count++;
}

return [{ json: {
  loop_completed: true,
  client_counts: clientCounts,
  completion_timestamp: new Date().toISOString(),
  workflow_name: 'SCX-Sheet-Sync'
} }];
```

---

### STEP 12b-filter — Per-Client Pending Calculation

```javascript
const fetchData = $input.first().json;
const all_rda = fetchData.list || [];

const step11aData = $('Step 11a - Ensure Output').first().json;
const clientCounts = step11aData.client_counts || {};

const pendingByClient = {};
for (let i = 0; i < all_rda.length; i++) {
  const rec = all_rda[i];
  const approvalStatus = rec['Approval Status'];
  const publishedTimestamp = rec['Published Timestamp'];
  const clientId = rec['Client ID'];

  const isPending = (approvalStatus === 'Pending' || approvalStatus === 'Pending-Elevated');
  const isNotPublished = (publishedTimestamp === null || publishedTimestamp === undefined);
  const hasClient = (clientId !== null && clientId !== undefined && clientId !== '');

  if (isPending && isNotPublished && hasClient) {
    if (!pendingByClient[clientId]) pendingByClient[clientId] = 0;
    pendingByClient[clientId]++;
  }
}

const resultByClient = {};
const clientKeys = Object.keys(pendingByClient);
for (let i = 0; i < clientKeys.length; i++) {
  const cid = clientKeys[i];
  const totalPending = pendingByClient[cid];
  const newToday = (clientCounts[cid] && clientCounts[cid].records_processed) ? clientCounts[cid].records_processed : 0;
  const previousPending = totalPending - newToday;

  resultByClient[cid] = { total_pending: totalPending, pending_previous_days: previousPending, new_today: newToday };
}

return [{ json: { pending_by_client: resultByClient } }];
```

---

### STEP 12c — Per-Client Email Variables (INTERNAL ONLY)

```javascript
const dateData = $('Step 12a - Count Platform Records').first().json;
const pendingData = $('Step 12b-filter - Filter Previous Pending').first().json;
const step11aData = $('Step 11a - Ensure Output').first().json;
const clientConfig = $('STEP 3c — BUILD SHEET MAP').first().json;

const clientCounts = step11aData.client_counts || {};
const pendingByClient = pendingData.pending_by_client || {};
const clientMap = clientConfig.client_sheet_map || {};
const clientKeys = Object.keys(clientMap);

const output = [];
for (let i = 0; i < clientKeys.length; i++) {
  var cid = clientKeys[i];
  var info = clientMap[cid];
  var counts = clientCounts[cid] || {};
  var pending = pendingByClient[cid] || {};

  output.push({
    json: {
      date: dateData.date,
      prior_date: dateData.prior_date,
      google_count: counts.google_count || 0,
      opentable_count: counts.opentable_count || 0,
      yelp_count: counts.yelp_count || 0,
      tripadvisor_count: counts.tripadvisor_count || 0,
      other_count: counts.other_count || 0,
      t3_count: counts.t3_count || 0,
      records_processed_today: counts.records_processed || 0,
      pending_previous_days: pending.pending_previous_days || 0,
      total_pending: pending.total_pending || 0,
      approval_contact_email: 'miguel@solofella.com',
      from_email: 'intelligence@vryoh.com',
      client_name: info.client_name || cid,
      client_id: cid,
      sheet_url: 'https://docs.google.com/spreadsheets/d/' + info.sheet_id + '/edit'
    }
  });
}

return output;
```

⚠️ **Locked (Chat #84):** `approval_contact_email` is hardcoded to `miguel@solofella.com` regardless of Client Config's actual value. This report is operator-only. Never route to client contacts — see Section 10.

---

## 5. STEP 12d/12e/12f — EMAIL CHAIN (loops all 6 items)

Step 12d builds VRYOH-branded HTML email per client (subject includes client name), Step 12e sends via Brevo API per item, Step 12f logs delivery per item. All three correctly iterate `$input.all()` — no single-item bottleneck.

---

## 6. ERROR HANDLER — 3 NODES

```
ERR1: Error Trigger
ERR2: Build error record (workflow, message, node, timestamp, retry action)
ERR3: Send error email to fallback contact
```

---

## 7. NODE MAP — v2.0 (FULL)

```
[01] Schedule Trigger — 5am UTC
[02] NocoDB GET — Fetch Pending RDA
[02b] Code — Filter (no client hardcode)
[03] IF — Records Exist?
[03b] HTTP GET — Fetch Client Config
[03c] Code — Build Sheet Map
[04] HTTP GET — Fetch ALA (bulk)
[05] Code — Build ALA Map
[06a] Code — Explode Client Sheet Map (6 items)
[06-loop] SplitInBatches (batch=1) ⚠️ CRITICAL FIX
[06] Google Sheets — Fetch Existing Rows (per client, dynamic)
[07] Code — Build Combined Dedup Map
[07b] Code — Merge All Data (attach client_id/sheet_id/tab_name)
[08] SplitInBatches (batch=1) — main loop
[09] Code — Consolidated Logic (dedup, ALA lookup, line-break fix). alwaysOutputData: ON
[09-check] IF Node ⚠️ NEW — skip empty items, route back to Step 8
[09b] Wait — 2 sec rate limit
[10a] Code — Build Sheet Payload (guard clause, OVERWRITE mode)
[10b] HTTP Request — Append to Sheet (Service Account). alwaysOutputData: ON
[11a] Code — Per-Client Platform Counts
[11] Code — Build Summary
[12] Set — Log Completion
[12a] Code — Pass dates + counts
[12b1] HTTP GET — Fetch All Pending RDA (fresh)
[12b-filter] Code — Per-Client Pending Calculation
[12c] Code — Per-Client Email Variables (internal-only recipient)
[12d] Code — Build Email Body (loops all items, VRYOH branding)
[12e] HTTP Request — Send via Brevo (loops per item)
[12f] Code — Log Email Delivery (loops per item)

── Error Handler ──
[ERR1] Error Trigger
[ERR2] Code — Build error record
[ERR3] Email — Send error notification

TOTAL: ~33 nodes
```

---

## 8. CREDENTIALS

| Credential | Type | Use |
|------------|------|-----|
| `Subtext-CX-GoogleSheets-ServiceAccount` (id: `Pf4MiR7hQF3eu3ts`) | Google Service Account API | Steps 6, 10b. Must have "Set up for use in HTTP Request node" enabled. |
| `xc-token` (id: `DT9tnRgqYpPc3rXo`) | NocoDB Header Auth | All NocoDB reads |
| `Subtext-CX-Brevo` (`api-key`) | Header Auth | Step 12e |
| ~~`Subtext-CX-GoogleSheets`~~ | OAuth2 | NOT USED — fails in scheduled/task-runner mode |
| ~~`Subtext-CX-Google-OAuth2`~~ | Generic OAuth2 | NOT USED |

---

## 9. NocoDB TABLE IDS

| Table | ID |
|-------|-----|
| RDA | `mr1v67cszcklwns` |
| ALA | `m57efwbtrvwohhr` |
| Client Config | `m95cmabjfyb94ps` |

Client Config field IDs: Client ID=`cuchy7c4qdoxxww`, Client Name=`cqjammv6531we7q`, Approval Contact Email=`cy6btcle1yhe09s`, Sheet ID=`cwmclp9syachfpu`, Sheet Tab Name=`cap99rmhaygpj88`.

---

## 10. INTERNAL vs CLIENT-FACING REPORTING — LOCKED (Chat #84)

**SCX-Sheet-Sync's "Daily Review Response Summary"** (Step 12c/12d) is **operator-only**. Sent exclusively to `miguel@solofella.com`, regardless of what Client Config's `Approval Contact Email` field contains. Purpose: internal ops visibility across all 6 sheets.

**Clients receive a separate report entirely** — MRA's "24-Hour Review Summary" and "Weekly Intelligence Brief," built in a different workflow, with different content (signal direction, expression mode, pain domains) and different cadence (8am daily, Monday weekly). MRA report bugs are out of scope for this document.

**Do not merge these two reports or redirect SCX-Sheet-Sync's internal email to client contacts.**

---

## 11. SHEET ACCESS MODEL — LOCKED (Chat #84)

| Role | Access Level | Can Edit |
|------|-------------|----------|
| Service Account | Editor (unrestricted) | All columns, via API |
| Human approvers | Editor + Protected Range | Status, Edited Response only |
| Protected range restriction | `intelligence@vryoh.com` + Service Account only | N/A |

Protected ranges block human UI edits but do NOT block Service Account API writes — confirmed safe for daily automated appends to continue alongside human-restricted editing.

---

## 12. SCALABILITY — ADDING A NEW CLIENT

1. Create Google Sheet, 12-column template, pre-formatted rows
2. Share with `scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com` as Editor
3. Add Client Config row: Client ID, Client Name, Sheet ID (bare ID, not full URL), Sheet Tab Name, Approval Contact Email
4. No code changes — picked up automatically next run

---

## 13. KNOWN LIMITATIONS

| Limitation | Detail |
|------------|--------|
| ALA bulk fetch limit | `?limit=100` — no pagination beyond 100 |
| Sheet formatting is template-dependent | Pre-formatted rows must be manually maintained/extended |
| Step 8a spread operator | `...item.json` — flagged task-runner rule violation, unresolved, monitor |
| Sheet ID format assumption | Step 3c does raw pass-through, assumes bare ID string — will break if a future entry is a full URL |

---

## 14. NOT YET BUILT — APPROVAL FEEDBACK LOOP (Planned, Chat #84)

Human approvers can change Status in the sheet, but nothing currently writes that change back to RDA. Planned solution (Option A, chosen but not built):

- Google Apps Script `onEdit` trigger on each sheet → webhook → new n8n workflow `SCX-Sheet-Approval-Sync` → PATCH RDA
- Confirmed from RDA (Chat #97 answers): no separate "Edited Response" field in RDA — edited text must overwrite `Public Response Draft` directly; `Published Timestamp` must be set by this new workflow (RDA never sets it); `Approval Notes` field exists and is unused — appropriate for human commentary; no "approved by" field exists in RDA schema.

**Status: designed, not implemented.**

---

## 15. CRITICAL DEFECT LOG — v2.0 (Chat #83–85)

| # | Defect | Root Cause | Fix |
|---|--------|-----------|-----|
| 1 | Extra 7th loop iteration in Step 6a/6-loop | Empty row accidentally present in Client Config | Deleted empty row — confirmed exactly 6 |
| 2 | Step 6 only ever read PAK-001's sheet, 6× | Native Google Sheets node evaluates Document/Sheet expressions ONCE using the FIRST input item when fed multiple items at once — documented directly in n8n's own node UI notes | Wrapped Step 6 in SplitInBatches (batch=1) — "Step 6-loop" — so it only ever receives 1 item per execution round |
| 3 | Step 10a 404 "resource not found" on duplicate items | `alwaysOutputData: true` + Step 9's legitimate `return []` on dedup match → phantom empty item → Step 10a built URL from `undefined` values | Guard clause in Step 10a to check for empty/invalid input |
| 4 | Cross-brand record mixing across all 6 sheets | Stale leftover data written during the Defect #2 period (before Step 6-loop fix), not a live/current bug | Resolved automatically once Defect #2 fixed; confirmed via clean re-test |
| 5 | New sheet rows broke pre-existing formatting | `insertDataOption=INSERT_ROWS` inserts and shifts rows, destroying formatting | Changed to `insertDataOption=OVERWRITE` — writes into next empty formatted row without shifting |
| 6 | Step 10b 401 `CREDENTIALS_MISSING` | Service Account credential lacked "Set up for use in HTTP Request node" enabled | Enabled that setting on the credential |
| 7 | Entire execution silently halted on the FIRST duplicate record each run — all subsequent new records (including all new AJI records) never processed | Step 9's `alwaysOutputData: true` correctly passed an empty item forward on dedup match, but nothing downstream handled it — it reached Step 10b, which crashed ("URL parameter must be a string, got undefined") on an empty item, and n8n halts entire execution on unhandled node error, never returning to the loop | Added new **Step 9-check IF node** immediately after Step 9 — routes empty items directly back to Step 8, skipping 9b/10a/10b entirely. Also enabled `alwaysOutputData: ON` on Step 10b as a secondary safety net. **Confirmed fixed and working, Chat #85.** |
| 8 | Spanish response drafts showed literal `\n` text instead of line breaks | RDA/BRA drafting chain stores literal two-character `\n` sequence in some language paths instead of a real newline | `.replace(/\\n/g, '\n')` normalization added in Step 9 — language-agnostic, safe for English too |

---

## 16. INFRASTRUCTURE

| Component | Value |
|-----------|-------|
| n8n | 161.35.133.49:5678 / n8n.solofella.com, self-hosted 2.4.6 |
| NocoDB | 161.35.133.49:8080 (external), http://nocodb:8080 (internal) |
| NocoDB Base ID | `pq249fix22t3ofv` |
| Google Drive — PAK sheet | solofellausa@gmail.com |
| Google Drive — AJI sheets | intelligence@vryoh.com |
| Service Account project | solofella-cmh-project |
| Service Account email | scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com |

---

**VRYOH Intelligence · SCX_Sheet_Sync_HOW_v2.0 · Chat #85 · July 19, 2026 · Solofella LLC**
```
