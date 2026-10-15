Chat #388 · October 14, 2026

**Verification status:** Direct observation — I opened the current official Anthropic, OpenAI, and DigitalOcean pages today. ✅ **HIGH** for facts cited below. I found several areas where the documentation does **not** support a stronger statement; I flag those rather than infer.

Two important findings before the detail:

- **Anthropic and OpenAI standard paid APIs do not use API inputs/outputs for model training by default.**
- **DigitalOcean Droplets require careful wording:** DigitalOcean says Droplet **virtual disks are not encrypted at rest**, and automated Droplet **backups are not encrypted at rest**. Do not publish a blanket statement that VRYOH's DigitalOcean-hosted data is encrypted at rest.

---

# A. ANTHROPIC — Claude API

## 1. Is standard API data used to train or improve models?

**CLAIM:** No, not by default.

**EVIDENCE FOR — page states:** Anthropic's Commercial Terms expressly prohibit model training on Customer Content. Its current Privacy Center also says commercial/API inputs and outputs are not used for training by default. Explicit feedback or voluntary data-sharing can be used.

> “By default, we will not use your inputs or outputs from our commercial products… to train our models.”

**Source:** [https://privacy.claude.com/en/articles/7996868-is-my-data-used-for-model-training](https://privacy.claude.com/en/articles/7996868-is-my-data-used-for-model-training?utm_source=chatgpt.com)  
**Page date:** August 18, 2026. [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/7996868-is-my-data-used-for-model-training)

Also:

> “Anthropic may not train models on Customer Content from Services.”

**Source:** [https://www.anthropic.com/legal/commercial-terms](https://www.anthropic.com/legal/commercial-terms?utm_source=chatgpt.com)  
**Effective:** June 17, 2025. [Anthropic](https://www.anthropic.com/legal/commercial-terms)

**EVIDENCE AGAINST / exception:** Anthropic says data may be used if you explicitly submit feedback/bugs or otherwise choose to allow such use. [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/7996868-is-my-data-used-for-model-training)

**CONCLUSION:** ✅ **HIGH.** For a normal standard pay-as-you-go API account, you do **not** need to opt out of model training. The default is no training. I cannot see VRYOH's actual Anthropic account settings, so I cannot verify whether someone has affirmatively joined an optional data-sharing program.

---

## 2. Is input/output retained? How long, why, and who can access it?

**CLAIM:** Standard API inputs and outputs are normally deleted from Anthropic's backend within 30 days.

> “For Anthropic API users, we automatically delete inputs and outputs on our backend within 30 days of receipt or generation…”

**Source:** [https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data?utm_source=chatgpt.com)  
**Page date:** July 1, 2026. [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data)

The documented exceptions are longer-retention features such as Files API, an agreed alternative such as ZDR, enforcement of the Usage Policy, and compliance with law. [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data)

On access, Anthropic's Commercial Terms classify Customer Content as confidential and restrict disclosure:

> “Recipient may only share… Confidential Information to… employees, agents, and advisors that have a need to know…”

**Source:** [https://www.anthropic.com/legal/commercial-terms](https://www.anthropic.com/legal/commercial-terms?utm_source=chatgpt.com)  
**Effective:** June 17, 2025. [Anthropic](https://www.anthropic.com/legal/commercial-terms)

**Inference:** Anthropic does **not** publicly identify the exact internal job titles or teams that may manually inspect standard API review text. I therefore would not write “only the safety team can see our data.”

**CONCLUSION:** ✅ **HIGH** for the 30-day default and documented exceptions. ⚠️ **MEDIUM** for describing specific human-access roles beyond need-to-know authorized personnel/agents because Anthropic does not publish the same detailed access-role wording OpenAI does.

---

## 3. Is Zero Data Retention available?

**CLAIM:** Yes, but it is not part of the ordinary self-service API default.

Anthropic states:

> “Some enterprise API customers, subject to Anthropic approval, may have arrangements under which Anthropic does not store their inputs or outputs…”

It also says those arrangements remain subject to legal/misuse exceptions and retain UserSafety classifier results.

**Source:** [https://privacy.claude.com/en/articles/8956058-i-have-a-zero-data-retention-agreement-with-anthropic-what-products-does-it-apply-to](https://privacy.claude.com/en/articles/8956058-i-have-a-zero-data-retention-agreement-with-anthropic-what-products-does-it-apply-to?utm_source=chatgpt.com)  
**Page displays:** “Updated over 2 weeks ago”; it does not provide an exact calendar date in the current page metadata. [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/8956058-i-have-a-zero-data-retention-agreement-with-anthropic-what-products-does-it-apply-to?utm_source=chatgpt.com)

Anthropic explicitly directs current or prospective enterprise API customers with ZDR questions to its Sales Team. [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/8956058-i-have-a-zero-data-retention-agreement-with-anthropic-what-products-does-it-apply-to?utm_source=chatgpt.com)

**CONCLUSION:** ✅ **HIGH.** Your ordinary VRYOH pay-as-you-go account should **not be described as ZDR**. Anthropic describes ZDR as an approved arrangement for some enterprise API customers. The verified request path is Anthropic Sales.

---

## 4. Do these rules apply to standard API accounts without an enterprise contract?

**CLAIM:** Yes.

> “They govern Customer's use of Anthropic API keys…”

The Commercial Terms specifically cover commercial API use, distinguish it from consumer Claude products, and provide standard pricing-page billing. [Anthropic](https://www.anthropic.com/legal/commercial-terms)

**Source:** [https://www.anthropic.com/legal/commercial-terms](https://www.anthropic.com/legal/commercial-terms?utm_source=chatgpt.com)  
**Effective:** June 17, 2025.

**CONCLUSION:** ✅ **HIGH.** These core API terms apply to VRYOH's described standard commercial/API usage.

---

## 5. Is there a DPA? Is it automatic?

**CLAIM:** Yes. The Anthropic DPA is automatically incorporated into the Commercial Terms; no separate signature is required for ordinary first-party API use.

> “When you accept Anthropic's Commercial Terms of Service, you also accept our DPA.”

**Source:** [https://privacy.claude.com/en/articles/7996862-how-do-i-view-and-sign-your-data-processing-addendum-dpa](https://privacy.claude.com/en/articles/7996862-how-do-i-view-and-sign-your-data-processing-addendum-dpa?utm_source=chatgpt.com)  
**Page date:** March 16, 2026. [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/7996862-how-do-i-view-and-sign-your-data-processing-addendum-dpa)

The actual DPA says:

> “This Data Processing Addendum… is incorporated into and forms part of the Anthropic Commercial Terms of Service…”

**Source:** [https://www.anthropic.com/legal/data-processing-addendum](https://www.anthropic.com/legal/data-processing-addendum?utm_source=chatgpt.com)  
**Effective:** February 24, 2025. [Anthropic](https://www.anthropic.com/legal/data-processing-addendum)

**CONCLUSION:** ✅ **HIGH.**

---

## 6. Do they publish subprocessors?

**CLAIM:** Yes.

**Source:** [https://trust.anthropic.com/subprocessors](https://trust.anthropic.com/subprocessors?utm_source=chatgpt.com)

Anthropic's Trust Center currently lists subprocessors including Google Cloud Platform, AWS, Microsoft Azure and Cloudflare. [Anthropic Trust Center](https://trust.anthropic.com/?gad=1\&utm_source=chatgpt.com)

Importantly, the list **changed today**:

> “We've updated our subprocessor list with two additions…”

The additions announced October 14, 2026 are Parallel Web Systems and hCaptcha/Intuition Machines. [Anthropic Trust Center](https://trust.anthropic.com/?gad=1\&utm_source=chatgpt.com)

**Last update verified:** October 14, 2026.

**CONCLUSION:** ✅ **HIGH.** The subprocessor list is current and changed on the same date as this review.

---

## 7. Legal-hold or court-order exceptions?

**CLAIM:** Yes, although I did not find Anthropic using the exact phrase “legal hold.”

Anthropic permits retention beyond 30 days “in compliance with the law.” [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data)

Its Commercial Terms also say:

> “Recipient may disclose… Confidential Information to the extent it is required by law, or court or administrative order…”

**Source:** [https://www.anthropic.com/legal/commercial-terms](https://www.anthropic.com/legal/commercial-terms?utm_source=chatgpt.com)  
**Effective:** June 17, 2025. [Anthropic](https://www.anthropic.com/legal/commercial-terms)

**CONCLUSION:** ✅ **HIGH** for legal/court-order exceptions. Do not specifically say “legal hold” unless you phrase it more generally as “where retention is required by law.”

---

# B. OPENAI — API

## 1. Is standard API data used to train/improve models?

**CLAIM:** No, not by default.

> “OpenAI will not use Customer Content to develop or improve the Services, unless Customer explicitly agrees to such use.”

**Source:** [https://openai.com/policies/services-agreement/](https://openai.com/policies/services-agreement/?utm_source=chatgpt.com)  
**Updated:** December 1, 2025. **Effective:** January 1, 2026. [OpenAI](https://openai.com/policies/services-agreement/)

OpenAI's current API data-sharing article is even more explicit:

> “By default, data sharing for inputs and outputs is disabled for all organizations.”

**Source:** [https://help.openai.com/en/articles/10306912-sharing-feedback-evaluation-and-fine-tuning-data-and-api-inputs-and-outputs-with-openai](https://help.openai.com/en/articles/10306912-sharing-feedback-evaluation-and-fine-tuning-data-and-api-inputs-and-outputs-with-openai?utm_source=chatgpt.com)  
**Page displays:** “Updated: 6 days ago”; no absolute calendar date is displayed. [OpenAI Help Center](https://help.openai.com/en/articles/10306912-sharing-feedback-evaluation-and-fine-tuning-data-and-api-inputs-and-outputs-with-openai)

**CONCLUSION:** ✅ **HIGH.** No opt-out action is required under the default configuration. Opt-in data sharing can be enabled by an organization owner, so I cannot verify VRYOH's actual account setting from public documentation.

---

## 2. Is API data retained? For how long, why, and who can access it?

**CLAIM:** By default, abuse-monitoring logs may retain API customer content for up to 30 days.

OpenAI states:

> “OpenAI may securely retain API inputs and outputs for up to 30 days to provide the services and to identify abuse.”

**Source:** [https://openai.com/enterprise-privacy/](https://openai.com/enterprise-privacy/?utm_source=chatgpt.com)  
**Updated:** January 8, 2026. [OpenAI](https://openai.com/enterprise-privacy/)

The API-specific data controls documentation confirms `/v1/chat/completions` and `/v1/responses` have 30-day abuse-monitoring retention by default. [OpenAI Developers](https://developers.openai.com/api/docs/guides/your-data)

**Important endpoint caveat:** the Responses API can also create **application state**. OpenAI says Responses API state has a 30-day retention period by default in applicable configurations. Other endpoints such as files, conversations, assistants or vector stores have different persistence rules. [OpenAI Developers](https://developers.openai.com/api/docs/guides/your-data)

Who may see retained API content is unusually clear:

> “Our access to API business data… is limited to (1) authorized employees… and (2) specialized third-party contractors…”

Those employees may need access for engineering support, abuse investigations and legal compliance; specialized contractors are used for abuse/misuse review. [OpenAI](https://openai.com/enterprise-privacy/)

**CONCLUSION:** ✅ **HIGH.** For a VRYOH privacy policy, “up to 30 days” is safe for ordinary API abuse-monitoring retention, but do **not** claim all OpenAI API data is always deleted exactly after 30 days regardless of endpoint/configuration.

---

## 3. Zero Data Retention?

**CLAIM:** Available only to approved eligible customers and supported endpoints.

> “Get in touch with our sales team to learn more about these offerings and inquire about eligibility.”

**Source:** [https://developers.openai.com/api/docs/guides/your-data](https://developers.openai.com/api/docs/guides/your-data?utm_source=chatgpt.com)  
**Last-updated date:** no last-updated date is displayed on the current documentation page. [OpenAI Developers](https://developers.openai.com/api/docs/guides/your-data)

`/v1/chat/completions` and `/v1/responses` are currently listed as ZDR-eligible, subject to limitations. OpenAI requires approval; after approval the organization gets Data Retention controls in Settings. [OpenAI Developers](https://developers.openai.com/api/docs/guides/your-data)

**CONCLUSION:** ✅ **HIGH.** ZDR is **not** the default for your standard self-service account. I could not verify a publicly stated revenue, contract-size or other precise eligibility threshold. Do not claim VRYOH qualifies until OpenAI approves it.

---

## 4. Do these terms apply to ordinary self-serve API accounts?

**CLAIM:** Yes.

> “This OpenAI Services Agreement only applies to use of OpenAI's APIs… for customers who are businesses and developers…”

The Agreement is accepted by clicking “I agree,” accepting an Order Form **or using the Services** and expressly includes usage-based services. [OpenAI](https://openai.com/policies/services-agreement/)

**Source:** [https://openai.com/policies/services-agreement/](https://openai.com/policies/services-agreement/?utm_source=chatgpt.com)  
**Updated:** December 1, 2025. **Effective:** January 1, 2026.

**CONCLUSION:** ✅ **HIGH.** An enterprise contract is not required for these API contractual terms to apply.

---

## 5. DPA: automatic or separately signed?

There is a nuance here.

**CLAIM:** The current Services Agreement incorporates the DPA when personal data is processed.

> “If Customer uses the Services to process Personal Data… the DPA… is incorporated by this reference into the Agreement.” [OpenAI](https://openai.com/policies/services-agreement/)


The DPA itself says:

> “This OpenAI Data Processing Addendum… supplements, and is incorporated into, the OpenAI Services Agreement…”

**Source:** [https://openai.com/policies/data-processing-addendum/](https://openai.com/policies/data-processing-addendum/?utm_source=chatgpt.com)  
**Updated:** December 1, 2025. **Effective:** January 1, 2026. [OpenAI](https://openai.com/policies/data-processing-addendum/)

It also says customers agree by clicking “I agree,” accepting the Order Form, or using the Services. [OpenAI](https://openai.com/policies/data-processing-addendum/)

OpenAI separately offers a DPA form for customers who want to formally execute one. [OpenAI](https://openai.com/enterprise-privacy/)

**CONCLUSION:** ✅ **HIGH** that the published DPA is contractually incorporated for API processing of personal data. ⚠️ **MEDIUM** on describing this as “no signing process is ever needed,” because OpenAI also provides an execution form. For a privacy policy, simply say **“OpenAI's Data Processing Addendum applies to its processing of personal data under the Services Agreement.”**

---

## 6. Subprocessors?

**CLAIM:** Yes, OpenAI publishes a detailed list and identifies which subprocessors apply to the API.

> “OpenAI engages the following entities to provide processing activities for Customer Data…”

**Source:** [https://openai.com/policies/sub-processor-list/](https://openai.com/policies/sub-processor-list/?utm_source=chatgpt.com)  
**Last updated:** **October 14, 2026 — today.** [OpenAI](https://openai.com/policies/sub-processor-list/)

The table explicitly identifies API providers such as Cloudflare, Microsoft and CoreWeave and their processing purposes/locations. [OpenAI](https://openai.com/policies/sub-processor-list/)

**CONCLUSION:** ✅ **HIGH.** This list changed or was refreshed today; I did not independently diff it against the previous version, so I cannot state exactly what changed.

---

## 7. Legal-hold / court-order retention exceptions?

**CLAIM:** Yes.

OpenAI's API policy says data normally removed after 30 days can be retained when legally required. [OpenAI](https://openai.com/enterprise-privacy/)

The Services Agreement also states:

> “OpenAI will delete all Customer Content… within thirty days, unless… OpenAI is legally required to retain it…”

**Source:** [https://openai.com/policies/services-agreement/](https://openai.com/policies/services-agreement/?utm_source=chatgpt.com)  
**Updated:** December 1, 2025. [OpenAI](https://openai.com/policies/services-agreement/)

OpenAI also documents exceptions reasonably necessary to protect its services or third parties from harm. [OpenAI Developers](https://developers.openai.com/api/docs/guides/your-data)

**CONCLUSION:** ✅ **HIGH.** As with Anthropic, “required by law” is verified. I would not substitute the narrower legal term “legal hold” unless counsel wants that wording.

---

# C. DIGITALOCEAN

## 1. Does DigitalOcean access/use/analyze Droplet contents?

**CLAIM:** DigitalOcean receives a limited license to handle customer content to provide the service, and it expressly reserves access in specified circumstances.

> “…host, store, transfer… your Services Content solely for the purpose of providing the Services to you and for no other reason.”

**Source:** [https://www.digitalocean.com/legal/terms-of-service-agreement](https://www.digitalocean.com/legal/terms-of-service-agreement?utm_source=chatgpt.com)  
**Last updated:** August 22, 2026. [DigitalOcean](https://www.digitalocean.com/legal/terms-of-service-agreement)

But DigitalOcean also expressly reserves the right to:

> “access, read, preserve, and disclose any information as we reasonably believe is necessary…”

The listed purposes include law/government requests, enforcing its TOS, fraud/security/technical issues, customer support and protecting rights/safety. [DigitalOcean](https://www.digitalocean.com/legal/terms-of-service-agreement)

Its DPA additionally says:

> “DigitalOcean will Process Personal Data only in accordance with Customer's instructions…”

Those instructions are to provide the Services and comply with applicable law. [DigitalOcean](https://www.digitalocean.com/legal/data-processing-agreement)

**Inference:** I found **no official statement supporting “DigitalOcean never accesses or analyzes Droplet contents.”** That would be false or at least materially incomplete.

**CONCLUSION:** ✅ **HIGH.** Safe language is that DigitalOcean processes hosted data to provide the service and may access it under specified legal, security, enforcement or support circumstances.

---

## 2. DigitalOcean DPA: automatic?

**CLAIM:** Yes.

DigitalOcean's current Terms incorporate the DPA by reference. [DigitalOcean](https://www.digitalocean.com/legal/terms-of-service-agreement)

Its GDPR FAQ states explicitly:

> “By agreeing to our terms of service, you are automatically accepting our DPA and do not need to sign a separate document.”

**Source:** [https://www.digitalocean.com/legal/gdpr-faq](https://www.digitalocean.com/legal/gdpr-faq?utm_source=chatgpt.com)  
**Last-updated date:** no explicit last-updated date is displayed on the current page. [DigitalOcean](https://www.digitalocean.com/legal/gdpr-faq)

Current DPA:

**Source:** [https://www.digitalocean.com/legal/data-processing-agreement](https://www.digitalocean.com/legal/data-processing-agreement?utm_source=chatgpt.com)  
**Last updated:** **October 14, 2026 — today.** [DigitalOcean](https://www.digitalocean.com/legal/data-processing-agreement)

**CONCLUSION:** ✅ **HIGH.** Automatic; no separate signature required.

---

# 3. Storage location, backups, encryption and deletion

### Geographic storage

DigitalOcean says you can specify a geographic region for Services Content, but its current TOS contains an important reservation:

> “DigitalOcean may transfer and store your Services Content in other geographic regions or jurisdictions at its sole discretion.”

That applies unless otherwise agreed in writing. [DigitalOcean](https://www.digitalocean.com/legal/terms-of-service-agreement)

**Source:** [https://www.digitalocean.com/legal/terms-of-service-agreement](https://www.digitalocean.com/legal/terms-of-service-agreement?utm_source=chatgpt.com)  
**Last updated:** August 22, 2026.

**CONCLUSION:** ✅ **HIGH.** Do **not** state “our DigitalOcean data never leaves the selected region.”

---

### Main Droplet encryption at rest

This is the most important DigitalOcean finding.

**CLAIM:** A normal Droplet's virtual disk is **not encrypted at rest**.

> “The virtual disks for Droplets stored on the hypervisor's local storage are not encrypted at rest.”

**Source:** [https://www.digitalocean.com/security/shared-responsibility-model-droplets](https://www.digitalocean.com/security/shared-responsibility-model-droplets?utm_source=chatgpt.com)  
**Last-updated date:** no last-updated date is displayed on this page. [DigitalOcean](https://www.digitalocean.com/security/shared-responsibility-model-droplets)

DigitalOcean separately says the **physical storage devices** underlying its infrastructure are encrypted at rest. [DigitalOcean](https://www.digitalocean.com/security/infrastructure-security?utm_source=chatgpt.com)

Those are not contradictory: physical-drive encryption and VM virtual-disk encryption are different security layers.

**CONCLUSION:** ✅ **HIGH.** Do not publish **“DigitalOcean encrypts VRYOH's Droplet data at rest”** without additional encryption you implement yourself.

---

### Automated Droplet backups

**CLAIM:** Automated Droplet backups are **not encrypted at rest**.

> “Backups are not encrypted at rest, but they are not externally accessible.”

**Source:** [https://docs.digitalocean.com/products/backups/details/limits/](https://docs.digitalocean.com/products/backups/details/limits/?utm_source=chatgpt.com)  
**Last verified:** June 17, 2025. [DigitalOcean](https://docs.digitalocean.com/products/backups/details/limits/)

**CONCLUSION:** ✅ **HIGH.**

---

### Droplet snapshots

DigitalOcean's current documentation states that Droplet snapshots are full disk images, are backed up offsite and remain until manually deleted. [DigitalOcean](https://docs.digitalocean.com/products/snapshots/details/features/?utm_source=chatgpt.com)

However:

**I could not verify from an official DigitalOcean page that ordinary Droplet snapshots are encrypted at rest.**

DigitalOcean explicitly states **Volume snapshots** are encrypted, but that is a different product. [DigitalOcean](https://docs.digitalocean.com/products/snapshots/how-to/snapshot-volumes/?utm_source=chatgpt.com)

**CONCLUSION:** ❌ **LOW / DO NOT CLAIM** that Droplet snapshots are encrypted at rest.

---

### Destroying a Droplet

> “Deleting a Droplet permanently and irreversibly destroys the Droplet.”

But:

> “Destroying a Droplet does not destroy its automated backups.”

**Source:** [https://docs.digitalocean.com/products/droplets/how-to/destroy/](https://docs.digitalocean.com/products/droplets/how-to/destroy/?utm_source=chatgpt.com)  
**Last verified:** July 13, 2026. [DigitalOcean](https://docs.digitalocean.com/products/droplets/how-to/destroy/)

Associated snapshots, volumes and volume snapshots also are **not destroyed by default** unless selected. [DigitalOcean](https://docs.digitalocean.com/products/droplets/how-to/destroy/)

Current backup documentation says:

> “We retain weekly backups for four weeks, daily backups for seven days… We retain snapshots until you delete them.”

**Source:** [https://docs.digitalocean.com/products/backups/getting-started/quickstart/](https://docs.digitalocean.com/products/backups/getting-started/quickstart/?utm_source=chatgpt.com)  
**Last verified:** September 3, 2026. [DigitalOcean](https://docs.digitalocean.com/products/backups/getting-started/quickstart/)

**CONCLUSION:** ✅ **HIGH.**

---

### Closing/deactivating the DigitalOcean account

DigitalOcean's current DPA states:

> “DigitalOcean will delete all Personal Data no later than 30 days following the date of de-activation…”

**Source:** [https://www.digitalocean.com/legal/data-processing-agreement](https://www.digitalocean.com/legal/data-processing-agreement?utm_source=chatgpt.com)  
**Last updated:** October 14, 2026. [DigitalOcean](https://www.digitalocean.com/legal/data-processing-agreement)

But archived backup copies may remain according to automated retention/disposal schedules and must be isolated from further active processing. [DigitalOcean](https://www.digitalocean.com/legal/data-processing-agreement)

**CONCLUSION:** ✅ **HIGH** for Personal Data covered by the DPA. Do not convert this into “every byte in every DigitalOcean system is deleted within 30 days.”

---

# D. Statements I consider safe for VRYOH's privacy policy

These are the versions I would be comfortable publishing based strictly on today's official documentation:

1. **Anthropic:** “When VRYOH uses Anthropic's commercial API, Anthropic does not use API inputs or outputs to train its models by default.”  
   Source: [Anthropic model-training policy](https://privacy.claude.com/en/articles/7996868-is-my-data-used-for-model-training?utm_source=chatgpt.com) [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/7996868-is-my-data-used-for-model-training)

2. **Anthropic:** “Anthropic ordinarily deletes API inputs and outputs from its backend within 30 days, subject to documented legal, safety, feature-specific, or contractual exceptions.”  
   Source: [Anthropic retention policy](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data?utm_source=chatgpt.com) [Anthropic Privacy Center](https://privacy.anthropic.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data)

3. **Anthropic:** “Anthropic's Data Processing Addendum is incorporated into its Commercial Terms of Service.”  
   Source: [Anthropic DPA](https://www.anthropic.com/legal/data-processing-addendum?utm_source=chatgpt.com) [Anthropic](https://www.anthropic.com/legal/data-processing-addendum)

4. **OpenAI:** “OpenAI does not use API customer inputs and outputs to train or improve its models unless the customer explicitly opts in.”  
   Source: [OpenAI Services Agreement](https://openai.com/policies/services-agreement/?utm_source=chatgpt.com) [OpenAI](https://openai.com/policies/services-agreement/)

5. **OpenAI:** “OpenAI may retain API inputs and outputs for up to 30 days for service provision and abuse monitoring, subject to endpoint-specific and legal exceptions.”  
   Source: [OpenAI API privacy information](https://openai.com/enterprise-privacy/?utm_source=chatgpt.com) [OpenAI](https://openai.com/enterprise-privacy/)

6. **OpenAI:** “Access to stored API business data is restricted to authorized personnel with specified operational needs and specialized contractors performing abuse/misuse review.”  
   Source: [OpenAI API privacy information](https://openai.com/enterprise-privacy/?utm_source=chatgpt.com) [OpenAI](https://openai.com/enterprise-privacy/)

7. **OpenAI:** “OpenAI's Data Processing Addendum is incorporated into its Services Agreement for processing personal data.”  
   Source: [OpenAI Services Agreement](https://openai.com/policies/services-agreement/?utm_source=chatgpt.com) [OpenAI](https://openai.com/policies/services-agreement/)

8. **DigitalOcean:** “VRYOH uses DigitalOcean infrastructure to host its server and database. DigitalOcean processes hosted personal data to provide its services and as required by applicable law.”  
   Sources: [DigitalOcean TOS](https://www.digitalocean.com/legal/terms-of-service-agreement?utm_source=chatgpt.com) and [DigitalOcean DPA](https://www.digitalocean.com/legal/data-processing-agreement?utm_source=chatgpt.com). [DigitalOcean](https://www.digitalocean.com/legal/terms-of-service-agreement)

9. **DigitalOcean:** “DigitalOcean's DPA forms part of its customer contractual terms.”  
   Source: [DigitalOcean DPA](https://www.digitalocean.com/legal/data-processing-agreement?utm_source=chatgpt.com) [DigitalOcean](https://www.digitalocean.com/legal/data-processing-agreement)

10. **DigitalOcean deletion:** “Deleting a Droplet permanently destroys that Droplet, although backups, snapshots or associated storage can have separate retention periods.”  
    Source: [DigitalOcean Droplet deletion documentation](https://docs.digitalocean.com/products/droplets/how-to/destroy/?utm_source=chatgpt.com) [DigitalOcean](https://docs.digitalocean.com/products/droplets/how-to/destroy/)

---

# E. Statements I recommend **NOT** publishing

These are either false, unsupported, or too broad:

- **“Anthropic never retains our API data.”** False for standard API accounts; ordinary retention is up to 30 days.
- **“VRYOH has Anthropic Zero Data Retention.”** Not verified and not the standard self-service configuration.
- **“Anthropic employees can never see our API data.”** Not supported.
- **“OpenAI stores no VRYOH data.”** False for standard API usage.
- **“OpenAI immediately deletes every API request after processing.”** False under ordinary retention.
- **“VRYOH has OpenAI Zero Data Retention.”** Not verified.
- **“All OpenAI API data is always deleted exactly after 30 days.”** Too broad because application-state and endpoint-specific rules differ.
- **“DigitalOcean never accesses our server contents.”** Contradicted by its TOS.
- **“All VRYOH data stays permanently in the DigitalOcean region we selected.”** Not supported by current TOS.
- **“DigitalOcean encrypts our Droplet virtual disk at rest.”** **False according to DigitalOcean's current Shared Responsibility page.**
- **“DigitalOcean automated Droplet backups are encrypted at rest.”** **False according to DigitalOcean documentation.**
- **“DigitalOcean Droplet snapshots are encrypted at rest.”** **I could not verify this.**
- **“Closing our DigitalOcean account immediately deletes every copy of our data.”** Too broad; archived backup exceptions exist.

---

# F. Are any of the verified protections NOT true for VRYOH's standard self-serve accounts?

**Anthropic:** The no-training default, 30-day standard retention, Commercial Terms and automatic DPA **do apply** to standard commercial API accounts. **ZDR does not automatically apply**; Anthropic describes it as an approved arrangement for some enterprise API customers. ✅

**OpenAI:** The no-training default, ordinary abuse-monitoring retention, Services Agreement and incorporated DPA **do apply** to standard API use. **ZDR does not automatically apply** and requires approval. ✅

**DigitalOcean:** The TOS and DPA **do apply** to ordinary DigitalOcean customers. Nothing I found indicates that a normal Droplet gets enterprise-only encryption protections that would change the findings above. In particular, DigitalOcean currently says standard Droplet virtual disks and automated Droplet backups are not encrypted at rest. ✅ [DigitalOcean](https://www.digitalocean.com/security/shared-responsibility-model-droplets)

### Recent-change warning

There have been very recent documentation/legal updates: **Anthropic updated its subprocessor list today, October 14, 2026; OpenAI's subprocessor list is also dated October 14, 2026; DigitalOcean's DPA was updated today, and its Privacy Policy was updated yesterday, October 13.** DigitalOcean's current Privacy Policy explicitly says it does **not** govern customer content processed through its services; that content is governed principally by the TOS/DPA. [Anthropic Trust Center](https://trust.anthropic.com/?gad=1\&utm_source=chatgpt.com)

**Overall conclusion: ✅ HIGH.** The largest privacy-policy risk I found is not Anthropic or OpenAI training. It is making an inaccurate security claim about DigitalOcean encryption. Based on DigitalOcean's own current documentation, VRYOH should **not** tell users that its Droplet data or automated Droplet backups are encrypted at rest unless VRYOH itself has implemented an additional encryption layer.
