Act as a careful reviewer of subscription terms, privacy disclosures, and Stripe payment requirements. Review VRYOH’s current alignment and unresolved issues using current official Stripe documentation. Distinguish verified facts, recommendations, contractual requirements, and settings that still need testing. Do not certify legal compliance or invent operational promises.

BUSINESS AND PAYMENT CONTEXT
VRYOH is operated by Solofella LLC, a Wyoming LLC. It provides review-response drafting and guest-signal intelligence for businesses. A person must approve each reply before publication. Support: intelligence@vryoh.com.

Policies:
https://vryoh.com/terms
https://vryoh.com/privacy

Plans: USD $30 per location per month, or $300 per location per year, billed annually in advance. Both have a 14-day free trial. Paid access requires an account and an active or trialing subscription. The website offers English and Spanish, but the legal policies are currently English-only.

WHAT THE AUDIT CONFIRMED AS ALIGNED
- Terms identify the business, describe the service, provide contact details, and disclose USD prices and per-location billing.
- Terms disclose automatic renewal, cancellation, continued access through the paid period, and a non-refundable policy with exceptions where law requires otherwise or VRYOH agrees otherwise.
- Privacy identifies Stripe and says VRYOH does not store full card numbers.
- Both Stripe recurring prices were read directly and confirmed active in TEST MODE: $30 monthly and $300 annually.
- The payment code requests a 14-day trial.
- An existing test Checkout Session had payment-method collection set to “always.”
- No policies or Stripe settings were changed during the audit, and nothing was charged.

IMPORTANT VERIFIED PAYMENT-SETUP FINDING
The existing test Checkout Session had Stripe Managed Payments enabled, automatic tax enabled with Stripe liability, and payment methods “card” and “cashapp.” This is not merely ordinary Stripe payment processing: Stripe’s documentation says Stripe is the merchant of record, and customers see “Sold through Link.” This finding applies to the inspected test session; do not assume live-mode settings have been verified.

ISSUES STILL TO RESOLVE

1. Merchant-of-record wording
Terms Section 5.2 says customers authorize Solofella LLC to charge their payment method. Review how this should be worded if Managed Payments remains enabled. Explain the distinction between VRYOH providing the service and Stripe/Link handling the sale and payment. Do not assume the owner intended Managed Payments or silently recommend keeping it.

2. Refund wording
Terms Section 5.6 says fees are non-refundable except where law requires otherwise or VRYOH agrees otherwise. The subscription page says simply “No refunds.” Stripe’s Managed Payments documentation says Stripe may issue refunds in certain cases, including certain refunds within 60 days, and may refund without the business’s approval if it fails to respond to a support escalation within 48 hours. Review wording that acknowledges applicable law and the applicable Stripe/Link payment terms without promising a blanket 60-day refund entitlement.

3. Free-trial disclosures
The subscription page mentions the 14-day trial, but the Terms do not explain it. Proposed wording should clearly cover payment details required, no subscription charge during the trial, automatic conversion to the selected paid plan, recurring amount based on location quantity, first charge timing, and how to cancel before the first charge. Distinguish “payment method required” from “card required,” because the inspected test checkout also allowed Cash App.

4. Cancellation and location changes
Terms Section 5.3 describes cancellation by email, with access continuing through the paid period. Section 5.4 describes location changes by email, with added locations billed at the next renewal and removals taking effect at the end of the paid period.
The account page offers a “Manage subscription” button that attempts to open Stripe’s Customer Portal. A read-only API check returned ZERO test-mode Customer Portal configurations. Managed Payments also provides management through Link.
Cancellation timing, trial cancellation, quantity changes, and prorations have not been verified end to end. Do not promise next-renewal billing or period-end cancellation until those behaviors are confirmed.

5. Renewal and trial emails
Terms Section 5.2 promises an annual renewal reminder 30 days beforehand. No reminder-sending process was found in the website code; a separate process may exist but is unverified.
Managed Payments documentation says it sends trial-start and trial-ending emails, with trial-ending reminders for trials longer than seven days. Its documented default anniversary reminder is 15 days beforehand, with configurable timing. Do not claim these settings satisfy the promised 30-day reminder without checking them. Separate documented behavior from actual delivery, especially in test mode.

6. Payment-data privacy
Privacy Section 4 describes using information only for drafting, approved publication, and results; it should also address account administration, billing, subscription management, and relevant security purposes.
Section 6 describes providers as processing data on VRYOH’s behalf. Stripe may act as an independent controller as well as a processor, and as merchant of record under Managed Payments.
Section 8 promises account/data deletion within 30 days except records required by law. Clarify that Stripe may independently retain payment records for legal, tax, accounting, and fraud-prevention purposes, and that VRYOH cannot guarantee deletion of all Stripe-held information within its own deadline.
Distinguish VRYOH’s own cookies/browser storage from Stripe/Link technologies. Do not assert that authentication or hosted checkout uses no storage. Exact current storage items remain to be audited.
Do not extend the policy’s AI-provider “generally within 30 days” statement to Stripe financial records.

7. Account-email matching and duplicate subscriptions
The payment code sends customer_email to prefill Checkout, rather than reusing an existing Stripe Customer with a valid email. Stripe documents non-editable email behavior when an existing Customer with a valid email is supplied; prefilling alone does not establish this behavior.
Subscriptions are looked up by account email, taking the first matching Stripe Customer. Checkout can create a new Customer, and the server does not prevent duplicate active subscriptions or repeated trials. Explain the account-access and duplicate-billing risks. These are implementation issues, not policy wording fixes or automatically Stripe-policy violations.

8. Consent evidence
Email signup records policy acceptance and a timestamp. Google/Apple signup requires checking the consent box before starting, but durable recording of that acceptance was not found. The inspected Checkout Session had consent_collection set to null; this does not by itself prove that no consent was obtained elsewhere.
Review what affirmative recurring-payment consent and retained evidence are needed. Distinguish policy acceptance from optional newsletter/promotional consent.

EXISTING DELETION CONSTRAINT
Do not propose or assert a 30-day maximum retention period for Google review content as an existing practice. That policy change is pending until the owner confirms the operational purge is live. Preserve the existing Google Limited Use and individual reply-approval provisions unless a specific problem is identified.

OFFICIAL SOURCES TO CHECK
https://docs.stripe.com/get-started/checklist/website
https://docs.stripe.com/payments/managed-payments/how-it-works
https://docs.stripe.com/billing/subscriptions/trials/manage-trial-compliance
https://docs.stripe.com/payments/checkout/free-trials
https://docs.stripe.com/api/checkout/sessions/create
https://docs.stripe.com/payments/save-during-payment
https://stripe.com/privacy
https://stripe.com/legal/dpa
https://stripe.com/legal/ssa-service-terms
https://stripe.com/legal/restricted-businesses

REQUESTED OUTPUT
A. A concise table: aligned, needs revision, or unverified, with reasons and official citations.
B. Prioritized issues that should be resolved before accepting real payments.
C. Proposed English replacement wording for affected Terms and Privacy sections, clearly labeled as drafts requiring owner approval. Separate wording for ordinary Stripe processing versus Managed Payments wherever necessary.
D. A checklist of payment settings and customer flows to test, including trial cancellation, annual renewal notices, location-change charges, refunds, and correct account access after Checkout.
E. A short list of decisions or operational facts the owner must confirm before finalizing the policies.

Preserve the agreed prices and business facts. Do not invent refund commitments, retention schedules, reminder systems, or cancellation behavior. Explain where professional legal review or Stripe confirmation is still needed.
