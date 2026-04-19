CLIENT CONFIG v1.0 — PAK-001 RECORD SPECIFICATION
For NocoDB Table ID: m95cmabjfyb94ps
Effective: Chat #76 · April 19, 2026
Status: READY FOR NOCODB ENTRY
FieldTypeValueNotesClient IDSingleLineTextPAK-001Unique identifier. Locked. No changes.Client NameSingleLineTextPark Avenue Kitchen by David BurkeDisplay name for internal reference + audit trails + approval emails.Tone DescriptorsLongTextchef-driven, warm, polished, unpretentious, New YorkSYNTHETIC PLACEHOLDER. Christine (GM, PAK) received Brand Voice Discovery questionnaire in Chat #75. Await her response before updating. Current values are calibrated examples only.Vocabulary RestrictionsLongTextper your request, moving forward, at this timeSYNTHETIC PLACEHOLDER. Await Christine's response. These are hospitality-common restrictions. Update when questionnaire returns.Formality LevelNumber (1–5)4SYNTHETIC PLACEHOLDER. Chef-driven, fine-dining context = high formality (4 out of 5). Adjust downward if Christine indicates casual/warm preference. Range: 1 (highly casual) to 5 (highly formal).Person PreferenceSingleLineTextour teamSYNTHETIC PLACEHOLDER. Fine-dining brands typically use "our team" or "we" rather than brand name alone. Confirm with Christine.Brand Phrases To IncludeLongTextThank you for dining with us, We'd love to welcome you back, We take great pride in every guest experienceSYNTHETIC PLACEHOLDER. Three phrases provided. RDA constraint: ONE phrase maximum per draft, selected for contextual naturalness only. Await Christine's actual brand voice preferences.Brand Phrases To AvoidLongTextWe apologize for any inconvenience, As per, Please be advised, Unfortunately, complaint, issue, problem, incident, We take this very seriously, Loop you inLOCKED. From RDA v3.1 ABSOLUTE PROHIBITIONS. "We apologize for any inconvenience" is permanently banned for all clients (T3 governance rule). Other terms are hospitality-standard restrictions. Do not remove. Add client-specific restrictions from Christine if needed.Language (fallback)SingleLineTextenEnglish. Fallback only—record-level lang from ALA pipeline takes priority. PAK is US-based. Confirm with Christine.SEO KeywordsLongTextPark Avenue Kitchen, David Burke, East Midtown dining, contemporary American cuisine, New York restaurant, chef-driven dining, Midtown Manhattan restaurant, special occasion dining New YorkSYNTHETIC PLACEHOLDER. Eight keywords provided for v3.1 SEO feature (Step 7f). RDA weaves 1–2 naturally into public response drafts. Await Christine's SEO priorities. If she has existing keyword targets, replace entirely.Approval Contact EmailEmailmarellano@solofella.comPLACEHOLDER — CRITICAL BLOCKING ISSUE. This field is REQUIRED before first PAK pilot batch runs. Christine must provide the email address of the person who will review and approve response drafts. Typical: GM, Operations Manager, or Marketing Manager responsible for review reputation. UPDATE REQUIRED before Step 15 (NocoDB POST in RDA workflow).

FIELD-BY-FIELD ENTRY NOTES FOR NOCODB UI
When entering this record into NocoDB m95cmabjfyb94ps:

Client ID: Type exactly PAK-001 (case-sensitive in filter queries)
Client Name: Type full legal name for audit trail clarity
Tone Descriptors: Copy-paste from above; separate by comma; no trailing commas
Vocabulary Restrictions: Copy list; use "," as separator; these words trigger RDA governance warnings (monitored but not auto-blocked)
Formality Level: Enter numeric value 4 (single integer, no text)
Person Preference: Enter exactly our team (matches RDA template engine lookups)
Brand Phrases To Include: Copy three-phrase list; RDA selects max ONE per draft based on contextual fit
Brand Phrases To Avoid: Do NOT edit this list. Lock it. This is operational governance. If Christine adds restrictions, append to a NEW field or document separately
Language (fallback): Enter en (two-letter ISO code)
SEO Keywords: Copy list; format as comma-separated; RDA Step 7f weaves 1–2 naturally or omits if no natural fit
Approval Contact Email: BLOCKING. Cannot proceed to live pilot without this email address.


DEPENDENCY CHAIN — BEFORE FIRST PAK PILOT RUN
#DependencyStatusOwnerBlocker?1Client Config PAK-001 record created in NocoDB⏳ PENDINGMiguelNO — can exist without values2Brand Voice Discovery questionnaire returned by Christine⏳ PENDINGChristine (PAK GM)YES — blocks quality baseline3Approval Contact Email populated⏳ PENDINGChristineYES — STRUCTURAL BLOCKER4First real review batch (OpenTable + Google) provided to Miguel⏳ PENDINGChristineNO — synthetic data can run for testing5RDA v3.1 workflow verified with PAK-001 client_id⏳ PENDINGMiguelYES — blocks pilot launch
Critical path: Approval Contact Email must arrive before RDA Step 18 (Human Approval Notification) can fire.

MULTI-CLIENT ARCHITECTURE PATTERN
This PAK-001 record is the template pattern for EDO-001 and AJI-001:
For EDO-001 (EDO Restaurants, Peru):
- Client ID: EDO-001
- Client Name: EDO Restaurants
- Language: es (Spanish pilot)
- Approval Contact Email: [TBD]
- Tone Descriptors: [TBD — Latin American hospitality norms]
- Other fields: follow same schema

For AJI-001 (Aji Ceviche, Florida):
- Client ID: AJI-001
- Client Name: Aji Ceviche (location: Orlando FL)
- Language: es (Spanish-dominant brand)
- Tone Descriptors: [TBD — ceviche restaurant context]
- SEO Keywords: [TBD — Orlando local SEO focus noted as differentiator]
- Approval Contact Email: [TBD]
No code changes required. One new NocoDB row per client. RDA reads client_id from pipeline payload, fetches the correct row, applies that client's brand voice.

VERSION CONTROL
VersionDateChangesSourcev1.0Chat #76 · Apr 19 2026Initial entry specification. Synthetic placeholders identified. Approval Contact Email flagged as structural blocker.MCD v7.4 Section 11 + RDA v3.1 Section 8 + Dashboard Brief Section 3
Next update trigger: When Christine returns Brand Voice Discovery questionnaire (Chat #77 or later).

GOVERNANCE LOCK
DO NOT EDIT Brand Phrases To Avoid list without explicit Miguel approval. This list enforces RDA v3.1 governance constraints locked in Chat #74–75.
DO EDIT all synthetic placeholder fields once real data arrives:

Tone Descriptors
Vocabulary Restrictions
Formality Level
Person Preference
Brand Phrases To Include
SEO Keywords
Approval Contact Email (CRITICAL — REQUIRED)
