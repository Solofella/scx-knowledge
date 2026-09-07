CLAIM: Account B (VRYOH's Google account) can authenticate via OAuth using its own credentials to access and manage reviews for a restaurant owned by Account A, provided Account B has been granted an Owner or Manager role on that location via the Business Profile UI.
EVIDENCE FOR: Standard Google Business Profile API OAuth 2.0 architecture and Account Management API specs. The access token represents the identity and direct permissions of the consenting Google user (Account B), not the original listing creator (Account A).
EVIDENCE AGAINST: None.
CONCLUSION: ✅ HIGH — The claim from the previous chat is fully accurate based on Google's permission model. Account A does not need to participate in the OAuth consent flow.

```

---

### 1. How Google Evaluates Access in the OAuth Model

Google's authorization model uses a **Two-Tier Permission Model**:

```
[Tier 1: Resource Authority]          [Tier 2: API Consent Scope]
Is Account B a Manager/Owner   AND    Has Account B authorized your OAuth
of Location X on Google?              App ID with 'business.manage' scope?
         │                                            │
         └──────────────────────┬─────────────────────┘
                                ▼
                       Access Granted (200 OK)

```

1. **Identity & Role Evaluation:** Google checks: *"Does the Google Account identified by the Bearer Access Token (Account B) have an explicit Manager or Owner role on Location X?"*
2. **OAuth Scope Evaluation:** Google checks: *"Did Account B grant the `[https://www.googleapis.com/auth/business.manage](https://www.googleapis.com/auth/business.manage)` scope to this specific GCP Client ID?"*

Google **does not** track whether Account A approved your GCP App. Because Account A added Account B as a Manager via `business.google.com`, Account A explicitly delegated native Google resource control to Account B. Once delegated, Account B holds native permissions to access that listing programmatically using its own OAuth credentials.

---

### 2. Account B OAuth Grant Behavior

```
CLAIM: Account B can grant OAuth access to its managed locations without Account A's involvement.
EVIDENCE FOR: Google OAuth 2.0 protocol specifications for resource delegates.
EVIDENCE AGAINST: None.
CONCLUSION: ✅ HIGH

```

When Account B completes the OAuth flow:

* Account B logs into Google and clicks **Allow** on the consent screen (`business.manage` scope).
* Google issues an Access Token / Refresh Token pair tied strictly to **Account B**.
* When making an API request, Google checks Account B's list of accessible locations.
* Because Account A added Account B as a Manager through the Business Profile UI, Location X appears in Account B's resource list. The API call succeeds.

---

### 3. Role Permission vs. OAuth Scope

They are two independent permission checks that must both be satisfied:

* **Resource Role (Google UI Level):** Authorizes *who* can touch the data (Account B is a Manager).
* **OAuth Scope (API / GCP Level):** Authorizes *what software application* can act on behalf of that user (Your n8n app with `business.manage` scope).

Having the Manager role alone gives Account B manual UI access. Completing the OAuth flow authorizes your server application to act as Account B programmatically.

---

### 4. Step-by-Step API Discovery Sequence

Once Account B completes the OAuth flow, run this sequence in n8n to discover the `accountId` and `locationId`:

#### Step 1: Discover Account B's Google Business Account ID

Make an HTTP `GET` request to fetch the account resource name:

* **Endpoint:** `GET [https://mybusinessaccountmanagement.googleapis.com/v1/accounts](https://mybusinessaccountmanagement.googleapis.com/v1/accounts)`
* **Headers:** `Authorization: Bearer <ACCOUNT_B_ACCESS_TOKEN>`
* **Response:**

```json
{
  "accounts": [
    {
      "name": "accounts/109876543210987654321",
      "accountName": "VRYOH Agency Account",
      "type": "PERSONAL"
    }
  ]
}

```

* Extract `accounts/109876543210987654321` (this is your `accountId`).

#### Step 2: List Locations Managed by Account B

Use the `accountId` from Step 1 to list all associated locations (including client locations where Account B is a Manager):

* **Endpoint:** `GET [https://mybusinessbusinessinformation.googleapis.com/v1/accounts/109876543210987654321/locations?readMask=name,title,storefrontAddress](https://mybusinessbusinessinformation.googleapis.com/v1/accounts/109876543210987654321/locations?readMask=name,title,storefrontAddress)`
* **Headers:** `Authorization: Bearer <ACCOUNT_B_ACCESS_TOKEN>`
* **Response:**

```json
{
  "locations": [
    {
      "name": "locations/12345678901234567890",
      "title": "Orlando Restaurant Location",
      "storefrontAddress": { ... }
    }
  ]
}

```

* Extract `locations/12345678901234567890` (this is your `locationId`).

#### Step 3: Fetch Reviews for the Location

Fetch reviews using the legacy My Business Reviews endpoint host:

* **Endpoint:** `GET [https://mybusiness.googleapis.com/v4/accounts/109876543210987654321/locations/12345678901234567890/reviews](https://mybusiness.googleapis.com/v4/accounts/109876543210987654321/locations/12345678901234567890/reviews)`
* **Headers:** `Authorization: Bearer <ACCOUNT_B_ACCESS_TOKEN>`

#### Step 4: Publish Reply

Publish an approved draft reply to a specific review:

* **Endpoint:** `PUT [https://mybusiness.googleapis.com/v4/accounts/109876543210987654321/locations/12345678901234567890/reviews/](https://mybusiness.googleapis.com/v4/accounts/109876543210987654321/locations/12345678901234567890/reviews/)<REVIEW_ID>/reply`
* **Headers:** `Authorization: Bearer <ACCOUNT_B_ACCESS_TOKEN>`
* **Body:**

```json
{
  "comment": "Thank you for visiting Orlando Restaurant! We appreciate your review."
}

```

---

### 5. Architectural Takeaway

Your original plan to build a complex end-user OAuth consent flow for the restaurant owner was unnecessary. Because Account B is already added as a Manager on the Google Business Profile, self-authenticating Account B once via OAuth grants your n8n workflow full, ongoing API access to the restaurant's reviews.
