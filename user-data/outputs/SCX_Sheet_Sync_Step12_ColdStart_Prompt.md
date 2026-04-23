# Cold Start Prompt — SCX-Sheet-Sync Chat
## Implement Step 12: Notify Email (7:05am Daily)

---

**Date:** April 21, 2026  
**Chat:** SCX-Sheet-Sync (dedicated workflow chat)  
**Task:** Add Step 12 (Notify Email) to SCX-Sheet-Sync workflow  
**Context:** SubtextCX / Solofella LLC / PAK-001 pilot  

---

## WHAT YOU NEED TO DO

Add 5 new nodes to the existing SCX-Sheet-Sync workflow. These nodes will send a daily email notification to Christine (PAK-001 approver) at 7:05am UTC after the sheet population completes.

The email summarizes:
- How many new review responses were added (by platform: Google, OpenTable, Yelp)
- How many require elevated attention (T3 — dignity-risk)
- How many are still pending from previous days
- Total pending records awaiting approval
- Direct link to the Google Sheet
- 48-hour SLA reminder

---

## CURRENT SCX-SHEET-SYNC STATE

You already have build 11 steps (14 nodes) in that SCX-Sheet-Sync workflow

---

## NEW NODES TO ADD (Step 12 through Step 12d)

After the loop completes and all records are appended to the sheet, add these 5 nodes:

### NODE 12 — Set Node (Log Completion)

**Node Title:** `Step 12 - Log Completion`  
**Node Type:** Set Node  
**Wire FROM:** Step 11 (Google Sheets POST)  
**Wire TO:** Step 12a AND Step 12b (parallel)

**Configuration:**
```
Include Other Input Fields: ON

Add field:
  Name: completion_message
  Value: SCX-Sheet-Sync completed. Records appended to sheet.
```

---

### NODE 12a — Code Node (Count Platform Records)

**Node Title:** `Step 12a - Count Platform Records`  
**Node Type:** Code Node  
**Wire FROM:** Step 12 (Log Completion)  
**Wire TO:** Step 12c (Merge counts)

**Code:**

```javascript
const rdaRecords = $input.all().json;

if (!rdaRecords || rdaRecords.length === 0) {
  return [{ json: {
    date: new Date().toISOString().split('T')[0],
    prior_date: new Date(Date.now() - 86400000).toISOString().split('T')[0],
    google_count: 0,
    openTable_count: 0,
    yelp_count: 0,
    t3_count: 0,
    records_processed: 0
  } }];
}

let googleCount = 0;
let openTableCount = 0;
let yelpCount = 0;

for (let i = 0; i < rdaRecords.length; i++) {
  const platform = rdaRecords[i].platform || 'Unknown';
  if (platform === 'Google') {
    googleCount++;
  } else if (platform === 'OpenTable') {
    openTableCount++;
  } else if (platform === 'Yelp') {
    yelpCount++;
  }
}

let t3Count = 0;
for (let i = 0; i < rdaRecords.length; i++) {
  if (rdaRecords[i].approval_status === 'Pending-Elevated') {
    t3Count++;
  }
}

const today = new Date();
const yesterday = new Date(today.getTime() - 86400000);
const dateToday = today.toISOString().split('T')[0];
const datePrior = yesterday.toISOString().split('T')[0];

return [{ json: {
  date: dateToday,
  prior_date: datePrior,
  google_count: googleCount,
  openTable_count: openTableCount,
  yelp_count: yelpCount,
  t3_count: t3Count,
  records_processed: rdaRecords.length
} }];
```

**Purpose:** Counts how many records were added in THIS execution, broken down by platform. Also counts T3 (dignity-risk) records.

---

### NODE 12b — HTTP Request (Fetch Previous Days Pending)

**Node Title:** `Step 12b - Fetch Previous Days Pending`  
**Node Type:** HTTP Request  
**Wire FROM:** Step 12 (Log Completion) — **PARALLEL to 12a**  
**Wire TO:** Step 12c (Merge counts)

**Configuration:**

```
Method: GET
URL: http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mr1v67cszcklwns

Query Parameters (add each as separate parameter):
  where: (RDA Timestamp,lt,{{new Date(Date.now() - 86400000).toISOString().split('T')[0]}}T00:00:00Z)~and(Approval Status,in,Pending,Pending-Elevated)~and(Client ID,eq,PAK-001)~and(Published Timestamp,is,null)
  limit: 100
  sort: -RDA Timestamp

Authentication: None

Headers (add these):
  Name: xc-token
  Value: [Your NocoDB xc-token credential value]

Response Format: JSON
```

**Purpose:** Queries RDA table for records created BEFORE today that are still pending (not yet approved/published). This gives us the count of "old pending" records.

---

### NODE 12c — Code Node (Merge Counts + Build Email Variables)

**Node Title:** `Step 12c - Merge Counts Build Email Variables`  
**Node Type:** Code Node  
**Wire FROM:** Step 12a AND Step 12b (receives input from BOTH)  
**Wire TO:** Step 12d (Send email)

**Code:**

```javascript
const current = $("Step 12a - Count Platform Records").first().json;
const previousData = $("Step 12b - Fetch Previous Days Pending").first().json;

const pendingPreviousDays = previousData.pageInfo?.totalRows || 0;

const totalPending = current.records_processed + pendingPreviousDays;

const approvalContactEmail = 'marellano@solofella.com';

const emailVariables = {
  date: current.date,
  prior_date: current.prior_date,
  google_count: current.google_count,
  openTable_count: current.openTable_count,
  yelp_count: current.yelp_count,
  t3_count: current.t3_count,
  pending_previous_days: pendingPreviousDays,
  total_pending: totalPending,
  approval_contact_email: approvalContactEmail,
  sheet_url: 'https://docs.google.com/spreadsheets/d/1JT0jt_l4NthR2n3Eu0-DUcWkhsMLV6mKaVJ5FFgIU6w/edit#gid=0',
  client_name: 'Park Avenue Kitchen by David Burke',
  client_id: 'PAK-001'
};

return [{ json: emailVariables }];
```

**Purpose:** Merges the platform counts from Step 12a with the previous-days-pending count from Step 12b. Calculates total pending. Builds complete object with all email template variables.

---

### NODE 12d — Email Node (Send Notify Email)

**Node Title:** `Step 12d - Send Notify Email`  
**Node Type:** Email (Send Email)  
**Wire FROM:** Step 12c (Merge Counts Build Email Variables)  
**Wire TO:** (End — no more nodes after this)

**Configuration:**

```
Credential: Subtext-CX-Email (or your configured email credential)

From Email: noreply@solofella.com
To Email: {{$json.approval_contact_email}}

Subject: 
SubtextCX - New Review Responses Ready for Approval — {{$json.date}}

Email Type: HTML

Email Body: [paste the HTML below]
```

**Email Body HTML Template:**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
    .summary { background-color: #f3f4f6; padding: 16px; border-radius: 8px; margin: 16px 0; }
    .summary-item { margin: 8px 0; }
    .button { display: inline-block; background-color: #3b82f6; color: white; padding: 12px 24px; text-decoration: none; border-radius: 6px; margin: 16px 0; }
    .sla { background-color: #fef3c7; padding: 12px; border-left: 4px solid #f59e0b; margin: 16px 0; }
    .footer { color: #6b7280; font-size: 12px; margin-top: 24px; border-top: 1px solid #e5e7eb; padding-top: 16px; }
  </style>
</head>
<body>

<p>Hi Christine,</p>

<p>Here is your daily platform reviews list from yesterday <strong>{{$json.prior_date}}</strong>.</p>

<div class="summary">
  <h3>📊 Summary:</h3>
  
  {{#if $json.google_count}}
  {{#if $json.google_count > 0}}
  <div class="summary-item">- {{$json.google_count}} new response(s) added today (Google)</div>
  {{/if}}
  {{/if}}
  
  {{#if $json.openTable_count}}
  {{#if $json.openTable_count > 0}}
  <div class="summary-item">- {{$json.openTable_count}} new response(s) added today (OpenTable)</div>
  {{/if}}
  {{/if}}
  
  {{#if $json.yelp_count}}
  {{#if $json.yelp_count > 0}}
  <div class="summary-item">- {{$json.yelp_count}} new response(s) added today (Yelp)</div>
  {{/if}}
  {{/if}}
  
  <div class="summary-item">- {{$json.t3_count}} require elevated attention (T3 — dignity-risk)</div>
  <div class="summary-item">- {{$json.pending_previous_days}} pending from previous days (still awaiting approval)</div>
  
  <hr style="margin: 12px 0; border: none; border-top: 1px solid #d1d5db;">
  <div class="summary-item" style="font-weight: bold;">Total pending your review: {{$json.total_pending}}</div>
</div>

<p>SubtextCX detects and interprets signals only. Operational decisions remain with your team.</p>

<a href="{{$json.sheet_url}}" class="button">👉 Review & Approve Here</a>

<div class="sla">
  <strong>⏱️ SLA: 48 hours from now</strong>
</div>

<p>Questions? Reply to this email.</p>

<div class="footer">
  <p>SubtextCX by Solofella LLC</p>
  <p>{{$json.client_name}} ({{$json.client_id}})</p>
</div>

</body>
</html>
```

**Purpose:** Sends the daily notification email to Christine with all platform counts, pending totals, and sheet link.

---

## WIRING SUMMARY

```
Step 11 (Google Sheets POST)
         ↓
Step 12 (Set Node - Log Completion)
         ↓
         ├──────────────┬──────────────┐
         ↓              ↓              
   Step 12a        Step 12b            
   Code Node       HTTP Request        
   Count           Fetch Previous      
   Platforms       Pending             
         ↓              ↓              
         └──────┬───────┘              
                ↓                      
           Step 12c                    
           Code Node                   
           Merge Counts                
                ↓                      
           Step 12d                    
           Email Node                  
           Send Email                  
                ↓                      
              [END]                    
```

**Key wiring notes:**
- Step 12 connects to BOTH Step 12a and Step 12b (parallel paths)
- Step 12a and Step 12b BOTH connect to Step 12c (merge point)
- Step 12c connects to Step 12d (final email send)
- Step 12d is the end of the workflow

---

## TESTING AFTER BUILD

After you add these 5 nodes:

1. **Manual Execute:** Click "Execute Workflow" in n8n
2. **Verify counts:** Check Step 12a output (should show google_count, openTable_count, etc.)
3. **Verify query:** Check Step 12b output (should show pending records from previous days)
4. **Verify merge:** Check Step 12c output (should show all variables including total_pending)
5. **Check email:** Look in marellano@solofella.com inbox for test email
6. **Verify email content:**
   - Subject line correct
   - Platform counts show only if > 0
   - Total pending is sum of new + old
   - Sheet link works
   - Governance note present

---

## VARIABLES REFERENCE (Email Template)

| Variable | Source | Example |
|----------|--------|---------|
| date | Step 12a | 2026-04-21 |
| prior_date | Step 12a | 2026-04-20 |
| google_count | Step 12a | 3 |
| openTable_count | Step 12a | 2 |
| yelp_count | Step 12a | 0 |
| t3_count | Step 12a | 3 |
| pending_previous_days | Step 12b → 12c | 8 |
| total_pending | Step 12c | 13 |
| approval_contact_email | Step 12c | marellano@solofella.com |
| sheet_url | Step 12c | https://docs.google.com/... |
| client_name | Step 12c | Park Avenue Kitchen... |
| client_id | Step 12c | PAK-001 |

---

## EXPECTED BEHAVIOR

**Every day at 7:00 AM UTC:**
1. SCX-Sheet-Sync fires
2. Steps 1-11 append new records to Google Sheet
3. Step 12 logs completion
4. Step 12a counts platforms from current execution
5. Step 12b queries old pending records
6. Step 12c merges both counts
7. Step 12d sends email to marellano@solofella.com

**Email arrives at ~7:05 AM UTC** with:
- "X new responses from Google, Y from OpenTable, Z from Yelp"
- "N require elevated attention (T3)"
- "M pending from previous days"
- "Total pending: X+M"
- Link to sheet
- 48-hour SLA reminder

---

## IMPORTANT NOTES

- **Test recipient:** marellano@solofella.com (for testing)
- **Production recipient:** Will change to Christine's actual email when provided
- **Email fires every day** even if 0 new records (email will show "0 new responses")
- **Handlebars conditionals:** `{{#if google_count > 0}}` ensures platforms only show if count > 0
- **Governance note:** "SubtextCX detects and interprets signals only" is LOCKED in every email

---

## READY TO BUILD?

You now have everything needed to add Step 12 (Notify Email) to SCX-Sheet-Sync. Build nodes 12, 12a, 12b, 12c, and 12d in that order, wire them as shown, test the workflow, and verify the email arrives.

After successful test, publish the workflow and wait for tomorrow's 7am UTC scheduled run.

---

**End of Cold Start Prompt**
