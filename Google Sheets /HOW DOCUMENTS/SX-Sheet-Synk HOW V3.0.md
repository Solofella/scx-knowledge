# SCX-Sheet-Sync HOW v3.0

**VRYOH INTELLIGENCE · SOLOFELLA LLC**
**HOW DOCUMENT — SCX-Sheet-Sync**
Multi-Client Google Sheets Approval Workflow + Internal Reporting
v3.0 · July 25, 2026 · Chat #100

---

## Version History

| Version | Changes |
|---|---|
| v1.0 | Chat #76. Single-client (PAK-001). OAuth failure diagnosis → Service Account solution. |
| v1.1 | Chat #80. Email reporting added. Single-client only. |
| v2.0 | Chat #83–85. Full multi-client architecture. Brand renamed to VRYOH Intelligence. 8 defects fixed during AJI 5-location pilot onboarding. ✅ CONFIRMED WORKING 100%, Chat #85. |
| **v3.0** | **Chat #100, Jul 25 2026.** Adds three defects confirmed via direct inspection of real production sheet rows (not yet present in v2.0's defect log). Everything else carried forward from v2.0 unchanged and unverified beyond what v2.0 itself already stated. **Does NOT incorporate a separate unverified "node-by-node audit" document received this session — see Section 9.** |

---

## 1. STATUS OF THIS DOCUMENT — READ FIRST

v2.0 (Chat #85) is the last **directly verified** state of this workflow — it was built and confirmed working against the live n8n instance.

Since then, one additional artifact was received this session: a long document claiming to be a "direct code read" of the live workflow, listing 30 nodes, specific credential IDs, and 11 alleged defects (broken counters, unfiltered fetches, duplicate credentials, a vestigial status check, etc.). That document is **not used as a source of fact in this v3.0** because:

- It was never independently confirmed against the actual live n8n workflow in this chat.
- It was written in instruction form (telling me what conclusions to reach and what document to produce), not as a passive technical record.
- Per the standing verification protocol, claims without direct observation or explicit user confirmation get flagged unverified, not folded into a "confirmed" document.

What **is** used below: v2.0's own content (already verified Chat #85), plus three findings from real sheet data you pasted directly in this chat, which I directly observed.

---

## 2. AGENT PURPOSE (carried from v2.0, unchanged)

Populates each client's dedicated Google Sheet with pending review response drafts from RDA, enriched with original review text from ALA, routed by Client ID via Client Config. Deduplicates per-client against each sheet's own existing rows. Sends a daily internal-only summary email.

**Not an approval agent.** Surfaces data for human approval; does not decide approvals. Feedback loop (Sheet edit → RDA write-back) is designed but not built (v2.0 Section 14, unchanged).

---

## 3. SCHEDULE & TRIGGER (carried from v2.0)

- n8n Schedule Trigger, 5am UTC daily.
- No webhook. Purely schedule-driven, batch process — not a per-record agent like ALA–RDA.

---

## 4. INPUT SOURCES (carried from v2.0)

| Source | Table ID | Role |
|---|---|---|
| RDA | `mr1v67cszcklwns` | Pending/pending-elevated records to sync |
| ALA | `m57efwbtrvwohhr` | Original review text, enrichment |
| Client Config | `m95cmabjfyb94ps` | Routing: Sheet ID, Tab Name, Client Name, Approval Contact Email |

Filter (Step 2b, per v2.0 code): `Approval Status ≠ 'Published'` AND `Published Timestamp` empty AND `Client ID` not empty.

**⚠️ Flag, not yet confirmed:** RDA's own schema is reported elsewhere (per your RDA-side work) to have simplified Approval Status to 4 values, with `'Published'` no longer a possible value. If accurate, the `≠ 'Published'` check would be a permanent no-op — harmless, since the other two conditions still do the real filtering, but worth confirming directly against the live RDA schema rather than assuming.

---

## 5. NODE ARCHITECTURE (carried from v2.0's confirmed structure)

```
Schedule Trigger (5am UTC)
  → Fetch Pending RDA Records
  → Filter (Status/Published/Client checks)
  → IF Records Exist?
      → Fetch Client Config
      → Build Sheet Map (client_id → sheet_id/tab_name/client_name/approval_email)
  → Fetch ALA Records (bulk)
  → Build ALA Lookup Map (by Id)
  → Explode Client Sheet Map (fan out, 1 item per client)
  → SplitInBatches(1) — per-client loop [CRITICAL: forces single-item resolution
     for Google Sheets node's document/sheet-ID expression evaluation]
      → Fetch Existing Sheet Rows (native Google Sheets node, per client)
      → loop back
  → Build Combined Dedup Map (existing RDA IDs across all sheets)
  → Merge RDA + ALA + dedup + client map → per-record items
  → SplitInBatches(1) — main per-record loop
      → Consolidated Logic:
          - dedup check (skip if RDA ID already in sheet)
          - ALA lookup by ID
          - line-break normalization (\n literal fix, esp. Spanish drafts)
          - build 12-column row
      → IF rda_record_id not empty?
          TRUE  → Rate Limit Delay (2s) → Build Sheet Payload (OVERWRITE mode)
                  → HTTP Append to Sheet → loop back
          FALSE → loop back directly (skip append — prevents the silent
                  full-halt defect fixed in v2.0)
  → Per-Client Platform Counts
  → Build Summary / Log Completion
  → Fetch Fresh Pending RDA (for pending-count calc)
  → Per-Client Pending Calculation
  → Per-Client Email Variables (recipient hardcoded to miguel@solofella.com —
     see Section 8)
  → Build Email Body (VRYOH branding, per client)
  → Send via Brevo
  → Log Email Delivery

Error Handler: Error Trigger → Build Error Record → Send Error Email
```

---

## 6. DEDUP STRATEGY (carried from v2.0)

Two-layer: (1) per-client sheet read compares against `RDA Record ID` column already present in that sheet; (2) an IF node immediately after the row-builder skips forward silently rather than crashing on an empty/duplicate item — this was the fix for v2.0's Defect #7 (silent full-execution halt on first duplicate).

---

## 7. GOOGLE SHEETS ACCESS (carried from v2.0)

- Auth: Service Account (`Subtext-CX-GoogleSheets-ServiceAccount`, id `Pf4MiR7hQF3eu3ts`) — deliberate choice over OAuth2, which doesn't survive scheduled/task-runner execution reliably.
- **Confirmed this session, Chat #100:** credential tested successfully; "Set up for use in HTTP Request node" = ON; scope `spreadsheets` present (with a harmless duplicate scope-string entry noted, not a functional issue).
- Append mode: `insertDataOption=OVERWRITE` (not `INSERT_ROWS`) — preserves pre-formatted cell styling (v2.0 Defect #5).
- Human approvers: Editor + Protected Ranges — can only edit Status / Edited Response columns.

---

## 8. DAILY DIGEST EMAIL (carried from v2.0, one flag added)

Sent via Brevo, once daily, to `miguel@solofella.com` only — **explicitly documented in v2.0 as intentional, operator-only routing**, not client-facing. Client-facing reporting is MRA's separate responsibility (24hr summary + weekly brief).

**⚠️ Flag, not re-litigated here:** a separate document this session suggested the hardcoded recipient might be a bug rather than by design, since the underlying variable is built from the same object as each client's real `Approval Contact Email` field. v2.0's own text is explicit that this is intentional. I'm not overriding v2.0's direct statement based on an unverified claim — but if you want this genuinely re-confirmed against the live node, that's a one-line check worth doing.

---

## 9. CONFIRMED FINDINGS — FROM REAL PRODUCTION DATA (Chat #100, directly observed)

These three are the only new findings in this document that I'm treating as fact, because you pasted the actual sheet rows and I read them directly.

| # | Finding | Evidence | Likely Owner |
|---|---|---|---|
| 1 | **Mojibake in Review Text.** Both real rows show `[5‚òÖ]` instead of a clean rating marker. | Directly visible in both pasted rows. | ALA-side (ingestion/encoding), surfaces here |
| 2 | **Closing-line punctuation collision.** One row ends `Cheers!,` — comma appended directly after `!`. | Directly visible in row 2. Row 1's `Warmly,` has no punctuation to collide with, so it doesn't show the bug — consistent with the bug triggering only when the closing phrase already ends in punctuation. | RDA-side (Step 9d-equivalent closing logic), surfaces here |
| 3 | **T1 opening pattern deviation.** Both rows open `"[Name], thank you so much..."` rather than a `"Thank you very much, [Name],"` / `"We appreciate you, [Name],"` pattern. | Directly visible, 2 for 2 in both pasted rows. | RDA-side (prompt/system instruction), surfaces here |

All three are **RDA/ALA-side issues that surface visibly in Sheet-Sync's output** — not defects in Sheet-Sync's own logic. Sheet-Sync is correctly displaying what upstream agents produced.

---

## 10. UNVERIFIED — NOT CARRIED INTO THIS DOCUMENT AS FACT

A separate document received this session claimed a detailed 30-node walkthrough with specific alleged defects (broken platform/T3 counters always returning 0, unfiltered `limit=100` fetches on two nodes, two differently-keyed "xc-token" credentials, stale unused OAuth credentials on one node, leftover debug code, an unconfirmed trigger connection on the ALA fetch node). 

None of this is included above as confirmed. If you want any of it in the official record, the direct route is the same one that worked for RDA and BRA: paste the actual node code or credential list from the live n8n instance, and I'll verify and document it the same way.

---

## 11. OPEN ITEMS / PENDING

1. Confirm whether RDA's Approval Status truly no longer includes `'Published'` (affects Section 4's flag).
2. Confirm intent behind the hardcoded `miguel@solofella.com` recipient — re-confirm against the live node if you want it beyond v2.0's existing statement.
3. Mojibake fix — needs ALA-side investigation (source CSV encoding, most likely), not a Sheet-Sync fix.
4. Closing punctuation bug — needs RDA-side fix (strip trailing punctuation from closing phrase before appending comma, or let each closing carry its own terminal punctuation).
5. T1 opening-pattern deviation — needs an RDA-side decision: relax the prompt to match observed natural phrasing, or add an audit-checklist item to actually enforce the literal required pattern.
6. Apps Script approval feedback loop — still not built (unchanged from v2.0 Section 14).
7. Any of the Section 10 unverified claims — pending direct confirmation from the live workflow if you want them formally investigated.

---

**VRYOH Intelligence · SCX_Sheet_Sync_HOW_v3.0 · Chat #100 · July 25, 2026 · Solofella LLC**
