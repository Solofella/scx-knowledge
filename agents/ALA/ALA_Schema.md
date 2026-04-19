# ALA Schema v2.1

**NocoDB Table ID:** `m57efwbtrvwohhr`  
**Base ID:** `pq249fix22t3ofv`  
**Last Updated:** Chat #76 · April 19, 2026  
**Status:** VERIFIED · 19 fields · Chat #74 changes applied

---

## Table Fields (19 Total)

| Field Name | Type | Required | Purpose | Notes |
|------------|------|----------|---------|-------|
| **ALA Record ID** | AutoNumber | YES | Primary key | Auto-increment. Travels through entire 8-agent pipeline. |
| **Client ID** | SingleLineText | YES | Client identifier | Chat #74 addition. Source: CSV column 6. PAK-001, EDO-001, AJI-001, etc. Required for all records. |
| **Platform** | SingleSelect | YES | Source platform | Options: Google, Yelp, OpenTable, TripAdvisor, Manual. Immutable after POST. |
| **Date Posted** | DateTime | YES | Review publication date | ISO 8601 format. Original timestamp from platform source. |
| **Star Rating** | Number | YES | Guest rating | Integer 1–5 only. Null rejected. |
| **Review Text** | LongText | YES | Original review content | Unmodified guest text. Preserved for audit trail. Never edited. |
| **Reviewer Handle** | SingleLineText | NO | Guest identifier | Name, email, or anonymized handle from platform. Null permitted. Used by RDA Step 7a for opening construction. |
| **Raw Tex** | LongText | YES | ALA output — normalized text | **CURRENT FIELD NAME (truncated).** Do NOT rename to "Raw Text" until post-pilot. Contains GPT-cleaned, semantically-preserved review text. Passed to EIP as primary input. |
| **Language Detected** | SingleLineText | YES | ISO 639-1 code | en, es, pt, etc. Detected by GPT at ALA processing. Phase 1: en, es only. |
| **Character Count** | Number | YES | Normalized text length | Used for complexity analysis downstream. Calculated after normalization. |
| **Dedup Hash** | SingleLineText | YES | Duplicate detection key | SHA256(Reviewer Handle + Date Posted + Platform + Review Text). Checked before INSERT. |
| **Ingestion Status** | SingleLineText | YES | Processing status | Options: Pending, Processed, Duplicate, Error. Updated throughout pipeline. |
| **Created At (ALA)** | DateTime | YES | ALA record timestamp | Auto-generated at webhook receive. ISO 8601 UTC. |
| **Error Log** | LongText | NO | Processing error details | Null if clean; populated only on failure. Non-blocking — batch continues. |
| **Reserved Field 1** | SingleLineText | NO | (Unused) | Reserved for future Phase 1b integrations. Do not repurpose. |
| **Reserved Field 2** | SingleLineText | NO | (Unused) | Reserved for future Phase 1b integrations. Do not repurpose. |
| **Reserved Field 3** | LongText | NO | (Unused) | Reserved for future Phase 1b integrations. Do not repurpose. |
| **Reserved Field 4** | SingleLineText | NO | (Unused) | Reserved for future Phase 1b integrations. Do not repurpose. |
| **Reserved Field 5** | SingleLineText | NO | (Unused) | Reserved for future Phase 1b integrations. Do not repurpose. |

**Total: 19 fields (14 active + 5 reserved)**

---

## Field Traceability — CSV Source to NocoDB

**Input Source (Phase 1: Manual CSV Upload)**

| CSV Column | Column Name | Type | NocoDB Destination | Status |
|---|---|---|---|---|
| 1 | Date | Date | Date Posted | Mapped |
| 2 | Platform | Text | Platform | Mapped |
| 3 | Star Rating | Number | Star Rating | Mapped |
| 4 | Review Text | LongText | Review Text | Mapped |
| 5 | Reviewer Handle | Text | Reviewer Handle | Mapped |
| **6** | **Client ID** | **Text** | **Client ID** | **Mapped (Chat #74 addition)** |
| 7 | Ingestion Status | Text | Ingestion Status | Mapped |

**Locked Rule (Chat #74):** Client ID must be a CSV column. Cannot be introduced mid-pipeline. All 24 ALA nodes depend on Client ID existing at Source.

---

## Processing Guardrails — n8n Implementation

### GPT Model Configuration
```
Model: gpt-5.2
max_completion_tokens: 400  ← CRITICAL: NOT max_tokens (gpt-5.2 blocks max_tokens, throws 400 error)
temperature: 0
Retry: true, Max tries: 3
Wait Between Tries: 5000ms
```

### Batch Processing
```
SplitInBatches size: 1 (LOCKED Chat #74 — prevents simultaneous API calls)
Last processing node wires back to: SplitInBatches input (not webhook trigger)
Never exceed batch size of 1 in production
```

**Reason:** Tests with batch size > 1 lost 15 records per 40-record test due to concurrent API rate limiting.

### CSV/Sheet Line Endings
```javascript
// CORRECT — handles Windows (CRLF) and Unix (LF)
const lines = csvText.split(/\r?\n/);

// WRONG — fails on Windows line endings
const lines = csvText.split("\n");
```

### Payload Construction
```javascript
// CORRECT — key-by-key (task runner blocks spread operator)
return [{ json: {
  client_id: parsed.client_id,
  platform: parsed.platform,
  date_posted: parsed.date_posted,
  // ... all fields individually assigned
} }];

// WRONG — spread operator not permitted in n8n task runner
return [{ json: { ...parsed } }];
```

### Webhook Configuration
- **Auth:** NONE (all agent webhooks)
- **Body Type:** JSON (not RAW)
- **Response:** 200 immediately (fire-and-forget to EIP)

---

## Relationships

### Upstream
None. ALA is the pipeline entry point.

### Downstream (All Locked)

| Downstream Agent | Field Passed | Field Name in Downstream Table | Used For |
|---|---|---|---|
| **EIP** | ALA Record ID | ALA Record ID | FK traceability, emotion analysis context |
| EIP | Client ID | client_id | Routing, brand voice lookup |
| EIP | Raw Tex | raw_input | Dictionary matching, pain point detection |
| EIP | Star Rating | star_rating | Sentiment anchor for emotion validation |
| EIP | Language Detected | lang | Model language selector |
| **ESS** | ALA Record ID | ALA Record ID | Signal context traceability |
| ESS | Client ID | client_id | Client isolation |
| **HSI** | ALA Record ID | ALA Record ID | Traceability anchor |
| HSI | Client ID | client_id | Client isolation |
| **RDA** | Reviewer Handle | reviewer_handle | Opening construction (Step 7a) |
| RDA | ALA Record ID | ALA Record ID | Audit fetch (Step 9a) |

**Path:** ALA → EIP (primary) → ESS → HSI → SIA (parallel) → BRA → RDA

---

## Processing Logic — Normalization

### GPT gpt-5.2 Tasks (400 tokens max)
1. Remove HTML tags, markup, extra whitespace
2. Standardize punctuation (smart quotes → ASCII, em dashes consistent)
3. Fix obvious typos without changing semantic intent
4. Preserve original meaning, tone, emphasis
5. Detect primary language — output ISO code

### Duplicate Detection Algorithm
```
Hash key: SHA256(Reviewer Handle + Date Posted + Platform + Review Text)
Query: SELECT COUNT(*) WHERE Dedup Hash = calculated_hash
If count > 0: Mark as Duplicate, skip EIP trigger
If count = 0: Proceed to normalization
```

**Critical Rule:** Use `pageInfo.totalRows` (NOT `list.length`) for accurate NocoDB count.

### Error Handling
- Failed records logged to Error Log field (LongText)
- Pipeline does NOT block on single record errors
- Batch processing continues for remaining records
- Error flag triggers downstream error handler at RDA

---

## Data Flow Diagram

```
CSV Upload (Manual Phase 1)
    ↓
ALA Webhook Trigger
    ↓
[Step 1-4] Parse Payload + Validate Client ID + Duplicate Check
    ↓
[Step 5-18] Normalize (GPT) + Detect Language + Calculate Hash
    ↓
[Step 19-21] Write to ALA NocoDB (19-field record)
    ↓
[Step 22-24] Trigger EIP Webhook (fire-and-forget, rich payload)
    ↓
EIP Processing (parallel, non-blocking)
```

**Phase 1b Future:** Replace CSV upload with Google Sheet polling (unified 7-column schema). BCA reads from ALA NocoDB, drips at 10 records/batch to prevent rate limit loss.

---

## Known Issues & Mitigations

| Issue | Status | Mitigation |
|-------|--------|-----------|
| **Raw Tex field truncated from "Raw Text"** | Post-pilot | Do NOT rename until after first 3 pilots. Renaming breaks existing n8n node references. Scheduled cleanup: November 2026 post-stabilization. |
| **SplitInBatches=1 performance** | Accepted trade-off | Sequential processing is slower but prevents API loss. At 10–15 records/day pilot volume, acceptable latency. Optimize post-Phase 1. |
| **No batch rate limiting (BCA pending)** | Deferred | Option B design documented. Phase 1 manual uploads capped at ~50 records/run to stay under OpenAI rate limits. BCA build scheduled after pilot data exists. |
| **Client ID not in original design** | Chat #74 fix | Now required at Source. All new clients must include Client ID column in CSV. Old spreadsheets will fail validation. |

---

## Related Documents

- **HOW Document:** [SCX_ALA_HOW_v4.1](SCX_ALA_HOW_v4.1.md) — Full 24-node decomposition, code examples, prompts
- **Changelog:** [SCX_ALA_CHANGELOG.md](SCX_ALA_CHANGELOG.md) — Build history (Chat #70 → #76)
- **Pre-Build Protocol:** [../../protocols/SCX_PreBuild_Protocol_v1.0.md](../../protocols/SCX_PreBuild_Protocol_v1.0.md) — Field Traceability Map standard
- **Schema Registry:** [../../schemas/Schema_Registry_v2.md](../../schemas/Schema_Registry_v2.md) — All 8-agent schema cross-reference
- **Master Continuity Doc:** [../../MCD_v7.4.md](../../MCD_v7.4.md) — Cold start package, locked decisions
- **Downstream Agent:** [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md) — Emotion Intelligence Processor

---

**Subtext CX · ALA_Schema.md v2.1 · Chat #76 · April 19, 2026 · Solofella LLC**

---

## CHANGES (v2.0 → v2.1)

✅ **Schema Completeness:** Added 7 missing field definitions (Dedup Hash, Ingestion Status, Reserved 1–5) to reach locked total of 19 fields  
✅ **Field Name Accuracy:** Corrected primary output field from "Normalized Text" to "Raw Tex" (truncated — do not rename until post-pilot)  
✅ **Chat #74 Lock:** Client ID elevated to YES/Required + CSV Source column 6 + locked rule that it cannot be introduced mid-pipeline  
✅ **Field Traceability:** Added explicit CSV-to-NocoDB mapping table (Chat #74 protocol)  
✅ **n8n Guardrails:** Added model parameters, batch size rule, line-ending handling, payload construction rules with examples  
✅ **Downstream Relationships:** Documented exact field names and usage in each downstream agent  
✅ **Processing Logic:** Clarified dedup hash algorithm + pageInfo.totalRows rule + error handling (non-blocking)  
✅ **Known Issues:** Added Raw Tex truncation issue + mitigation timeline + Client ID requirement note  
✅ **Versioned Links:** Updated all related document cross-references to v4.1, MCD v7.4, PreBuild Protocol v1.0

**No structural changes to NocoDB table architecture.** All updates are documentation corrections, field additions (to match locked 19-field count), and n8n rule clarifications from Chat #74 verification.
