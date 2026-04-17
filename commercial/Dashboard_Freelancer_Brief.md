# SubtextCX Dashboard - Freelancer Implementation Brief

**Project:** Client-facing dashboard for SubtextCX intelligence platform  
**Deliverable:** Single-page web application (one-page client portal)  
**Timeline:** End Q2 2026 target  
**Freelancer Audience:** Frontend developer (intermediate to advanced)

---

## Technical Stack (Fixed)

**Backend:**
- Express.js (Node.js)
- Hosted on DigitalOcean droplet (port 3000)
- Ubuntu 24.04, 4GB RAM

**Frontend:**
- Vanilla HTML (no React/Vue/Angular)
- Tailwind CSS via CDN (utility-first styling)
- Chart.js via CDN (data visualization)

**Data Source:**
- NocoDB API (self-hosted on same droplet)
- Base ID: `pq249fix22t3ofv`
- Authentication: xc-token header auth

**URL Structure:**
- Format: `subtextcx.[parentbrand].com/[client-id]`
- Example: `subtextcx.venuiq.com/PAK-001`
- Subdomain routing via Nginx
- SSL via Let's Encrypt (free, auto-renewing)

---

## Authentication

**Method:** Magic link + NocoDB token table

**Flow:**
1. Client receives email with unique magic link
2. Link contains single-use token (expires 24hr)
3. Token validated against NocoDB token table
4. Session cookie set (7-day expiry)
5. Subsequent visits use session cookie

**No password login.** No username/password storage.

**Token Table Schema (NocoDB):**
- Client ID
- Token (UUID)
- Created At
- Expires At
- Used (boolean)

---

## Dashboard Layout (Single Page)

### Page Structure (6 sections)

```
┌─────────────────────────────────────────────┐
│ 1. HEADER                                   │
│    Client Name | Period Toggle              │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ 2. SIGNAL PULSE                             │
│    T-NEG / T-AMB / T-POS distribution       │
│    (Horizontal bar chart)                   │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ 3. SIGNAL DISTRIBUTION                      │
│    Domain breakdown (pie chart)             │
│    Food Quality | Service | Ambiance | etc. │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ 4. RESPONSE DRAFT STATUS                    │
│    Approved: 12 | Pending: 3                │
│    (Simple count, no draft text shown)      │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ 5. TREND SIGNALS                            │
│    Week-over-week changes                   │
│    ↑ Service +15% | ↓ Wait Time -8%         │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ 6. SEO SIGNAL (Phase 1)                     │
│    Keyword Hits: 47                         │
│    Top Keyword: "birthday celebration"      │
│    Coverage Rate: 82%                       │
│    Response Rate: 94%                       │
│    Avg Velocity: 18 hours                   │
└─────────────────────────────────────────────┘
```

---

## Section Details

### 1. Header
**Elements:**
- Client name (fetched from Client Config NocoDB table)
- Period toggle: **This Week** / **Last Month** (buttons, active state styled)
- Date range displayed (e.g., "April 10-17, 2026")

**Styling:**
- Clean, minimal
- Client logo upload NOT in Phase 1 (text name only)

---

### 2. Signal Pulse (Horizontal Bar Chart)

**Data Source:** SIA NocoDB table

**Displays:**
- T-NEGATIVE count & percentage (red bar)
- T-AMBIGUOUS count & percentage (yellow bar)
- T-POSITIVE count & percentage (green bar)

**Example:**
```
T-NEGATIVE   ████░░░░░░  12 (18%)
T-AMBIGUOUS  ██████░░░░  23 (35%)
T-POSITIVE   ██████████  31 (47%)
```

**Chart.js config:**
- Horizontal bar chart
- Color-coded (red/yellow/green)
- Percentage labels on bars
- Responsive (mobile-friendly)

---

### 3. Signal Distribution (Pie Chart)

**Data Source:** SIA NocoDB table (domain breakdown)

**Displays:**
- Domain percentages (Food Quality, Service Quality, Ambiance, Value, Wait Time, etc.)
- Top 5 domains shown in pie chart
- Remaining grouped as "Other"

**Example:**
```
Food Quality: 32%
Service Quality: 28%
Ambiance: 18%
Value: 12%
Wait Time: 10%
```

**Chart.js config:**
- Pie chart (or doughnut chart, designer's choice)
- Color-coded by domain
- Legend below chart
- Hover tooltips show exact counts

---

### 4. Response Draft Status

**Data Source:** RDA NocoDB table

**Displays:**
- Approved count (drafts marked "Approved" in Google Sheet)
- Pending count (drafts not yet reviewed)
- NOT ACCEPTED count (optional, show if >0)

**Example:**
```
✓ Approved: 12
⏳ Pending: 3
✗ Not Accepted: 1
```

**Styling:**
- Simple text with icons
- Green checkmark for Approved
- Yellow clock for Pending
- Red X for Not Accepted

**Critical:** Do NOT show actual draft text. Only counts.

---

### 5. Trend Signals

**Data Source:** SIA NocoDB table (current period vs previous period comparison)

**Displays:**
- Week-over-week OR month-over-month changes (depends on period toggle)
- Top 3 domains with biggest changes (up or down)

**Example:**
```
↑ Service Quality +15% (23 → 26 reviews)
↓ Wait Time Issues -8% (12 → 11 reviews)
↑ Food Quality +5% (18 → 19 reviews)
```

**Styling:**
- Green up arrow for increases
- Red down arrow for decreases
- Percentage change + absolute count change shown

---

### 6. SEO Signal (Phase 1)

**Data Source:** Computed from RDA NocoDB table + review metadata

**Phase 1 Metrics (no Google Search Console API yet):**

**Keyword Hits:** Count of reviews containing tracked keywords (e.g., "birthday", "anniversary", "celebration", "date night")

**Top Keyword:** Most frequently mentioned tracked keyword in reviews

**Coverage Rate:** Percentage of reviews that received a response draft

**Response Rate:** Percentage of approved drafts actually published (manual Miguel update)

**Avg Velocity:** Average time from review posted to response approved (in hours)

**Example:**
```
Keyword Hits: 47
Top Keyword: "birthday celebration"
Coverage Rate: 82%
Response Rate: 94%
Avg Velocity: 18 hours
```

**Phase 2 (Future):** Google Search Console API integration for actual search impression/click data

---

## Period Toggle Logic

**Two states:**
1. **This Week:** Last 7 days of data
2. **Last Month:** Last 30 days of data

**Backend API calls:**
- `/api/dashboard/:clientId?period=week`
- `/api/dashboard/:clientId?period=month`

**Frontend:**
- Toggle button sets period state
- All charts/metrics re-fetch with new period parameter
- Active state styled (darker background, white text)

---

## Data Flow

```
User visits URL (magic link)
  ↓
Token validation (NocoDB token table)
  ↓
Session cookie set
  ↓
Frontend loads (HTML + Tailwind + Chart.js)
  ↓
JavaScript fetch: GET /api/dashboard/:clientId?period=week
  ↓
Backend queries NocoDB (SIA, RDA, Client Config tables)
  ↓
JSON response returned
  ↓
Chart.js renders visualizations
  ↓
Period toggle click → re-fetch with ?period=month
```

---

## API Endpoint Specification

**Endpoint:** `GET /api/dashboard/:clientId`

**Query Params:**
- `period` (required): `week` or `month`

**Response JSON Structure:**

```json
{
  "client_name": "Park Avenue Kitchen by David Burke",
  "period": "week",
  "date_range": "April 10-17, 2026",
  "signal_pulse": {
    "t_negative": { "count": 12, "percentage": 18 },
    "t_ambiguous": { "count": 23, "percentage": 35 },
    "t_positive": { "count": 31, "percentage": 47 }
  },
  "signal_distribution": [
    { "domain": "Food Quality", "count": 21, "percentage": 32 },
    { "domain": "Service Quality", "count": 18, "percentage": 28 },
    { "domain": "Ambiance", "count": 12, "percentage": 18 },
    { "domain": "Value", "count": 8, "percentage": 12 },
    { "domain": "Wait Time", "count": 7, "percentage": 10 }
  ],
  "response_status": {
    "approved": 12,
    "pending": 3,
    "not_accepted": 1
  },
  "trend_signals": [
    { "domain": "Service Quality", "change": "+15%", "direction": "up", "count_change": "23 → 26" },
    { "domain": "Wait Time", "change": "-8%", "direction": "down", "count_change": "12 → 11" }
  ],
  "seo_signal": {
    "keyword_hits": 47,
    "top_keyword": "birthday celebration",
    "coverage_rate": "82%",
    "response_rate": "94%",
    "avg_velocity_hours": 18
  }
}
```

---

## Styling Guidelines (Tailwind CSS)

**Design Principles:**
- Clean, minimal, professional
- Mobile-responsive (works on phone, tablet, desktop)
- No animations or transitions (simple, fast-loading)
- Consistent spacing (Tailwind spacing scale: p-4, m-6, etc.)

**Color Palette:**
- **Primary:** Blue-gray (professional, neutral)
- **T-NEGATIVE:** Red (#EF4444)
- **T-AMBIGUOUS:** Yellow (#FBBF24)
- **T-POSITIVE:** Green (#10B981)
- **Background:** White or light gray (#F9FAFB)
- **Text:** Dark gray (#1F2937)

**Typography:**
- **Headers:** Font size 2xl-3xl, font-semibold
- **Body:** Font size base, font-normal
- **Metrics:** Font size lg-xl, font-medium

**Example Tailwind Classes:**
```html
<div class="bg-white shadow-md rounded-lg p-6 mb-4">
  <h2 class="text-2xl font-semibold text-gray-900 mb-4">Signal Pulse</h2>
  <canvas id="signalPulseChart"></canvas>
</div>
```

---

## What NOT to Show

**Critical privacy/governance constraints:**

**Do NOT display:**
- Individual review text
- Guest names
- Response draft text (only counts: Approved/Pending)
- Operational recommendations (dashboard is descriptive only, never prescriptive)

**Why:** Dashboard shows WHAT signals exist, not WHAT to do about them. Governance principle: detect/interpret only.

---

## Deliverables Expected from Freelancer

1. **Frontend HTML file** (`dashboard.html`)
   - Single-page layout
   - Tailwind CSS via CDN
   - Chart.js via CDN
   - JavaScript fetch to backend API

2. **Backend Express.js server** (`server.js`)
   - `/api/dashboard/:clientId` endpoint
   - NocoDB API integration
   - Token validation middleware
   - Session cookie handling

3. **Nginx config** (subdomain routing)
   - Route `subtextcx.[parentbrand].com` to port 3000
   - SSL certificate setup (Let's Encrypt)

4. **README** (deployment instructions)
   - Environment variables (NocoDB URL, xc-token, session secret)
   - How to run locally for testing
   - How to deploy to DigitalOcean droplet

---

## Testing Data

**Sample client IDs for testing:**
- PAK-001 (Park Avenue Kitchen)
- EDO-001 (EDO Restaurants)
- AJI-001 (AJI Ceviche)

**Backend can use mock data** for initial development (NocoDB API integration can be added after layout is approved).

---

## Timeline & Milestones

**Milestone 1 (Week 1):**
- Frontend layout complete (HTML + Tailwind)
- Static data (hardcoded JSON for visual approval)

**Milestone 2 (Week 2):**
- Backend API endpoint complete
- NocoDB integration working
- Period toggle functional

**Milestone 3 (Week 3):**
- Magic link authentication working
- Nginx subdomain routing configured
- SSL certificate active

**Milestone 4 (Week 4):**
- Testing with PAK-001 pilot client
- Bug fixes and refinements
- Final deployment to production

**Total estimated time:** 4 weeks

---

## Questions for Freelancer to Ask

1. **Preferred Chart.js version?** (Latest stable recommended)
2. **Mobile breakpoint priorities?** (Desktop-first or mobile-first approach)
3. **Browser support requirements?** (Modern browsers only, or IE11 support needed?)
4. **Hosting SSH access?** (Will freelancer deploy directly or hand off code?)
5. **Revision rounds included?** (1-2 rounds of visual tweaks expected)

---

## Contact & Handoff

**Project Owner:** Miguel Arellano (Solofella LLC)  
**Email:** solofellausa@gmail.com  
**Handoff Method:** GitHub repository access OR ZIP file delivery

**Code ownership:** All code becomes property of Solofella LLC upon delivery and payment.

---

**End of Dashboard Freelancer Brief**
