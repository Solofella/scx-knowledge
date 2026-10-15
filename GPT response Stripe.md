Chat #388 · October 15, 2026

# VRYOH — Stripe Subscription Billing Capability Audit
**Verification date:** October 15, 2026  
**Sources:** Official Stripe documentation and API references only  
**Account type:** Existing U.S. Stripe account; products and Checkout not configured

## Executive conclusion

**Stripe supports VRYOH's core subscription model:** monthly and annual recurring charges, anniversary billing, per-location quantities, prorated subscription changes, and customer self-service cancellation.

However, **Stripe's billing capabilities are not the same as a complete subscription-compliance system**.

Three requirements must be treated separately until verified in your actual account:

1. **Explicit renewal consent:** Stripe supports a required Terms of Service acceptance checkbox. That is not automatically proof of a separate, independently worded automatic-renewal consent.
2. **Annual renewal reminders:** I could verify a configurable upcoming-invoice event, but not a native Stripe email that reliably sends exactly 30 days before annual renewals only.
3. **VRYOH location management:** Stripe can change subscription quantities, but it does not automatically create or delete corresponding restaurant-location records in NocoDB.

### Recommended architecture

| Component | Responsibility |
|---|---|
| **Lovable** | Marketing website, signup interface, customer account area |
| **Stripe Checkout** | Collect payment and initiate recurring subscriptions |
| **Stripe Billing** | Renewals, invoices, payment collection, quantity-based charges |
| **Stripe Customer Portal** | Payment-method changes, subscription quantity updates, cancellation |
| **n8n** | Connect billing events to VRYOH, send custom reminders when needed |
| **NocoDB** | Record locations, subscription relationships, and service entitlements |

This architecture is a **proposed design**, not a tested integration. I have NOT verified that it works end-to-end with your existing Lovable, n8n, and NocoDB installations. The direct verification method is to configure it using Stripe test mode and run complete subscription scenarios.

**Documentation dates:** Stripe's current documentation pages were opened during this review. Most of the referenced pages do **not** display a calendar date labeled “last updated.” I identify those as *not displayed*, rather than inventing publication dates. The current documentation does not establish that the configuration is enabled in your particular Stripe account.

---

# 1. Checkout consent

## Can customers be required to check a box before paying?

**✅ HIGH — Supported for Terms of Service acceptance.**

Stripe's Payment Links API documents:

> “If set to `required`, it requires customers to accept the terms of service before being able to pay.”

**Official source:**  
https://docs.stripe.com/api/payment_links/payment_links/object

**Page last updated:** Not displayed.

The documented `consent_collection.terms_of_service` setting supports `required` or `none`. Stripe also provides a configurable `custom_text.terms_of_service_acceptance` field, allowing custom text in place of the default acceptance wording. [Stripe Docs](https://docs.stripe.com/api/payment_links/payment_links/object?bsft_clkid=grintkjx\&utm_source=chatgpt.com)

### What that means for VRYOH

You want an acknowledgment such as:

> My subscription renews automatically each month or year until I cancel.

Stripe documents customization of the Terms acceptance text, but I could not verify that a separate, additional mandatory automatic-renewal checkbox can be created independently of the Terms checkbox using Dashboard-only Checkout or Payment Links.

**Status:**
- Required Terms checkbox: **SUPPORTED**
- Custom Terms acceptance wording: **SUPPORTED through the documented API**
- Separate renewal-consent checkbox: **COULD NOT VERIFY**
- Dashboard-only customization of the precise checkbox wording: **COULD NOT VERIFY**

### Is accepting the Terms the same as separate automatic-renewal consent?

**No—not necessarily.**

A Terms acceptance checkbox can contain renewal language, but whether that satisfies a statute requiring distinct consent is a legal question.

Stripe's technical ability to collect acceptance does not establish that the resulting consent meets every applicable law.

### Can Stripe store proof of consent?

The API exposes consent-related fields, but I could not verify from the opened documentation that Stripe provides a complete, downloadable legal-evidence record containing all of the following:

- Exact renewal-consent wording accepted.
- Version of the Terms.
- Date and time of affirmative acceptance.
- Identity of the accepting business representative.
- A separately retrievable renewal-consent acknowledgment.

**Confidence: LOW for the complete evidence requirement.**

**VRYOH implication:** Do not publish “Stripe permanently records proof of the customer's separate automatic-renewal consent” based on the documentation reviewed.

### Can Checkout display the subscription price and frequency?

**✅ HIGH — Supported.**

Stripe documents recurring prices with monthly and yearly intervals.

> “how much they cost, what currency you accept for payments, and the service interval for subscriptions.”

**Source:**  
https://docs.stripe.com/billing/subscriptions/metered-billing/thresholds

**Page last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/billing/subscriptions/metered-billing/thresholds?__=\&utm_source=chatgpt.com)

However, I could not verify that a standard no-code Checkout page always displays the **exact next renewal date** before the customer pays.

Stripe calculates billing anchors and subscription periods, but that does not establish that every date is displayed in the hosted Checkout UI.

**Conclusion:** Price and billing frequency are supported. Exact next-charge-date display requires testing of your chosen checkout configuration.

---

# 2. Subscription confirmation and receipts

Stripe provides customer communication settings and supports transaction-related emails.

Its customer portal documentation states:

> “Email settings are applied to all emails sent from Stripe to your customers.”

**Official source:**  
https://docs.stripe.com/customer-management/configure-portal

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/customer-management/configure-portal?utm_source=chatgpt.com)

Stripe's no-code customer portal also provides a link that can appear in customer emails.

> “We’ll also add the link to your customer emails.”

**Source:**  
https://docs.stripe.com/customer-management/activate-no-code-customer-portal

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/customer-management/activate-no-code-customer-portal?utm_source=chatgpt.com)

### What is verified

✅ Stripe supports configurable customer emails and subscription-related billing communications.

✅ Stripe invoices contain subscription-related charge information.

✅ Stripe supports branding adjustments for customer emails.

### What I could not verify

I could not establish from the official pages opened that Stripe's standard initial-payment receipt automatically contains all these elements together:

- Explicit automatic-renewal acknowledgment.
- Date of the next renewal.
- Full contractual cancellation language.
- A legally sufficient cancellation link.
- Your exact custom English and Spanish renewal disclosures.

I also could not verify that every subscription activation produces a particular confirmation email regardless of payment method and Dashboard settings.

### Recommendation for VRYOH

Use Stripe's native receipts for payment documentation.

For a **subscription agreement confirmation**, use an additional email containing the complete subscription terms.

That email could be generated by n8n after confirmed subscription activation, but it would require an integration and a verified email-delivery service.

**Implementation classification:** Stripe emails are configurable without code; a guaranteed VRYOH-specific legal confirmation email requires additional automation.

---

# 3. Annual renewal reminders

This is one of the most important findings for your proposed Terms of Service.

## Does Stripe provide an upcoming-invoice event?

**✅ HIGH — Yes.**

Stripe documents the `invoice.upcoming` event:

> “Occurs X number of days before a subscription is scheduled to create an invoice”

Stripe states that X is determined by subscription settings.

**Official source:**  
https://docs.stripe.com/api/events

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/api/events/retrieve?payment_intent_object-status=\&utm_source=chatgpt.com)

### Does this prove Stripe sends an automatic reminder email?

**No.**

A webhook event is a technical notification to an integration. It is not evidence that Stripe sends a customer-facing email.

I could not verify from the official documentation retrieved that Stripe's standard Dashboard supports all of these requirements simultaneously:

| Requirement | Verification |
|---|---|
| Email 30 days before annual renewal | **COULD NOT VERIFY** |
| Reminder only for annual subscriptions | **COULD NOT VERIFY** |
| No equivalent reminder for monthly plans | **COULD NOT VERIFY** |
| Different reminder schedules by price interval | **COULD NOT VERIFY** |
| Required renewal date, price, period, and cancellation link in one native email | **COULD NOT VERIFY** |
| Delivery to all existing subscriptions | **COULD NOT VERIFY** |
| Exact suppression rules for missing customer email | **COULD NOT VERIFY** |

I also could not verify an exact Dashboard list of available native reminder intervals such as 3, 7, or 30 days. I will not invent those options.

### What VRYOH would need

If the Terms promise a reminder 30 days before each annual renewal, VRYOH needs a verified method for delivering that message.

**Proposed implementation:** A daily n8n workflow checks annual subscription renewal dates and generates the required reminder when a subscription reaches the 30-day threshold.

That is an engineering approach, not a claim that Stripe's `invoice.upcoming` event natively implements the requirement.

For reliability, the workflow would need to account for duplicate sends, canceled subscriptions, unavailable email addresses, and the subscription's current renewal date.

**Conclusion: HIGH** that an upcoming-invoice event exists. **LOW** that the native email capability you require has been established.

Do not make Stripe's unverified native reminder email the basis of Section 5.6.

---

# 4. Customer cancellation portal

## Can customers cancel without contacting VRYOH?

**✅ HIGH — YES.**

Stripe states:

> “Cancel subscriptions immediately or at the end of the current billing period”

**Official source:**  
https://docs.stripe.com/customer-management

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/billing/subscriptions/customer-portal?utm_source=chatgpt.com)

This matches VRYOH's proposed policy:

*Cancellation stops future renewals, while service continues through the paid period.*

## Can you configure this without code?

**✅ HIGH — Yes.**

Stripe provides Dashboard configuration of the customer portal.

**Source:**  
https://docs.stripe.com/customer-management/configure-portal

The page documents cancellation controls and subscription management settings.

Its configuration table shows cancellation enabled by default, although the exact immediate-versus-period-end configuration still must be inspected in your Dashboard. [Stripe Docs](https://docs.stripe.com/customer-management/configure-portal?utm_source=chatgpt.com)

### Can customers enter the portal from VRYOH?

**✅ HIGH — Yes, without a custom API integration.**

Stripe provides a no-code customer portal login link.

> “They can log in to the portal with their email address.”

Stripe then emails a login link.

**Source:**  
https://docs.stripe.com/customer-management/activate-no-code-customer-portal

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/customer-management/activate-no-code-customer-portal?utm_source=chatgpt.com)

You can place this URL behind a **Manage Billing** link in Lovable.

**Alternative:** Create an authenticated portal session for each customer using the Stripe API.

That alternative requires integration work. Stripe states that customer portal sessions are temporary and should be created when a customer wants to manage billing. [Stripe Docs](https://docs.stripe.com/billing/subscriptions/customer-portal?utm_source=chatgpt.com)

### Does the portal support changing locations?

**✅ HIGH — Yes, if locations correspond to subscription quantity.**

Stripe explicitly documents:

> “Let your customer increase or decrease the quantity of a subscription.”

The quantity-update option is **off by default** in the portal configuration.

**Source:**  
https://docs.stripe.com/customer-management/configure-portal

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/customer-management/configure-portal?utm_source=chatgpt.com)

For VRYOH:

- One location = quantity 1.
- Three locations = quantity 3.
- Five locations = quantity 5.

But Stripe changes the **billing quantity**, not the corresponding NocoDB location records.

A quantity change therefore needs to be synchronized with VRYOH's own records before the additional locations can be considered fully provisioned.

### Cancellation confirmation email

I verified that the customer can cancel through the portal.

I could not verify that the portal always sends a specific cancellation confirmation email containing every element VRYOH wants.

**Conclusion:** Native self-service cancellation is supported. The exact confirmation email must be tested.

---

# 5. Spanish-language support

## Stripe customer portal

**✅ HIGH — Spanish is supported.**

Stripe explicitly lists:

> “Spanish, Spain (es)”

and:

> “Spanish, Latin America (es-419)”

**Source:**  
https://docs.stripe.com/customer-management

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/billing/subscriptions/customer-portal?utm_source=chatgpt.com)

Stripe states that the portal automatically localizes according to the customer's preferred browser language.

### Checkout

Stripe's Checkout API supports language configuration, including automatic locale selection. The hosted Checkout language can therefore be handled separately from your Lovable website's language.

**API reference:**  
https://docs.stripe.com/api/checkout/sessions/create

**Page last updated:** Not displayed.

**Confidence: MEDIUM** because I did not complete an opened-page verification of the current `locale` parameter's precise supported values during this review.

### Receipts, invoices, and reminder emails

I could not verify that changing Lovable's language automatically changes all Stripe-generated emails into the same language.

I also could not verify that every custom Stripe text field is translated automatically.

**Critical distinction:** Automatically translated Stripe interface labels do not imply that merchant-written English terms are translated into Spanish.

### VRYOH implementation judgment

Your website should keep its own selected language separate from the Stripe-hosted interface language.

If VRYOH generates contractual emails through n8n, it can choose an English or Spanish message template according to a language preference stored in NocoDB.

That proposed workflow requires implementation and testing.

---

# 6. Adding locations during a billing cycle

**✅ HIGH — Stripe supports subscription quantity changes and proration.**

Stripe's API states:

> “We also prorate when you make quantity changes.”

**Official source:**  
https://docs.stripe.com/api/subscriptions/update

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/api/subscriptions/update?utmx=\&utm_source=chatgpt.com)

However, the default behavior of the Subscriptions API and the default settings of the **customer portal** are not identical.

## 6.1 Stripe's API default

The API uses `create_prorations` by default.

When the subscription quantity increases, Stripe calculates an adjustment for the unused portion of the current period.

That adjustment is not necessarily collected immediately.

Stripe documents three proration behaviors:

| Setting | Effect |
|---|---|
| `create_prorations` | Creates applicable prorated adjustments, ordinarily collected on a later invoice |
| `always_invoice` | Creates applicable adjustments and attempts to invoice and collect them immediately |
| `none` | Disables prorated adjustments |

**Source:** https://docs.stripe.com/api/subscriptions/update [Stripe Docs](https://docs.stripe.com/api/subscriptions/update?utmx=\&utm_source=chatgpt.com)

### Example: Monthly subscription

A customer subscribes on October 10.

They have two locations:

$30 × 2 = **$60 per month**.

On October 20, they add a third location.

With proration enabled, Stripe calculates a partial-period charge for the additional location.

The amount depends on the remaining time in the billing period and Stripe's calculation.

With `create_prorations`, that amount may be carried to the next invoice.

With `always_invoice`, Stripe attempts to collect it immediately.

At the next full renewal, the recurring amount becomes $90 per month.

**The same general principle applies to an annual subscription**, except the prorated addition may cover many remaining months.

## 6.2 Customer portal defaults

**Important distinction:**

Stripe's portal documentation identifies the following defaults:

| Setting | Documented default |
|---|---|
| Customer can change quantity | Off |
| Portal proration option | Off |
| Customer can cancel | On |

**Official source:**  
https://docs.stripe.com/customer-management/configure-portal

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/customer-management/configure-portal?utm_source=chatgpt.com)

Therefore, do not assume the API's `create_prorations` default means the customer portal will automatically charge prorated additions.

## 6.3 What I recommend for VRYOH

**Proposed policy:** When a customer adds a location, charge the prorated amount immediately for the remainder of the current period.

This is operationally preferable if VRYOH activates the additional location immediately.

However, the portal's exact behavior must be tested before adopting this policy.

For annual subscriptions, the effect is especially important.

A location added halfway through a $300 annual period could generate an approximately half-year charge, while the next renewal would charge the full $300 for that location.

This is an illustration, not a guaranteed Stripe calculation.

### Suggested Terms wording

> Additional locations added during a billing period may result in a prorated charge for the remaining portion of that period. The updated recurring subscription amount will apply at subsequent renewals.

Use this only after confirming that your configured portal or API integration actually prorates additions.

If you configure immediate collection, the wording should explicitly say so.

---

# 7. Monthly subscriptions beginning on the 29th, 30th, or 31st

**✅ HIGH — Verified.**

Stripe states:

> “A monthly subscription with a billing cycle anchor date of January 31 bills the last day of the month”

**Official source:**  
https://docs.stripe.com/billing/subscriptions/billing-cycle

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/billing/subscriptions/billing-cycle?utm_source=chatgpt.com)

Stripe's example explicitly covers February in ordinary and leap years.

### What happens

For a subscription anchored on January 31:

| Month | Renewal date |
|---|---|
| January | 31 |
| February | 28 or 29 |
| March | 31 |
| April | 30 |
| May | 31 |

The billing anchor is preserved.

**The date does not permanently drift to the 28th.**

For subscriptions anchored on the 29th or 30th, shorter months similarly require a shortened billing date.

### VRYOH Terms

Instead of promising that renewal always occurs on the exact numerical signup date, use:

> Monthly subscriptions renew on the corresponding calendar day each month. If a month does not contain that day, renewal occurs on the last day of that month.

For annual subscriptions, the analogous leap-day treatment should be verified before promising an exact February 29 renewal date in every year.

---

# 8. Failed subscription payments

**✅ HIGH — Stripe supports automated retries and failed-payment notifications.**

Stripe describes Smart Retries:

> “Smart Retries chooses the best times to retry failed payment attempts”

**Official source:**  
https://docs.stripe.com/billing/revenue-recovery/smart-retries

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/billing/revenue-recovery/smart-retries?utm_source=chatgpt.com)

Stripe also documents available retry policies:

> “1 week, 2 weeks, 3 weeks, 1 month, or 2 months.”

The documentation describes a recommended configuration of eight attempts within two weeks. This is **not proof that your account currently uses that configuration**.

### Dashboard configuration

Stripe directs merchants to the subscription billing settings to configure failed-payment communications:

https://dashboard.stripe.com/settings/billing/automatic

The documentation identifies:

> “Send emails when card payments fail”

as a setting that enables automatic customer emails.

**Source:** https://docs.stripe.com/invoicing/automatic-collection [Stripe Docs](https://docs.stripe.com/billing/automatic-collection?utm_source=chatgpt.com)

### What happens to subscription status?

Stripe's APIs support statuses such as `active`, `past_due`, `unpaid`, and `canceled`.

A failed payment does not always mean the subscription immediately becomes canceled.

The eventual status depends on the payment failure, retry settings, and configured action after recovery attempts are exhausted.

**Source:**  
https://docs.stripe.com/api/subscriptions/object

**Last updated:** Not displayed.

### VRYOH's unresolved policy

You need to decide whether VRYOH will:

- Continue service while Stripe retries payment.
- Suspend service after a defined grace period.
- Terminate access once Stripe marks the subscription canceled or unpaid.

**I could not verify which behavior your current Stripe settings will produce**, because your account has not been configured.

Stripe manages the billing state; VRYOH must separately enforce the corresponding service-access policy.

That requires a connection between Stripe subscription status and VRYOH's customer records.

### Technical implementation

Stripe provides subscription and invoice events, including:

- `invoice.payment_failed`
- `invoice.payment_succeeded`
- `customer.subscription.updated`
- `customer.subscription.deleted`

**Source:** https://docs.stripe.com/api/events [Stripe Docs](https://docs.stripe.com/api/events?event_types-invoice.paid=\&utm_source=chatgpt.com)

n8n could receive those webhooks and synchronize subscription information with NocoDB.

This integration has **not been tested** in your environment.

---

# 9. Testing subscriptions without waiting

## Test mode and test clocks

**✅ HIGH — Supported.**

Stripe states:

> “A test clock enables deterministic control over objects in testmode.”

**Official source:**  
https://docs.stripe.com/api/test_clocks

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/api/test_clocks?event_types-invoice.payment_succeeded=\&utm_source=chatgpt.com)

A test clock lets you advance simulated time and inspect changes to compatible test objects.

### What can be tested?

| Scenario | Verification |
|---|---|
| Create a test subscription | SUPPORTED |
| Simulate a monthly renewal | SUPPORTED |
| Simulate an annual renewal | SUPPORTED |
| Inspect resulting invoices and subscription states | SUPPORTED |
| Simulate subscription quantity changes | SUPPORTED through test subscriptions |
| Test cancellation | SUPPORTED |
| Observe compatible webhook events | SUPPORTED |
| Prove native renewal-reminder email delivery | COULD NOT VERIFY |
| Prove all customer emails are delivered in test mode | COULD NOT VERIFY |

Test clocks have compatibility and time-advancement restrictions. I did not verify the current maximum permitted advancement or object-limit values, so those limits remain unresolved.

### Important distinction

A simulated webhook event is not the same as an email successfully arriving in the customer's inbox.

**I could not verify that advancing a test clock sends Stripe's ordinary renewal-reminder emails.**

Do not use a successful test-clock renewal as proof that the promised 30-day reminder was delivered.

For that requirement, an actual delivered test email and its content must be inspected.

---

# 10. Subscription compliance and Stripe's responsibilities

Stripe provides payment-processing and billing tools. Using them does not, by itself, establish that a merchant satisfies every applicable automatic-renewal law.

## Documented Stripe capabilities

✅ Recurring billing and automatic payment collection  
✅ Subscription interval configuration  
✅ Checkout terms acceptance  
✅ Customer cancellation through the portal  
✅ Subscription quantity changes  
✅ Invoice and payment records  
✅ Customer communication settings  
✅ Webhooks for subscription events

### What VRYOH still controls

VRYOH is responsible for deciding and implementing its commercial terms, including which legal disclosures appear before purchase, which agreement is accepted, how changes in locations affect service and billing, and how cancellation affects access.

Stripe's API and Dashboard settings can support those practices, but they do not determine whether VRYOH's contractual language is legally sufficient.

**Stripe documentation:**  
https://docs.stripe.com/payments/checkout  
https://docs.stripe.com/billing/subscriptions/overview  
https://docs.stripe.com/customer-management

**Last updated:** Not displayed on the documentation examined.

### Stripe terms and card-network requirements

I did not complete verification of the currently applicable Stripe merchant agreement and specific Visa/Mastercard recurring-payment rules using official Stripe legal documentation.

Consequently:

**I could not verify** the complete list of Stripe-contractual or card-network disclosure requirements applicable to VRYOH's U.S. recurring subscriptions as of October 15, 2026.

I would not characterize Stripe's standard Checkout as automatically satisfying every card-network disclosure, consent, and cancellation requirement.

That should remain an explicit compliance-review item.

---

# 11. Minimum setup for VRYOH

**Objective:** Launch recurring subscriptions with the least custom code possible.

**Recommendation:** Start with Stripe Payment Links or Stripe's hosted Checkout, use the no-code customer portal, and reserve n8n for requirements that Stripe's native configuration cannot reliably satisfy.

This is an implementation recommendation based on the verified product capabilities—not a successfully tested VRYOH deployment.

## Step 1 — Create the subscription product

**SUPPORTED WITHOUT CODE — HIGH**

Open Stripe's Product Catalog:

https://dashboard.stripe.com/products

Create one product:

**VRYOH — Per Location Subscription**

Add two recurring prices:

| Price | Amount | Frequency |
|---|---|---|
| Monthly | $30 | Monthly |
| Annual | $300 | Yearly |

Use a per-unit quantity-based pricing model.

Stripe documents creating products and recurring prices in the Dashboard.

**Official documentation:**  
https://docs.stripe.com/billing/subscriptions/metered-billing/thresholds

**Last updated:** Not displayed. [Stripe Docs](https://docs.stripe.com/billing/subscriptions/metered-billing/thresholds?__=\&utm_source=chatgpt.com)

**Expected result:** One product, two recurring prices, with quantity representing the number of business locations.

## Step 2 — Create Checkout or Payment Links

**SUPPORTED WITHOUT CODE for basic recurring checkout — HIGH**

Open:

https://dashboard.stripe.com/payment_links

Create one subscription Payment Link for each plan.

Configure customer-adjustable quantity if that option is available for your chosen Payment Link.

Set the default quantity to one location.

**Official source:**  
https://docs.stripe.com/payment-links

**Last updated:** Not displayed.

**Important:** Basic hosted Payment Links require no custom code. An authenticated Checkout flow that associates the payment with a specific VRYOH account may require an API-created Checkout Session.

For VRYOH's eventual self-service account system, the authenticated integration is preferable, but it requires additional work.

## Step 3 — Configure checkout disclosures

**PARTIALLY VERIFIED**

Configure the available Terms of Service acceptance and display the applicable renewal language.

A separate, independently recorded automatic-renewal checkbox remains unverified.

**Source:**  
https://docs.stripe.com/api/payment_links/payment_links/object

Before publishing a statement that acceptance is collected separately, inspect the actual hosted checkout.

## Step 4 — Enable customer emails

**SUPPORTED WITHOUT CODE for Stripe's configurable billing communications**

Open:

https://dashboard.stripe.com/settings/billing/automatic

Review the available subscription email settings, receipts, and failed-payment notifications.

Also review:

https://dashboard.stripe.com/settings/emails

**Source:**  
https://docs.stripe.com/customer-management/configure-portal

Do not assume that enabling Stripe receipts satisfies your complete subscription-confirmation requirements.

## Step 5 — Activate the Stripe Customer Portal

**SUPPORTED WITHOUT CODE — HIGH**

Open:

https://dashboard.stripe.com/settings/billing/portal

Configure:

1. Customer cancellation.
2. Cancellation at the end of the current billing period.
3. Subscription quantity updates.
4. Proration behavior.
5. Billing information and payment-method updates.
6. VRYOH branding and Terms of Service URL.

**Source:**  
https://docs.stripe.com/customer-management/configure-portal

## Step 6 — Connect the portal to Lovable

**SUPPORTED WITHOUT CODE for a general portal login link — HIGH**

Activate Stripe's no-code portal login link.

Add a **Manage Billing** link to the VRYOH account area.

Stripe authenticates the customer through their registered email address.

**Source:**  
https://docs.stripe.com/customer-management/activate-no-code-customer-portal

This is different from creating a dedicated, authenticated Stripe portal session from a VRYOH login.

The latter requires API integration.

## Step 7 — Configure Spanish

**SUPPORTED WITHOUT CODE for the Stripe customer portal — HIGH**

Use the browser-language localization supported by Stripe.

**Source:**  
https://docs.stripe.com/customer-management

For Checkout, verify locale configuration in the actual integration.

Do not assume merchant-written renewal text is translated automatically.

## Step 8 — Connect Stripe to NocoDB through n8n

**REQUIRES INTEGRATION — NOT TESTED**

The minimum useful integration should synchronize:

| Stripe event | VRYOH action |
|---|---|
| Successful subscription activation | Associate a Stripe subscription with the business |
| Successful recurring payment | Update payment state |
| Quantity change | Reconcile purchased location quantity |
| Cancellation scheduled | Record upcoming termination |
| Subscription ended | Update service access |
| Failed payment | Record payment issue |

**Stripe source:**  
https://docs.stripe.com/api/events

A webhook can notify n8n about billing events.

However, the n8n → NocoDB implementation and permission controls have not been verified in this conversation.

**I have NOT verified this works.** The direct method is an end-to-end test using a Stripe test subscription, n8n's receiving webhook, and a dedicated NocoDB test record.

## Step 9 — Implement annual reminders

**REQUIRES ADDITIONAL AUTOMATION unless a suitable native feature is verified**

Your Terms promise a reminder approximately 30 days before annual renewal.

Stripe's `invoice.upcoming` event is verified, but its existence does not guarantee an appropriate customer email.

A separate reminder workflow may therefore be necessary.

Do not activate a contractual promise that depends on this workflow until actual delivery has been tested.

## Step 10 — Test before accepting live subscriptions

**SUPPORTED through Stripe testing tools — HIGH**

Test:

- Monthly signup.
- Annual signup.
- Multiple locations.
- Adding a location mid-cycle.
- Subscription renewal.
- Failed recurring payment.
- Cancellation at period end.
- English and Spanish checkout.
- Confirmation-email delivery.
- Annual reminder delivery.
- Corresponding changes in NocoDB.

**Source:**  
https://docs.stripe.com/api/test_clocks

Only the Stripe testing capability is verified; these VRYOH-specific scenarios have not yet been executed.

---

# 12. Final capability classification

The categories below refer to Stripe's documented capabilities, not whether the integration is already operational in VRYOH.

| Capability | Classification | Confidence | Official documentation |
|---|---|---|---|
| Create monthly recurring price | **SUPPORTED WITHOUT CODE** | HIGH | [Recurring pricing](https://docs.stripe.com/billing/subscriptions/metered-billing/thresholds) |
| Create annual recurring price | **SUPPORTED WITHOUT CODE** | HIGH | [Recurring pricing](https://docs.stripe.com/billing/subscriptions/metered-billing/thresholds) |
| Anniversary-based billing | **SUPPORTED WITHOUT CODE** for ordinary subscription creation | HIGH | [Billing cycles](https://docs.stripe.com/billing/subscriptions/billing-cycle) |
| Charge by number of locations | **SUPPORTED WITHOUT CODE** | HIGH | [Quantities](https://docs.stripe.com/billing/subscriptions/quantities) |
| Required Terms checkbox | **SUPPORTED**; precise Dashboard configuration not verified | HIGH for API | [Payment Link object](https://docs.stripe.com/api/payment_links/payment_links/object) |
| Separate automatic-renewal checkbox | **COULD NOT VERIFY** | LOW | [Consent fields](https://docs.stripe.com/api/payment_links/payment_links/object) |
| Store complete legal proof of separate renewal consent | **COULD NOT VERIFY** | LOW | [Consent fields](https://docs.stripe.com/api/payment_links/payment_links/object) |
| Show recurring price and frequency | **SUPPORTED WITHOUT CODE** | HIGH | [Recurring pricing](https://docs.stripe.com/billing/subscriptions/metered-billing/thresholds) |
| Guarantee exact next-charge date on hosted checkout | **COULD NOT VERIFY** | LOW | [Checkout](https://docs.stripe.com/payments/checkout) |
| Configure Stripe billing emails | **SUPPORTED WITHOUT CODE** | HIGH | [Customer emails](https://docs.stripe.com/customer-management/configure-portal) |
| Guarantee custom renewal terms in standard receipt | **COULD NOT VERIFY** | LOW | [Customer emails](https://docs.stripe.com/customer-management/configure-portal) |
| Exact 30-day annual-only reminder email | **COULD NOT VERIFY** | LOW | [Stripe events](https://docs.stripe.com/api/events) |
| Upcoming invoice event | **SUPPORTED WITH CODE/WEBHOOK** | HIGH | [Events](https://docs.stripe.com/api/events) |
| No-code cancellation portal | **SUPPORTED WITHOUT CODE** | HIGH | [Portal activation](https://docs.stripe.com/customer-management/activate-no-code-customer-portal) |
| Cancel at paid-period end | **SUPPORTED WITHOUT CODE** | HIGH | [Portal configuration](https://docs.stripe.com/customer-management/configure-portal) |
| General portal login link | **SUPPORTED WITHOUT CODE** | HIGH | [Portal activation](https://docs.stripe.com/customer-management/activate-no-code-customer-portal) |
| Customer-specific authenticated portal session | **SUPPORTED WITH CODE/API** | HIGH | [Portal integration](https://docs.stripe.com/customer-management/integrate-customer-portal) |
| Customer changes location quantity | **SUPPORTED WITHOUT CODE** | HIGH | [Portal configuration](https://docs.stripe.com/customer-management/configure-portal) |
| Automatically synchronize NocoDB locations | **SUPPORTED WITH CUSTOM INTEGRATION**; untested | MEDIUM | [Events](https://docs.stripe.com/api/events) |
| Spanish customer portal | **SUPPORTED WITHOUT CODE** | HIGH | [Customer portal](https://docs.stripe.com/customer-management) |
| Spanish Checkout | **SUPPORTED WITH LOCALE CONFIGURATION**; exact setup not fully verified | MEDIUM | [Checkout Sessions](https://docs.stripe.com/api/checkout/sessions/create) |
| Guarantee all emails and custom text in Spanish | **COULD NOT VERIFY** | LOW | [Email configuration](https://docs.stripe.com/customer-management/configure-portal) |
| Proration on quantity changes | **SUPPORTED WITHOUT CODE** through configurable portal | HIGH | [Portal configuration](https://docs.stripe.com/customer-management/configure-portal) |
| Immediately invoice prorated additions | **SUPPORTED WITH API**; portal behavior needs testing | HIGH for API | [Subscription update](https://docs.stripe.com/api/subscriptions/update) |
| Handle 29th–31st billing anchors | **SUPPORTED AUTOMATICALLY** | HIGH | [Billing cycles](https://docs.stripe.com/billing/subscriptions/billing-cycle) |
| Failed-payment retries | **SUPPORTED WITHOUT CODE** | HIGH | [Smart Retries](https://docs.stripe.com/billing/revenue-recovery/smart-retries) |
| Failed-payment emails | **SUPPORTED WITHOUT CODE** | HIGH | [Automatic collection](https://docs.stripe.com/invoicing/automatic-collection) |
| Simulated subscription renewals | **SUPPORTED WITH TEST CLOCKS** | HIGH | [Test clocks](https://docs.stripe.com/api/test_clocks) |
| Guaranteed reminder-email delivery in test mode | **COULD NOT VERIFY** | LOW | [Test clocks](https://docs.stripe.com/api/test_clocks) |
| Automatically establish legal compliance | **NOT SUPPORTED AS A GUARANTEE** | HIGH | [Stripe Checkout](https://docs.stripe.com/payments/checkout) |

**Availability note:** The core recurring billing, customer portal, API quantity updates, and test-clock capabilities are documented as standard Stripe features. I found no indication in the cited pages that these core capabilities are currently beta-only for U.S. accounts. That does **not** verify eligibility for every advanced Billing feature, automation, payment method, or pricing tier.

I could not verify account-specific feature access without inspecting your Dashboard.

---

# 13. What VRYOH's Terms should not promise yet

Based on the verification results, these statements would be premature:

**Do not publish:**

- “Customers separately consent to automatic renewal through a dedicated Stripe checkbox.”
- “Stripe records complete, retrievable legal evidence of each customer's renewal consent.”
- “Stripe automatically emails annual renewal reminders exactly 30 days in advance.”
- “All Stripe subscription emails include the full renewal terms and cancellation instructions.”
- “Every Stripe confirmation and reminder is automatically delivered in English or Spanish according to the VRYOH website language.”
- “Every newly added location is charged immediately on a prorated basis.”
- “Canceling or changing a Stripe subscription automatically updates VRYOH's location database.”
- “Stripe automatically suspends or restores VRYOH access according to payment status.”

These are either unverified native capabilities or functions requiring integration with VRYOH's own system.

### Statements supported by documentation

You can configure Stripe to support the following commercial practices:

**Automatic renewal:** Monthly and annual subscriptions renew automatically on their billing anniversaries, subject to cancellation and billing-anchor rules.

**Customer cancellation:** Customers can cancel through Stripe's customer portal, with cancellation configured to take effect at the end of the paid billing period.

**Location-based billing:** Subscription charges are calculated according to the number of subscribed locations.

**Additional locations:** Subscription quantity changes can result in prorated adjustments according to the configured billing rules.

**Failed payments:** Stripe can retry unsuccessful subscription payments according to configured recovery settings.

These should still be checked in test mode before publishing them as descriptions of VRYOH's implemented service.

---

# 14. What to verify inside your Stripe Dashboard

The official documentation cannot establish your account's actual settings.

| Setting | What you must verify |
|---|---|
| Product Catalog | One product with $30 monthly and $300 annual recurring prices |
| Checkout / Payment Links | Correct quantity selection and subscription disclosures |
| Terms acceptance | Required checkbox and actual wording |
| Customer emails | Enabled messages and actual confirmation content |
| Customer portal | Cancellation enabled and effective at period end |
| Quantity updates | Enabled, with the intended proration settings |
| Portal login | Enabled and working with the customer email address |
| Revenue recovery | Retry schedule and final unsuccessful-payment action |
| Customer language | English and Spanish rendering |
| Annual reminders | Actual delivery mechanism and timing |
| Stripe webhooks | Events received and processed by n8n |
| Service synchronization | NocoDB records correctly updated |
| Billing mode | No unexpected billing-anchor reset following changes |
| Test environment | Checkout, cancellation, renewal, and failed-payment scenarios |

## Final recommendation

**CLAIM:** VRYOH can launch its basic recurring billing using Stripe's hosted, low-code tools.

**EVIDENCE FOR:** Stripe officially documents recurring prices, quantities, hosted payment flows, self-service customer portals, subscription cancellation, and billing recovery.

**EVIDENCE AGAINST:** The full workflow—including separate renewal consent, legal confirmation emails, exact annual reminders, and synchronization with NocoDB—has not been demonstrated in your account.

**CONCLUSION: ⚠️ MEDIUM — the basic billing architecture is supported, but the complete VRYOH implementation is unverified.**

For the least custom code, I recommend **Stripe Payment Links or hosted Checkout + Stripe Customer Portal + a minimal n8n integration**.

The two most important features to validate before finalizing Section 5 are:

**1. Annual renewal reminders:** Whether Stripe can actually deliver the promised 30-day annual-only notices, or VRYOH must send them through n8n.

**2. Adding locations:** Whether your customer portal immediately collects prorated charges or defers them to the next invoice.

Those two decisions directly determine the billing language that VRYOH can safely publish.

**Overall verification status:** Core Stripe billing features — ✅ HIGH. Complete VRYOH checkout and compliance workflow — ⚠️ UNVERIFIED until configured and tested.
