# SCX_SIA_HOW_v5.0

**Agent Name:** SIA (Signal Intelligence Aggregator)
**Version:** 5.0
**Last Updated:** Chat #104 · July 25, 2026
**Model:** None (Pure JavaScript — zero AI calls)
**Status:** Live-audited via Claude Code direct read of n8n workflow JSON (Chat #102). Superseded v4.0's doc-only claims on several points, particularly trigger hours and node count. 8 open items unresolved, listed in Section 11.

**Source basis for this version:** This document is built from a Claude Code audit that read the live n8n workflow JSON directly — not from re-derivation of v4.0's prose. Where v4.0 and the live audit disagreed, the live audit is treated as current source of truth until Miguel confirms intent. Confidence per claim is marked inline (✅ = confirmed against live code/data, ⚠️ = observed but intent unconfirmed, ❌ = known gap).

---

## 1. Purpose

SIA is a zero-cost, schedule-triggered batch aggregation agent. It has no webhook and no per-record trigger — it runs on three independent n8n Schedule Triggers (daily, weekly, monthly), each producing time-windowed cluster intelligence written to its own NocoDB table for downstream reporting consumption.

SIA reads ALA records (filtered by **guest review creation date**, not pipeline processing time), matches them to EIP records via `ALA Record ID`, groups matched records by `Client ID + Domain + Signal Tier`, computes trend direction against the prior run of the same window type, and writes the resulting clusters back to NocoDB in a single bulk POST.

**SIA makes zero AI calls.** No Claude or GPT calls occur anywhere in this workflow — confirmed directly from the live n8n JSON, not inferred. ✅

SIA is architecturally distinct from every other agent in the pipeline (ALA, EIP, ESS, HSI, BRA, RDA): those are per-record, webhook-chained agents; SIA is a scheduled, multi-cadence batch process with no webhook in and no trigger out. It is terminal and self-contained.

---

## 2. Three-Schedule Architecture

| Schedule | Node | Trigger Config | Window | Status |
|---|---|---|---|---|
| Daily | Step 1a | `triggerAtHour: 7` | 2 days (not 1) | ✅ Hour matches v4.0 doc. ⚠️ 2-day window not confirmed deliberate vs. accidental (Open Item 6). |
| Weekly | Step 1b | `triggerAtDay: [1]` (Monday), `triggerAtHour: 8` | Prior complete calendar week | ❌ v4.0 doc said 6am — live code says 8am. Disagreement unresolved (Open Item 1). |
| Monthly | Step 1c | `triggerAtHour: 9` (1st of month, implicit) | Rolling 30 days | ❌ v4.0 doc said 6am — live code says 9am. Disagreement unresolved (Open Item 1). |

**Neither the documented (6am/6am) nor the live (8am/9am) values for weekly/monthly have been independently confirmed as the intended correct value.** This document records what the live code currently does; it does not assert that this is correct.

Each of the three schedule triggers feeds a dedicated Set node (Steps 2a/2b/2c) that stamps `window_type` (`daily`/`weekly`/`monthly`) and `window_days` (2/7/30). All three converge into the same downstream chain starting at Step 3.

---

## 3. Window Calculation Logic (Step 3)

**Step 3 — Set Window Parameters** (Code node)

- For `weekly`: computes the **previous complete calendar week**, Monday 00:00:00.000 UTC → Sunday 23:59:59.999 UTC. ✅ Confirmed matches live code's boundary math exactly, including correct handling of edge cases when the run executes on a Sunday (`currentDay = 0`) or Monday (`currentDay = 1`).
- For `daily`/`monthly`: simple rolling window — `now` minus `window_days`.
- Sets fixed constants: `trend_threshold: 0.20` (20% change threshold separating Stable from Growing/Declining), `min_cluster_size: 2`.
- Generates `sia_run_id` in format `SIA-YYYYMMDD-HHMMSS`.

**Step 4 — Set Window Bounds** (Code node)

❌ **Confirmed complete no-op.** Copies every input field to output unchanged; adds or transforms nothing. Exists for architectural symmetry only. Recommend deletion (Open Item 3) — not actioned here per Diagnose-Only Rule.

---

## 4. Data Sources & Fetch Strategy

SIA does not receive webhooks. It queries NocoDB directly on each schedule fire.

**Step 5 — Fetch ALL ALA Records**
`GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/m57efwbtrvwohhr?limit=1000&sort=Date`

❌ **Known risk (Open Item 2):** No WHERE filter — fetches every ALA record across every client, relying entirely on Step 6's in-memory date filter. Hardcoded `limit=1000`. Sort is `Date` with no `-` prefix, which in NocoDB's convention typically means ascending (oldest-first) — unconfirmed, but if true, this fetch returns the **oldest** 1000 rows once total ALA records exceed 1000, silently starving every window of recent data with no error surfaced anywhere. Flagged as a "ticking time bomb": inert today at current volume, catastrophic once the threshold is crossed.

**Step 6 — Filter ALA by Date** (Code node)
- Parses ALA `Date` field, format `M/D/YY` (no leading zeros, 2-digit year always interpreted as `2000 + YY`).
- Filters to `[window_start, window_end)`.
- Builds `ala_id_list` (filtered ALA IDs) and `ala_client_map` (ALA ID → Client ID) for downstream matching.
- Sets `empty_run: filteredAla.length === 0`.

❌ **Known gap (Open Item 7):** `empty_run` cannot distinguish "genuinely no reviews this period" from "Step 5's fetch missed the relevant records." Both conditions currently look identical downstream.

**Step 7 — Fetch ALL EIP Records**
`GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mhicpnrahaesxmy?where=(ESS Status,eq,Complete)&limit=1000`

❌ Same systemic risk as Step 5, **arguably worse**: no sort parameter at all — relies entirely on NocoDB's undocumented default ordering. Same time-bomb concern once completed-EIP-record count exceeds 1000 across all clients.

**Step 8 — Filter EIP by ALA Match** (Code node)
- Cross-references fetched EIP records against `ala_id_list` via `ALA Record ID`.
- Attaches `client_id` from `ala_client_map` to each matched record.
- This is the actual mechanism scoping EIP records to the target window — its correctness is entirely dependent on Steps 5 and 7 having fetched the right underlying data.

---

## 5. Prior-Run Lookup for Trend Comparison

**Step 9 — Load Prior SIA Clusters**
`GET http://nocodb:8080/api/v1/db/data/noco/pq249fix22t3ofv/mdn68l4lm609fve?where=(Report Window Type,eq,{window_type})&limit=1000&sort=-Run Timestamp`

✅ Correctly filtered to the matching `Report Window Type` and correctly sorted descending (properly `-`-prefixed, unlike Step 5's ambiguous sort).

**Step 10 — Build Prior N Map** (Code node)
- Builds `prior_n_map` keyed by `ClientID||Domain||SignalTier`.
- Captures each key's most recent prior `N`, using "only set if not already present" logic — correct given the input is already sorted most-recent-first.

---

## 6. Signal Tier Mapping

`getSignalTier()` (inside Step 11):

| EIP `Signal Type` value | Mapped Tier |
|---|---|
| Dignity-Risk Signal | T-NEGATIVE |
| Negative Signal | T-NEGATIVE |
| Masked Negative Signal | T-NEGATIVE |
| Ambiguous Negative Signal | T-AMBIGUOUS |
| Mixed Signal | T-AMBIGUOUS |
| Positive Signal | T-POSITIVE (default) |

✅ Confirmed to exhaustively and correctly map all 6 of EIP's actual Signal Type enum values — verified directly against EIP's confirmed enum, no drift.

---

## 7. Cluster Aggregation & Trend Comparison Logic (Step 11)

**Step 11 — Build Clusters** (Code node) — the core aggregation logic.

- Groups EIP records into clusters keyed by `ClientID||Domain||SignalTier`.
- Counts occurrences per cluster; builds `Enriched Pain Point Breakdown` and `Enriched Emotion Breakdown` as JSON-stringified frequency maps, sorted descending by count.
- **Trend calculation:**
  ```
  pct = |n_delta| / priorN   (or 1 if no prior N)
  Stable    if pct <= 0.20
  Growing   if pct > 0.20 and n_delta > 0
  Declining if pct > 0.20 and n_delta < 0
  New       if no prior data exists for that composite key
  ```
  ✅ Verified correct against a real production row: `n=1, prior_n=3` → `pct ≈ 0.667` → correctly classified `Declining`.
- Computes `is_singleton: n < min_cluster_size` (i.e., `n < 2`) per cluster.
- Computes each cluster's percentage of total records in the run.
- Computes run-level aggregates: counts by tier, by trend direction, singleton vs. active counts.

❌ **Real-data observation (Open Item 5), not a code defect:** an actual live daily run produced 5 clusters, **all with n=1** — every cluster a singleton, none reaching `min_cluster_size=2`. This suggests daily-cadence review volume per client per domain may be structurally too low to ever produce a non-singleton cluster for at least the smaller clients in the roster. Open question, not yet resolved: is daily-cadence clustering analytically meaningful as designed, or does it only become useful at weekly/monthly cadence?

**Step 12 — Split Clusters** (Code node)
Fans the `clusters` array out into individual n8n items, each carrying its own cluster fields plus the run-level aggregate stats duplicated across every item.

---

## 8. Multi-Client Composite Key Design

Every cluster carries a `Client ID`. The composite trend-comparison key is:

```
key = client_id + '||' + domain + '||' + signal_tier
```

✅ Confirmed in live code, including an inline comment `✅ FIXED: Include client_id in key` — suggesting this correctness fix (preventing cross-client trend contamination, e.g. PAK-001's numbers being compared against EDO-001's) was itself a relatively recent addition to the workflow, not part of the original design.

Without Client ID in the key, PAK-001's Service Quality T-NEGATIVE trend could be silently compared against a different client's prior count for the same Domain+Tier — this is now closed off.

---

## 9. NocoDB Write Strategy

**Step 13 — Build Bulk POST Body** (Code node)
Maps each split item (from Step 12) into a NocoDB-field-named record: `Client ID`, `SIA Run ID`, `Run Timestamp`, `Report Window Type`, `Window Days`, `Domain`, `Signal Tier`, `N`, `Prior N`, `N Delta`, `Trend Direction`, `Enriched Pain Point Breakdown`, `Enriched Emotion Breakdown`, `Is Singleton`, `Percentage`. JSON-stringifies the full array as one bulk payload.

**Step 14 — NocoDB Bulk POST Clusters**
`POST http://nocodb:8080/api/v1/db/data/bulk/noco/pq249fix22t3ofv/mdn68l4lm609fve`

This is the **only** bulk-insert endpoint anywhere in the entire pipeline audit — every other agent (ALA, EIP, ESS, HSI, BRA, RDA) uses single-record POSTs.

❌ **Confirmed gap (Open Item 4):** No success/failure verification exists after this node. Every other agent has a dedicated "Capture Record ID" step that explicitly throws if the write doesn't return an expected ID. SIA has none. A hard network/HTTP-level failure would still throw via n8n's default error behavior, but a silent **partial** success — NocoDB accepting some rows and rejecting others due to a data issue on one record — would go completely unnoticed. Because this is a scheduled batch job with no immediate downstream consumer watching it, a failed or partial run could go unnoticed for an extended period.

Step 14 is terminal. No further connection exists downstream in this workflow.

---

## 10. Zero-AI-Cost Model

- Zero OpenAI API calls.
- Zero Anthropic API calls.
- Zero dictionary/knowledge-structure queries.
- Only operations: NocoDB GET/POST calls and in-memory JavaScript (date parsing, grouping, counting, percentage math).
- Cost: infrastructure only (NocoDB query time + n8n execution time).

---

## 11. Full Node Flow (18 Nodes)

```
[Step 1a Daily Trigger 7am]  → Step 2a (window_type=daily, window_days=2)   ─┐
[Step 1b Weekly Trigger Mon 8am] → Step 2b (window_type=weekly, window_days=7) ─┼→ Step 3
[Step 1c Monthly Trigger 9am] → Step 2c (window_type=monthly, window_days=30) ─┘
Step 3 (window_start/end + constants + sia_run_id)
  → Step 4 (no-op, flagged for removal)
  → Step 5 (fetch ALL ALA, limit=1000, risk flagged)
  → Step 6 (filter ALA by Date, build ala_client_map, empty_run flag)
  → Step 7 (fetch ALL EIP where ESS Status=Complete, limit=1000, risk flagged)
  → Step 8 (match EIP↔ALA via ALA Record ID, attach client_id)
  → Step 9 (fetch prior SIA clusters, filtered by window_type, sorted -Run Timestamp)
  → Step 10 (build prior_n_map)
  → Step 11 (build clusters: group, tier-map, trend calc, singleton/pct)
  → Step 12 (split array → items)
  → Step 13 (build bulk POST body)
  → Step 14 (bulk POST — terminal, no verification)
```

---

## 12. Access & Infrastructure Facts

| Item | Value |
|---|---|
| NocoDB base | `http://nocodb:8080`, base ID `pq249fix22t3ofv` |
| ALA table | `m57efwbtrvwohhr` |
| EIP table | `mhicpnrahaesxmy` |
| SIA output table | `mdn68l4lm609fve` |
| NocoDB auth | `httpHeaderAuth`, `xc-token`, credential ID `DT9tnRgqYpPc3rXo` (same credential as main pipeline — unlike SCX-Sheet-Sync, which uses a separate, differently-scoped credential) |
| Webhook | None — three independent Schedule Triggers are the only entry points |
| Downstream trigger | None — SIA does not call another workflow. Terminal/self-contained. |
| Presumed consumer of output table | MRA (per architecture docs) — **not independently confirmed in this audit** (Open Item 8) |

---

## 13. SIA NocoDB Table Schema (Live-Verified, 2026-07-21)

Table: `mdn68l4lm609fve` — 20 total columns = 5 NocoDB system fields + 15 data fields.

System fields: `Id` (ceb4nca3tpxu5cc), `CreatedAt` (c1yaat85uzcigip), `UpdatedAt` (cl2iszqp3lyjm1f), `nc_created_by` (cjun6zwj0p0rqt2), `nc_updated_by` (chrm8q3j7hp3f1d), `nc_order` (cijhjhwamkehqp7).

Data fields:

| Field | Field ID | Type |
|---|---|---|
| SIA Run ID | caj8uqmy4gt0jht | SingleLineText |
| Run Timestamp | cxefar2jdak52pe | DateTime |
| Window Days | ctkp2xbv2m1lrg7 | Number |
| Domain | ca689w3tscfrv64 | SingleLineText |
| Signal Tier | cc9kh6gzletpjw5 | SingleSelect |
| N | c0c8dsniaizq9lv | Number |
| Prior N | cmbi5zhj5r0gsgx | Number |
| N Delta | csl5ii7qd2ke832 | Number |
| Trend Direction | cvakjryjsn7mlci | SingleSelect |
| Enriched Pain Point Breakdown | cu28t85sieke73d | LongText |
| Enriched Emotion Breakdown | c1ji5p1skotcwnp | LongText |
| Is Singleton | cesjlze5ihmgiyg | Checkbox |
| Percentage | cu5j3xgtefxidzt | Decimal |
| Report Window Type | c5wrrw2729hlwa0 | SingleSelect |
| Client ID | cqxifo3lpi5m0x6 | SingleLineText |

✅ Confirmed against direct NocoDB export (Chat #101), zero discrepancy with live workflow field usage.

---

## 14. Known Issues (Consolidated)

| # | Issue | Severity | Status |
|---|---|---|---|
| 1 | Weekly/monthly trigger hours disagree between v4.0 doc (6am/6am) and live code (8am/9am) | Medium — operational timing, not data-correctness | Unresolved |
| 2 | Step 5/Step 7 unfiltered `limit=1000` fetches, ambiguous or missing sort | High (latent) — silent wrong-window data once row counts exceed 1000 | Unresolved, not yet triggered |
| 3 | Step 4 confirmed no-op | Low — cleanup only | Unresolved |
| 4 | No post-write verification after Step 14 bulk POST | Medium-High — silent partial-write failure possible, no downstream watcher | Unresolved |
| 5 | Daily-cadence clustering may never produce non-singleton clusters at current volume | Design question, not a defect | Unresolved |
| 6 | Step 2a's 2-day daily window — deliberate buffer or accidental? | Low-Medium | Unresolved |
| 7 | `empty_run` flag can't distinguish true-zero from fetch-miss | Medium — masks issue #2 if it occurs | Unresolved |
| 8 | Downstream consumer of SIA output table not independently confirmed | Low-Medium — architectural assumption (MRA) unverified | Unresolved |

---

## 15. Open Items Requiring Miguel's Decision (Not Actioned — Diagnose-Only)

1. Confirm the correct weekly and monthly trigger hours — live code and documentation disagree; neither independently verified as intended.
2. Address Step 5/Step 7 unfiltered-fetch scalability risk (server-side date filter, explicit descending sort, or pagination) — no fix proposed here.
3. Confirm whether Step 4 (no-op) should be removed.
4. Confirm whether post-bulk-write verification should be added, matching the Capture-Record-ID pattern used elsewhere in the pipeline.
5. Resolve whether daily-cadence clustering is analytically meaningful given real per-client review volume — a reporting/design conversation, not a code fix.
6. Confirm whether Step 2a's 2-day daily window is deliberate.
7. Confirm whether `empty_run` flag logic should be improved to distinguish true-zero from fetch-miss.
8. Confirm the actual downstream consumer(s) of SIA's cluster output table.

---

**End of SIA HOW v5.0**
