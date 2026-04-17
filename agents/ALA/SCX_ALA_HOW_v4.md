# SCX_ALA_HOW_v4.0

**Agent Name:** ALA (Acquisition & Language Agent)  
**Version:** 4.0  
**Last Updated:** Chat #74 · April 4, 2026  
**Model:** GPT gpt-5.2  
**Status:** Verified operational - 24 nodes complete

---

## Purpose

ALA is the first agent in the SubtextCX pipeline. It normalizes incoming guest reviews from multiple platforms, standardizes text formatting, detects language, and prepares clean input for downstream classification by EIP.

---

## Input Sources

- **Google Business Profile** (API - Phase 1b)
- **Yelp** (API - Phase 1b)
- **OpenTable** (partner application pending)
- **TripAdvisor** (developer program pending)
- **Manual CSV upload** (Phase 1 pilot method)

**Phase 1 Pilot:** Manual CSV upload per client (PAK-001, EDO-001, AJI-001)

---

## Processing Logic

### Node Flow (24 Nodes Total)

**INIT Nodes (1-4):** Webhook receive, payload parse, client validation, duplicate check

**Processing Nodes (5-18):** 
- Text normalization via GPT gpt-5.2
- Language detection
- Character count
- Format cleanup (remove HTML, standardize punctuation)
- Preserve semantic meaning

**Output Nodes (19-24):** 
- Write to NocoDB ALA table
- Pass record to EIP webhook
- Error logging if failures occur

---

## NocoDB Schema

**Table ID:** `m57efwbtrvwohhr`

**Fields:**
- ALA Record ID (auto-increment, primary key)
- Client ID (PAK-001, EDO-001, etc.)
- Platform (Google, Yelp, OpenTable, Manual)
- Date Posted (review publication date)
- Star Rating (1-5)
- Review Text (original)
- Reviewer Handle (guest identifier)
- Normalized Text (ALA output - clean, standardized)
- Language Detected (ISO code)
- Character Count
- Created At (timestamp)

---

## Token Budget

**~400 tokens per record** (GPT normalization only, no dictionary access)

---

## Downstream Handoff

**ALA → EIP:** 
- Normalized text passed via webhook
- ALA Record ID travels through entire pipeline for traceability
- EIP receives clean input ready for emotion/pain point classification

---

## Key Design Decisions

### Duplicate Detection
Uses `pageInfo.totalRows` from NocoDB API (not `list.length`) to accurately count existing records before insertion.

### Language Support
Phase 1: English + Spanish (US + Peru markets)

### Error Handling
All processing errors logged to ALA table error field. Failed records do not block batch processing.

---

## Open Issues

**BCA Integration (Deferred):**
Option B design documented - BCA will read from ALA NocoDB table and drip records to EIP at controlled rate (10 records per batch) to prevent API rate limit losses.

Currently: Direct pass-through from ALA to EIP (no batch control). Risk: High-volume uploads may lose records if rate limits hit.

---

## n8n Workflow Details

**Workflow Name:** SCX-ALA  
**Trigger:** Webhook POST (JSON body)  
**Credential:** NocoDB xc-token, OpenAI API key  
**Rate Limit Handling:** Wait Between Tries = 5000ms max

**Critical Rules:**
- No spread operator in Code Nodes (key-by-key assignment only)
- IF node both branches fire → gate FALSE branch with `return []`
- Boolean conditions → use Code Node (IF node unreliable)
- Payload parse: `typeof body === 'string' ? JSON.parse(body) : body`

---

## Related Documents

- **Changelog:** [SCX_ALA_CHANGELOG.md](SCX_ALA_CHANGELOG.md)
- **Schema:** [ALA_Schema.md](ALA_Schema.md)
- **Pre-Build Protocol:** [../../protocols/SCX_PreBuild_Protocol_v1.0.md](../../protocols/SCX_PreBuild_Protocol_v1.0.md)
- **Downstream Agent:** [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md)
