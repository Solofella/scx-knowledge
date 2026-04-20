# SCX-Sheet-Sync HOW v1.0

**SUBTEXT CX · SOLOFELLA LLC**  
**HOW DOCUMENT — SCX-Sheet-Sync**  
**Google Sheets Approval Workflow Automation**  
Complete Step Decomposition · Node Logic · Credentials · Data Flow  
v1.0 · April 20, 2026 · Chat #76

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0 | Initial build — Chat #76 · April 20, 2026. Complete workflow specification. OAuth credential failure diagnosis (Chat #76). Service Account solution integrated. 11 nodes. |

---

**PURPOSE:** This document specifies every step, node, code block, credential configuration, and data field for the SCX-Sheet-Sync (Google Sheets Approval Population) n8n workflow. A developer or Miguel must be able to build or repair SCX-Sheet-Sync from this document alone with zero prior context.

---

## Summary Grid

| Property | Value |
|----------|-------|
| **Workflow Name** | SCX-Sheet-Sync |
| **Purpose** | Populate PAK-001 Response Approvals Google Sheet with pending RDA records on scheduled cadence |
| **Trigger Type** | Schedule — 7am UTC daily |
| **Trigger Cadence** | Daily at 07:00 UTC |
| **Source Data** | RDA NocoDB table (pending/pending-elevated records only) |
| **Destination** | Google Sheet (PAK-001 Response Approvals) |
| **Destination Sheet ID** | 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w |
| **Destination Range** | Sheet1!A:H (8 columns) |
| **Credential Type** | Google Service Account (JSON key — non-expiring) |
| **Credential Status** | TO BE IMPLEMENTED (Chat #77) |
| **Total n8n Nodes** | 11 main + error handler (3 nodes) = 14 |
| **Data Model** | Rich join (RDA + ALA) before sheet write |
| **Idempotency** | Fetch existing sheet rows, filter new records before append |
| **Error Handling** | Error trigger → email notification + log |
| **Phase 1 Scope** | Manual CSV upload + Apps Script approval gate. SCX-Sheet-Sync automates sheet population only. |
| **Phase 2 Scope** | Google Business Profile API + Yelp Fusion API ingestion (not in this workflow). |
| **Active Clients** | PAK-001 (Park Avenue Kitchen) — Live |
| **Doc Version** | v1.0 — April 2026 |

---

## 1. AGENT PURPOSE

### SCX-Sheet-Sync — Google Sheets Approval Workflow Automation

Automates the daily population of the client's Google Sheet approval interface with pending review response drafts from the RDA NocoDB table. Enriches RDA records with original review text from ALA. Deduplicates against existing sheet rows to prevent duplicate appends. Enables human approvers (e.g., Christine at Park Avenue Kitchen) to review and approve drafted responses in a familiar Google Sheets interface rather than navigating n8n or NocoDB directly.

**NOT an approval agent.** SCX-Sheet-Sync does not make approval decisions. It surfaces data for human approval. The Apps Script onEdit trigger listens for approval status changes in the sheet and fires a webhook to update RDA NocoDB records.

**AUTOMATION BOUNDARY:** SCX-Sheet-Sync runs on schedule. Apps Script (approval gate) runs on edit. No circular dependencies.

### SCX-Sheet-Sync PRODUCES

- Google Sheet rows appended to PAK-001 Response Approvals sheet
- Each row contains: Date, Platform, Stars, Review Text, Proposed Response, Edited Response, Approval Status, RDA Record ID
- Prevents duplicates via RDA ID deduplication
- Error log + email notification on failure

### SCX-Sheet-Sync DOES NOT

- Make approval decisions
- Modify NocoDB RDA records (that is Apps Script → n8n webhook)
- Filter by approval status post-write (filtering happens pre-write)
- Execute on-demand (scheduled only — no manual trigger)
- Handle credential refresh (Service Account uses static JSON key)

---

## 2. DATA ARCHITECTURE

### Source 1 — RDA NocoDB Table (Primary Source)

**Table ID:** `mr1v67cszcklwns`  
**Filter:** `Approval Status` ∈ [Pending, Pending-Elevated] AND `Published Timestamp` = null AND `Client ID` = 'PAK-001'

Fields read from RDA:
| Field | Type | Use |
|-------|------|-----|
| Id | AutoNumber | Row identifier for dedup |
| RDA Record ID | SingleLineText | Sheet column RDA-ID (primary dedup key) |
| RDA Timestamp | DateTime | Sheet column Date |
| Confirmed Response Tier | SingleSelect | Internal context (not written to sheet) |
| Public Response Draft | LongText | Sheet column Proposed (protected field) |
| Approval Status | SingleSelect | Sheet column Status (Pending / Pending-Elevated) |
| Client ID | SingleLineText | Filter condition (PAK-001 only) |
| ALA Record ID | Number | Foreign key to fetch original review |

### Source 2 — ALA NocoDB Table (Enrichment Source)

**Table ID:** `m57efwbtrvwohhr`  
**Fetch:** HTTP GET by ALA Record ID (passed from RDA)

Fields read from ALA:
| Field | Type | Use |
|-------|------|-----|
| Id | AutoNumber | Fetch key |
| Raw Tex (or Raw Text) | LongText | Sheet column Review |
| Star Rating | Number | Sheet column Stars |
| Platform | SingleSelect | Sheet column Platform |
| Ingestion Date | DateTime | Reference (not primary date) |

### Destination — Google Sheet (PAK-001 Response Approvals)

**Sheet ID:** `1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w`  
**Sheet Name:** `Sheet1`  
**Range:** `A:H` (8 columns)

| Column | Header | Type | Source | Protected | Editable |
|--------|--------|------|--------|-----------|----------|
| A | Date | Text (ISO 8601) | RDA Timestamp | NO | NO |
| B | Platform | Text | ALA Platform | NO | NO |
| C | Stars | Number | ALA Star Rating | NO | NO |
| D | Review | LongText | ALA Raw Text | NO | NO |
| E | Proposed | LongText | RDA Public Response Draft | YES | NO |
| F | Edited | LongText | (Human edit) | NO | YES |
| G | Status | SingleSelect | RDA Approval Status | NO | YES |
| H | RDA-ID | Text | RDA Record ID | NO | NO |

**Sheet Protection:**
- Column E (Proposed) — protected from editing by Apps Script. Prevents accidental override of AI draft.
- Columns G (Status) and F (Edited) — editable. Christine updates here.
- Dropdown validation on Column G: Pending / Pending-Elevated / Approved / Edited-Approved / Not Accepted

---

## 3. WORKFLOW ARCHITECTURE

### Data Flow — Per Execution

```
Schedule Trigger (7am UTC)
         ↓
Step 1: Fetch Pending RDA Records (NocoDB GET, filtered)
         ↓
Step 2: Loop through RDA records
         ├─→ Step 3: Fetch ALA record (HTTP GET by ALA Record ID)
         ├─→ Step 4: Join RDA + ALA fields in memory
         ├─→ Step 5: Fetch existing sheet rows (Google Sheets GET, idempotency check)
         ├─→ Step 6: Filter new records (Code Node — RDA ID not in existing sheet)
         ├─→ Step 7: Build sheet row values (Code Node — format 8 columns)
         ├─→ Step 8: Write row to sheet (Google Sheets API append)
         └─→ (continue loop or exit)
         ↓
Step 9: Log completion
         ↓
[End]

[Error Handler]
         ↓
ERR1: Error Trigger
         ↓
ERR2: Build error record + email body
         ↓
ERR3: Send error email to approval_contact_email
```

### Credential Architecture

**OLD (v1.0 — FAILED):** OAuth2 credential `Subtext-CX-GoogleSheets`
- **Failure mode:** n8n 2.4.6 task runner cannot access refreshToken in scheduled mode
- **Error:** "refreshToken is required" at Step 8 (Google Sheets API call)
- **Symptom:** Manual execution works (UI has access to token), scheduled execution fails (task runner isolated)

**NEW (v1.1 — Service Account):** Google Service Account JSON key
- **Key type:** Non-expiring JSON (not OAuth2)
- **Credential method:** Header Auth → raw JSON in n8n credential
- **No refresh cycle:** Static key valid indefinitely (until manually rotated)
- **Task runner compatible:** JSON key accessible in all execution contexts

---

## 4. GOOGLE SERVICE ACCOUNT SETUP (REQUIRED FOR CHAT #77)

### Prerequisites
- Google Cloud project: `solofella-cmh-project`
- Project ID: `806396262251`
- Service Account creation permission

### Step 1 — Create Service Account

```bash
# In Google Cloud Console:
# Navigate to: APIs & Services > Credentials > Create Credentials > Service Account
# Service account name: scx-sheet-sync
# Service account ID: scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com
# Description: "Autonomous Google Sheets write access for SubtextCX approval workflows"
# DO NOT grant Compute Engine or other roles — only Sheets API access
```

### Step 2 — Generate JSON Key

```bash
# In Google Cloud Console:
# Service account > Keys > Add Key > Create new key
# Key type: JSON
# Downloads as: solofella-cmh-project-scx-sheet-sync-key.json
# File contents (example):
{
  "type": "service_account",
  "project_id": "solofella-cmh-project",
  "private_key_id": "key-id-here",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com",
  "client_id": "service-account-number",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/scx-sheet-sync%40solofella-cmh-project.iam.gserviceaccount.com"
}
```

### Step 3 — Share PAK-001 Sheet with Service Account

```bash
# Sheet ID: 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w
# In Google Sheets:
# Share button > Add scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com
# Permission level: Editor
# Uncheck "Notify people"
# Share
```

### Step 4 — Add Service Account Credential to n8n

```
n8n Credentials > New > Google Sheets
Name: Subtext-CX-GoogleSheets-ServiceAccount
Authentication: Service Account (JSON)
JSON key: [Paste entire JSON key from Step 2]
Save
```

### Step 5 — Enable Google Sheets API

```bash
# In Google Cloud Console:
# APIs & Services > Enabled APIs & Services
# Search: Google Sheets API
# If not enabled: Click > Enable
# (Note: Drive API also required if creating new sheets, but not needed for append-only)
```

---

## 5. STEP-BY-STEP DECOMPOSITION — 11 NODES

### STEP 1 — SCHEDULE TRIGGER

**n8n:** Schedule Node

```
Trigger type: Every day
Time: 07:00 (UTC)
Timezone: UTC
Execute: Every execution
```

**Output:** Empty payload (schedule marker only)

---

### STEP 2 — FETCH PENDING RDA RECORDS

**n8n:** NocoDB GET

```
Table: RDA (mr1v67cszcklwns)
Filter: (Approval Status,neq,Published) AND (Published Timestamp,is_empty) AND (Client ID,eq,PAK-001)
Limit: 100 (safe batch size)
Sort: RDA Timestamp (ascending)
```

**Output fields read:**
- Id, RDA Record ID, RDA Timestamp, Confirmed Response Tier, Public Response Draft, Approval Status, Client ID, ALA Record ID

**Validation in Code Node after this step:**
```javascript
const rdaRecords = $input.first().json.list || [];
if (rdaRecords.length === 0) {
  // No new records — halt workflow gracefully
  return []; // Prevents downstream errors
}
// Continue if records exist
return [{ json: { rda_records: rdaRecords } }];
```

---

### STEP 3 — LOOP THROUGH RDA RECORDS (BATCH PROCESSOR)

**n8n:** SplitInBatches Node

```
Input array: {{$json.rda_records}}
Batch size: 1
```

**Purpose:** Process one RDA record at a time. Prevents simultaneous API calls that could cause data loss.

**Output:** Single RDA record per loop iteration

---

### STEP 4 — FETCH ALA RECORD (ENRICHMENT)

**n8n:** NocoDB GET

```
Table: ALA (m57efwbtrvwohhr)
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr/{{$json.ala_record_id}}
Auth: xc-token
```

**Output:** ALA record with Raw Text, Platform, Star Rating

**Error handling:** If ALA record not found (404), log and continue (review text marked as "N/A")

---

### STEP 5 — JOIN RDA + ALA (CODE NODE)

**n8n:** Code Node

```javascript
const rda = $input.first().json; // Current RDA record from loop
const alaData = $("Step 4 - Fetch ALA Record").first().json; // ALA enrichment

const joined = {
  rda_id: rda['RDA Record ID'],
  rda_timestamp: rda['RDA Timestamp'],
  approval_status: rda['Approval Status'],
  public_response_draft: rda['Public Response Draft'],
  // ALA enrichment
  review_text: alaData['Raw Tex'] || alaData['Raw Text'] || 'Review text unavailable',
  platform: alaData['Platform'] || 'Unknown',
  star_rating: alaData['Star Rating'] || 0,
  ala_id: rda['ALA Record ID']
};

return [{ json: joined }];
```

**Output:** Flattened object with all needed fields for sheet write

---

### STEP 6 — FETCH EXISTING SHEET ROWS (IDEMPOTENCY CHECK)

**n8n:** Google Sheets GET

```
Spreadsheet ID: 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w
Range: Sheet1!H:H (Column H = RDA-ID column only)
Auth: Subtext-CX-GoogleSheets-ServiceAccount (NEW credential)
Major dimension: ROWS
```

**Output:** Array of all existing RDA-IDs in sheet (for dedup check)

**Code Node validation after fetch:**
```javascript
const sheetRows = $input.first().json.values || [];
const existingRdaIds = sheetRows.flat(); // Extract RDA IDs from column H

const currentRdaId = $("Step 5 - Join RDA + ALA").first().json.rda_id;
const isNewRecord = !existingRdaIds.includes(currentRdaId);

return [{ json: {
  is_new_record: isNewRecord,
  existing_rda_ids: existingRdaIds,
  current_rda: $("Step 5 - Join RDA + ALA").first().json
} }];
```

---

### STEP 7 — FILTER NEW RECORDS (DETERMINISTIC)

**n8n:** IF Node

```
Condition: {{$json.is_new_record}} === true
TRUE branch: Continue to Step 8 (write)
FALSE branch: Return empty array (skip duplicate)
```

**FALSE branch code:**
```javascript
// Record already in sheet — skip
return [];
```

---

### STEP 8 — BUILD SHEET ROW VALUES (CODE NODE)

**n8n:** Code Node

```javascript
const current = $input.first().json.current_rda;

// Format 8 columns for Sheet1!A:H
const rowValues = [
  // Column A: Date (ISO 8601 from RDA Timestamp)
  current.rda_timestamp ? new Date(current.rda_timestamp).toISOString().split('T')[0] : '',
  
  // Column B: Platform
  current.platform || 'Unknown',
  
  // Column C: Stars
  current.star_rating || '',
  
  // Column D: Review
  current.review_text || '',
  
  // Column E: Proposed (RDA public response draft — protected in sheet)
  current.public_response_draft || '',
  
  // Column F: Edited (blank initially — Christine fills on edit)
  '',
  
  // Column G: Status (Approval Status)
  current.approval_status || 'Pending',
  
  // Column H: RDA-ID
  current.rda_id || ''
];

return [{ json: {
  values: [rowValues], // Google Sheets expects 2D array
  rda_id: current.rda_id
} }];
```

**Output:** Google Sheets API append body

---

### STEP 9 — WRITE ROW TO SHEET (GOOGLE SHEETS APPEND)

**n8n:** Google Sheets API (HTTP POST)

```
Credential: Subtext-CX-GoogleSheets-ServiceAccount
Method: POST
URL: https://sheets.googleapis.com/v4/spreadsheets/1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w/values/Sheet1!A:H:append?valueInputOption=USER_ENTERED

Headers:
  Authorization: Bearer {{$nodeExecutionContext.auth.token}} (auto-managed by credential)
  Content-Type: application/json

Body (RAW):
{
  "values": {{$json.values}}
}

Retry: true
Max retries: 3
Wait between retries: 5000ms
```

**Success response:** `{ "updates": { "updatedRows": 1, "updatedColumns": 8 } }`

**Error handling:** If 403 (permission denied) or 404 (sheet not found), throw error → ERR1 trigger

---

### STEP 10 — MERGE RESULTS (AFTER LOOP)

**n8n:** Code Node (after SplitInBatches loop completes)

```javascript
// Summarize execution
const totalProcessed = $input.all().length;
const successCount = $input.all().filter(item => item.json?.updates?.updatedRows > 0).length;

return [{ json: {
  execution_status: 'complete',
  total_records_processed: totalProcessed,
  successfully_written: successCount,
  completion_timestamp: new Date().toISOString()
} }];
```

---

### STEP 11 — LOG COMPLETION (OPTIONAL)

**n8n:** Set Node

```
Include Other Input Fields: ON
Message: "SCX-Sheet-Sync completed. {{$json.successfully_written}} records written."
```

**Output:** Completion log (no external action)

---

## 6. ERROR HANDLER — 3 NODES

### ERR1 — ERROR TRIGGER

**n8n:** Error Trigger Node

Fires if any node throws:
- "refreshToken is required" (credential failure)
- 403 Forbidden (permission denied)
- 404 Not Found (sheet deleted)
- Network timeout
- NocoDB connection failure

---

### ERR2 — BUILD ERROR RECORD

**n8n:** Code Node

```javascript
const error = $input.first().json;

const errorRecord = {
  error_message: error.message || 'Unknown error',
  error_node: error.node?.name || 'Unknown',
  error_timestamp: new Date().toISOString(),
  workflow_name: 'SCX-Sheet-Sync',
  client_id: 'PAK-001',
  retry_action: error.message.includes('refreshToken') 
    ? 'Check Google Service Account credential in n8n' 
    : 'Investigate sheet permissions or NocoDB connection'
};

return [{ json: errorRecord }];
```

---

### ERR3 — SEND ERROR EMAIL

**n8n:** Email Node

```
To: {{$nodeExecutionContext.approval_contact_email}} 
    (from PAK-001 Client Config, fallback: marellano@solofella.com)
Subject: [ERROR] SCX-Sheet-Sync Workflow Failed — {{$json.error_timestamp}}

Body:
Workflow: SCX-Sheet-Sync
Status: FAILED
Error: {{$json.error_message}}
Failed Node: {{$json.error_node}}
Time: {{$json.error_timestamp}}
Client: PAK-001
Action: {{$json.retry_action}}

Next automatic attempt: Tomorrow at 7am UTC
```

---

## 7. NODE MAP — SCX-Sheet-Sync

```
[01] Schedule Trigger — 7am UTC daily
[02] NocoDB GET — Fetch Pending RDA Records (PAK-001, not published)
[03] IF Node — Records exist?
     |–– YES →
     +–– NO → Return empty (halt gracefully)
[04] SplitInBatches Node — Process one record per loop iteration
[05] NocoDB GET — Fetch ALA Record (review text enrichment)
[06] Code Node — Join RDA + ALA fields
[07] Google Sheets GET — Fetch existing RDA-IDs (idempotency)
[08] Code Node — Build idempotency check + filter logic
[09] IF Node — Record already in sheet?
     |–– YES → Return [] (skip)
     +–– NO →
[10] Code Node — Format 8-column row values
[11] Google Sheets POST — Append row to sheet (returns control to loop)
[12] (Loop continues or exits if no more records)
[13] Code Node — Merge results after loop
[14] Set Node — Log completion message

── Error Handler ─────────────────────────────────────────────
[ERR1] Error Trigger
[ERR2] Code Node — Build error record
[ERR3] Email Node — Send error notification

TOTAL: 14 nodes (11 main + 3 error)
Google Sheets API calls: 2 per record (GET idempotency + POST append)
NocoDB calls: 2 per record (RDA fetch, ALA enrichment)
Batch mode: SplitInBatches = 1 (serial, prevents race conditions)
Task runner compatible: Service Account credential works in all contexts
```

---

## 8. CREDENTIALS + CONFIGURATION

| Item | Value |
|------|-------|
| **Workflow Name** | SCX-Sheet-Sync |
| **Trigger** | Schedule — 07:00 UTC daily |
| **Google Sheets Credential (OLD)** | Subtext-CX-GoogleSheets (OAuth2 — DEPRECATED) |
| **Google Sheets Credential (NEW)** | Subtext-CX-GoogleSheets-ServiceAccount (Service Account JSON) |
| **Service Account Email** | scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com |
| **Service Account Project** | solofella-cmh-project (806396262251) |
| **NocoDB Credential** | xc-token (Header Auth, Name: xc-token) |
| **NocoDB URL (internal)** | http://nocodb:8080 — never localhost |
| **RDA Table ID** | mr1v67cszcklwns |
| **ALA Table ID** | m57efwbtrvwohhr |
| **Client Config Table ID** | m95cmabjfyb94ps |
| **Sheet ID** | 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w |
| **Sheet Range** | Sheet1!A:H |
| **Sheet Columns** | Date, Platform, Stars, Review, Proposed, Edited, Status, RDA-ID |
| **Column E (Proposed)** | Protected from editing — AI draft only |
| **Column G (Status)** | Editable by approver — dropdown validation |
| **Column F (Edited)** | Editable by approver — human edits |
| **Approval Contact Email (PAK-001)** | Placeholder: marellano@solofella.com (update when Christine responds) |
| **Infrastructure** | DigitalOcean n8n-Solofella — NYC3 — Ubuntu 24.04 — 4GB RAM — IP: 161.35.133.49 |
| **Max retries on 5xx errors** | 3 |
| **Wait between retries** | 5000ms |
| **Batch size (SplitInBatches)** | 1 (serial execution — prevents race conditions) |
| **Google Sheets API rate limit** | 60-100 requests/min (safe for daily run with 10-30 records) |

---

## 9. DATA FLOW — FULL CYCLE EXAMPLE

### Example: PAK-001 processes 3 pending RDA records

```
7:00 AM UTC — Schedule fires
   ↓
Step 2: Query RDA table → returns 3 pending records
   {id: 1, rda_record_id: "RDA-20260420-070015-001", ala_record_id: 42, ...}
   {id: 2, rda_record_id: "RDA-20260420-070016-002", ala_record_id: 43, ...}
   {id: 3, rda_record_id: "RDA-20260420-070017-003", ala_record_id: 44, ...}
   ↓
Loop Iteration 1:
   Step 4: Fetch ALA #42 → {review_text: "Great service...", platform: "Google", stars: 5}
   Step 6: Join RDA + ALA → {rda_id: "RDA-...001", review: "Great service...", ...}
   Step 7: GET Sheet Column H → ["RDA-...100", "RDA-...101", "RDA-...102"] (existing)
   Step 8: Check "RDA-...001" in existing? → NO (new record)
   Step 10: Format row → ["2026-04-20", "Google", "5", "Great service...", "Thank you...", "", "Pending", "RDA-...001"]
   Step 11: POST to sheet → Append successful (updateRows: 1)
   ↓
Loop Iteration 2:
   [Same process for RDA #2]
   ↓
Loop Iteration 3:
   [Same process for RDA #3]
   ↓
Loop exits
   ↓
Step 13: Merge results → {total_records_processed: 3, successfully_written: 3}
   ↓
Sheet now contains 3 new rows. Christine sees them in the sheet at ~7:05 AM UTC.
Apps Script onEdit trigger watches for Column G edits.
When Christine updates Status, Apps Script fires → n8n SCX-Sheet-Approval webhook → PATCH RDA NocoDB.
```

---

## 10. KNOWN ISSUES + OPEN ITEMS

### CRITICAL (Chat #77 — In Progress)

| Issue | Status | Impact | Fix |
|-------|--------|--------|-----|
| **OAuth credential failure** | IN PROGRESS | Scheduled execution fails at Step 9 "refreshToken is required" | Implement Google Service Account (JSON key). Chat #77. |
| **Approval Contact Email blank** | Open | Error emails fail (no recipient). PAK-001 Client Config shows placeholder. | Update PAK-001 Client Config with Christine's actual email before next test run. |

### MEDIUM PRIORITY

| Item | Status | Notes |
|------|--------|-------|
| ALA record not found (404) | Handled | If review deleted from ALA, sheet row shows "Review text unavailable". Non-blocking. |
| Sheet column protected state | Configured | Column E (Proposed) is protected. Verified. No changes needed. |
| Google Sheets API rate limits | Low risk | Workflow appends ~10-30 rows/day. Google limit is 60-100 req/min. Safe. |
| EDO-001 sheet integration | Not started | Q2 2026. Will need separate Sheet ID + Service Account email grant. |
| Yelp/OpenTable ingestion | Phase 1b | Not in scope for Chat #77. Focus on Google/TripAdvisor first. |

### PHASE 2 (Post-Pilot)

| Item | Timeline | Notes |
|------|----------|-------|
| Google Business Profile API ingestion | Phase 1b | Auto-populate Column D (Review) from GBP instead of manual CSV. |
| Yelp Fusion API ingestion | Phase 1b | Multi-platform review source automation. |
| Apps Script → Google Sheet approval interface | Phase 2 | Currently manual. Could auto-populate dropdown from RDA Approval Status values. |
| MRA integration | Post-pilot | Append weekly/monthly metrics to client sheet. Separate workflow. |
| EDO-001 + AJI-001 sheet setup | Q2 2026 | Replicate SCX-Sheet-Sync for each client. No code changes — credential + Sheet ID only. |

---

## 11. DEPLOYMENT CHECKLIST — CHAT #77

Before publishing SCX-Sheet-Sync with Service Account credential:

**Pre-deployment (Setup):**
- [ ] Create Google Service Account in solofella-cmh-project
- [ ] Generate JSON key (download, secure locally)
- [ ] Share PAK-001 Response Approvals sheet with service account email (Editor)
- [ ] Verify Google Sheets API enabled in solofella-cmh-project
- [ ] Add Service Account credential to n8n (Subtext-CX-GoogleSheets-ServiceAccount)

**Credential Migration:**
- [ ] Update Step 9 (Google Sheets POST) to use NEW credential
- [ ] Verify old OAuth credential can be safely retired (no other workflows use it)
- [ ] Publish workflow after credential change

**Testing:**
- [ ] Manual Execute: Create 1 test RDA record, run workflow, verify row appended to sheet
- [ ] Manual Execute: Run again, verify no duplicate row (idempotency check)
- [ ] Scheduled Execute: Wait for 7am UTC, verify auto-run completes
- [ ] Error test: Break credential temporarily, verify ERR3 email fires
- [ ] Apps Script test: Edit Status column in sheet, verify webhook fires and RDA updates

**Post-deployment:**
- [ ] Update PAK-001 Client Config Approval Contact Email (from placeholder to Christine's actual email)
- [ ] Document Service Account key location (secure storage — not in GitHub)
- [ ] Add SCX-Sheet-Sync to GitHub (agents/ingestion/ folder)
- [ ] Update MCD v7.5 with new Service Account credential architecture

**First Live Run (April 21, 7am UTC):**
- [ ] Monitor workflow execution
- [ ] Verify ~50 pending RDA records append to sheet
- [ ] Confirm no duplicates (idempotency)
- [ ] Check error log email (should be clean)
- [ ] Have Christine test approval workflow (edit Status, verify RDA updates)

---

## 12. CRITICAL BUILD RULES (FROM CHAT #76 LESSONS)

| Rule | Reason |
|------|--------|
| No OAuth2 in scheduled workflows (n8n 2.4.6) | Task runner cannot access refreshToken. Use Service Account instead. |
| Service Account JSON must be pasted exactly | Double-encryption bug: any formatting change causes 400 error. |
| SplitInBatches = 1 (never >1 in this workflow) | Prevents simultaneous API calls that cause data loss. Serial is safe. |
| Google Sheets GET before POST for idempotency | Dedup by RDA ID prevents duplicate rows on retry. |
| Fallback for missing ALA record (404) | Review might be deleted. Show "N/A" rather than fail. |
| Error handler before publish | Catch failures early. Email approver immediately. |
| No circular dependency with Apps Script | SCX-Sheet-Sync appends. Apps Script edits. One-way flow. |
| Batch size 1 in SplitInBatches | Prevents race conditions in concurrent executions. |

---

## 13. REFERENCE DOCUMENTS

| Document | Purpose | Location |
|----------|---------|----------|
| SCX_HOW_RDA_v3.1+ | RDA workflow (produces records for sheet) | GitHub: agents/RDA/ |
| SCX_HOW_ALA_v4 | ALA workflow (source for review text) | GitHub: agents/ALA/ |
| Dashboard Freelancer Brief | Client UI (reads approved/published records) | GitHub: frontend/ |
| SCX_PreBuild_Protocol_v1.0 | Field traceability standard | GitHub: protocols/ |
| MCD_v7.4+ | Project continuity (updates after each chat) | GitHub: docs/ |

---

## 14. INFRASTRUCTURE

| Component | Value |
|-----------|-------|
| **Droplet** | DigitalOcean n8n-Solofella · NYC3 · Ubuntu 24.04 |
| **IP** | 161.35.133.49 |
| **n8n version** | 2.4.6 (Self-hosted) |
| **n8n URL** | http://161.35.133.49:5678 |
| **NocoDB URL (external)** | http://161.35.133.49:8080 |
| **NocoDB URL (inside n8n)** | http://nocodb:8080 |
| **Docker compose** | v1 hyphen syntax (`docker-compose`, not `docker compose`) |
| **Database** | SQLite at /var/lib/docker/volumes/n8n_n8n_data/_data/database.sqlite |
| **Monthly cost** | ~$63 (DigitalOcean) + Google Cloud API (free tier) |

---

**Subtext CX · SCX_Sheet_Sync_HOW_v1.0 · Chat #76 · April 20, 2026 · Solofella LLC**
