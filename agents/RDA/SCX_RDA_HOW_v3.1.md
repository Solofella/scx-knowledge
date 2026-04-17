# SCX_RDA_HOW_v3.1

**Agent Name:** RDA (Response Drafting Agent)  
**Version:** 3.1  
**Last Updated:** Chat #73 · March 2026  
**Model:** Claude claude-sonnet-4-6  
**Status:** Verified operational - 18 nodes complete

---

## Purpose

RDA is the final agent in the SubtextCX pipeline. It generates client-ready response drafts with brand voice integration, tiered personalization (T1/T2/T3), and conditional guest name handling. Outputs go to Google Sheet approval workflow where clients mark Approved/Modify/Not Accepted.

---

## Input Sources

**Upstream Agent:** BRA (Brand Response Agent)

**Receives:**
- BRA Response Draft (template-based for T1, Claude-generated for T2/T3)
- HSI Hospitality Signal ID
- Star Rating (1-5)
- Review Text (original)
- Guest Name (if available)
- Client ID
- Brand Voice Settings (from Client Config NocoDB table)

---

## Processing Logic

### Node Flow (18 Nodes Total)

**INIT Nodes (1-3):** Webhook receive, payload parse, brand voice config fetch

**Brand Voice Integration (4-9):**
- Load client-specific brand voice settings from NocoDB Client Config table
- Apply brand voice mode: Standard / Warm / Professional / Casual
- Inject brand-specific phrases (max 1 per draft, variation constraint)

**Conditional Guest Name Handling (10-12):**
- **If guest name available:** Direct personal greeting variations
  - T1: Warm acknowledgment with 6 structural variations (gratitude lead, pleasure lead, guest word lead, specific detail lead, occasion lead, belonging lead)
  - T2/T3: "Hi [name]" or "Hello [name]"
- **If no guest name:** Open with specific signal detail from review (never generic "Thank you for your feedback")

**Draft Generation (13-16):**
- Claude API call with governance-constrained prompt
- System prompt enforces: detect/interpret only, never prescribe operational actions
- Output: `response_draft` field (client-facing) + `internal_followup_draft` field (signal meaning description for internal use)

**Output Nodes (17-18):**
- Write to RDA NocoDB table
- Pass to Google Sheet approval workflow via n8n webhook

---

## NocoDB Schema

**Table ID:** `mr1v67cszcklwns`

**Fields:**
- RDA Record ID (auto-increment, primary key)
- ALA Record ID (traceability through entire pipeline)
- Client ID
- Response Draft (final client-facing text)
- Internal Followup Draft (signal meaning description, not shown to client)
- Brand Voice Mode (Standard/Warm/Professional/Casual)
- Approval Status (Pending/Approved/Modify/Not Accepted)
- Guest Name Present (boolean)
- Tier (T1/T2/T3)
- Created At (timestamp)

---

## Brand Voice Integration

### Brand Voice Modes (Client Config)

**Standard:** Professional, service-focused, clear acknowledgment  
**Warm:** Personal, appreciative, relationship-building  
**Professional:** Formal, detail-oriented, quality commitment  
**Casual:** Friendly, conversational, approachable

### Brand Phrase Injection (Current Rule)

**Max 1 brand phrase per draft** (reduced from 3 to prevent over-repetition)

**Variation constraint:** RDA system prompt includes instruction to vary phrasing even when using brand terms

**Examples:**
- "At [Brand], we..." → vary to "Our team at [Brand]...", "[Brand] is committed to...", "We at [Brand]..."

---

## Conditional Guest Name Logic

### When Guest Name Available

**T1 drafts (warm acknowledgment):**

6 structural variations rotate:
1. **Gratitude lead:** "Thank you, [Name], for..."
2. **Pleasure lead:** "It's wonderful to hear from you, [Name]..."
3. **Guest word lead:** "Your feedback, [Name], means..."
4. **Specific detail lead:** "[Name], we're so glad you enjoyed..."
5. **Occasion lead:** "[Name], celebrating with us was..."
6. **Belonging lead:** "Welcome back, [Name]..."

**T2/T3 drafts:**
- Simple: "Hi [Name]," or "Hello [Name],"

### When Guest Name NOT Available

**Open with specific signal detail** from review instead of name:

**Good examples:**
- "We're glad the ribeye exceeded expectations..."
- "Your point about the noise level is important..."
- "Celebrating a milestone with us is always special..."

**Bad examples (never use):**
- "Thank you for your feedback" (too generic)
- "We appreciate your review" (template language)

---

## Governance Constraints (System Prompt)

**Critical rule enforced in RDA prompt:**

**Detect and interpret only. Never prescribe operational actions.**

**Internal followup draft** must describe what the signal means, not what the operator should do.

**Good internal draft:**
- "This guest's frustration with wait time suggests a capacity management signal during weekend dinner service."

**Bad internal draft (violates governance):**
- "Add more servers during weekend dinner rush." ← This is prescriptive, not allowed

---

## Token Budget

**~3,000 tokens per record**

Includes:
- Brand voice config fetch
- BRA draft input
- Review text context
- Claude API call for final draft generation

---

## Approval Workflow Integration

**RDA output → Google Sheet:**

Each draft written to client-specific Google Sheet with columns:
- Date / Platform / Stars / Review / Proposed (protected) / Edited / Status / RDA-ID

**Status values:**
- **Approved:** Client accepts draft as-is
- **Modify:** Client edits in "Edited" column
- **Not Accepted:** Client rejects, no response sent

**Apps Script onEdit webhook** fires to n8n → PATCHes RDA NocoDB Approval Status field

**Miguel marks Published** in dashboard (daily manual update)

---

## Open Issues

### Issue 1: Internal Brief Quality

**Problem:** `internal_followup_draft` sometimes includes operational prescriptions (violates governance)

**Fix needed:** Strengthen system prompt to describe signal meaning only

**Example fix:**
Current: "Add this item to the menu"
Fixed: "Guest expressed interest in menu item not currently offered - potential demand signal"

### Issue 2: Brand Voice Variation

**Problem:** Brand phrases repeat too frequently (currently allows up to 3 per draft)

**Fix needed:** Reduce to 1 phrase max, add variation constraint to system prompt

**Status:** Documented, not yet implemented

---

## Key Design Decisions

### Hybrid Guest Name Strategy

**Why conditional logic instead of always using names?**

Testing showed:
- T1 guests respond better to personalized openings when name available
- T2/T3 guests prefer substance over formality
- When no name available, leading with specific signal detail performs better than generic "Thank you for your review"

### Brand Voice as Enhancement, Not Override

Brand voice settings enhance the draft but never override:
- Governance constraints (no prescriptive language)
- Signal accuracy (don't fabricate positives)
- Tier-appropriate tone (T3 stays formal even in "Casual" brand mode)

---

## Downstream Handoff

**RDA → Google Sheet Approval Workflow:**
- Draft written to sheet (protected "Proposed" column)
- Client edits in "Edited" column if needed
- Status tracked (Approved/Modify/Not Accepted)
- Webhook fires on edit → n8n updates RDA NocoDB

**RDA → Dashboard (read-only):**
- Approval counts displayed (Approved vs Pending)
- Draft text NOT shown to client in dashboard (only in Google Sheet)

---

## Related Documents

- **Changelog:** [SCX_RDA_CHANGELOG.md](SCX_RDA_CHANGELOG.md)
- **Upstream Agent:** [../BRA/SCX_BRA_HOW_v3.2.md](../BRA/SCX_BRA_HOW_v3.2.md)
- **Client Config Table:** Client brand voice settings stored in NocoDB `m95cmabjfyb94ps`
- **Approval Workflow:** Google Sheet per client (shared with solofellausa@gmail.com)

---

## n8n Workflow Details

**Workflow Name:** SCX-RDA  
**Trigger:** Webhook POST from BRA  
**Credentials:** NocoDB xc-token, Anthropic API key

**Critical Rules:**
- Claude API manual headers: `anthropic-version:2023-06-01` + `Content-Type:application/json`
- Brand voice config must be fetched BEFORE Claude call (not cached, loaded per record)
- Guest name conditional: IF node checks for name presence, branches to different prompt variations

---

**End of RDA HOW v3.1**
