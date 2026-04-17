# Schema Registry v2

**Last Updated:** April 4, 2026  
**Base ID:** `pq249fix22t3ofv` (all tables)

---

## NocoDB Table Registry

| Agent | Table Name | Table ID | Record Count (Apr 2026) |
|-------|------------|----------|-------------------------|
| ALA | ALA | `m57efwbtrvwohhr` | ~300 |
| EIP | EIP | `mhicpnrahaesxmy` | ~300 |
| ESS | ESS | `m5yektnbtxf8evk` | ~300 |
| HSI | HSI | `mb8nv8t3nk6xzed` | ~300 |
| SIA | SIA | `mdn68l4lm609fve` | ~90 (clusters, not records) |
| BRA | BRA | `mwqejw7swhd2cf4` | ~300 |
| RDA | RDA | `mr1v67cszcklwns` | ~300 |
| MRA | MRA | `mybieof2em75t6e` | ~30 (reports, not records) |
| — | Template Library | `mafv9by73ebama7` | ~50 templates |
| — | Client Config | `m95cmabjfyb94ps` | ~3 clients |
| — | Emotion Dictionary | `mrrscb955j1d2i7` | 161 entries |
| — | Pain Point Master | `meavqh37mdqgl4d` | 336 entries |

---

## Critical Field IDs

### HSI Table
- **BRA Status:** `cl1250sz39sm45l` (updated by BRA after draft generation)

### BRA Table
- **Error Log:** `c3crsub617hnuee` (captures template selection or Claude API failures)

### MRA Table
- **MRA Run ID:** `czljrhljau7a8iv`
- **Report Type:** `cyqw1ki969a0vq0`
- **Client ID:** `chp1m03x1v1t3mv`
- **Total Records:** `cr3fr2dpxdncllz`
- **Avg Star Rating:** `cohbxugfjq590n1`
- **SLA Rate:** `cauxg0fskprxy70`
- **T-NEGATIVE:** `c7qu39qw9puutdq`
- **T-AMBIGUOUS:** `corkoggtozkumne`
- **T-POSITIVE:** `cg0u8la22vt06lc`
- **Delivery Status:** `ci6hytuki6u9i2j`
- **Magic Link Token:** `can5u62qbnmin1x`
- **Error Log:** `c5b2bybjo0equeb`
- **Platform Counts JSON:** `cixk8vliloo1rao`

---

## Data Flow Relationships
ALA (source)
↓
EIP (classifies emotions + pain points)
↓
ESS (analyzes expression) + HSI (synthesizes narrative)
↓                              ↓
SIA (aggregates)              BRA (generates draft)
↓
RDA (refines draft)
↓
Google Sheet (approval)
↓
MRA (reports)

---

## Pass-Through Fields

**Fields that travel through pipeline without modification:**

| Field | Origin | Passes Through | Purpose |
|-------|--------|----------------|---------|
| ALA Record ID | ALA | ALL agents | Traceability |
| Client ID | ALA | ALL agents | Client isolation |
| Review Text | ALA | EIP, ESS, HSI, BRA, RDA | Context for analysis |
| Star Rating | ALA | EIP, ESS, HSI, BRA, RDA | Tier assignment, severity calculation |
| Date Posted | ALA | SIA, MRA | Time-based aggregation |
| Platform | ALA | SIA, MRA | Platform distribution reporting |

---

## Computed Fields Registry

**Fields created by agents (not passed through):**

### ALA Outputs
- Normalized Text
- Language Detected
- Character Count

### EIP Outputs
- Enriched Emotion Tag
- Core Emotion
- Need State
- Cognitive Driver
- Enriched Emotion Breakdown JSON
- Pain Point Sub-Category
- Pain Point Domain Confirmed
- Signal Weight
- Enriched Pain Point
- Enriched Pain Point Breakdown JSON
- Signal Type (T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE)

### ESS Outputs
- Expression Mode
- Emotional Clarity
- Narrative Alignment Score

### HSI Outputs
- Hospitality Signal ID
- Signal Severity Score (1-10)
- Behavioral Narrative
- Operator Insight

### SIA Outputs
- Domain (cluster key)
- Signal Tier (cluster key)
- Count
- Percentage
- Enriched Pain Points JSON (frequency-sorted)
- Enriched Emotions JSON (frequency-sorted)
- Trend Direction (UP/DOWN/STABLE/NEW)

### BRA Outputs
- Tier Assignment (T1/T2/T3)
- Template ID (T1 only)
- Response Draft (initial)
- Generation Method (Template/Claude)

### RDA Outputs
- Response Draft (final, brand voice integrated)
- Internal Followup Draft
- Brand Voice Mode
- Guest Name Present (boolean)
- Approval Status (synced from Google Sheet)

### MRA Outputs
- Report Type (48hr/Weekly/Monthly)
- Total Records
- Avg Star Rating
- SLA Rate
- Platform Counts JSON
- Magic Link Token
- Delivery Status

---

## Field Naming Conventions

**Established patterns:**

- **IDs:** `[Agent]_Record_ID`, `[Entity]_ID`
- **Timestamps:** `Created_At`, `Updated_At`, `Approved_At`
- **Booleans:** `[Attribute]_Present`, `Is_[State]`
- **JSON:** `[Content]_Breakdown_JSON`, `[Content]_Counts_JSON`
- **Enums:** Single word or hyphenated (T-NEGATIVE, not T_NEGATIVE)

---

## Schema Modification Protocol

**Before modifying any schema:**

1. **Check downstream impact** - which agents read this field?
2. **Update Field Traceability Map** - document new field sources
3. **Version increment** - Schema Registry v2 → v3
4. **Test in isolation** - verify field addition doesn't break queries
5. **Deploy sequentially** - update agents in pipeline order (ALA → EIP → ESS → HSI → SIA/BRA → RDA → MRA)

---

**End of Schema Registry v2**
