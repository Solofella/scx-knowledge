# SCX-Sheet-Sync HOW v4.0

**VRYOH INTELLIGENCE · SOLOFELLA LLC**
**HOW DOCUMENT — SCX-Sheet-Sync**
**Multi-Client Google Sheets Approval Workflow Automation + Email Reporting**
Complete Step Decomposition · Node Logic · Credentials · Data Flow
v4.0 · August 26, 2026 · Chat #102

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0 | Initial build — Chat #76. Single-client (PAK-001). OAuth credential failure diagnosis. Service Account solution integrated. 11 nodes. |
| v1.1 | Chat #80. Email reporting architecture added. Fixed platform count case-sensitivity bug. Fixed pending calculation (date-based → math-based). 28 nodes. |
| v2.0 | Chat #83–85. Multi-client architecture (PAK-001 + AJI-001–005). Brand renamed Subtext CX → VRYOH Intelligence. 8 defects fixed during AJI 5-location pilot onboarding. ~33 nodes. Confirmed working, Chat #85. |
| v3.0 | Chat #100. Three defects confirmed via real production sheet data (mojibake, closing-line punctuation, T1 opening-pattern deviation) — flagged RDA/ALA-side. Declined an unverified third-party "node audit" document per source-hierarchy rule. |
| **v4.0** | **Chat #102, Aug 26 2026.** Full rebuild from 6 directly-read source documents: HOW v1.1 (template/history) + 5 live n8n JSON export batches covering Steps 1–12f end to end. Documents Google Cloud Service Account migration Solofella→VRYOH (Chat #100–101). Corrects single-client architecture assumptions with confirmed live multi-client structure (per-client Sheet Map, per-client loop, per-client email). Node count, credentials, filters, and email logic all re-verified against live code, not doc-only claims. |

---

**PURPOSE:** This document specifies every step, node, code block, credential configuration, and data field for the SCX-Sheet-Sync (Google Sheets Approval Population) n8n workflow. A developer or Miguel must be able to build or repair SCX-Sheet-Sync from this document alone with zero prior context.

---

## Summary Grid

| Property | Value |
|----------|-------|
| **Workflow Name** | SCX-Sheet-Sync |
| **Purpose** | Populate each active client's Google Sheet with pending RDA response drafts, enriched with ALA review text, on daily schedule |
| **Trigger Type** | Schedule — 5am UTC daily |
| **Trigger Cadence** | Daily at 05:00 UTC |
| **Source Data** | RDA NocoDB table (pending records, all active clients) + ALA NocoDB table (enrichment) + Client Config NocoDB table (routing) |
| **Destination** | One Google Sheet per active client (multi-client, dynamic routing) |
| **Destination Sheet ID(s)** | Resolved dynamically per client from Client Config `Sheet ID` field — not a single fixed ID |
| **Destination Range** | `{tab_name}!A:L` (12 columns, dynamic tab name per client) |
| **Credential Type** | Google Service Account (JSON key — non-expiring) |
| **Credential Status** | ✅ LIVE — `VRYOH-GoogleSheets-ServiceAccount`, migrated from Solofella project Chat #100–101 |
| **Total n8n Nodes** | Not independently recounted node-by-node in this version; confirmed live sequence spans Steps 1–12f across 6 source documents (see §5, §8) |
| **Data Model** | Rich join (RDA + ALA), per-client routing via Client Config, per-client dedup, per-client email |
| **Idempotency** | Fetch existing sheet rows per client, build dedup map from column K (RDA Record ID), filter before append |
| **Error Handling** | Not confirmed live in the 6 documents received this session — see §7, carried as open item |
| **Phase 1 Scope** | Multi-client live sync — PAK-001 + AJI-001–005, 6 clients |
| **Phase 2 Scope** | Not addressed in documents received this session |
| **Active Clients** | PAK-001 (Park Avenue Kitchen), AJI-001–005 (Aji Ceviche Bar, 5 locations) — 6 total |
| **Doc Version** | v4.0 — August 2026 |

---

## 1. AGENT PURPOSE

### SCX-Sheet-Sync — Multi-Client Google Sheets Approval Workflow Automation

Automates the daily population of each active client's Google Sheet approval interface with pending review response drafts from the RDA NocoDB table. Enriches RDA records with original review text from ALA. Deduplicates per-client against each sheet's own existing rows to prevent duplicate appends. Enables human approvers to review and approve drafted responses in a familiar Google Sheets interface rather than navigating n8n or NocoDB directly.

**Relationship to preceding agents:** SCX-Sheet-Sync sits entirely downstream of the ALA→EIP→ESS→HSI→SIA→BRA→RDA pipeline. It consumes RDA's finished, governed response drafts (`Public Response Draft`) and ALA's original review text (`Raw Tex`) — it does not call or trigger any upstream agent, and nothing in the ALA–RDA chain is aware SCX-Sheet-Sync exists.

**Relationship to subsequent process:** SCX-Sheet-Sync produces the human-facing approval surface. The approval decision itself (Approve/Edit/Reject) is meant to flow back to RDA's NocoDB record via an Apps Script `onEdit` trigger → webhook → separate PATCH workflow. That return path is designed but not built (§11).

**NOT an approval agent.** SCX-Sheet-Sync does not make approval decisions. It surfaces data for human approval.

**AUTOMATION BOUNDARY:** SCX-Sheet-Sync runs on a fixed daily schedule only. No webhook exists anywhere in this workflow. It is a **batch process**, not a per-record, event-triggered agent like ALA–RDA.

### SCX-Sheet-Sync PRODUCES

- Google Sheet rows appended to each active client's sheet
- Each row contains 12 columns: SCX Date, Review Date, Platform, Star Rating, Reviewer Handle, Review Text, Proposed Response, Status, Edited Response, ALA Record ID, RDA Record ID, Sync Status
- Prevents duplicates via per-client RDA-ID deduplication
- One daily email summary per active client, with platform breakdown, pending totals, and T3 dignity-risk alerts
- A delivery-status log per email sent (Step 12f)

### SCX-Sheet-Sync DOES NOT

- Make approval decisions
- Modify NocoDB RDA records
- Filter by approval status post-write (filtering happens pre-write)
- Execute on-demand (scheduled only — no manual/webhook trigger exists)
- Handle credential refresh (Service Account uses a static JSON key — see §4)

---

## 2. DATA ARCHITECTURE

### Source 1 — RDA NocoDB Table (Primary Source)

**Table ID:** `mr1v67cszcklwns`
**Fetch (Step 2):** native n8n NocoDB node, `getAll`, `returnAll: true`, sorted by `RDA Timestamp` — no fixed numeric limit (differs from v1.1's `limit=100`).
**Filter (Step 2b, Code node, applied post-fetch):** `Approval Status !== 'Published'` AND `Published Timestamp` is null/undefined AND `Client ID` is present/non-empty. Contains live debug instrumentation (`debug_count`, `debug_first`, full `debug` object) still present in production output.

Fields read from RDA (per Step 2b/7b/9 code):
| Field | Type | Use |
|-------|------|-----|
| Id | AutoNumber | Used as `rda_record_id` for dedup (Step 9) |
| RDA Timestamp | DateTime | Sheet column SCX Date (sliced to first 10 chars) |
| Public Response Draft | LongText | Sheet column Proposed Response (protected field), `\n`-literal-normalized |
| Approval Status | SingleSelect | Sheet column Status |
| Client ID | SingleLineText | Routing key into Client Config map |
| ALA Record ID | Number | Foreign key into ALA map |
| Reviewer Handle | SingleLineText | Fallback source for Sheet column Reviewer Handle if ALA lacks it |
| Published Timestamp | DateTime | Filter condition only |

### Source 2 — ALA NocoDB Table (Enrichment Source)

**Table ID:** `m57efwbtrvwohhr`
**Fetch (Step 4):** raw HTTP GET, `limit=500`, **no WHERE filter, no client scoping** — bulk-fetches up to 500 rows regardless of which are actually referenced by the current pending RDA batch.

Fields read from ALA (Step 5 map-build):
| Field | Type | Use |
|-------|------|-----|
| Id | AutoNumber | Map key (also tried as string/parseInt for type-mismatch tolerance in Step 9) |
| Raw Tex (or Raw Text) | LongText | Sheet column Review Text |
| Platform | SingleSelect | Sheet column Platform |
| Star Rating | Number | Sheet column Star Rating (cast to string) |
| Reviewer Handle | SingleLineText | Sheet column Reviewer Handle |
| Ingestion Date (or Date) | DateTime | Sheet column Review Date |

### Source 3 — Client Config NocoDB Table (Routing Source, NEW since v1.1)

**Table ID:** `m95cmabjfyb94ps`
**Fetch (Step 3b):** raw HTTP GET, `limit=500`.
**Purpose:** Builds `client_sheet_map`, keyed by Client ID, containing `sheet_id`, `tab_name`, `client_name`, `approval_email`. Any client record missing `Sheet ID` or `Sheet Tab Name` is silently skipped from the map (defensive behavior, confirmed live Step 3c).

### Destination — Google Sheet (Per Active Client, Dynamic)

**Sheet ID / Tab Name:** resolved per client at runtime from Client Config, not fixed.
**Range:** `{tab_name}!A:L` (12 columns — expanded from v1.1's 8-column A:H layout).

| Column | Index | Header (per Step 10a build order) | Source | Protected | Editable |
|--------|-------|-----------------------------------|--------|-----------|----------|
| A | 0 | SCX Date | RDA Timestamp (sliced) | NO | NO |
| B | 1 | Review Date | ALA Ingestion Date/Date | NO | NO |
| C | 2 | Platform | ALA Platform | NO | NO |
| D | 3 | Star Rating | ALA Star Rating | NO | NO |
| E | 4 | Reviewer Handle | ALA Reviewer Handle (fallback: RDA) | NO | NO |
| F | 5 | Review Text | ALA Raw Tex/Raw Text | NO | NO |
| G | 6 | Proposed Response | RDA Public Response Draft (`\n`-normalized) | YES | NO |
| H | 7 | Status | RDA Approval Status | NO | YES |
| I | 8 | Edited Response | (Human edit) | NO | YES |
| J | 9 | ALA Record ID | ALA Id | NO | NO |
| K | 10 | RDA Record ID | RDA Id (dedup key — read back by Step 7) | NO | NO |
| L | 11 | Sync Status | Literal `'pending_sync'` | NO | NO |

**Sheet Protection:** consistent with v1.1/v2.0's documented model — Proposed Response protected from editing; Status and Edited Response editable by the human approver. Protected-range allow-lists were updated for all 6 sheets during the Aug 2026 credential migration to reference the new Service Account email.

---

## 3. WORKFLOW ARCHITECTURE

### Data Flow — Per Execution (Confirmed Live, Steps 1–12f)

```
Schedule Trigger (5am UTC)
         ↓
Step 1: Schedule Trigger fires
         ↓
Step 2: Fetch Pending RDA Records1 (native NocoDB node, all clients, sorted by RDA Timestamp)
         ↓
Step 2b: Filter Pending Records (Code — status/published/client checks + debug echo)
         ↓
Step 3: Check Records Exist (IF — pageInfo.totalRows > 0)
         ├─→ TRUE: Step 3b → Fetch Client Config (all active clients)
         │         Step 3c → Build Sheet Map (client_id → sheet_id/tab_name/client_name/approval_email)
         └─→ FALSE: no wire — implicit halt
         ↓
Step 4: Fetch ALA Records (bulk, unfiltered, limit=500)
         ↓
Step 5: Build ALA Map (Code — keyed by Id)
         ↓
Step 6a: Explode Client Sheet Map (Code — fan out one item per client)
         ↓
Step 6-loop: Per Sheet Batch (SplitInBatches — one client at a time)
         ├─→ Step 6: Fetch Existing Sheet Rows (native Google Sheets node, per client,
         │           dynamic documentId/sheetName, Service Account auth)
         │           → loops back to Step 6-loop
         ↓ (after per-client sheet reads complete)
Step 7: Build Dedup Map (Code — extract RDA Record ID from column K of each client's
         existing rows into existing_rda_ids set)
         ↓
Step 7b: Merge All Data (Code — fan out one item per pending RDA record, attaching
         full ala_map, existing_rda_ids, client_id, sheet_id, tab_name)
         ↓
Step 8a: Pre-Loop Platform Counter (Code — computes summary_stats; confirmed reading
         wrong field level, see §8)
         ↓
Step 8: Loop RDA Records (SplitInBatches — one RDA record at a time)
         ├─→ Step 9: Consolidated Logic (Code — dedup check, ALA lookup with
         │           triple-fallback keys, \n-literal fix, builds 12-field row object)
         │           ↓
         │         If (rda_record_id notEmpty?)
         │           ├─→ TRUE:  Step 9b: Rate Limit Delay (Wait, 2 sec)
         │           │          → Step 10a: Build Sheet Payload (Code — manual Sheets
         │           │            API v4 :append REST body, USER_ENTERED + OVERWRITE)
         │           │          → Step 10b: Append Row to Sheet (HTTP POST)
         │           │            → (loop continues)
         │           └─→ FALSE: (skip append — loop continues)
         └─→ (continue loop or exit)
         ↓
Step 11a: Ensure Output (Code — recomputes PER-CLIENT counts: records_processed,
         google/opentable/yelp/tripadvisor/other, t3_count)
         ↓
         ├─→ Step 11: Build Summary (Code — total_records_processed; dead-end/logging only)
         └─→ Step 12: Log Completion (Set — completion_message/workflow_name/execution_timestamp)
                ↓
              Step 12a: Count Platform Records (Code — today/yesterday ISO dates,
              passes through client_counts)
                ↓
              Step 12b1: Fetch Previous Pending RDA (HTTP GET, unfiltered, limit=500 —
              second bulk fetch instance)
                ↓
              Step 12b-filter: Filter Previous Pending (Code — computes total_pending/
              pending_previous_days/new_today PER CLIENT)
                ↓
              Step 12c: Merge Counts Build Email Variables (Code — fans out one item per
              ACTIVE CLIENT; builds full email variable set; hardcodes
              approval_contact_email — see §6)
                ↓
              Step 12d: Build Email Body (Code — full HTML email per client, conditional
              platform lines, conditional T3 block, builds Brevo payload)
                ↓
              Step 12e: HTTP Send Email (POST to Brevo)
                ↓
              Step 12f: Log Email Delivery (Code — delivery-status log; terminal, not
              wired further)
         ↓
[End]

[Error Handler]
         — NOT confirmed present in the 6 documents received this session.
         — v1.1 documented a 3-node Error Trigger → Build Error Record → Send Error
           Email chain; this was NOT visible in any of Batches #1–5. Carried as an
           open item (§11) rather than assumed still present or assumed removed.
```

### Credential Architecture

**OLD (v1.0 — FAILED):** OAuth2 credential `Subtext-CX-GoogleSheets`
- **Failure mode:** n8n 2.4.6 task runner cannot access refreshToken in scheduled mode
- **Error:** "refreshToken is required" at the sheet-append step
- **Symptom:** Manual execution works (UI has token access), scheduled execution fails (task runner isolated)

**SUPERSEDED (v1.1–v3.0):** Google Service Account JSON key, project `solofella-cmh-project`
- Credential name: `Subtext-CX-GoogleSheets-ServiceAccount` (id `Pf4MiR7hQF3eu3ts`)
- Retained, unused, as rollback fallback after the Aug 2026 migration

**CURRENT (v4.0 — confirmed live):** Google Service Account JSON key, project `vryoh-sheet-sync`
- Service Account email: `scx-sheet-sync@vryoh-sheet-sync.iam.gserviceaccount.com`
- GCP project under the `vryoh.com` GCP organization
- n8n credential name: `VRYOH-GoogleSheets-ServiceAccount`, id `g8FQe3X7BBSGCPZy` (confirmed from live JSON export, Batches #2 and #4 — not independently read back verbally during the migration chat itself, so this specific ID is sourced from live code, the higher-trust source per standing hierarchy)
- Scope: `https://www.googleapis.com/auth/spreadsheets` — corrected during migration from an initial `http://` misconfiguration that caused a live 401
- "Set up for use in HTTP Request node" = ON (required specifically for Step 10b's raw HTTP node; native nodes like Step 6 don't need this)
- No refresh cycle: static key valid indefinitely until manually rotated
- Task runner compatible: confirmed working in scheduled execution, Chat #101

---

## 4. GOOGLE SERVICE ACCOUNT SETUP (AS PERFORMED, Chat #100–101)

### Prerequisites (Confirmed)
- Google Cloud organization: `vryoh.com` (pre-existing, Workspace-backed, confirmed by Miguel directly)
- New GCP project created under that org: `vryoh-sheet-sync`

### Step 1 — Create GCP Project

Created under the `vryoh.com` organization (confirmed via console — org shown in project picker before creation).

### Step 2 — Enable Google Sheets API

Enabled via APIs & Services → Library → Google Sheets API → Enable. Confirmed live.

### Step 3 — Create Service Account

Name: `scx-sheet-sync`. Resulting email: `scx-sheet-sync@vryoh-sheet-sync.iam.gserviceaccount.com`. No project-level IAM role granted — access is entirely via per-sheet Google Sheets sharing, not GCP IAM.

### Step 4 — Generate JSON Key — BLOCKER ENCOUNTERED AND RESOLVED

**Blocker:** Org policy `iam.managed.disableServiceAccountKeyCreation` was enforced by default on the new `vryoh.com` org (Google's "Secure by Default" auto-enforcement on new organizations — not a deliberate setting Miguel made). This blocked all Service Account JSON key creation org-wide.

**Note on a near-miss during diagnosis:** a visually similar constraint, `iam.managed.disableServiceAccountApiKeyCreation` (governs API-Key-to-Service-Account bindings, a different feature entirely), was initially found and inspected — this was NOT the blocking constraint. The correct constraint has no "Api" in its name and governs resource type `iam.googleapis.com/ServiceAccountKey` specifically.

**Fix applied:** IAM & Admin → Organization Policies → `iam.managed.disableServiceAccountKeyCreation` → Override parent's policy → Rule 1 → set to **Not enforced** → Set Policy. Applied **organization-wide** (the console view available at the time did not offer a narrower per-project override path). This is a known tradeoff — the security control this policy provides is now off for the entire `vryoh.com` org, not scoped to just this one project.

Key generated successfully after the policy change.

### Step 5 — Share All Active Client Sheets with Service Account

All 6 active client sheets (PAK-001, AJI-001–005) shared with `scx-sheet-sync@vryoh-sheet-sync.iam.gserviceaccount.com` as Editor.

### Step 6 — Update Protected Ranges

Each sheet's Protected Range allow-list updated: old Service Account email (`...@solofella-cmh-project...`) removed, new email added.

### Step 7 — Add/Update Service Account Credential in n8n

```
n8n Credentials > New > Google Service Account
Name: VRYOH-GoogleSheets-ServiceAccount
Service Account Email: scx-sheet-sync@vryoh-sheet-sync.iam.gserviceaccount.com
Private Key: [pasted from downloaded JSON key file]
Set up for use in HTTP Request node: ON
Scope(s): https://www.googleapis.com/auth/spreadsheets
```

Created as a **new** credential rather than editing the old one in place, specifically to preserve rollback capability.

**Sub-blocker encountered during this step:** initial private-key paste attempts failed with `secretOrPrivateKey must be an asymmetric key when using RS256` — root cause was pasting only the Key ID (a short hex string shown in the GCP console) rather than the actual `private_key` field value from the downloaded JSON file, which is a much longer Base64-encoded block including `-----BEGIN PRIVATE KEY-----`/`-----END PRIVATE KEY-----` markers. Resolved once the correct field was located and pasted in full.

**Second sub-blocker:** credential tested successfully in isolation, but Step 10b threw a live 401 UNAUTHENTICATED on first real execution. Root cause: the Scope(s) field had been entered as `http://www.googleapis.com/auth/spreadsheets` (invalid — Google OAuth scopes are always `https://`). Corrected; re-test succeeded end-to-end.

### Step 8 — Attach Credential to Workflow Nodes

Confirmed live in JSON export: Step 6 (native Google Sheets node) and Step 10b (HTTP Request node) both reference credential id `g8FQe3X7BBSGCPZy`, name `VRYOH-GoogleSheets-ServiceAccount`.

### Step 9 — Manual Test Run

Executed manually in n8n; confirmed real row appended correctly to at least one sheet before trusting the unattended 5am schedule.

---

## 5. STEP-BY-STEP DECOMPOSITION — CONFIRMED LIVE NODES

### STEP 1 — SCHEDULE TRIGGER

**n8n:** `n8n-nodes-base.scheduleTrigger`, typeVersion 1.3

```
Trigger type: Interval
triggerAtHour: 5
```

**Output:** Empty payload (schedule marker only).

---

### STEP 2 — FETCH PENDING RDA RECORDS1

**n8n:** native NocoDB node, `n8n-nodes-base.nocoDb`, typeVersion 3

```
Authentication: nocoDbApiToken
Operation: getAll
Project ID: pq249fix22t3ofv
Table: mr1v67cszcklwns
Return All: true
Sort: RDA Timestamp
Credential: xc-token, id Deo2xEE2XImtA7ci
```

**Note:** this credential ID (`Deo2xEE2XImtA7ci`) is distinct from the `DT9tnRgqYpPc3rXo` xc-token credential used by every other raw-HTTP NocoDB call in this same workflow and across the rest of the pipeline. Both are named identically ("xc-token") in the n8n UI. Not resolved as same-underlying-token-or-not in this version — carried as an open item.

---

### STEP 2b — FILTER PENDING RECORDS

**n8n:** Code Node

```javascript
const all_input = $input.all();
const debug_count = all_input.length;
const debug_first = all_input.length > 0 ? all_input[0].json : null;

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
  pageInfo: { totalRows: filtered.length },
  debug: {
    input_count: debug_count,
    filtered_count: filtered.length,
    first_status: debug_first ? debug_first['Approval Status'] : 'none',
    first_published: debug_first ? debug_first['Published Timestamp'] : 'none',
    first_client: debug_first ? debug_first['Client ID'] : 'none'
  }
} }];
```

---

### STEP 3 — CHECK RECORDS EXIST

**n8n:** IF Node, typeVersion 2.3

```
Condition: {{ $json.pageInfo.totalRows }} > 0
TRUE branch → Step 3b
FALSE branch → not wired (implicit halt)
```

---

### STEP 3b — FETCH CLIENT CONFIG

**n8n:** HTTP Request Node

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m95cmabjfyb94ps?limit=500
Authentication: genericCredentialType / httpHeaderAuth
Credential: xc-token, id DT9tnRgqYpPc3rXo
Timeout: 30000ms
```

---

### STEP 3c — BUILD SHEET MAP

**n8n:** Code Node

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

---

### STEP 4 — FETCH ALA RECORDS

**n8n:** HTTP Request Node

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr?limit=500
Authentication: genericCredentialType / httpHeaderAuth
Credential: xc-token, id DT9tnRgqYpPc3rXo
Timeout: 30000ms
```

**No WHERE filter applied.** See §8, item 2.

---

### STEP 5 — BUILD ALA MAP

**n8n:** Code Node

```javascript
const ala_input = $input.first().json;
const ala_records = ala_input.list || [];
const ala_map = {};

for (let i = 0; i < ala_records.length; i++) {
  let ala = ala_records[i];
  ala_map[String(ala['Id'])] = {
    review_text: ala['Raw Tex'] || ala['Raw Text'] || '',
    platform: ala['Platform'] || '',
    star_rating: String(ala['Star Rating'] || ''),
    reviewer_handle: String(ala['Reviewer Handle'] || ''),
    review_date: ala['Ingestion Date'] || ala['Date'] || ''
  };
}

return [{ json: { ala_map: ala_map, ala_total: ala_records.length } }];
```

---

### STEP 6a — EXPLODE CLIENT SHEET MAP

**n8n:** Code Node

```javascript
const mapData = $('STEP 3c — BUILD SHEET MAP').first().json;
const clientSheetMap = mapData.client_sheet_map || {};
const output = [];
const keys = Object.keys(clientSheetMap);

for (let i = 0; i < keys.length; i++) {
  const clientId = keys[i];
  output.push({ json: {
    client_id: clientId,
    sheet_id: clientSheetMap[clientId].sheet_id,
    tab_name: clientSheetMap[clientId].tab_name
  } });
}

return output;
```

---

### STEP 6-loop — PER SHEET BATCH

**n8n:** SplitInBatches, typeVersion 3. Default batch settings (batch size not explicitly overridden in this node's parameters, per live export).

Output 0 (done): not wired in the received batches.
Output 1 (item): → Step 6.

---

### STEP 6 — FETCH EXISTING SHEET ROWS

**n8n:** native Google Sheets node, typeVersion 4.7

```
Authentication: serviceAccount
Document ID: {{ $json.sheet_id }} (dynamic, mode: id)
Sheet Name: {{ $json.tab_name }} (dynamic, mode: name)
alwaysOutputData: true
retryOnFail: true
waitBetweenTries: 5000ms
Credential: googleApi id g8FQe3X7BBSGCPZy, name VRYOH-GoogleSheets-ServiceAccount
```

Loops back to Step 6-loop after each client's sheet is read.

---

### STEP 7 — BUILD DEDUP MAP

**n8n:** Code Node

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

### STEP 7b — MERGE ALL DATA

**n8n:** Code Node

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
  output.push({ json: {
    rda_record: rda,
    ala_map: ala_map,
    existing_rda_ids: existing_rda_ids,
    client_id: clientId,
    sheet_id: sheetInfo.sheet_id || '',
    tab_name: sheetInfo.tab_name || ''
  } });
}

return output;
```

---

### STEP 8a — PRE-LOOP PLATFORM COUNTER

**n8n:** Code Node

```javascript
const items = $input.all();
let googleCount = 0, openTableCount = 0, yelpCount = 0, tripAdvisorCount = 0, otherCount = 0, t3Count = 0;

for (let i = 0; i < items.length; i++) {
  const item = items[i].json;
  if (!item) continue;
  const platform = (item.platform || '').toLowerCase();
  if (platform === 'google') googleCount++;
  else if (platform === 'opentable') openTableCount++;
  else if (platform === 'yelp') yelpCount++;
  else if (platform === 'tripadvisor') tripAdvisorCount++;
  else if (platform !== '') otherCount++;
  const approvalStatus = item['Approval Status'] || item.status || '';
  if (approvalStatus === 'Pending-Elevated') t3Count++;
}

return items.map(item => ({ json: {
  ...item.json,
  summary_stats: {
    total_google: googleCount, total_opentable: openTableCount, total_yelp: yelpCount,
    total_tripadvisor: tripAdvisorCount, total_other: otherCount, total_t3: t3Count,
    total_records: items.length
  }
} }));
```

**Confirmed field-path issue** — see §8, item 1.

---

### STEP 8 — LOOP RDA RECORDS

**n8n:** SplitInBatches. Output 0 (done): not wired in received batches. Output 1 (item) → Step 9.

---

### STEP 9 — CONSOLIDATED LOGIC

**n8n:** Code Node, `alwaysOutputData: true`

```javascript
const input = $input.first().json;
const rda = input.rda_record;
const ala_map = input.ala_map || {};
const existing_rda_ids = input.existing_rda_ids || {};
const rda_record_id = String(rda['Id'] || '');

if (existing_rda_ids[rda_record_id]) {
  return [];
}

const ala_id_raw = rda['ALA Record ID'];
const ala = ala_map[ala_id_raw] || ala_map[String(ala_id_raw)] || ala_map[parseInt(ala_id_raw)] || {};
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

---

### IF (Step 9's gate)

**n8n:** IF Node, typeVersion 2.3

```
Condition: {{ $json.rda_record_id }} notEmpty
TRUE → Step 9b
FALSE → not wired in received batches (implicit skip)
```

---

### STEP 9b — RATE LIMIT DELAY

**n8n:** Wait Node

```
Amount: 2 (seconds)
```

**Purpose:** throttle between Google Sheets API append calls to avoid tripping per-user/per-project write-rate quotas. Positioned after the dedup/empty check, so skipped duplicates never incur the delay — only records about to trigger a real write do.

---

### STEP 10a — BUILD SHEET PAYLOAD

**n8n:** Code Node, `alwaysOutputData: true`

```javascript
const input = $input.first().json;
if (!input || !input.rda_record_id || input.rda_record_id === '') {
  return [];
}
const sheetId = input.sheet_id;
const tabName = encodeURIComponent(input.tab_name);
const range = tabName + '!A:L';
const url = 'https://sheets.googleapis.com/v4/spreadsheets/' + sheetId + '/values/' + range + ':append?valueInputOption=USER_ENTERED&insertDataOption=OVERWRITE';

const rowValues = [
  input.scx_date || '', input.review_date || '', input.platform || '', input.star_rating || '',
  input.reviewer_handle || '', input.review_text || '', input.proposed_response || '',
  input.status || '', input.edited_response || '', input.ala_record_id || '',
  input.rda_record_id || '', input.sync_status || ''
];

const body = JSON.stringify({ values: [rowValues] });

return [{ json: {
  append_url: url, append_body: body,
  client_id: input.client_id || '', rda_record_id: input.rda_record_id || ''
} }];
```

---

### STEP 10b — APPEND ROW TO SHEET

**n8n:** HTTP Request Node, `retryOnFail: true`

```
Method: POST
URL: {{ $json.append_url }}
Authentication: predefinedCredentialType / nodeCredentialType: googleApi
Body: raw, application/json, {{ $json.append_body }}
Timeout: 30000ms
Credentials attached: xc-token (unused/stale), Subtext-CX-Google-OAuth2 (unused/stale),
  Subtext-CX-GoogleSheets (unused/stale), VRYOH-GoogleSheets-ServiceAccount (active)
```

Not wired further in the received batches — presumed to loop back to Step 8 based on documented v1.1/v2.0 architecture; not independently confirmed live in this session's documents.

---

### STEP 11a — ENSURE OUTPUT

**n8n:** Code Node

```javascript
const loopItems = $input.all();
const clientCounts = {};

for (let i = 0; i < loopItems.length; i++) {
  const item = loopItems[i].json;
  if (!item || Object.keys(item).length === 0) continue;
  const clientId = item.client_id || 'UNKNOWN';
  if (!clientCounts[clientId]) {
    clientCounts[clientId] = {
      records_processed: 0, google_count: 0, opentable_count: 0, yelp_count: 0,
      tripadvisor_count: 0, other_count: 0, t3_count: 0
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
  loop_completed: true, client_counts: clientCounts,
  completion_timestamp: new Date().toISOString(), workflow_name: 'SCX-Sheet-Sync'
} }];
```

Fans out to both Step 11 and Step 12.

---

### STEP 11 — BUILD SUMMARY

**n8n:** Code Node, `alwaysOutputData: true`

```javascript
const results = $input.all();
return [{ json: {
  execution_status: 'complete',
  total_records_processed: results.length,
  completion_timestamp: new Date().toISOString(),
  workflow: 'SCX-Sheet-Sync'
} }];
```

Not wired further — dead-end/logging purposes only.

---

### STEP 12 — LOG COMPLETION

**n8n:** Set Node

```
completion_message: "SCX-Sheet-Sync completed. Records appended to sheet."
workflow_name: "SCX-Sheet-Sync"
execution_timestamp: {{new Date().toISOString()}}
Include Other Input Fields: ON
```

---

### STEP 12a — COUNT PLATFORM RECORDS

**n8n:** Code Node

```javascript
const input = $input.first().json;
const clientCounts = input.client_counts || {};
const today = new Date();
const yesterday = new Date(today.getTime() - 86400000);
const date = today.toISOString().split('T')[0];
const prior_date = yesterday.toISOString().split('T')[0];

return [{ json: { date: date, prior_date: prior_date, client_counts: clientCounts } }];
```

---

### STEP 12b1 — FETCH PREVIOUS PENDING RDA

**n8n:** HTTP Request Node

```
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns?limit=500
Authentication: genericCredentialType / httpHeaderAuth
Credential: xc-token, id DT9tnRgqYpPc3rXo
```

Second unfiltered bulk-fetch instance — see §8, item 2.

---

### STEP 12b-filter — FILTER PREVIOUS PENDING

**n8n:** Code Node

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

### STEP 12c — MERGE COUNTS BUILD EMAIL VARIABLES

**n8n:** Code Node

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
  output.push({ json: {
    date: dateData.date, prior_date: dateData.prior_date,
    google_count: counts.google_count || 0, opentable_count: counts.opentable_count || 0,
    yelp_count: counts.yelp_count || 0, tripadvisor_count: counts.tripadvisor_count || 0,
    other_count: counts.other_count || 0, t3_count: counts.t3_count || 0,
    records_processed_today: counts.records_processed || 0,
    pending_previous_days: pending.pending_previous_days || 0,
    total_pending: pending.total_pending || 0,
    approval_contact_email: 'miguel@solofella.com',
    from_email: 'intelligence@vryoh.com',
    client_name: info.client_name || cid, client_id: cid,
    sheet_url: 'https://docs.google.com/spreadsheets/d/' + info.sheet_id + '/edit'
  } });
}

return output;
```

**Confirmed hardcoded recipient** — see §6, §8 item 3.

---

### STEP 12d — BUILD EMAIL BODY

**n8n:** Code Node. Builds full HTML email per client (conditionally-rendered platform lines, conditionally-rendered T3 warning, governance-principle footer line, SLA reminder, Approve/Review button linking to `sheet_url`), then constructs and stringifies a Brevo API payload (`sender`, `to`, `subject`, `htmlContent`).

---

### STEP 12e — HTTP SEND EMAIL

**n8n:** HTTP Request Node

```
Method: POST
URL: https://api.brevo.com/v3/smtp/email
Authentication: genericCredentialType / httpHeaderAuth
Credential: api-key, id uJbXCL9WY5rfhkcv
Headers: Content-Type: application/json
Body: raw, {{$json.brevo_body}}
```

---

### STEP 12f — LOG EMAIL DELIVERY

**n8n:** Code Node

```javascript
const emailResponse = $input.first().json;
const deliveryLog = {
  workflow_name: 'SCX-Sheet-Sync',
  email_sent_at: new Date().toISOString(),
  recipient: emailResponse.recipient_email || 'unknown',
  subject: `VRYOH Intelligence — New Review Responses Ready (${emailResponse.date})`,
  brevo_message_id: emailResponse.messageId || null,
  delivery_status: emailResponse.messageId ? 'sent' : 'failed',
  client_id: emailResponse.client_id || 'UNKNOWN',
  total_pending_count: emailResponse.total_pending || 0
};
return [{ json: deliveryLog }];
```

Terminal — not wired further. Log visible only in n8n's own execution history.

---

## 6. EMAIL REPORTING ARCHITECTURE

### Purpose

Generate one daily email summary per active client with:
- Total new responses appended today (per client)
- Platform breakdown (Google, OpenTable, Yelp, TripAdvisor, Other)
- T3 dignity-risk count (Pending-Elevated records)
- Pending from previous days
- Total pending for review

### Critical Design Decision (carried from v1.1, still true live)

**Original approach (historical, failed):** filter by an `SCX Date` field — RDA's NocoDB table does not have this field.

**Current approach (confirmed live):** math-based calculation, now computed **per client** rather than globally:
- `total_pending[client]` = count of that client's pending RDA records (Step 12b-filter)
- `new_today[client]` = that client's `records_processed` count (Step 11a)
- `pending_previous_days[client]` = `total_pending - new_today`

### Confirmed Hardcoded Recipient

```javascript
approval_contact_email: 'miguel@solofella.com',
from_email: 'intelligence@vryoh.com',
```

Every client's daily email — regardless of that client's real `Approval Contact Email` field value, which is present in `client_sheet_map` but not referenced here — currently sends to the same single address. Diagnosis only, no fix proposed, per Diagnose-Only Rule.

### Email Output Format (Reconstructed from Live Step 12d Template)

```
Daily Review Response Summary
Good morning,
Here is your daily platform reviews summary from [prior_date].

📊 New Responses Added Today:
Google: [n] response(s)
OpenTable: [n] response(s)     (only rendered if > 0)
Yelp: [n] response(s)          (only rendered if > 0)
TripAdvisor: [n] response(s)   (only rendered if > 0)
Total new responses: [records_processed_today]

⚠️ [n] require elevated attention (T3 — dignity-risk signals)   (only rendered if t3_count > 0)

📋 [n] pending from previous days (still awaiting approval)

Total Pending Your Review: [total_pending]

VRYOH Intelligence detects and interprets signals only. Operational decisions remain with your team.

👉 Review & Approve Responses → [sheet_url]

⏱️ SLA Reminder: 48-hour approval window from first notification

VRYOH Intelligence by Solofella LLC
[client_name] ([client_id])
Sent: [date] at 05:00 UTC
```

### Confirmed Behavior (Live Code Logic)

Per-client fan-out confirmed at Step 12c: one item per key in `client_sheet_map`, meaning **one email is built and sent per active client**, each with that client's own counts — not a single combined email across all clients.

---

## 7. ERROR HANDLER

**STATUS: NOT CONFIRMED PRESENT IN LIVE WORKFLOW.**

v1.1 documented a 3-node error handler (Error Trigger → Build Error Record → Send Error Email). None of the 5 live JSON export batches received this session (Steps 1 through 12f, covering the entire main workflow body) contained any node of type `n8n-nodes-base.errorTrigger`, nor any node resembling ERR1/ERR2/ERR3.

Per verification protocol: this is not asserted as "removed" or "still present" — it is unconfirmed either way from the documents available. If the error handler still exists as a separate branch not captured in these 5 batches (e.g., attached at the workflow level rather than the main node chain), it would not show up in a Steps-1-through-12f-scoped export. Flagged as an open item (§11) requiring direct confirmation.

---

## 8. NODE MAP — SCX-Sheet-Sync v4.0

```
[01] Schedule Trigger — 5am UTC daily
[02] NocoDB native GET — Fetch Pending RDA Records1 (all clients, sort by RDA Timestamp)
[02b] Code — Filter Pending Records (status/published/client + debug echo)
[03] IF — Records exist? (pageInfo.totalRows > 0)
     |── TRUE →
     +── FALSE → not wired (implicit halt)
[03b] HTTP GET — Fetch Client Config (limit=500)
[03c] Code — Build Sheet Map (client_id → sheet_id/tab_name/client_name/approval_email)
[04] HTTP GET — Fetch ALA Records (limit=500, unfiltered)
[05] Code — Build ALA Map (keyed by Id)
[06a] Code — Explode Client Sheet Map (fan out per client)
[06-loop] SplitInBatches — Per Sheet Batch
[06] Google Sheets native GET — Fetch Existing Sheet Rows (per client, dynamic)
[07] Code — Build Dedup Map (from column K / index 10)
[07b] Code — Merge All Data (fan out per pending RDA record)
[08a] Code — Pre-Loop Platform Counter (confirmed field-path issue)
[08] SplitInBatches — Loop RDA Records
[09] Code — Consolidated Logic (dedup check, ALA lookup, \n fix, row build)
[If] IF — rda_record_id notEmpty?
     |── TRUE →
     +── FALSE → not wired (implicit skip)
[09b] Wait (2 sec) — Rate Limit Delay
[10a] Code — Build Sheet Payload (manual Sheets API v4 append body)
[10b] HTTP POST — Append Row to Sheet
[11a] Code — Ensure Output (per-client counts)
[11] Code — Build Summary (dead-end/logging)
[12] Set — Log Completion
[12a] Code — Count Platform Records (today/yesterday dates)
[12b1] HTTP GET — Fetch Previous Pending RDA (limit=500, unfiltered — 2nd instance)
[12b-filter] Code — Filter Previous Pending (per-client math)
[12c] Code — Merge Counts Build Email Variables (per-client fan-out, hardcoded recipient)
[12d] Code — Build Email Body (per-client HTML + Brevo payload)
[12e] HTTP POST — Send Email (Brevo)
[12f] Code — Log Email Delivery (terminal)

── Error Handler ─────────────────────────────────────────────
NOT CONFIRMED PRESENT — see §7

TOTAL NODES: not independently recounted as a single figure in this version;
  full sequence spans the above list across Steps 1–12f.
Google Sheets API calls: 1 GET per client (idempotency, Step 6) + 1 POST per new record (Step 10b)
NocoDB calls: 1 (RDA fetch) + 1 (Client Config) + 1 (ALA bulk) + 1 (previous-pending re-fetch, Step 12b1)
Batch mode: SplitInBatches used at Step 6-loop (per client) and Step 8 (per record)
Task runner compatible: Service Account credential confirmed working in scheduled execution
Email frequency: Daily at 5am UTC, one email per active client
```

---

## 9. CREDENTIALS + CONFIGURATION

| Item | Value |
|------|-------|
| **Workflow Name** | SCX-Sheet-Sync |
| **Trigger** | Schedule — 05:00 UTC daily |
| **Google Sheets Credential (retired, OAuth2)** | Subtext-CX-GoogleSheets (DEPRECATED, v1.0 era) |
| **Google Sheets Credential (rollback, Solofella)** | Subtext-CX-GoogleSheets-ServiceAccount, id Pf4MiR7hQF3eu3ts — intact, unused |
| **Google Sheets Credential (ACTIVE, VRYOH)** | VRYOH-GoogleSheets-ServiceAccount, id g8FQe3X7BBSGCPZy |
| **Service Account Email (active)** | scx-sheet-sync@vryoh-sheet-sync.iam.gserviceaccount.com |
| **Service Account Project (active)** | vryoh-sheet-sync, under vryoh.com GCP org |
| **NocoDB Credential (Step 2 only)** | xc-token, id Deo2xEE2XImtA7ci — distinct ID from all other NocoDB calls in this workflow |
| **NocoDB Credential (all other steps)** | xc-token, id DT9tnRgqYpPc3rXo — same as rest of pipeline |
| **NocoDB URL (internal)** | http://nocodb:8080 — never localhost |
| **RDA Table ID** | mr1v67cszcklwns |
| **ALA Table ID** | m57efwbtrvwohhr |
| **Client Config Table ID** | m95cmabjfyb94ps |
| **Sheet ID(s)** | Dynamic per client, resolved from Client Config |
| **Sheet Range** | {tab_name}!A:L (12 columns) |
| **Sheet Columns** | SCX Date, Review Date, Platform, Star Rating, Reviewer Handle, Review Text, Proposed Response, Status, Edited Response, ALA Record ID, RDA Record ID, Sync Status |
| **Column G (Proposed Response)** | Protected from editing — AI draft only |
| **Column H (Status)** | Editable by approver |
| **Column I (Edited Response)** | Editable by approver |
| **Approval Contact Email (email sender logic)** | Hardcoded: miguel@solofella.com — NOT read from Client Config despite field being populated (§6) |
| **Brevo Credential** | api-key, id uJbXCL9WY5rfhkcv |
| **Brevo Sender** | intelligence@vryoh.com |
| **Infrastructure** | DigitalOcean — NYC3 — Ubuntu 24.04 — 4GB RAM — IP: 161.35.133.49 |
| **Wait between sheet appends** | 2000ms (Step 9b) |
| **Timeout, Client Config / ALA fetches** | 30000ms |
| **Google Sheets read retry (Step 6)** | retryOnFail: true, waitBetweenTries: 5000ms |
| **Google Sheets write retry (Step 10b)** | retryOnFail: true |

---

## 10. DATA FLOW — FULL CYCLE EXAMPLE

### Example: 2 active clients, one with 2 pending records, one with 1

```
5:00 AM UTC — Schedule fires
   ↓
Step 2/2b: Query + filter RDA table → returns 3 pending records across 2 clients
   {id: 5568, client_id: "AJI-001", ala_record_id: 9012, ...}
   {id: 5569, client_id: "AJI-001", ala_record_id: 9013, ...}
   {id: 5570, client_id: "PAK-001", ala_record_id: 9014, ...}
   ↓
Step 3/3b/3c: Confirm records exist → fetch Client Config → build sheet map for
   AJI-001 and PAK-001 (sheet_id/tab_name/client_name/approval_email each)
   ↓
Step 4/5: Fetch ALA bulk (limit=500) → build ala_map keyed by Id
   ↓
Step 6a/6-loop/6: Explode into 2 client items → loop → for each, fetch that
   client's existing sheet rows via Service Account
   ↓
Step 7/7b: Build combined dedup set from both clients' existing rows →
   merge into 3 per-record items, each carrying its own client_id/sheet_id/tab_name
   ↓
Step 8a: Pre-loop counter runs (field-path issue — produces summary_stats that
   Step 9's actual row-build logic does not consume)
   ↓
Step 8 loop, iteration 1 (record 5568):
   Step 9: dedup check passes (new) → ALA lookup finds #9012 → builds row object
   If: rda_record_id notEmpty → TRUE
   Step 9b: wait 2 sec
   Step 10a: builds append URL for AJI-001's sheet_id/tab_name, 12-value row array
   Step 10b: POST → append succeeds
   ↓
Step 8 loop, iteration 2 (record 5569): same process, same AJI-001 sheet
   ↓
Step 8 loop, iteration 3 (record 5570): same process, PAK-001's own sheet_id/tab_name
   ↓
Loop exits
   ↓
Step 11a: per-client counts — AJI-001: records_processed=2, PAK-001: records_processed=1
   ↓
Step 12a/12b1/12b-filter: fetch all pending RDA again → compute per-client
   total_pending and pending_previous_days
   ↓
Step 12c: fan out 2 items (one per client in client_sheet_map) — both hardcoded
   to approval_contact_email: 'miguel@solofella.com'
   ↓
Step 12d/12e: build + send 2 separate emails, one per client, to the same
   single recipient address
   ↓
Step 12f: log 2 delivery records

Sheets now contain the new rows. Human approvers open their respective sheets
and update Status/Edited Response. Apps Script feedback loop (Sheet → RDA
write-back) is NOT built — see §11.
```

---

## 11. KNOWN ISSUES + OPEN ITEMS

### CONFIRMED LIVE THIS SESSION (Chat #102)

| Issue | Status | Impact | Confirmed Via |
|-------|--------|--------|----------------|
| **Step 8a platform/T3 counter reads wrong field level** | Open | `item.platform`/`item['Approval Status']` don't exist at the level Step 7b actually outputs; counts computed here are not the ones that matter downstream (Step 11a recomputes correctly per-client from the loop's real output) | Batch #3, direct code read |
| **Step 4 and Step 12b1 unfiltered bulk fetches** | Open | `limit=500`, no WHERE/date/client filter on either — scalability ceiling as ALA/RDA tables grow | Batches #2, #5 |
| **Hardcoded email recipient** | Open | Every client's daily email sent to `miguel@solofella.com`, not each client's real `Approval Contact Email` | Batch #5, Step 12c |
| **Step 8/Step 6-loop "done" branches not wired in received batches** | Unconfirmed (gap, not asserted defect) | Cannot confirm live how the workflow converges after either loop's completion | Batches #3, #4 — absent from connections blocks provided |
| **Debug instrumentation in Step 2b** | Open | `debug_count`/`debug_first`/`debug` object still returned in production output | Batch #1 |
| **Multiple stale credentials on Step 10b** | Open, cosmetic/cleanup | xc-token, Subtext-CX-Google-OAuth2, Subtext-CX-GoogleSheets all attached but unused given `predefinedCredentialType`/`googleApi` auth mode | Batch #4 |
| **Error Handler not confirmed present** | Unconfirmed | v1.1 documented 3 nodes (ERR1/2/3); none appeared in any of the 5 live batches covering the full main chain | Batches #1–5, absence |
| **Two differently-keyed "xc-token" NocoDB credentials** | Open | Step 2 uses id `Deo2xEE2XImtA7ci`; every other NocoDB call in this workflow uses id `DT9tnRgqYpPc3rXo` — same display name, unconfirmed whether same underlying token | Batches #1–5 |

### CARRIED FROM v3.0 — NOT RE-VERIFIED THIS SESSION

| Issue | Status | Notes |
|-------|--------|-------|
| Mojibake in Review Text (`[5‚òÖ]`) | Not re-checked | ALA-side, last confirmed via real sheet data Chat #100 |
| T1 opening-pattern deviation | Not re-checked | RDA-side, last confirmed via real sheet data Chat #100 |
| Closing-line punctuation collision (`Cheers!,`) | Not re-checked | RDA-side, last confirmed via real sheet data Chat #100 |

### NOT YET BUILT

| Item | Status | Notes |
|------|--------|-------|
| Apps Script `onEdit` → webhook → RDA PATCH feedback loop | Not built | Confirmed absent on all 6 sheets (Miguel, Chat #101) |

---

## 12. REFERENCE DOCUMENTS

| Document | Purpose | Location |
|----------|---------|----------|
| SCX-Sheet-Sync HOW v1.1 | Prior version, template baseline for this document | GitHub: Google Sheets/HOW DOCUMENTS/ |
| SCX-Sheet-Sync HOW v3.0 | Prior version, real-data defect findings | This chat history, Chat #100 |
| RDA HOW (referenced, not fetched this session) | Produces records this workflow syncs | GitHub: agents/RDA/ (not re-fetched) |
| ALA HOW (referenced, not fetched this session) | Source for review text | GitHub: agents/ALA/ (not re-fetched) |

---

## 13. INFRASTRUCTURE

| Component | Value |
|-----------|-------|
| **Droplet** | DigitalOcean — NYC3 — Ubuntu 24.04 |
| **IP** | 161.35.133.49 |
| **n8n version** | 2.4.6 (Self-hosted) |
| **NocoDB URL (internal)** | http://nocodb:8080 |
| **Base ID** | pq249fix22t3ofv |

---

**VRYOH Intelligence · SCX_Sheet_Sync_HOW_v4.0 · Chat #102 · August 26, 2026 · Solofella LLC**
