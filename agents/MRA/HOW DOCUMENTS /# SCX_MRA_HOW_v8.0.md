**Source:** v7.0 (read in full), plus everything you shared through Oct 1. Step 9 is confirmed live from your Oct 1 paste. I haven't re-checked the rest against a fresh workflow export.

**Three things I found while rewriting (diagnosis only, nothing fixed):**
1. The new approval-rate and SEO-on-approved-drafts numbers may never reach their destination. Steps 10a, 12 and 13 rebuild their output field by field, and none was updated in our chats.
2. Step 9 checks for the status "Modified", but an older note says "Modify". A wrong spelling would quietly inflate the approval rate.
3. The monthly window, as coded, skips the last day of the month. Monthly is off today.

---

# SCX_MRA_HOW_v8.0

**Agent:** MRA (Metrics & Reporting Agent)
**Version:** 8.0
**Compiled:** Chat #115 · October 6, 2026
**Built from:** v7.0 (Aug 22 full read of the live workflow) plus changes confirmed in Chats #108–#114
**Model:** Pure JavaScript. No AI calls, zero token cost.
**Node count:** 46, counted from the Aug 22 export (2 disabled). Steps 9 and 11 were edited afterward, with no nodes added or removed that I know of. This replaces the old "30 vs 38" question.
**Status:**
- **Daily:** one email per brand, running on the 8:00 AM Eastern schedule. The Oct 1 runs show it with client contacts as recipients.
- **Weekly:** per location. It runs and writes to NocoDB, but **email sending is paused**.
- **Monthly:** per location, trigger **disabled**.

**Changes since v7.0:**
- Signal-tier fix applied (Step 11): all six signal types are now counted.
- Dead "Published" metrics replaced. Step 9 now reports **approval rate**. SLA and speed metrics are retired by design.
- SEO scan now reads Approved drafts.
- EIP now outputs domain labels in English only.
- Daily window behavior documented: it reports the previous calendar day.
- Weekly/monthly direction: a per-client dashboard was chosen (separate chat).
- `parent_account_id` now holds the brand's display name.
- Step 13 passthrough claim from v7.0 corrected (§10).

---

## 1. Purpose and Role in the Pipeline

MRA is the last agent in the pipeline (ALA → EIP → ESS → HSI → SIA → BRA → RDA → MRA). It reads what the other agents produced and turns it into reports the client sees.

**What it reads:**
- **ALA (Step 6):** review counts, ratings, platforms. Filtered by the review's posting date (M/D/YY).
- **RDA (Step 8):** draft tiers and approval status. The statuses are Pending, Approved, Not Accepted and Modified (spelling check in §16). Per RDA HOW v7.1, users change status in their Google Sheet and an Apps Script sends it back to NocoDB (Sync Status column = "synced"). RDA has no approval timestamp, by design, so MRA does not measure approval speed.
- **SIA (Step 10b):** weekly/monthly signal clusters, read by named reference. SIA is also the source of the signal-tier rules that MRA's daily branch now copies.
- **EIP (Step 10d):** daily only. Per-review signal type and domain. Domain labels are English only (EIP fix, confirmed by your test and the SIA table).
- **ESS (Step 10c):** expression mode. It is computed but not shown in the daily email.
- **Client Config:** brand and location identity, recipients, Sheet link, grouping.

**Downstream:** no agent reads MRA. Output goes to Brevo emails and the NocoDB MRA table. Only weekly/monthly rows are saved to that table. Daily is not stored anywhere except the email itself. A planned dashboard will become the first reader (§13).

**Cadences:**
- **Daily:** one consolidated email per brand, covering all of its locations.
- **Weekly:** Monday brief, one per location. Sending paused (§13).
- **Monthly:** 1st-of-month report, one per location. Disabled.

---

## 2. Architectural Principles

1. **Filter by review date, never by processing time.** Step 9 uses ALA's `Date` field, so reports reflect when guests posted, not when the pipeline ran.
2. **Brand → location nested loop.** Client Config rows are grouped by `parent_account_id`. An outer loop goes through brands and an inner loop goes through each brand's locations. All three report types share it.
3. **One trigger, all clients.** One run processes every client.
4. **Measure quality, not speed.** "Approved" means the user confirmed the draft's quality and message. MRA tracks approval, not how fast a response goes out.

SCX-Sheet-Sync (5 AM UTC) shares NocoDB tables with MRA but has no execution link in either direction.

---

## 3. Triggers and Report Type

| Node | Schedule | Status |
|---|---|---|
| 1a Daily | Daily, hour 8 | Active |
| 1b Weekly | Mondays, hour 9 | Active (sends paused, §13) |
| 1c Monthly | 1st of month, hour 10 | **Disabled** |
| 2a / 2b / 2c | Set `report_type` to daily / weekly / monthly | 2c disabled |

Run records show 12:00 UTC, which is 8:00 AM Eastern, so schedule hours are Eastern.

---

## 4. Window Calculation (Step 3 and Step 9)

Step 3 sets the window and generates `mra_run_id` (`MRA-YYYYMMDD-HHMMSS`):

```javascript
daily:   start = now − 1 day,  end = now
weekly:  start = now − 7 days, end = now
monthly: start = 1st of last month, end = 1ms before 1st of this month
```

**What it actually covers:** Step 9 turns both ends into midnight and counts a review only if its date is on or after the start and before the end.

| Report | Covers |
|---|---|
| Daily | The full calendar day **before** the run date |
| Weekly | The 7 full calendar days before the run date |
| Monthly | Intended: all of last month. As coded, it stops one day early (§16). |

- **Same-day reviews never appear in same-day reports.** A review dated today shows up tomorrow.
- **Oct 1 incident:** Step 9 returned zeros because the test reviews were dated the run day. You confirmed it was a data-entry mistake, not a bug.
- **The Aug 28 PAK test** (6 same-day reviews, zero shown) fits the same cause, but this was not separately confirmed.
- **Testing rule:** date test reviews yesterday.

**Unresolved:** SIA's monthly window is a rolling 30 days, while MRA's is a calendar month. The label and the SIA data may cover different dates. This is moot while monthly is off.

---

## 5. Brand/Location Loop (Steps 4–4d)

Business driver: pilot feedback that five separate daily emails per brand were too much. The rebuild sends one email per brand, for any number of locations, with no data mixing between brands. Metrics and the approval mechanism are unchanged.

```
Step 4  Fetch Client Config (all rows, limit 100)
 → 4a   Group rows by parent_account_id (an ungrouped location is its own brand)
 → 4b   Outer loop: one pass per brand
    → 4c  One item per location, tagged is_first_in_account,
          report_type, parent_account_id, seo_keywords_raw
    → 4d  Inner loop: one pass per location. Reset = {{ $json.is_first_in_account }}
          Loop branch → Steps 5–13 → back to 4d
          Done branch → 16e-gate (§6)
```

**Why the Reset setting matters:** n8n's batch loop remembers its position for the whole run and does not notice a new batch from the outer loop. Without the reset, the second brand got zero iterations and inherited the first brand's leftovers. Tagging the first location of each brand and feeding that tag into Reset clears the state once per brand. Confirmed working with a 1-location and a 5-location brand.

**Grouping key:** `parent_account_id` now holds the brand's display name (e.g. "Aji Ceviche Bar", "Park Ave Kitchen", as seen in the August data). The daily email uses it as the account name. Every location of a brand must carry exactly the same text, because a typo or extra space would split the brand into two emails.

n8n state is isolated per run. Separate runs (8 AM and 9 AM) can never affect each other, and the earlier suspicion was ruled out.

---

## 6. Report-Type Gate (`16e-gate`)

The inner loop's Done branch is shared by all report types, so it needs a gate. Otherwise weekly runs would also send empty daily emails.

```
4d Done → 16e-gate (report_type = daily?)
    yes → 16e → 17a → 17d → 18 → 19 → 19b → If → 19c → 4b
    no  → 19c → 4b   (weekly/monthly emails were already built per location)
```

The gate's "no" wire to 19c was confirmed working by the Chat #106 weekly test. The batch export does not show it, because the export leaves out wires that cross batch boundaries.

---

## 7. `16e` — Daily Brand Totals

Runs once per brand, after all its locations finish.

```javascript
// Which brand is current? Take the latest entry from Step 4c's history.
const all4c = $('Step 4c ...').all().map(i => i.json);
const currentAccountId = all4c[all4c.length - 1].parent_account_id;
// Allow-list that brand's client_ids, then keep only those in Step 4d's history
...
// Sums: reviews, rating (weighted by review count), signals, pending
```

- **Why the filter:** `.all()` on any n8n node returns its whole run history, not just the current pass. Without the filter, one brand's locations leak into another's email.
- **"Drafts generated":** computed as the sum of the three signal counts. It is complete now that the tier fix is applied.
- **`13b - Accumulate Location Result`:** an earlier attempt at the same isolation problem. It is still wired in and writes to static data, but nothing reads it. It is harmless and purposeless (§16).

---

## 8. Step 11 — Signals, SEO, Expression

**Daily branch** reads EIP records for the window. FIX 3 is **applied**, and the branch now uses SIA's exact mapping:

| Signal Type | Tier |
|---|---|
| Dignity-Risk, Negative, Masked Negative | T-NEGATIVE |
| Ambiguous Negative, Mixed | T-AMBIGUOUS |
| Positive (and anything else) | T-POSITIVE |

```javascript
function getSignalTier(signal_type) {
  if (['Dignity-Risk Signal','Negative Signal','Masked Negative Signal'].includes(signal_type)) return 'T-NEGATIVE';
  if (['Ambiguous Negative Signal','Mixed Signal'].includes(signal_type)) return 'T-AMBIGUOUS';
  return 'T-POSITIVE';
}
```

- Before the fix, three of the six signal types, including **Dignity-Risk**, were silently uncounted.
- **Watch:** a blank or unrecognized signal type now counts as positive.
- This function is a **copy** of SIA's. If SIA's rules change, update both (§16).

**Weekly/monthly branch** reads SIA clusters (latest run only) and uses SIA's own tiers. No bug.

**SEO:** keyword hits are counted in **Approved** drafts (`approved_drafts` from Step 9). Coverage = drafts with a keyword ÷ approved count.

**Passed through from Step 9:** `approval_rate`, `approved_count`, `not_accepted_count`, `modified_count`, `decided_count`.

**Removed:** `sla_compliance_rate`, `seo_avg_velocity_hrs`, `seo_response_rate`, `published_count`.

It also computes top domains, the top 5 non-singleton trends (weekly/monthly only) and the expression-mode counts.

---

## 9. Step 9 — Records and Approval Metrics

Step 9 matches RDA to ALA by `ALA Record ID`, keeps records whose review date falls in the window (§4), and computes:
- `total_records`, `avg_star_rating`, `platform_counts`.
- **Approval rate** = Approved ÷ (Approved + Not Accepted + Modified) × 100, or 0 if nothing is decided.
- `approved_count`, `not_accepted_count`, `modified_count`, `decided_count`.
- **Tier breakdown** (T1/T2/T3): total, approved, pending.
- `approved_drafts`: lowercased draft text and tier, used for the SEO scan.
- `all_time_pending`: Pending across all of the client's RDA records, not just the window.
- `filtered_ala_record_ids`, used by Step 11.

**What changed and why:**
- v7.0 flagged a chain of metrics built on a "Published" status that no longer exists, so they were always zero.
- **Speed metrics are retired by design.** No approval timestamp exists, and you decided quality matters more than speed.
- The SLA/velocity fields and all references to "Published", "Edited-Approved" and "Pending-Elevated" were removed.
- `tier_breakdown.avg_approval_hrs` was removed.
- **Verified live:** your Oct 1 output shows exactly these fields.

**Still true:**
- **Step 7 is a placeholder.** It passes records through and writes zeros. Step 9 does the real work. Cosmetic only.
- **A blank star rating counts as 0** in the average. That is a data issue, not a calculation bug.

---

## 10. Field Passthrough

Code nodes that rebuild their output field by field silently drop anything not listed. Fetch nodes and plain IF nodes pass items through unchanged.

- **`sheet_id`:** carried from Step 5 onward.
- **`parent_account_id`:** carried through 5, 7, 7b, 9, 10a, 11 and 12. **Correction to v7.0:** v7.0 said Step 13 drops it, but the Step 13 code and output you pasted in Chats #103 and #105 both show it present. Treat that as a likely non-issue and confirm on the next export.
- **`report_type`:** added to Step 4c's payload, read from Step 3. The brand-grouping chain had no other way to know it.
- **`seo_keywords_raw`:** passed 4a → 4c, then parsed at Step 5 (split, trimmed, lowercased).
- **New approval fields:** the new Step 9 fields (`approval_rate`, `approved_drafts`, the counts) have to cross Steps 10a, 11, 12 and 13. Step 11 was updated. **10a, 12 and 13 were not updated in our chats**, so they likely drop these fields. Nothing visible breaks because the daily email doesn't display them. Check 10a first.

---

## 11. Recipients and Sheet Links

- **Recipients:** each location's `Approval Contact Email` is split on commas/semicolons. The daily email merges every location's list into one send. The field is NocoDB LongText, because the Email type allows only one address. A blank field falls back to miguel@solofella.com.
- **Approval buttons** (daily) open that location's own Google Sheet: `docs.google.com/spreadsheets/d/{sheet_id}/edit`.
- **Dead link:** the old `subtextcx.solofella.com` dashboard link leads to a blank page. `17a` no longer uses it. `17b` and `17c` still do (§13).

---

## 12. `17a` — Daily Email

**Layout:**
- **Header:** dark wordmark bar with sender address, then a terracotta title band (account name, "All N locations", date).
- **KPI strip:** New reviews, Avg rating, Drafts generated, Pending (all time). It uses alternating tones and is built as a table.
- **One row per location:** identity (name, reviews · stars · platform counts, top tier, top domain) | signal direction + pending | "View & Approve →". A colored stripe shows signal direction: green positive, amber mixed, red negative, sand no change. A negative-majority location gets a solid button.
- **Footer:** warm greige with the governance note.
- **Single-location brands** use the same row design, with a larger button label.

**Built for real email clients:**
- Tables, not flexbox, because Gmail's mobile app breaks flexbox.
- The identity column is fixed at 46% width so long names wrap instead of stretching the row.
- Stacked mobile layout. Confirmed on Apple Mail and Gmail mobile. Some clients drop mobile styling, and then the desktop layout shows.
- Sender link color is forced so clients don't turn it blue.

**Heads-up:**
- The email date is the run date, but the data is the previous calendar day, and the label says "last 24 hours" (§16).
- The email does not show approval rate or expression modes.

---

## 13. Weekly and Monthly

**Weekly sending is paused for the pilot (`18-gate`).**

```
17d → 18-gate (weekly?)
    yes → 19 (skips the Brevo send)
    no  → 18 Send → 19
```

- Weekly still processes, writes to NocoDB and advances the loop. Only the email is suppressed.
- Step 19 doesn't read Step 18's result, so skipping Step 18 is safe.
- **Side effect, accepted:** NocoDB marks paused weekly reports "sent".

**`17b` / `17c` are unchanged and out of date:**
- Both still link to the dead dashboard URL.
- `17b` shows a hardcoded "100%" Response Coverage and a trend arrow that never works (it looks for UP/DOWN, SIA says Stable/Growing/Declining/New).
- Both reference the retired SLA and speed fields.
- `17c` is unreachable while monthly is off.

**Direction for weekly/monthly:** you chose a **per-client, no-login dashboard** over a consolidated weekly email. It is a hosted page opened by a private link (like today's token pattern) that shows trends over time, filters and history. It is handed to a dedicated chat. What was settled:
- **Ruled out:** Google Workspace portals (internal-staff tools) and Looker Studio. Looker Studio has no NocoDB connector, its link filters can be edited by the viewer, true per-client security needs Google logins, and it can't create a report per new client automatically.
- **Kept as alternative:** self-hosted Metabase (fast, but a generic look).
- **Chosen:** a custom page, because the VRYOH look matters.
- **Data it would read:** SIA clusters (tier, count, prior count, trend, run time) and the weekly/monthly MRA rows. Daily isn't stored, and approval rate isn't stored yet.
- **Still undecided:** first-version scope, how much history, hosting (same server or separate), token strength, and whether it replaces the weekly email or sits beside it.

---

## 14. Delivery Status and Loop Advance

```
19 Update Delivery Status (always writes "sent")
 → 19b Gate Daily PATCH (name is a holdover; now passes everything through)
 → If (is there a real nocodb_record_id?)
    yes → 20 NocoDB PATCH → back to 4d    (weekly/monthly, next location)
    no  → 19c Advance → back to 4b        (daily, next brand)
```

The `If` node uses an explicit true/false expression. n8n's "not empty" check treated `null` as text "null", which sent every daily run down the wrong path.

---

## 15. Locked Lessons

1. Filter by ALA Date, never RDA Timestamp.
2. Step 11's weekly/monthly source must use named references, never `$input`.
3. `filtered_ala_record_ids` is built in Step 9 and used in Step 11's daily branch.
4. Wrap ALA Record ID in `String()` before comparing.
5. A loop-back node must output exactly one item every time.
6. Rebuilding Code nodes drop unlisted fields, so check every hop.
7. NocoDB field type limits data: Email = one address; use LongText for lists.
8. Sheet sharing is a Google permission, not a workflow setting.
9. Blank source data silently skews averages.
10. `records[0]`-only reads silently drop the other rows.
11. Nested batch loops need an explicit Reset on the inner loop.
12. `.all()` returns a node's whole run history, so filter by a per-pass key.
13. A shared loop exit needs a report-type gate.
14. Email layouts use tables, not flexbox.
15. n8n state never carries between separate runs. Odd cross-run behavior means a wiring bug.
16. n8n's "not empty" test treats `null` as non-empty. Use explicit true/false checks.
17. **Test data goes by the review date, not the run date.** Daily reports cover yesterday.
18. **When an agent upstream changes a field's allowed values, audit every MRA node that reads it.** The "Published" chain stayed dead for weeks.

---

## 16. Open Items

**Closed since v7.0:** signal-tier fix (applied). Dead "Published" metrics (replaced). SEO source (Approved drafts). Account display name (brand name in `parent_account_id`). Node count (46).

**Quick checks:**
1. **Status spelling:** Step 9 looks for "Modified". RDA v7.1 says "Modified", an older note says "Modify". Confirm the real value in NocoDB.
2. **Approval flow:** change one status in a Sheet and confirm NocoDB updates. RDA v7.1 says it's live, but an earlier remark in Chat #108 said nothing came back.
3. **Field carry-through:** confirm Steps 10a, 12 and 13 pass the new Step 9 fields. Likely not.
4. **Step 13:** confirm `parent_account_id` is present (§10).
5. **Old workflow retired:** confirm the original per-location MRA workflow is off (no duplicate daily emails), and record the date the new one went live.

**Decisions:**
6. **Approval rate:** where is it shown and stored? The MRA table has no column for it, and three columns (SLA Compliance Rate, SEO Response Rate, SEO Avg Velocity Hrs) are now always empty. Retire or repurpose them.
7. **Weekly/monthly rebuild and dashboard** (separate chat): includes cleaning `17b`/`17c` as listed in §13. Weekly sending stays paused until then.
8. **Monthly:** the last day of the month is skipped (derived from the code, untested), plus the SIA window mismatch. Fix both before re-enabling, and set a re-enable date.
9. **Token strength:** the private-link token (Step 12) uses `Math.random()`, which is not secure. This matters more if the dashboard reuses it.
10. **Delivery check:** confirm Brevo accepted the email before Step 19 writes "sent". Also relabel paused weekly items.
11. **SEO source:** RDA already stores "SEO Keywords Used" for each draft. Read that directly instead of re-scanning text?
12. **Email labels:** the email shows the run date and "last 24 hours" but contains the previous calendar day. Relabel, e.g. "Reviews from Sept 30"?
13. **Blank recipient fallback:** confirm miguel@solofella.com is the intended default.
14. **Shared logic:** the signal-tier rule now lives in SIA and MRA. Keep them in sync or centralize.
15. **Cleanup:** remove the idle `13b`, rename Step 7 and Step 19b to match what they do, and remove the "MRA-TEST" wording in Step 4c's error message.
16. **Brand-name typos:** a mismatch in `parent_account_id` splits a brand into two emails. Add a check.

---

*End of SCX_MRA_HOW_v8.0 — Chat #115, October 6, 2026. Full rewrite of v7.0 with the same 16 sections. Nothing was fixed or changed in the workflow while writing it.*
