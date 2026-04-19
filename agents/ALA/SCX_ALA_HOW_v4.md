# SCX_ALA_HOW_v4.1

**Agent Name:** ALA (Audience Listener Agent)  
**Version:** 4.1  
**Last Updated:** Chat #76 · April 19, 2026  
**Model:** GPT gpt-5.2 (LOCKED Chat #64)  
**Status:** VERIFIED — 24 nodes complete · All Chat #74 changes applied

---

## PURPOSE

ALA is the first agent in the SubtextCX pipeline. It ingests guest reviews from multiple sources, normalizes text, detects language, validates data integrity, and prepares clean records for emotional intelligence processing by EIP. Every record carries a Client ID and ALA Record ID through the entire pipeline for traceability.

---

## INPUT CONTRACT — PHASE 1

**Pilot Method (Q2–Q3 2026):**
- Manual CSV upload per client (PAK-001, EDO-001, AJI-001)
- One CSV per batch ingestion event

**Phase 1b (Planned — APIs):**
- Google Business Profile API
- Yelp Fusion API  
- OpenTable Partner Application (pending submission)
- TripAdvisor Developer Program (pending application)

**Phase 1b Unified Ingestion Architecture:**
All platform adapters normalize to one Google Sheet per client:

| Column | Type | Notes |
|--------|------|-------|
| Date | Date | Review publication date |
| Platform | Text | Google/Yelp/OpenTable/TripAdvisor/Manual |
| Star Rating | Number | 1–5 integer |
| Review Text | LongText | Full guest review text |
| Reviewer Handle | Text | Guest identifier or "Anonymous" |
| Client ID | Text | **REQUIRED** — PAK-001, EDO-001, etc. |
| Ingestion Status | Text | Pending/Processed/Duplicate/Error |

**Critical Rule (Chat #74):** Client ID must be a column in the source CSV/Sheet. Cannot be introduced mid-pipeline.

---

## PROCESSING LOGIC — 24 NODES

### Architecture
1. **INIT (Nodes 1–4):** Webhook receive, payload parse, client validation
2. **PROCESSING (Nodes 5–18):** Normalization, language detection, deduplication, format cleanup
3. **OUTPUT (Nodes 19–24):** NocoDB write, EIP trigger, error handling

### Key Processing Rules

**Batch Size:** SplitInBatches = 1 (Fix 1, Chat #74)
- Prevents simultaneous GPT calls that cause record loss at scale
- Each record processes sequentially through full pipeline

**Text Normalization:**
- Remove HTML tags, extra whitespace, non-UTF-8 characters
- Preserve semantic meaning — no aggressive truncation
- Character count calculation (ALA field)

**Language Detection:**
- ISO 639-1 codes (en, es, etc.)
- Phase 1: English + Spanish (US + Peru markets)
- Passes to downstream agents as `lang` field

**Deduplication:**
- Hash-based: `hash(Reviewer Handle + Date + Platform + Review Text)`
- Uses `pageInfo.totalRows` (NOT `list.length`) for accurate NocoDB count

**Client ID Routing:**
- Extracted from CSV column, required at Source
- Carried through all 24 nodes via key-by-key assignment (no spread operator)
- Stored in ALA NocoDB record — permanent audit trail

---

## NOCODB SCHEMA — m57efwbtrvwohhr

**Field Count:** 19 fields (Chat #74 verified)

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| ALA Record ID | AutoNumber | YES | Primary key. Travels through entire pipeline. |
| Client ID | SingleLineText | YES | **Chat #74 addition.** Required. Links to Client Config. |
| Platform | SingleLineText | YES | Source: Google / Yelp / OpenTable / TripAdvisor / Manual |
| Date Posted | DateTime | YES | Review publication date (ISO 8601) |
| Star Rating | Number | YES | 1–5 integer |
| Review Text (original) | LongText | YES | Guest-written text, unmodified |
| Reviewer Handle | SingleLineText | NO | Guest name or identifier from source |
| Raw Tex | LongText | YES | **CURRENT FIELD NAME (truncated).** DO NOT rename to "Raw Text" until post-pilot. Contains normalized, cleaned review text. |
| Language Detected | SingleLineText | YES | ISO 639-1 code (en, es, etc.) |
| Character Count | Number | YES | Length of normalized text |
| Dedup Hash | SingleLineText | YES | SHA256(Reviewer Handle + Date + Platform + Review Text) |
| Ingestion Status | SingleLineText | YES | Pending / Processed / Duplicate / Error |
| Created At (ALA) | DateTime | YES | Timestamp when record entered ALA |
| Error Log | LongText | NO | Null if clean; populated if processing fails |
| (Reserved 1–5) | — | — | Unused fields; do not repurpose |

---

## CRITICAL RULES — n8n IMPLEMENTATION

### GPT Model Parameters
```
Model: gpt-5.2
max_completion_tokens: 400 (NOT max_tokens — gpt-5.2 blocks max_tokens)
temperature: 0
Retry: true, Max tries: 3, Wait Between Tries: 5000ms
```

**Why:** gpt-5.2 parameter specification differs from Claude. Using `max_tokens` throws 400 error.

### Batch Processing
```
SplitInBatches node: size = 1 (LOCKED Chat #74)
Last processing node MUST wire back to SplitInBatches input
Never wire to webhook trigger — only to SplitInBatches
```

**Why:** Prevents simultaneous LLM calls. High-volume tests lost 15 records with batch size > 1 due to API rate limits.

### CSV Handling
```javascript
// Correct: handles both Windows (CRLF) and Unix (LF) line endings
const lines = csvText.split(/\r?\n/);

// Wrong: fails on Windows line endings
const lines = csvText.split("\n");
```

### Payload Construction
```javascript
// CORRECT: Key-by-key assignment
return [{ json: {
  ala_record_id: parsed.id,
  client_id: parsed.client_id,
  platform: parsed.platform,
  // ... all fields explicitly assigned
} }];

// WRONG: Spread operator (blocked by task runner)
return [{ json: { ...parsed } }];
```

### Webhook Authentication
- Auth: NONE (all agent webhooks use Auth = None)
- Body type: JSON (not RAW)

---

## DOWNSTREAM HANDOFF → EIP

**Payload Contents:**
- ALA Record ID (primary key for traceability)
- Client ID (routing)
- Normalized review text (`Raw Tex` field)
- Language code (`lang`)
- Star Rating
- Platform
- Reviewer Handle
- All metadata

**Trigger:** Step 14 → n8n Webhook POST to SCX-EIP webhook path
- Fire-and-forget architecture
- No blocking waits for EIP response
- Error handler logs failures to ALA Error Log

---

## TOKEN BUDGET & COST

**Per Record:** ~400 tokens (GPT normalization only)
- No dictionary injection
- No emotion analysis at ALA stage
- Pure text standardization

**Cost Model:** Negligible baseline cost. Scales linearly. No caching applied at ALA.

---

## FIELD TRACEABILITY MAP (Chat #74 Protocol)

**Source → Processing → Storage → Output → Feedback**

| Source Field | Node Declared | Nodes Used | NocoDB Field | Downstream Field |
|---|---|---|---|---|
| Date | CSV col 1 | Init → Norm | Date Posted | EIP Date |
| Platform | CSV col 2 | Init → Store | Platform | EIP Platform |
| Star Rating | CSV col 3 | Init → Store | Star Rating | EIP Star Rating |
| Review Text | CSV col 4 | Init → Norm → Clean | Review Text | EIP Raw input |
| Reviewer Handle | CSV col 5 | Init → Store | Reviewer Handle | RDA Opening (Step 7a) |
| **Client ID** | **CSV col 6** | **Init → All 24 nodes** | **Client ID** | **All agents (routing)** |
| Ingestion Status | CSV col 7 | Init → Update | Ingestion Status | Audit trail |

**Lesson Locked:** Client ID was required at RDA Output (approval notifications, Client Config lookup) but was never declared at ALA Source. This caused a pipeline break at Chat #74. New rule: **Any field required at Output must be declared at Source before Node 1.**

---

## KNOWN ISSUES & MITIGATIONS

| Issue | Status | Mitigation |
|-------|--------|-----------|
| BCA (Batch Controller Agent) — 10-record drip control | Deferred | Option B design documented. Not needed at 10–15 records/day pilot volume. |
| Raw Tex field name truncated | Post-pilot cleanup | Do not rename until after first 3 pilots complete. Renaming breaks existing workflows. |
| High-volume CSV uploads (>100 records/run) | Phase 1b | Phase 1 manual uploads capped at ~50 records per batch to stay under rate limits. |

---

## OPEN ITEMS

- [ ] Google Business Profile API integration (Phase 1b)
- [ ] Yelp Fusion API integration (Phase 1b)
- [ ] OpenTable Partner Application (submit to partners.opentable.com)
- [ ] TripAdvisor Developer Program (apply when ready)
- [ ] BCA Option B build (post-pilot) — reads from ALA NocoDB, drips to EIP
- [ ] Raw Tex → Raw Text rename (post-pilot cleanup)
- [ ] First Spanish-language batch test (EDO-001 or AJI-001)

---

## RELATED DOCUMENTS

- **MCD v7.4:** Master Continuity Document (Cold Start, Chat #75)
- **SCX_PreBuild_Protocol_v1.0:** Field Traceability Map requirement
- **SCX_EIP_HOW_v4.0:** Downstream agent (emotion processing)
- **Build Lessons (31 locked + Chat #74 additions):** n8n rules, API constraints

---

**Subtext CX · SCX_ALA_HOW_v4.1 · Chat #76 · April 19, 2026 · Solofella LLC**

---

## WHAT CHANGED (v4.0 → v4.1)

✅ **Added:** Client ID as required CSV column + all 24 nodes + NocoDB field  
✅ **Added:** Field Traceability Map (Chat #74 protocol)  
✅ **Added:** Phase 1b unified ingestion architecture (7-column Google Sheet pattern)  
✅ **Added:** Critical GPT parameter rule (max_completion_tokens, not max_tokens)  
✅ **Added:** SplitInBatches=1 rule with reasoning  
✅ **Added:** Batch processing wiring rule  
✅ **Clarified:** CSV line-ending handling (CRLF vs LF)  
✅ **Clarified:** Payload construction (no spread operator)  
✅ **Clarified:** Known issue about Raw Tex truncation  
✅ **Updated:** Status to reflect all Chat #74 changes verified  

**No structural changes to the 24-node workflow.** All updates are documentation corrections, parameter clarifications, and traceability rules locked at Chat #74.
