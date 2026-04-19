SubtextCX Client Signal Intelligence Dashboard
Freelancer Implementation Brief — UPDATED v1.1
Effective: Chat #79 · April 19, 2026
Project Owner: Miguel Arellano, Solofella LLC
Status: Ready for freelancer handoff
Audience: Frontend developer (intermediate to advanced, Node.js + Express + vanilla JS)

## 1. Purpose & Context

SubtextCX is a B2B emotional signal intelligence platform for hospitality operators. The dashboard is the client-facing window — a GM or owner logs in and sees guest signal patterns for the current week/month without accessing pipeline infrastructure.

**First pilot client:** Park Avenue Kitchen by David Burke, Manhattan NY (ID: `PAK-001`)  
**Dashboard readiness:** Must be professional enough for day-one pilot presentation to GM.

**Philosophy:** Dashboard shows WHAT signals exist (detection/interpretation). It never shows WHAT to do (governance principle locked).

---

## 2. Technical Stack (Fixed)

| Component | Specification |
|-----------|---|
| **Backend** | Express.js (Node.js), port 3000 |
| **Hosting** | DigitalOcean droplet: Ubuntu 24.04, 4GB RAM, IP: 161.35.133.49 |
| **Frontend** | Vanilla HTML, no build step. Tailwind CSS via CDN. Chart.js via CDN. |
| **Data source** | NocoDB API (self-hosted on same droplet, internal network http://nocodb:8080) |
| **Process manager** | PM2 (keeps Express running across reboots) |
| **CDN sources allowed** | cdnjs.cloudflare.com, cdn.jsdelivr.net, unpkg.com ONLY |
| **Auth** | Magic link (72-hour token expiry, stored in NocoDB) |
| **SSL** | Let's Encrypt via Certbot (free, auto-renewing) |
| **Reverse proxy** | Nginx (port 443 → Express port 3000) |

---

## 3. URL Structure & Domain Configuration

**Format:** `subtextcx.[parentbrand].com/[client-id]`

**Example:** `subtextcx.venuiq.com/PAK-001`

**Domain status:**
- `subtextcx.com` — owned, not yet live. Must go live before any outbound prospect contact.
- `[parentbrand].com` — TBD (brand naming session pending). Freelancer receives CNAME record value once domain selected.

**DNS Configuration (Miguel handles):**
- One CNAME record at domain registrar → points `subtextcx.[parentbrand].com` subdomain to droplet IP `161.35.133.49`
- Freelancer documents exact record needed in README

**Nginx Configuration (Freelancer creates):**
```nginx
server {
  listen 443 ssl http2;
  server_name subtextcx.*.com;
  ssl_certificate /etc/letsencrypt/live/subtextcx.[parentbrand].com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/subtextcx.[parentbrand].com/privkey.pem;
  
  location / {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```

Freelancer installs SSL cert via:
```bash
sudo certbot certonly --standalone -d subtextcx.[parentbrand].com
```

---

## 4. Authentication Flow — Magic Link

**Step 1: Token Generation**
- MRA workflow generates UUID token
- Writes to NocoDB MRA table with 72-hour expiry timestamp
- Sends email: "Your [weekday] brief is ready" + magic link

**Step 2: Link Click**
```
User clicks: https://subtextcx.venuiq.com/access?token=[UUID]
```

**Step 3: Express Validation**
- GET `/access?token=[UUID]`
- Queries NocoDB MRA table: `Token = [UUID]` AND `Expires > now` AND `Used = false`
- If valid: sets session cookie (7-day expiry) + marks token as `Used = true` + redirects to `/PAK-001`
- If invalid/expired: serves clean error page: "This link has expired. Your next brief arrives Monday."

**Step 4: Session Persistence**
- Subsequent visits to `/PAK-001` check session cookie
- Express validates session, filters all NocoDB queries by authenticated `client_id`
- Client browser never receives or sees `client_id` (server-side filtering only)

**Session Security:**
```javascript
// Express session middleware
const session = require('cookie-session');
app.use(session({
  name: 'scx-session',
  keys: [process.env.SESSION_SECRET],
  maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
}));

// Middleware: validate session
const requireAuth = (req, res, next) => {
  if (!req.session.client_id) return res.redirect('/access-expired');
  next();
};
```

---

## 5. Dashboard Visual Design System

### Color Palette (Dark Mode)

| Token | Hex | Usage |
|-------|-----|-------|
| **Page Background** | `#0f1117` | Main page bg |
| **Card Background** | `#1a1d27` | Card bg, container bg |
| **Primary Text** | `#ffffff` | Headlines, KPI numbers (26-28px) |
| **Muted Text** | `rgba(255,255,255,0.75)` | Body text (12-13px) |
| **Metadata Text** | `rgba(255,255,255,0.28)` | Footer, timestamps (10px) |
| **Border Color** | `#2a2d3a` | Card borders (0.5px solid) |
| **T-NEGATIVE** | `#f87171` (red) | Negative/dignity-risk signal |
| **T-AMBIGUOUS** | `#fbbf24` (amber) | Mixed/unclear signal |
| **T-POSITIVE** | `#4ade80` (green) | Positive guest signal |
| **SEO Accent** | `#6366f1` (indigo) | SEO section cards only |

### Typography

| Element | Spec |
|---------|------|
| **Brand wordmark** | 13px, uppercase, letter-spacing 0.08em, white |
| **Page title (client name)** | 19-20px, weight 500, white |
| **Section labels** | 10px, uppercase, letter-spacing 0.1em, muted color |
| **KPI numbers** | 26-28px, weight 500, white |
| **Body / table text** | 12-13px, weight 400, rgba(255,255,255,0.75) |
| **Subtitles / notes** | 10-11px, rgba(255,255,255,0.25-0.35) |
| **Glossary footer** | 10px, rgba(255,255,255,0.28) |

**Font family:** System sans-serif stack:
```css
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
```

### Layout Rules

- **Fixed top navigation bar:** Brand wordmark LEFT, period toggle RIGHT
- **Content area:** 24px padding on all sides
- **Cards:** bg #1a1d27, border 0.5px solid #2a2d3a, border-radius 10px, padding 16-18px
- **Section gaps:** 18px vertical
- **No horizontal scroll** above 768px viewport width

---

## 6. Dashboard Sections (Single Page, 7 Sections)

### Section 0: Header

**Elements:**
- **Left:** Brand wordmark (text, not logo)
- **Center:** Client name + period label
  - Example: "Park Avenue Kitchen · Manhattan | Week of Apr 10-17, 2026"
- **Right:** Period toggle buttons (3 options)
  - "This Week" | "Last Month" | "Custom"
- **Governance note (always visible, muted):**
  - *"SubtextCX detects and interprets signals only. Operational decisions remain with your team."*

---

### Section 1: Signal Pulse (Three KPI Cards)

**Data source:** ALA NocoDB table (m57efwbtrvwohhr)

**Layout:** Three equal-width cards in horizontal row

**Card 1 — Reviews Processed**
- **Value:** COUNT of ALA records in period
- **Label:** "Reviews Processed"
- **Delta:** ↑/↓ N vs prior period (green up, red down)
- **Example:** Large "47", label below, "+3 vs last week" in green

**Card 2 — Avg Star Rating**
- **Value:** AVG of Star Rating field (ALA table)
- **Label:** "Average Star Rating"
- **Delta:** ↑/↓ X.X vs prior period
- **Example:** Large "4.2", label below, "+0.1 vs last week" in green

**Card 3 — SLA Compliance**
- **Value:** % of RDA records where `(Published Timestamp - ALA Timestamp) ≤ 48 hours`
- **Label:** "SLA Compliance" with static note: "Drafts approved within 48hr"
- **Delta:** No arrow, static percentage
- **Example:** Large "96%", label below, "No change" in muted text

---

### Section 2: Platform Inputs (Three Cards, Conditional)

**Data source:** ALA NocoDB table, Platform field

**Layout:** One card per detected platform (Google / OpenTable / Yelp only)

**Card per platform shows:**
- Platform name (e.g., "Google")
- Platform icon/initial in branded color (Google: #4285F4, OpenTable: #DA3743, Yelp: #D32323)
- Review count: COUNT where Platform = [name]
- Avg star rating: AVG where Platform = [name]

**Conditional rendering:**
- Only show platforms with ≥1 record in period
- If no Yelp records this week, Yelp card hidden

**Example layout:**
```
┌─────────────┐  ┌──────────────┐  ┌────────┐
│ G Google    │  │ O OpenTable  │  │ Y Yelp │
│ 23 reviews  │  │ 18 reviews   │  │ 6 revi │
│ ★ 4.5       │  │ ★ 4.2        │  │ ★ 4.7  │
└─────────────┘  └──────────────┘  └────────┘
```

---

### Section 3: Signal Distribution + Response Draft Status (Two-Column Row)

**This is a 2-COLUMN LAYOUT. Do NOT make it full-width.**

#### Left Card — Signal Distribution (Donut Chart)

**Data source:** SIA NocoDB table (mdn68l4lm609fve)

**Chart:** Chart.js doughnut with cutout: 68%

**Legend (custom HTML, no Chart.js legend):**
- Below canvas, row-by-row
- Each row: colored dot + tier name (left-aligned, 86px fixed width) | count + percentage (right-aligned)
- Thin horizontal bar below each row showing percentage fill

**Tier breakdown:**
| Tier | Color | Source |
|------|-------|--------|
| **T-NEGATIVE** | #f87171 | COUNT of SIA records where Signal Tier = T-NEGATIVE |
| **T-AMBIGUOUS** | #fbbf24 | COUNT of SIA records where Signal Tier = T-AMBIGUOUS |
| **T-POSITIVE** | #4ade80 | COUNT of SIA records where Signal Tier = T-POSITIVE |

**Example:**
```
Donut chart (visual)
Below chart:
● T-NEGATIVE    12 (18%)  ████░░░░░░
● T-AMBIGUOUS   23 (35%)  ██████░░░░
● T-POSITIVE    31 (47%)  ███████░░░
```

#### Right Card — Response Draft Status (Table)

**Data source:** RDA NocoDB table (mr1v67cszcklwns)

**Table structure (5 columns):**
| Column | Source | Notes |
|--------|--------|-------|
| **Tier** | Confirmed Response Tier field | "T1 Standard" / "T2 Calibrated" / "T3 Dignity-restoration" |
| **Total** | COUNT of RDA records by Confirmed Response Tier in period | All records for that tier |
| **Approved** | COUNT where Approval Status = "Approved" OR "Edited-Approved" | Published or ready |
| **Pending** | COUNT where Approval Status = "Pending" OR "Pending-Elevated" | Awaiting review |
| **Avg approval** | AVG hours between RDA Timestamp and Published Timestamp (for approved records only) | Approval velocity |

**Styling:**
- Column headers: uppercase, muted label style (10px)
- Numbers: right-aligned
- Thin dividers between rows
- One row per response tier (3 rows total)

**Example:**
```
Tier             Total  Approved  Pending  Avg approval
T1 Standard       28      26         2       12 hours
T2 Calibrated     18      15         3       18 hours
T3 Dignity-rest    8       7         1       24 hours
```

---

### Section 4: Signal Volume by Domain (Full-Width Card)

**Data source:** SIA NocoDB table (mdn68l4lm609fve)

**Layout:** 2-column GRID inside card

**Subtitle under section label:** *"Each domain shows signal split across all three tiers — not all signals are concerns."*

**Domain cells (up to 13 possible, typically 5-8 shown):**

Each domain cell contains:
- **Domain name (12px, weight 500, white)** + total signal count right-aligned in muted text ("N signals")
- **Three rows** (one per tier):
  - Colored dot | Tier label (86px fixed width) | Proportional bar | Count
- **Thin divider** between each domain cell

**Bar width calculation:** Relative to highest count WITHIN that domain (not global max)

**All 13 possible domains:**
1. Service Quality & Delivery
2. Food & Beverage Quality
3. Value & Pricing Perception
4. Ambience & Environment
5. Staff Behavior & Attitude
6. Hygiene & Cleanliness
7. Expectation vs. Reality
8. Communication & Responsiveness
9. Reservation & Booking Experience
10. Technology & Digital Experience
11. Loyalty & Recognition
12. Accessibility & Inclusion
13. Safety & Wellbeing

**Rendering rule:** Hide domains with zero records in period.

**Example visual:**
```
┌────────────────────────────────────────┐
│ Each domain shows signal split...      │
├────────────────────────────────────────┤
│ Service Quality  12 signals             │
│ ● T-NEGATIVE     ████░░░░░░  5         │
│ ● T-AMBIGUOUS    ███░░░░░░░  3         │
│ ● T-POSITIVE     █████░░░░░  4         │
│                                        │
│ Food Quality     8 signals              │
│ ● T-NEGATIVE     ██░░░░░░░░  1         │
│ ● T-AMBIGUOUS    ███░░░░░░░  2         │
│ ● T-POSITIVE     █████░░░░░  5         │
└────────────────────────────────────────┘
```

---

### Section 5: Trend Signals (Full-Width Card)

**Data source:** SIA NocoDB table (comparison of current vs prior period)

**Layout:** Up to 5 rows max

**Sorting:** By Signal Tier (T-NEGATIVE first) then by count descending

**Visibility:** Exclude singletons (Is Singleton = true in SIA records)

**Each row contains:**
- **Tier badge (left):** Small pill, colored by tier (red/amber/green), font 9px bold
- **Domain name** + **record count** + **prior count** + **trend direction arrow/label**

**Trend indicators:**
- **↑ Growing:** Red arrow (count increased >10%)
- **↓ Declining:** Green arrow (count decreased >10%)
- **→ Stable:** Muted dash (change ±10%)
- **● New:** Indigo dot (first appearance, no prior data)

**Example:**
```
🔴 T-NEGATIVE · Service Quality · 12 records (prior: 8) · ↑ Growing +50%
🟡 T-AMBIGUOUS · Food Quality · 6 records (first detected) · ● New
🟢 T-POSITIVE · Ambiance · 15 records (prior: 16) · ↓ Declining -6%
```

**Timestamp (bottom right, muted):** "Data as of [Run Timestamp in ISO format]"

---

### Section 6: SEO Performance (Full-Width Card, Phase 1)

**Data source:** RDA NocoDB table (published records) + Client Config keywords

**Layout:** Three KPI sub-cards (indigo accent) + keyword list below

**Styling for sub-cards:**
- Background: `rgba(99,102,241,0.08)` (very light indigo)
- Border: `rgba(99,102,241,0.2)` (light indigo)
- Visually separated from signal cards by color

**KPI 1 — Keyword Placements**
- **Label:** "Keyword Placements"
- **Value:** COUNT of published RDA records containing ≥1 SEO keyword from Client Config
- **Example:** "47"

**KPI 2 — Coverage Rate**
- **Label:** "Coverage Rate"
- **Value:** (Keyword Placements ÷ Total Published Records) × 100, formatted as %
- **Example:** "82%"

**KPI 3 — Avg Response Velocity**
- **Label:** "Avg Response Velocity"
- **Value:** AVG hours from ALA Timestamp → Published Timestamp for published records
- **Example:** "18 hours"

**Keyword List (below sub-cards):**
- Top 5-8 keywords placed, sorted by frequency
- Format: "keyword_text: N placements"
- Example:
  ```
  birthday celebration: 12
  anniversary dinner: 8
  special occasion: 6
  date night: 5
  ```

**Phase 2 Note (static, muted, bottom of card):**
*"Phase 2 · Google Search Console integration — keyword rank positions and impression trends coming Q3 2026"*

---

## 7. Glossary Footer (Always Visible)

**Positioning:** Permanently at bottom of page, below all content

**Styling:** Thin top border, low-contrast font (10px, rgba(255,255,255,0.28))

**Layout:** Flex-wrap row

**Term styling:** Term name slightly brighter (rgba(255,255,255,0.40), weight 500)

**Seven terms:**
| Term | Definition |
|------|-----------|
| **T1 Standard** | Review with no elevated concern. Standard polished response. |
| **T2 Calibrated** | Mixed or ambiguous signal. Response requires careful tone calibration. |
| **T3 Dignity-restoration** | Negative or dignity-risk signal. Highest attention required before approval. |
| **SLA compliance** | % of drafts reviewed and approved within the 48-hour service commitment. |
| **T-NEGATIVE** | Confirmed negative or dignity-risk guest signal. |
| **T-AMBIGUOUS** | Signal with unclear intent. Monitoring required. |
| **T-POSITIVE** | Positive guest signal. Not all signals in a domain are concerns. |

---

## 8. Period Toggle & Date Picker

**Buttons:** Three toggle buttons in top-right navigation

1. **"This Week"** — Last 7 calendar days
2. **"Last Month"** — Last 30 calendar days
3. **"Custom"** — Reveals two date inputs (from / to) inline in top bar

**Active state styling:**
- Selected button: `background: rgba(255,255,255,0.10)`, white text
- Inactive buttons: transparent background, muted text

**Custom date range:**
- Two native HTML date inputs (`<input type="date">`)
- Apply button sends API call with `?from=YYYY-MM-DD&to=YYYY-MM-DD`
- Dashboard re-renders all 7 sections with new data (no page reload)

**API calls:**
```
GET /api/dashboard/PAK-001?period=week
GET /api/dashboard/PAK-001?period=month
GET /api/dashboard/PAK-001?from=2026-04-10&to=2026-04-17
```

---

## 9. Data Flow & API Specification

### Client-Side Flow
```
User clicks magic link (from email)
  ↓
Browser: GET /access?token=[UUID]
  ↓
Express validates token, sets session cookie
  ↓
Redirect: POST /PAK-001
  ↓
Frontend loads dashboard.html
  ↓
JavaScript: fetch('/api/dashboard/PAK-001?period=week')
  ↓
Chart.js renders donut, bars, tables
  ↓
User clicks "Last Month" button
  ↓
Fetch: '/api/dashboard/PAK-001?period=month'
  ↓
All sections re-render with new data
```

### Express API Endpoint

**Endpoint:** `GET /api/dashboard/:clientId`

**Query Parameters:**
- `period` (optional): `week` or `month` (default: `week`)
- `from` & `to` (optional): ISO date strings for custom range

**Response JSON Schema:**

```json
{
  "client_id": "PAK-001",
  "client_name": "Park Avenue Kitchen by David Burke",
  "period": "week",
  "date_range": "April 10-17, 2026",
  
  "signal_pulse": {
    "reviews_processed": {
      "count": 47,
      "delta": "+3",
      "delta_direction": "up"
    },
    "avg_star_rating": {
      "value": 4.2,
      "delta": "+0.1",
      "delta_direction": "up"
    },
    "sla_compliance": {
      "percentage": 96,
      "delta": null
    }
  },

  "platform_inputs": [
    {
      "platform": "Google",
      "platform_icon_color": "#4285F4",
      "review_count": 23,
      "avg_star_rating": 4.5
    },
    {
      "platform": "OpenTable",
      "platform_icon_color": "#DA3743",
      "review_count": 18,
      "avg_star_rating": 4.2
    },
    {
      "platform": "Yelp",
      "platform_icon_color": "#D32323",
      "review_count": 6,
      "avg_star_rating": 4.7
    }
  ],

  "signal_distribution": {
    "t_negative": { "count": 12, "percentage": 18 },
    "t_ambiguous": { "count": 23, "percentage": 35 },
    "t_positive": { "count": 31, "percentage": 47 }
  },

  "response_draft_status": [
    {
      "tier": "T1 Standard",
      "total": 28,
      "approved": 26,
      "pending": 2,
      "avg_approval_hours": 12
    },
    {
      "tier": "T2 Calibrated",
      "total": 18,
      "approved": 15,
      "pending": 3,
      "avg_approval_hours": 18
    },
    {
      "tier": "T3 Dignity-restoration",
      "total": 8,
      "approved": 7,
      "pending": 1,
      "avg_approval_hours": 24
    }
  ],

  "signal_volume_by_domain": [
    {
      "domain": "Service Quality & Delivery",
      "total_signals": 12,
      "t_negative": 5,
      "t_ambiguous": 3,
      "t_positive": 4
    },
    {
      "domain": "Food & Beverage Quality",
      "total_signals": 8,
      "t_negative": 1,
      "t_ambiguous": 2,
      "t_positive": 5
    }
  ],

  "trend_signals": [
    {
      "tier": "T-NEGATIVE",
      "domain": "Service Quality & Delivery",
      "current_count": 12,
      "prior_count": 8,
      "trend_direction": "up",
      "percentage_change": "+50%"
    },
    {
      "tier": "T-AMBIGUOUS",
      "domain": "Food & Beverage Quality",
      "current_count": 6,
      "prior_count": null,
      "trend_direction": "new",
      "percentage_change": null
    }
  ],

  "seo_signal": {
    "keyword_placements": 47,
    "coverage_rate": "82%",
    "avg_response_velocity_hours": 18,
    "top_keywords": [
      { "keyword": "birthday celebration", "count": 12 },
      { "keyword": "anniversary dinner", "count": 8 },
      { "keyword": "special occasion", "count": 6 }
    ]
  },

  "run_timestamp": "2026-04-17T08:30:00Z"
}
```

---

## 10. NocoDB Table References

**Base ID:** `pq249fix22t3ofv`

| Table | Table ID | Used by Section | Notes |
|-------|----------|-----------------|-------|
| ALA | m57efwbtrvwohhr | Signal Pulse, Platform Inputs | Raw reviews + metadata |
| SIA | mdn68l4lm609fve | Signal Distribution, Domain Volume, Trend Signals | Aggregated by domain + tier |
| RDA | mr1v67cszcklwns | Draft Status, SEO Signal | Response drafts + approval status |
| Client Config | m95cmabjfyb94ps | Header, all sections | Client brand voice + SEO keywords |
| MRA | (TBD) | Auth token table | Magic link tokens (72hr expiry) |

**Client Config Critical Fields:**
- `Client ID` (text): e.g., "PAK-001"
- `Client Name` (text): e.g., "Park Avenue Kitchen by David Burke"
- `SEO Keywords` (long text): comma-separated keywords
- `Approval Contact Email` (email): who receives draft approval notifications

---

## 11. Governance Rules — What Must NEVER Appear

**SubtextCX is detection/interpretation only. Dashboard NEVER prescribes actions.**

**Explicitly forbidden on dashboard:**
- ✕ Individual review text or guest-written content
- ✕ Guest names or reviewer handles
- ✕ Draft response text (only show counts: Approved/Pending)
- ✕ Any recommendation, suggestion, or action item
- ✕ Language: "you should", "we recommend", "action required"
- ✕ Staff names in operational context
- ✕ Any content interpreted as prescribing what client must do

**Governance note (always visible in header, muted):**
> *"SubtextCX detects and interprets signals only. Operational decisions remain with your team."*

---

## 12. Multi-Tenant Architecture

**Per-client routing:** `/[client-id]`

**Session security:**
- `client_id` stored in server session (never exposed to browser)
- All NocoDB queries filtered server-side: `WHERE client_id = [session.client_id]`
- Multi-tenant tokens validated: PAK-001 session cannot access `/edo-001`

**Adding a new client:**
1. Create one NocoDB row in Client Config table (Client ID + Client Name + brand voice fields)
2. No code changes
3. New client can access dashboard via magic link

---

## 13. Deliverables Expected from Freelancer

### Code Files

1. **`server.js`** — Express backend
   - Routes: `/`, `/access?token=[UUID]`, `/api/dashboard/:clientId`, `/logout`
   - NocoDB API proxy calls (filters by client_id server-side)
   - Session middleware + magic link token validation
   - Error handling + graceful degradation

2. **`dashboard.html`** — Frontend (single file)
   - All CSS inline (Tailwind via CDN)
   - All JavaScript inline
   - Chart.js via CDN for donut + bar charts
   - Responsive layout (768px mobile breakpoint minimum)

3. **`README.md`** — Deployment guide
   - Environment variables: `NOCODB_URL`, `NOCODB_TOKEN`, `SESSION_SECRET`, `PORT=3000`
   - Local testing: `npm install`, `node server.js`
   - Droplet deployment: PM2 start, Nginx config, Certbot SSL
   - Client management operations (add new client, change colors, rotate token, restart PM2)

### Documentation

4. **`Operations_Guide.md`** — Required
   - How to add a new client (create NocoDB row)
   - How to change client display name
   - How to update brand colors (Client Config table)
   - How to rotate NocoDB token (security)
   - How to restart PM2 if needed
   - Troubleshooting common errors

### Reference Materials Provided to Freelancer

- **Prototype HTML file** (`subtextcx_dashboard_v4.html`) — interactive mockup with synthetic PAK-001 data, visual reference only
- **PDF brief** (SubtextCX_Dashboard_Freelancer_Brief.pdf) — original specification (Chat #75)
- **This markdown brief** — authoritative spec (Chat #79)

**If prototype and brief conflict: Brief takes precedence.**

---

## 14. Freelancer Sourcing & Onboarding

**Where to post:** Upwork (or equivalent freelance marketplace)

**Key skills to filter for:**
- Node.js + Express (3+ years)
- Vanilla JavaScript (no React/Vue required)
- HTML + CSS fundamentals
- REST API integration (NocoDB)
- Session management + token validation
- Chart.js OR similar data visualization library
- Linux/Ubuntu + Nginx basics (for deployment instructions)

**Preferred experience (not required):**
- DigitalOcean or similar VPS deployment
- Let's Encrypt / Certbot SSL
- PM2 process manager
- NocoDB OR similar headless database

**Revision rounds included:** 1-2 visual/UX refinements expected (budget accordingly)

**Contract terms:**
- Code ownership: All code becomes property of Solofella LLC upon delivery and payment
- Handoff method: GitHub repository access OR ZIP file with all source
- Testing: Freelancer tests locally before delivery; Miguel final QA on droplet

---

## 15. Project Timeline & Milestones

| Milestone | Week | Deliverable | Owner |
|-----------|------|-------------|-------|
| M1 — Frontend layout | Week 1 | HTML + Tailwind complete, static layout approved | Freelancer |
| M2 — Backend + data | Week 2 | Express API working, NocoDB integration tested | Freelancer |
| M3 — Auth + SSL | Week 3 | Magic link auth + Nginx + SSL certificate active | Freelancer |
| M4 — Live testing | Week 4 | PAK-001 pilot test, bug fixes, deployment to prod | Freelancer + Miguel |

**Total estimated duration:** 4 weeks  
**Start date:** TBD (after freelancer onboarded)

---

## 16. Testing Data & Prototype

**Interactive prototype provided:** `subtextcx_dashboard_v4.html`
- Fully functional layout with synthetic PAK-001 data
- Visual reference for freelancer
- Demonstrates period toggle behavior, chart rendering, domain grid
- Run locally in browser to preview final product appearance

**Testing approach:**
- Freelancer develops with mock JSON data (hardcoded)
- Once layout approved, integrate live NocoDB queries
- Miguel provides sample NocoDB credentials for testing on droplet

---

## 17. Security & Privacy Checklist

- [ ] Session cookie secure flag set (HTTPS only)
- [ ] Magic link token validated (exists, not expired, not used)
- [ ] Client_id never exposed in frontend (server-side filtering only)
- [ ] NocoDB xc-token stored as environment variable (never in code)
- [ ] SESSION_SECRET stored as environment variable (never hardcoded)
- [ ] All NocoDB queries filtered by authenticated client_id
- [ ] No individual review text sent to frontend
- [ ] No guest names exposed to frontend
- [ ] HTTPS enforced (redirect http → https)
- [ ] CORS headers restricted (dashboard origin only)

---

## 18. Known Limitations & Phase 2 Features

**Phase 1 (Current):**
- SEO metrics are counts + frequency only (no Google Search Console API yet)
- No client logo upload (text name only)
- No email reply integration (magic link only)
- No scheduled report email (MRA not yet built)

**Phase 2 (Future):**
- Google Search Console API integration (keyword rankings, impressions, clicks)
- Client logo upload + white-label branding
- Scheduled email delivery of weekly/monthly reports
- Google Sheet approval interface (instead of NocoDB direct edit)
- Multi-location client dashboard (aggregate across branches)

---

## 19. Contact & Support

**Project Owner:** Miguel Arellano  
**Email:** solofellausa@gmail.com  
**Droplet Access:** Miguel provides SSH key after contract signed  
**GitHub Repo:** Freelancer given write access for code submission  

**Escalation path:**
1. Code review questions → async GitHub issues
2. Design questions → screenshare call (async preferred)
3. Blockers → Slack or email within 24hr

---

## 20. Final Notes

This dashboard is the client-facing window into SubtextCX. It must feel professional, fast, and intuitive. A GM should absorb the key intelligence in under 90 seconds on Monday morning.

**Design philosophy:**
- Show signals clearly
- Let the client interpret
- No prescriptions
- No confusion

The prototype is the visual north star. This brief is the functional specification. Build toward both.

---

**END OF UPDATED DASHBOARD FREELANCER BRIEF — v1.1**  
**Chat #79 · April 19, 2026**  
**Solofella LLC · SubtextCX**
