# ALA Schema

**NocoDB Table ID:** `m57efwbtrvwohhr`  
**Base ID:** `pq249fix22t3ofv`

---

## Table Fields

| Field Name | Type | Purpose | Notes |
|------------|------|---------|-------|
| ALA Record ID | Auto-increment | Primary key | Travels through entire pipeline for traceability |
| Client ID | SingleLineText | Client identifier | PAK-001, EDO-001, AJI-001, etc. |
| Platform | SingleSelect | Source platform | Google, Yelp, OpenTable, TripAdvisor, Manual |
| Date Posted | Date | Review publication date | Original timestamp from platform |
| Star Rating | Number | 1-5 rating | Integer value |
| Review Text | LongText | Original review content | Unmodified guest text |
| Reviewer Handle | SingleLineText | Guest identifier | Anonymized if needed for privacy |
| Normalized Text | LongText | ALA-processed output | Cleaned, standardized, semantic-preserving |
| Language Detected | SingleLineText | ISO language code | en, es, etc. |
| Character Count | Number | Review length | Used for complexity analysis |
| Error Log | LongText | Processing errors | Captures failures without blocking pipeline |
| Created At | DateTime | Record creation timestamp | Auto-generated |

---

## Relationships

**Upstream:** None (ALA is pipeline entry point)

**Downstream:** 
- **EIP** reads ALA records via NocoDB API query
- ALA Record ID passed through entire pipeline (ALA → EIP → ESS → HSI → SIA → BRA → RDA)

---

## Processing Logic

### Normalization (GPT gpt-5.2)
1. Remove HTML tags and formatting artifacts
2. Standardize punctuation and spacing
3. Preserve original semantic meaning
4. Fix common typos without changing intent
5. Detect primary language (ISO code)

### Duplicate Detection
Uses `pageInfo.totalRows` from NocoDB API to count existing records before insertion.

**Check logic:**
```javascript
const existingCount = nocodb_response.pageInfo.totalRows;
if (existingCount > 0) {
  return { duplicate: true, skip: true };
}
```

### Error Handling
Failed records logged to Error Log field. Pipeline continues processing remaining records.

---

## Token Budget

**~400 tokens per record**
- No dictionary access at ALA layer
- Only normalization and language detection

---

## Data Flow
CSV Upload / API Pull
↓
ALA Webhook (JSON payload)
↓
Parse & Validate Client ID
↓
Duplicate Check (NocoDB query)
↓
GPT Normalization
↓
Write to ALA Table
↓
Pass to EIP Webhook (ALA Record ID + Normalized Text)

---

## Key Design Decisions

### Pass-Through Architecture
ALA writes to NocoDB but also immediately passes data to EIP via webhook. This creates both:
- **Persistent record** (NocoDB table for audit trail)
- **Live processing** (webhook to EIP for real-time pipeline flow)

### BCA Integration Point (Future)
When BCA is built, it will:
1. Read from ALA NocoDB table (not CSV upload)
2. Drip records to EIP at controlled rate (10 per batch)
3. Prevent API rate limit losses at high volume

Currently: Direct ALA → EIP pass-through (no batch control)

---

## Related Documents

- **HOW Document:** [SCX_ALA_HOW_v4.md](SCX_ALA_HOW_v4.md)
- **Changelog:** [SCX_ALA_CHANGELOG.md](SCX_ALA_CHANGELOG.md)
- **Schema Registry:** [../../schemas/Schema_Registry_v2.md](../../schemas/Schema_Registry_v2.md)
- **Downstream Agent:** [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md)
