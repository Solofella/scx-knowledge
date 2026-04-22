SCX-Sheet-Sync HOW v1.1
SUBTEXT CX · SOLOFELLA LLC
HOW DOCUMENT — SCX-Sheet-Sync
Google Sheets Approval Workflow Automation
Complete Step Decomposition · Node Logic · Credentials · Data Flow · Protection Strategy
v1.1 · April 22, 2026 · Chat #78

Version History
VersionChangesv1.0Initial build — Chat #76 · April 20, 2026. Complete workflow specification. OAuth credential failure diagnosis. Service Account solution specified. 11 nodes.v1.1Chat #78 · April 22, 2026. Service Account implementation COMPLETE + verified. 14-node architecture (added [07b] Merge, [9b] Rate Limit, [9d] Rename Fields). Google Sheet protection strategy documented (Section 15). Deduplication mechanism locked. Critical debugging lessons from Chat #77-78 transcript integrated. First live sync: 50 PAK-001 RDA records successfully appended. Approval workflow ready for Christine.

PURPOSE: This document specifies every step, node, code block, credential configuration, data field, and sheet protection rule for the SCX-Sheet-Sync (Google Sheets Approval Population) n8n workflow. A developer or Miguel must be able to build, repair, or troubleshoot SCX-Sheet-Sync from this document alone with zero prior context.

Summary Grid
PropertyValueWorkflow NameSCX-Sheet-SyncVersionv1.1 — April 22, 2026PurposePopulate PAK-001 Response Approvals Google Sheet with pending RDA records on scheduled cadenceTrigger TypeSchedule — 7am UTC dailyTrigger CadenceDaily at 07:00 UTCSource DataRDA NocoDB table (pending records only)DestinationGoogle Sheet (PAK-001 Response Approvals)Destination Sheet ID1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6wDestination RangeSheet1!A:L (12 columns)Credential TypeGoogle Service Account (JSON key — non-expiring)Credential Status✅ COMPLETE — scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.comTotal n8n Nodes14 main + error handler (3 nodes) = 17 totalData ModelRich join (RDA + ALA) before sheet writeDeduplicationRDA Record ID lookup in existing sheet rows (Column K)Sheet ProtectionColumns A-F, J-L read-only. Columns G (Status), H (Edited Response), I (Proposed Response reference) editable.Error HandlingError trigger → email notification + logPhase 1 ScopeManual CSV upload → SCX-Sheet-Sync → Google Sheet approval gate → Apps Script webhook → RDA updatePhase 1b ScopeGoogle Business Profile API + Yelp Fusion API ingestion (separate workflow)Active ClientsPAK-001 (Park Avenue Kitchen) — Live + testedFirst Live SyncApril 21, 2026 — 50 records appended, zero duplicates, 100% success rateDoc Versionv1.1 — April 22, 2026

1. AGENT PURPOSE
SCX-Sheet-Sync — Google Sheets Approval Workflow Automation
Automates the daily population of the client's Google Sheet approval interface with pending review response drafts from the RDA NocoDB table. Enriches RDA records with original review text, platform, star rating, and reviewer handle from ALA. Deduplicates against existing sheet rows to prevent duplicate appends on retry. Enables human approvers (e.g., Christine at Park Avenue Kitchen) to review and approve drafted responses in a familiar Google Sheets interface rather than navigating n8n or NocoDB directly.
NOT an approval agent. SCX-Sheet-Sync does not make approval decisions. It surfaces data for human approval. The Apps Script onEdit trigger listens for approval status changes in the sheet and fires a webhook to update RDA NocoDB records.
AUTOMATION BOUNDARY: SCX-Sheet-Sync runs on schedule. Apps Script (approval gate) runs on edit. No circular dependencies.
SCX-Sheet-Sync PRODUCES

Google Sheet rows appended to PAK-001 Response Approvals sheet (12 columns)
Each row contains: SCX Date, Review Date, Platform, Star Rating, Reviewer Handle, Review Text, Proposed Response, Status, Edited Response, ALA Record ID, RDA Record ID, Sync Status
Prevents duplicates via RDA ID deduplication
Error log + email notification on failure
✅ First live run (April 21): 50 records, zero duplicates, 100% append success

SCX-Sheet-Sync DOES NOT

Make approval decisions
Modify NocoDB RDA records (that is Apps Script → n8n webhook)
Filter by approval status post-write (filtering happens pre-write)
Execute on-demand (scheduled only — no manual trigger)
Handle credential refresh (Service Account uses static JSON key)


2. DATA ARCHITECTURE
Source 1 — RDA NocoDB Table (Primary Source)
Table ID: mr1v67cszcklwns
Filter: Approval Status ∈ [Pending, Pending-Elevated] AND Published Timestamp = null AND Client ID = 'PAK-001'
Fields read from RDA:
FieldTypeUseIdAutoNumberRow identifier for dedupRDA Record IDSingleLineTextSheet column K (primary dedup key)RDA TimestampDateTimeSheet column A (SCX Date)Confirmed Response TierSingleSelectInternal context (not written to sheet)Public Response DraftLongTextSheet column G (Proposed Response — protected)Approval StatusSingleSelectSheet column H (Status: Pending / Approved / Edited-Approved / Not Accepted / Published)Client IDSingleLineTextFilter condition (PAK-001 only)ALA Record IDNumberForeign key to fetch original reviewReviewer HandleSingleLineTextFallback if ALA missing (rare)
Source 2 — ALA NocoDB Table (Enrichment Source)
Table ID: m57efwbtrvwohhr
Fetch: HTTP GET by ALA Record ID (passed from RDA)
Fields read from ALA:
FieldTypeUseIdAutoNumberFetch keyRaw TextLongTextSheet column F (Review Text)Star RatingNumberSheet column D (Star Rating)PlatformSingleSelectSheet column C (Platform)Reviewer HandleSingleLineTextSheet column E (Reviewer Handle)Review DateDateTimeSheet column B (Review Date)
Destination — Google Sheet (PAK-001 Response Approvals)
Sheet ID: 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w
Sheet Name: Sheet1
Range: A:L (12 columns)
ColumnHeaderTypeSourceProtectedEditableNotesASCX DateText (ISO 8601)RDA TimestampYESNOAutomated system dateBReview DateText (ISO 8601)ALA Review DateYESNOOriginal guest review dateCPlatformTextALA PlatformYESNOGoogle / Yelp / OpenTable / etc.DStar RatingNumberALA Star RatingYESNO1–5 starsEReviewer HandleTextALA Reviewer HandleYESNOGuest name or pseudonymFReview TextLongTextALA Raw TextYESNOFull original reviewGProposed ResponseLongTextRDA Public Response DraftYESNOAI-drafted response (reference only)HStatusSingleSelectRDA Approval StatusNOYESEDITABLE by approverIEdited ResponseLongText(Human edit)NOYESEDITABLE — Christine pastes modified response hereJALA Record IDTextRDA foreign keyYESNOTraceabilityKRDA Record IDTextRDA Record IDYESNOPrimary dedup keyLSync StatusTextn8n workflowYESNOpending_sync / synced
Sheet Protection Architecture (NEW — Section 15):

Columns A-F, J-L: Read-only (protected from editing)
Column G (Proposed Response): Read-only (reference only, protected from accidental override)
Column H (Status): EDITABLE with dropdown validation (Pending / Approved / Edited-Approved / Not Accepted / Published)
Column I (Edited Response): EDITABLE (Christine pastes modified response here)
Row 1 (Headers): Protected from deletion
Service Account exception: scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com has explicit edit permission on protected ranges A-F and J-L (allows n8n append)


3. WORKFLOW ARCHITECTURE
Data Flow — Per Execution
Schedule Trigger (7am UTC)
         ↓
[01] Fetch Pending RDA Records (NocoDB GET, filtered by Client ID + Approval Status)
         ↓
[02b] Filter Pending Records (Code Node — remove Published records + client mismatch)
         ↓
[03] IF Records Exist (Condition: pageInfo.totalRows > 0)
     FALSE → HALT (no new records)
     TRUE →
         ↓
[04] Fetch ALL ALA Records (HTTP GET with ?limit=100 pagination)
         ↓
[05] Build ALA Map (Code Node — index ALA by Id, extract needed fields)
         ↓
[06] Fetch Existing Sheet Rows (Google Sheets GET Column K — RDA Record IDs)
         ↓
[07] Build Dedup Map (Code Node — extract existing RDA IDs into object)
         ↓
[07b] Merge All Data (Code Node — combine RDA list + ALA map + existing IDs, expand to items)
         ↓
[08] Loop RDA Records (SplitInBatches, batch size 1)
     ├─→ [09] Consolidated Logic (Code Node — dedup check + ALA lookup + fallback)
     ├─→ [9b] Rate Limit Delay (Wait 1.5 seconds — Google Sheets API rate limit protection)
     ├─→ [9d] Rename Fields (Set Node — map flat fields to exact sheet column headers)
     ├─→ [10] Append Row to Sheet (Google Sheets API POST)
     └─→ (loop back to [08] or exit when done)
         ↓
[11] Build Summary (Code Node — count processed records)
         ↓
[12] Error Trigger (if any step throws)
[13] Build Error Email
[14] Send Error Email
         ↓
[End]
Credential Architecture
Service Account (VERIFIED COMPLETE):

Email: scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com
Key type: Non-expiring JSON (not OAuth2)
Credential method: Header Auth in n8n (JSON pasted directly)
Task runner compatible: YES — static key works in all execution contexts (scheduled + manual)
Sheet permissions: Editor access to PAK-001 sheet + explicit edit permission on protected ranges
Status: ✅ LIVE — First successful sync April 21, 2026


4. GOOGLE SERVICE ACCOUNT SETUP (COMPLETE)
✅ COMPLETED STEPS
Step 1 — Service Account Created
Service account name: scx-sheet-sync
Service account ID: scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com
Google Cloud project: solofella-cmh-project (806396262251)
Status: ACTIVE
Step 2 — JSON Key Generated
File: solofella-cmh-project-scx-sheet-sync-key.json
Key type: JSON (non-expiring)
Location: Secure local storage (NOT in GitHub)
Status: ✅ ADDED TO n8n as credential "Subtext-CX-GoogleSheets-ServiceAccount"
Step 3 — PAK-001 Sheet Shared
Sheet ID: 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w
Shared with: scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com
Permission: Editor
Status: ✅ VERIFIED — first append test successful
Step 4 — Protected Range Exception Added
Range A:F "Lock Columns A-F (Read-Only)" → Added service account exception → Can Edit
Range J:L "Lock Columns J-L (Read-Only)" → Added service account exception → Can Edit
Status: ✅ VERIFIED — workflow can append despite protection
Step 5 — Google Sheets API Enabled
Project: solofella-cmh-project
API: Google Sheets API v4
Status: ✅ ENABLED

5. SHEET PROTECTION STRATEGY (NEW — SECTION 15)
PROTECTION RULES — CLIENT-READY
Goal: Lock automated columns (A-F, J-L). Allow client to edit only:

Column H (Status) — dropdown: Pending / Approved / Edited-Approved / Not Accepted / Published
Column I (Edited Response) — freeform text (Christine pastes modified response here)
Column G (Proposed Response) — visible as reference, not editable (prevents accidental override)

SETUP PROCEDURE
STEP 1 — DELETE ALL EXISTING PROTECTIONS

Open sheet
Data → Protected sheets and ranges
Delete all existing protections (start fresh)

STEP 2 — PROTECT COLUMNS A-F (READ-ONLY)

Select range: A:F
Data → Protect sheets and ranges
Name: Lock Columns A-F (Read-Only)
Description: Automated data — cannot edit
Restrict who can edit: Select "Only you"
Click Create
Click "Set Permissions" (modify lock)
Add: scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com
Grant: Can Edit
Save

STEP 3 — PROTECT COLUMNS J-L (READ-ONLY)

Select range: J:L
Data → Protect sheets and ranges
Name: Lock Columns J-L (Read-Only)
Description: Traceability & sync status — cannot edit
Restrict who can edit: Select "Only you"
Click Create
Click "Set Permissions"
Add: scx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com
Grant: Can Edit
Save

STEP 4 — PROTECT ROW 1 (HEADERS)

Select range: 1:1
Data → Protect sheets and ranges
Name: Protect Headers
Restrict who can edit: Select "Only you"
Click Create
(No service account exception needed)

STEP 5 — LEAVE COLUMNS G, H, I UNPROTECTED
DO NOT protect these columns. They remain fully editable by anyone (approvers + service account append).
STEP 6 — ADD DATA VALIDATION TO COLUMN H (OPTIONAL)

Select column H
Data → Data validation
Criteria: List of items
Items: Pending, Approved, Edited-Approved, Not Accepted, Published
Show dropdown list: ON
Appearance: Show warning (or Reject input)
Save


6. STEP-BY-STEP DECOMPOSITION — 14 NODES
[01] SCHEDULE TRIGGER
n8n: Schedule Node
Trigger type: Every day
Time: 07:00 (UTC)
Timezone: UTC
Execute: Every execution
Output: Empty payload (schedule marker only)

[02] FETCH PENDING RDA RECORDS
n8n: NocoDB GET Many
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns
Credential: xc-token (Header Auth, Name: xc-token)
Query: NO filter formula in node
        (filtering happens in [02b] Code Node instead — NocoDB filter formula causes 422 error)
Limit: 100 (safe batch, will filter down further)
Output fields read:

Id, RDA Record ID, RDA Timestamp, Confirmed Response Tier, Public Response Draft, Approval Status, Client ID, ALA Record ID, Reviewer Handle

⚠️ CRITICAL: Do NOT add filter formula in this node. Use Code Node [02b] to filter.

[02b] FILTER PENDING RECORDS (Code Node)
n8n: Code Node
javascriptconst all_input = $input.all();
const filtered = [];
for (let i = 0; i < all_input.length; i++) {
  const rec = all_input[i].json;
  const status = rec['Approval Status'];
  const published = rec['Published Timestamp'];
  const client = rec['Client ID'];
  const status_ok = (status !== 'Published');
  const published_ok = (published === null || published === undefined);
  const client_ok = (client === 'PAK-001');
  if (status_ok && published_ok && client_ok) { 
    filtered.push(rec); 
  }
}
return [{ json: { list: filtered, pageInfo: { totalRows: filtered.length } } }];
Filtering logic:

Removes "Published" records (already sent)
Removes records with non-null Published Timestamp
Keeps only Client ID = 'PAK-001' (pilot records)
Returns: { list: [...], pageInfo: { totalRows: N } }

Output: Array of qualifying RDA records

[03] IF RECORDS EXIST
n8n: IF Node
Condition: {{$json.pageInfo.totalRows}} > 0
TRUE branch: Continue to [04]
FALSE branch: HALT (return empty array)
FALSE branch (Code Node):
javascript// No new records — no work to do
return [];

[04] FETCH ALL ALA RECORDS
n8n: HTTP GET
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr?limit=100
Auth: Set Headers manually
  Header 1: Name: xc-token, Value: {{$nodeExecutionContext.credentials.nocodb_xc_token}}
  (or use saved NocoDB credential if available in HTTP node)
Method: GET
Return all: ON
⚠️ CRITICAL: Add ?limit=100 to URL. NocoDB defaults to 25/page and will return incomplete data.
Output: Array of all ALA records (full enrichment data)

[05] BUILD ALA MAP (Code Node)
n8n: Code Node
javascriptconst ala_records = $input.first().json.list || [];
const ala_map = {};
let ala_total = 0;

for (let i = 0; i < ala_records.length; i++) {
  const rec = ala_records[i];
  const ala_id = rec['Id'] || rec['id'];
  ala_map[ala_id] = {
    review_date: rec['Review Date'] || '',
    platform: rec['Platform'] || 'Unknown',
    star_rating: rec['Star Rating'] || 0,
    reviewer_handle: rec['Reviewer Handle'] || '',
    review_text: rec['Raw Text'] || ''
  };
  ala_total++;
}

return [{ json: { ala_map: ala_map, ala_total: ala_total } }];
Output:
json{
  "ala_map": {
    "42": { "review_date": "2026-04-20", "platform": "Google", ... },
    "43": { "review_date": "2026-04-21", "platform": "Yelp", ... }
  },
  "ala_total": 50
}

[06] FETCH EXISTING SHEET ROWS
n8n: Google Sheets (Get Row(s))
Spreadsheet: 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w (by ID)
Sheet: Sheet1
Range: K:K (Column K only = RDA Record IDs)
Auth: Subtext-CX-GoogleSheets-ServiceAccount
Return All: ON
Always Output Data: ON
Output: Array of existing RDA Record IDs in column K
Note: On first run, returns empty array. Correct behavior.

[07] BUILD DEDUP MAP (Code Node)
n8n: Code Node
javascriptconst sheet_data = $input.first().json.values || [];
const existing_rda_ids = {};

for (let i = 0; i < sheet_data.length; i++) {
  const rda_id = sheet_data[i][0]; // Column K value
  if (rda_id) {
    existing_rda_ids[String(rda_id).trim()] = true;
  }
}

return [{ json: { 
  existing_rda_ids: existing_rda_ids, 
  existing_total: Object.keys(existing_rda_ids).length 
} }];
Output:
json{
  "existing_rda_ids": {
    "RDA-20260420-070015-001": true,
    "RDA-20260420-070016-002": true
  },
  "existing_total": 2
}

[07b] MERGE ALL DATA (Code Node)
n8n: Code Node
javascriptconst step_03 = $('Step 3 - Check Records Exist').first().json;
const step_05 = $('Step 5 - Build ALA Map').first().json;
const step_07 = $input.first().json;

const rda_list = step_03.list || [];
const ala_map = step_05.ala_map || {};
const existing_rda_ids = step_07.existing_rda_ids || {};

const output = [];
for (let i = 0; i < rda_list.length; i++) {
  output.push({ 
    json: { 
      rda_record: rda_list[i], 
      ala_map: ala_map, 
      existing_rda_ids: existing_rda_ids 
    } 
  });
}

return output; // MULTIPLE items (one per RDA record)
Output: Array of objects (one per RDA record), each carrying:

rda_record (single RDA record)
ala_map (entire ALA map)
existing_rda_ids (entire dedup map)

Purpose: Pre-expand RDA list so SplitInBatches [08] can process one at a time with full context.
Named references:

$('Step 3 - Check Records Exist') — Use Step 3 name, NOT Step 2b (Step 3 is the visible continuation)
$input.first() — Step 7 output (current node's input)


[08] LOOP RDA RECORDS (SplitInBatches)
n8n: SplitInBatches Node
Input: {{$json}} (implicit — processes items from [07b])
Batch size: 1
Options > Max iterations: (leave blank or set to 1000)
Output: 
  - Loop output (CONTINUE): connects to [09]
  - Done output (BREAK): connects to [11]
Purpose: Iterate over items from [07b] one per loop. Return to [08] after [10], or exit to [11] when no more items.
⚠️ CRITICAL: Batch size must be 1. Do not increase (prevents race conditions).

[09] CONSOLIDATED LOGIC (Code Node)
n8n: Code Node
javascriptconst input = $input.first().json;
const rda = input.rda_record;
const ala_map = input.ala_map || {};
const existing_rda_ids = input.existing_rda_ids || {};

// Extract RDA Record ID
const rda_record_id = String(rda['RDA Record ID'] || '');

// DEDUP CHECK: Skip if already in sheet
if (existing_rda_ids[rda_record_id]) {
  return []; // Skip this record, loop continues
}

// Extract ALA ID and lookup
const ala_id_raw = rda['ALA Record ID'];
const ala = ala_map[ala_id_raw] 
  || ala_map[String(ala_id_raw)] 
  || ala_map[parseInt(ala_id_raw)] 
  || {}; // Fallback to empty object if not found

// Fallback to RDA Reviewer Handle if ALA missing
const reviewer_handle = ala.reviewer_handle || String(rda['Reviewer Handle'] || '');

return [{ json: {
  scx_date: (rda['RDA Timestamp'] || '').slice(0, 10), // YYYY-MM-DD
  review_date: ala.review_date || '',
  platform: ala.platform || '',
  star_rating: ala.star_rating || '',
  reviewer_handle: reviewer_handle,
  review_text: ala.review_text || '',
  proposed_response: rda['Public Response Draft'] || '',
  status: rda['Approval Status'] || 'Pending',
  edited_response: '', // Blank initially — Christine fills on edit
  ala_record_id: String(ala_id_raw),
  rda_record_id: rda_record_id,
  sync_status: 'pending_sync'
} }];
Logic:

Dedup check: if RDA ID exists in sheet, return empty (skip)
ALA lookup: try 4 key formats (raw, string, int, empty fallback)
Fallback: if ALA missing, use RDA Reviewer Handle
Format all 12 field values for sheet write
Return single record object

Output: Flat object with 12 fields (or empty array if duplicate)

[9b] RATE LIMIT DELAY (Wait Node)
n8n: Wait Node
Execution type: Time
Delay: 1.5 seconds (1500ms)
Purpose: Prevent Google Sheets API rate limit (60-100 requests/min). With ~30 records/run, spacing allows safe margins.
Output: Pass-through (data unchanged, just delayed)

[9d] RENAME FIELDS FOR SHEET (Set Node)
n8n: Set Node
Mode: Keep Only Set Fields

Rename mappings (Input → Output):
  scx_date → "SCX Date"
  review_date → "Review Date"
  platform → "Platform"
  star_rating → "Star Rating"
  reviewer_handle → "Reviewer Handle"
  review_text → "Review Text"
  proposed_response → "Proposed Response"
  status → "Status"
  edited_response → "Edited Response"
  ala_record_id → "ALA Record ID"
  rda_record_id → "RDA Record ID"
  sync_status → "Sync Status"
Output: Object with sheet column names as keys (matches Sheet1 headers exactly)
⚠️ CRITICAL: Case-sensitive, space-sensitive, no trailing spaces.

[10] APPEND ROW TO SHEET
n8n: Google Sheets (Append Row)
Spreadsheet ID: 1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w
Sheet: Sheet1
Credential: Subtext-CX-GoogleSheets-ServiceAccount
Column Mapping Mode: Map Automatically
First Row Contains Headers: ON
Value Input Option: USER_ENTERED

Settings:
  On Error: Continue (don't halt on single append error)
  Retry: ON
  Max Retries: 3
  Wait Between Retries: 5000ms
Input: Flat object from [9d] with 12 fields
Output: Google Sheets API response: { "updates": { "updatedRows": 1, "updatedColumns": 12 } }
After append: Control returns to [08] SplitInBatches loop (continue or exit).

[11] BUILD SUMMARY (Code Node)
n8n: Code Node (after [08] DONE output)
javascriptconst all_results = $input.all();
const success_count = all_results.filter(
  item => item.json?.updates?.updatedRows > 0
).length;

return [{ json: {
  execution_status: 'complete',
  total_records_processed: all_results.length,
  successfully_written: success_count,
  completion_timestamp: new Date().toISOString(),
  workflow_name: 'SCX-Sheet-Sync',
  client_id: 'PAK-001'
} }];
Output: Summary object for logging/email

7. ERROR HANDLER — 3 NODES
[ERR1] ERROR TRIGGER
n8n: Error Trigger Node
Fires if any node throws exception:

"refreshToken is required" (credential failure — should not occur with Service Account)
403 Forbidden (permission denied on sheet)
404 Not Found (sheet deleted)
Network timeout
NocoDB connection failure


[ERR2] BUILD ERROR RECORD (Code Node)
n8n: Code Node
javascriptconst error = $input.first().json;

const errorRecord = {
  error_message: error.message || 'Unknown error',
  error_node: error.node?.name || 'Unknown',
  error_timestamp: new Date().toISOString(),
  workflow_name: 'SCX-Sheet-Sync',
  client_id: 'PAK-001',
  retry_action: 'Investigate sheet permissions, NocoDB connectivity, or Service Account access.'
};

return [{ json: errorRecord }];

[ERR3] SEND ERROR EMAIL
n8n: Email Node (or Brevo API)
To: marellano@solofella.com (fallback; update to Christine's email when available)
Subject: [ERROR] SCX-Sheet-Sync Workflow Failed — {{$json.error_timestamp}}

Body:
Workflow: SCX-Sheet-Sync
Status: FAILED
Error: {{$json.error_message}}
Failed Node: {{$json.error_node}}
Time: {{$json.error_timestamp}}
Client: {{$json.client_id}}
Action: {{$json.retry_action}}

Next automatic attempt: Tomorrow at 7am UTC

8. NODE MAP — SCX-Sheet-Sync v1.1
[01] Schedule Trigger — Daily 07:00 UTC
      ↓
[02] NocoDB GET — Fetch Pending RDA (no filter formula)
      ↓
[02b] Code Node — Filter Pending + Client Check
      ↓
[03] IF Node — Records exist?
      ├─ NO → HALT
      └─ YES ↓
[04] HTTP GET — Fetch ALL ALA Records (with ?limit=100)
      ↓
[05] Code Node — Build ALA Map (index by Id)
      ↓
[06] Google Sheets GET — Fetch Existing RDA IDs (Column K dedup)
      ↓
[07] Code Node — Build Dedup Map
      ↓
[07b] Code Node — Merge All Data (expand RDA list + carry context)
      ↓
[08] SplitInBatches — Loop RDA records (batch size 1)
      ├─ LOOP OUTPUT → [09]
      └─ DONE OUTPUT → [11]
      ↓
[09] Code Node — Consolidated Logic (dedup + ALA lookup + fallback)
      ↓
[9b] Wait Node — Rate limit delay (1.5 seconds)
      ↓
[9d] Set Node — Rename fields to sheet columns
      ↓
[10] Google Sheets POST — Append row to sheet
      ↓ (return to [08] loop or exit to [11])
[08] [continues loop]
      ↓
[11] Code Node — Build summary (count processed records)
      ↓
[END]

── ERROR HANDLER ──────────────────────────
[ERR1] Error Trigger
      ↓
[ERR2] Code Node — Build error record
      ↓
[ERR3] Email Node — Send error email
Total nodes: 14 main + 3 error = 17

9. CREDENTIALS + CONFIGURATION
ItemValueStatusWorkflow NameSCX-Sheet-Syncv1.1TriggerSchedule — 07:00 UTC daily✅ ACTIVEGoogle Sheets Credential (NEW)Subtext-CX-GoogleSheets-ServiceAccount✅ COMPLETEService Account Emailscx-sheet-sync@solofella-cmh-project.iam.gserviceaccount.com✅ LIVEService Account Projectsolofella-cmh-project (806396262251)✅ VERIFIEDNocoDB Credentialxc-token (Header Auth)✅ ACTIVENocoDB URL (internal)http://nocodb:8080✅ VERIFIEDRDA Table IDmr1v67cszcklwns✅ LIVEALA Table IDm57efwbtrvwohhr✅ LIVESheet ID1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w✅ LIVESheet RangeSheet1!A:L (12 columns)✅ CONFIGUREDProtected ColumnsA-F (lock), G (read-only ref), J-L (lock)✅ SET UPEditable ColumnsH (Status), I (Edited Response)✅ READYData Validation (Col H)Pending / Approved / Edited-Approved / Not Accepted / Published✅ OPTIONALApproval Contact Email (PAK-001)marellano@solofella.com (placeholder)⚠️ UPDATE WHEN CHRISTINE CONFIRMSInfrastructureDigitalOcean n8n-Solofella · NYC3 · Ubuntu 24.04 · IP: 161.35.133.49✅ LIVEFirst Live SyncApril 21, 2026 — 50 records appended, 100% success✅ VERIFIED

10. DATA FLOW — FULL CYCLE EXAMPLE (APRIL 21)
Example: PAK-001 processes 50 pending RDA records on first live sync
7:00 AM UTC — Schedule fires
   ↓
[02]: Query RDA table → returns 50 records (not yet filtered)
   ↓
[02b]: Filter → Remove Published + non-PAK-001 → 50 qualify
   ↓
[03]: Check records exist → 50 > 0 → TRUE, continue
   ↓
[04]: Fetch ALA records → HTTP GET ?limit=100 → returns 200+ ALA records
   ↓
[05]: Build ALA map → Index all 200+ by Id → {42: {...}, 43: {...}, ...}
   ↓
[06]: Fetch existing sheet → Column K returns [] (first run, sheet empty)
   ↓
[07]: Build dedup map → {} (no existing IDs)
   ↓
[07b]: Merge all data → Expand 50 RDA records into 50 items, each carrying full ALA map + dedup map
   ↓
[08]: Loop iteration 1–50
   Loop 1:
      [09]: RDA-001 → Check dedup (not in sheet) → Lookup ALA #42 → Success → Format 12 fields
      [9b]: Wait 1.5s
      [9d]: Rename to sheet columns
      [10]: POST append → updateRows: 1 ✓
      (return to [08])
   Loop 2:
      [09]: RDA-002 → Check dedup (not in sheet) → Lookup ALA #43 → Success → Format 12 fields
      [9b]: Wait 1.5s
      [9d]: Rename
      [10]: POST append → updateRows: 1 ✓
      (return to [08])
   [Loop 3–50 continue...]
   [08] DONE → Exit loop to [11]
   ↓
[11]: Merge results → {
  execution_status: 'complete',
  total_records_processed: 50,
  successfully_written: 50,
  completion_timestamp: '2026-04-21T07:05:30.123Z'
}
   ↓
Workflow completes. 50 rows now in PAK-001 sheet.
Christine opens sheet, sees 50 new rows.
She can now review and approve (edit Column H) or edit responses (Column I).
When she edits Status, Apps Script onEdit fires → webhook to RDA NocoDB.

11. KNOWN ISSUES + LESSONS (LOCKED FROM CHAT #77-78)
CRITICAL DEBUGGING LESSONS
LessonRoot CauseFixPreventionOAuth2 credential failure in scheduled moden8n 2.4.6 task runner cannot access refreshToken outside UI contextUse Google Service Account (static JSON key)Always use Service Account for scheduled workflows in n8n 2.4.6NocoDB filter formula → 422 errorn8n node config doesn't validate filter syntax against NocoDB schemaRemove filter formula, use Code Node post-fetchNever use filter formula in n8n NocoDB nodes — always post-fetch filterString().trim() on fields → silent failureNocoDB field read interception causes value lossRead fields directly without conversionAlways read NocoDB fields as-is, no type conversionNamed node references across branches failn8n 2.4.6 execution context limits $('NodeName') scopeUse only visible continuation path names (e.g., Step 3, not Step 2b if Step 3 is next)Test named references in actual branch path before deployingSplitInBatches batch >1 → data lossConcurrent loop iterations cause race conditions in API callsSet batch size to 1 alwaysAlways use batch size 1 for n8n SplitInBatches with API writesNocoDB pagination default 25/pageHTTP GET without limit returns incomplete dataAdd ?limit=100 to URLAlways specify limit in NocoDB HTTP GET callsGoogle Sheets "Map Each Column Manually" → 400 errorColumn name mismatch in manual mappingUse "Map Automatically" + Set Node for field renamingUse Set Node for field renaming, let Google Sheets auto-detect columnsSheet protection prevents appendProtection rule applied without service account exceptionAdd service account to protected range exceptionsAlways add service account to exceptions on protected rangesColumn header case/space sensitivity"Proposed Response" ≠ "proposed_response" ≠ "Proposed response "Match exact headers in Set Node outputUse echo "Column: 'Proposed Response'" to verify exact header textFirst run returns empty sheet rowsCorrect behavior — no existing records yetHandle empty array gracefullyAlways test dedup logic with empty sheet firstALA record lookup fails (404)Review deleted after RDA createdFallback to empty string or "N/A"Implement try-catch in ALA lookup, fall back to RDA fields

12. PHASE 2 OPEN ITEMS
ItemStatusNotesWorkflow Logging to NocoDBOpenCreate workflow_logs NocoDB table (Timestamp, Workflow Name, Status, Client ID, Records Processed, Error Log). Add Step [12] POST after [11] to log every execution.EDO-001 sheet setupPendingReplicate SCX-Sheet-Sync for EDO-001. Requires: separate Google Sheet + Service Account email grant + Client Config record. No code changes.AJI-001 sheet setupExploratoryConfirm pilot terms. Then replicate workflow.Apps Script auto-syncDeferredCurrently manual (Christine edits Status, Apps Script fires webhook). Could auto-sync published records back to Reviews sheet if needed.Dashboard real-time statusPhase 2Dashboard currently static. Add polling to refresh approval status from sheet every 5 minutes.Google Business Profile API ingestionPhase 1bSeparate workflow. Auto-populate Column F (Review Text) from GBP instead of ALA.Yelp Fusion API ingestionPhase 1bSeparate workflow. Auto-populate Columns C-F from Yelp reviews.

13. DEPLOYMENT CHECKLIST — CHAT #77-78 (✅ COMPLETE)
Pre-deployment (Setup)

[✅] Created Google Service Account in solofella-cmh-project
[✅] Generated JSON key (secure storage)
[✅] Shared PAK-001 sheet with service account (Editor)
[✅] Verified Google Sheets API enabled
[✅] Added Service Account credential to n8n

Credential Migration

[✅] Updated [10] (Google Sheets POST) to use new credential
[✅] Verified old OAuth credential deprecated
[✅] Published workflow with new credential

Testing

[✅] Manual Execute: 1 test RDA record → appended successfully
[✅] Manual Execute: Run again → no duplicate (dedup verified)
[✅] Scheduled Execute: Wait for 7am UTC → ✅ auto-run successful
[✅] Error test: Temporarily revoked credential → ERR3 email fired
[✅] Apps Script test: Edit Status → webhook verified → RDA updated
[✅] First live sync: 50 records, 100% success, zero duplicates

Post-deployment

[✅] PAK-001 Client Config Approval Contact Email — placeholder (update when Christine confirms)
[✅] Service Account key location — documented (secure local storage)
[✅] GitHub commit — SCX_Sheet_Sync_HOW_v1.1 (this document)
[⏳] Update MCD to v7.5 (pending Miguel approval)

First Live Run — April 21, 2026

[✅] Workflow executed at 7:00 AM UTC
[✅] 50 pending RDA records appended
[✅] Zero duplicates (idempotency verified)
[✅] All 12 columns populated correctly
[✅] Column protection working (A-F, J-L read-only)
[✅] Error log clean (no errors)
[✅] Christine can view + edit (Status + Edited Response editable)


14. CRITICAL BUILD RULES (FROM CHAT #77-78 TRANSCRIPT)
RuleReasonImpactNo OAuth2 in scheduled workflows (n8n 2.4.6)Task runner isolated, cannot refresh tokensUse Google Service Account onlyService Account JSON must paste exactly as-isJSON encoding strict, any formatting breaks authCopy-paste JSON, no editsSplitInBatches batch size = 1 alwaysPrevents race conditions in concurrent iterationsSerial execution onlyGoogle Sheets GET before POST for idempotencyDedup prevents duplicates on retryAlways fetch existing + filterFallback for missing ALA record (404)Reviews might be deleted post-RDA creationUse RDA Reviewer Handle fallbackError handler before publishCatch failures early, notify immediatelyAlways include [ERR1-3]No circular dependency with Apps ScriptOne-way flow prevents infinite loopsSCX-Sheet-Sync appends only, Apps Script edits onlyBatch size 1 in SplitInBatches (REPEAT)Concurrent API calls lose dataSerial is safe, tested, verifiedAdd ?limit=100 to NocoDB HTTP GETDefault 25/page returns incomplete dataAlways specify limit paramUse Set Node for field renaming, not manualManual mapping causes column mismatch errorsRename in Set, let Sheets auto-detectProtect sheet columns, exempt service accountPrevents accidental edits while allowing appendAlways add service account to exceptions

15. SHEET PROTECTION ARCHITECTURE (COMPLETE)
PROTECTION STRUCTURE (As of April 22, 2026)
ColumnRangeTypeProtectedExceptionEditable ByAA:ASCX DateYESService AccountService Account only (n8n append)BB:BReview DateYESService AccountService Account only (n8n append)CC:CPlatformYESService AccountService Account only (n8n append)DD:DStar RatingYESService AccountService Account only (n8n append)EE:EReviewer HandleYESService AccountService Account only (n8n append)FF:FReview TextYESService AccountService Account only (n8n append)GG:GProposed ResponseYESNoneRead-only (reference only, AI-generated)HH:HStatusNO—Everyone (Christine + service account) — dropdown validationII:IEdited ResponseNO—Everyone (Christine pastes modified response)JJ:JALA Record IDYESService AccountService Account only (n8n append)KK:KRDA Record IDYESService AccountService Account only (n8n append)LL:LSync StatusYESService AccountService Account only (n8n append)11:1Header RowYESNoneOwner only (prevents deletion)
CLIENT WORKFLOW WITH PROTECTION
Daily at ~7:05 AM UTC:

Christine opens PAK-001 Response Approvals sheet
Sees new rows in columns A-F, G, J-L (greyed out, read-only)
Views Column G (Proposed Response) as AI reference
Edits Column H (Status) → dropdown selects: Pending / Approved / Edited-Approved / Not Accepted / Published
If needed, pastes modified response in Column I (Edited Response)
onEdit trigger fires → Apps Script → webhook to RDA NocoDB
RDA record updated with new Status + Edited Response

Sheet remains immutable for automated columns (A-F, J-L).

16. INFRASTRUCTURE
ComponentValueDropletDigitalOcean n8n-Solofella · NYC3 · Ubuntu 24.04IP161.35.133.49n8n version2.4.6 (Self-hosted)n8n URLhttp://161.35.133.49:5678NocoDB URL (external)http://161.35.133.49:8080NocoDB URL (inside n8n)http://nocodb:8080Docker compose syntaxv1 hyphen (docker-compose, not docker compose)DatabaseSQLite at /var/lib/docker/volumes/n8n_n8n_data/_data/database.sqliteMonthly cost~$63 (DigitalOcean) + Google APIs (free tier)Status✅ LIVE — April 21 first sync successful

17. REFERENCE DOCUMENTS
DocumentPurposeLocationStatusSCX_HOW_RDA_v3.1RDA pipeline (produces records for SCX-Sheet-Sync)GitHub: agents/RDA/✅ LIVESCX_HOW_ALA_v4ALA pipeline (source for review enrichment)GitHub: agents/ALA/✅ LIVEDashboard Freelancer BriefClient UI (reads sheet data)GitHub: frontend/✅ COMPLETESCX_PreBuild_Protocol_v1.0Field traceability standardGitHub: protocols/✅ LOCKEDMCD_v7.4Project continuity (pre-update)GitHub: docs/⏳ UPDATE TO v7.5SCX_Sheet_Sync_HOW_v1.1This documentGitHub: agents/ingestion/✅ CURRENT

18. NEXT ACTIONS (AS OF CHAT #78)
ActionOwnerTimelineStatusUpdate MCD to v7.5MiguelImmediate⏳ PENDINGCreate EDO-001 Client ConfigMiguelQ2 2026OpenConfirm AJI-001 pilot termsMiguelQ2 2026ExploratoryUpdate PAK-001 Approval Contact EmailChristine (confirmation)ASAP⏳ PENDING INPUTCreate workflow_logs NocoDB tableMiguel/DevPhase 2PlannedPhase 1b: Google Business Profile APIDevPhase 1bQueuedPhase 1b: Yelp Fusion APIDevPhase 1bQueuedsubtextcx.com landing pageDevBefore outboundPending

Subtext CX · SCX_Sheet_Sync_HOW_v1.1 · Chat #78 · April 22, 2026 · Solofella LLC
STATUS: ✅ PRODUCTION LIVE — First sync verified April 21, 2026. 50 records appended, 100% success rate, zero duplicates. Approval workflow ready for client testing.
