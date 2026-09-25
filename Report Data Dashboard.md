Chat #105 · July 25, 2026

### Answers to Your Questions

**1. Does NocoDB itself cover this need natively?**
⚠️ **UNVERIFIED / PARTIAL.** NocoDB allows you to share individual **Grid/Table Views** publicly via unlisted links, which works natively for raw tabular numbers. However, native **Dashboards with charts** (bar, line, KPI widgets) are a feature restricted to Cloud/Paid plans, and public dashboard sharing is part of their paid feature set. Relying purely on self-hosted community NocoDB for *visual charts* directly via public links will hit feature limitations.

**2. What low-code BI/dashboard tools can connect directly to Postgres/REST API on your droplet?**
Metabase and Grafana can connect directly to your underlying Postgres database on `localhost` (bypassing NocoDB's REST API layer entirely). Retool and Appsmith are low-code app builders that require configuring UI widgets manually. For a non-developer seeking simple automated reporting, Metabase is the standard choice for direct Postgres read-access.

**3. Is Google Sheets + Google Looker Studio viable?**
✅ **HIGH.** This is the simplest, lowest-friction option for your current pilot setup. Since you already have n8n pushing report data to Google Sheets, Looker Studio connects to Google Sheets natively in seconds with zero code, zero backend setup, and native support for shareable unlisted dashboard links.

**4. Realistic setup time & hidden infrastructure requirements:**
No option requires building a custom API service or migrating data from NocoDB. However, self-hosting a new BI tool on your DigitalOcean droplet consumes RAM and requires configuring a reverse proxy (e.g., Nginx/Caddy) or exposing ports.

---

### Ranked Options (Lowest Effort/Complexity First)

| Rank | Option | Setup Time | Hidden Complexity / Pitfalls |
| --- | --- | --- | --- |
| **1** | **Google Sheets + Looker Studio** | **15–30 mins** | None. Reuses existing n8n-to-Sheets pipeline. No Docker or droplet changes required. |
| **2** | **NocoDB Public Grid Views** | **10 mins** | Extremely simple, but displays raw spreadsheet grids/numbers only—no native visual charts on community self-hosted. |
| **3** | **Metabase (Self-hosted on Droplet)** | **1–2 hours** | Requires running a Docker container on your droplet and connecting to Postgres port/credentials locally. |
| **4** | **Appsmith / Retool** | **3+ hours** | High manual UI layout effort; requires building queries and binding UI components line-by-line. |

---

### Option 1 Breakdown: Google Sheets + Looker Studio *(Recommended)*

CLAIM: Looker Studio connected to Google Sheets provides unlisted client dashboards with zero droplet modification.
EVIDENCE FOR: Documented native integration between Google Sheets and Looker Studio; user confirmed an existing Google Sheets pipeline in n8n.
EVIDENCE AGAINST: Dependent on Google ecosystem; requires n8n to append report metrics to a Google Sheet.
CONCLUSION: ✅ **HIGH.** Ideal for the pilot phase.

**Setup Steps:**

1. Maintain your existing n8n workflow appending daily/weekly summary rows to a designated Google Sheet.
2. Open **Looker Studio** (free), click **Create -> Report**, and select **Google Sheets** as the data source.
3. Drag and drop KPI metric cards, time-series line charts, and bar charts.
4. Add a filter control for `Location` or create 2 separate report pages (Page 1: Aji Ceviche Bar, Page 2: Park Avenue Kitchen).
5. Click **Share -> Anyone with the link can view** and email the unlisted link to pilot clients.

---

### Option 2 Breakdown: Metabase on DigitalOcean Droplet *(Alternative if hosted on-premise)*

CLAIM: Metabase can run alongside n8n/NocoDB and query Postgres directly.
EVIDENCE FOR: Metabase provides official Docker images and connects via standard Postgres credentials.
EVIDENCE AGAINST: Increases Droplet RAM usage (~1GB required); requires opening ports or configuring Nginx for public URLs.
CONCLUSION: ⚠️ **MEDIUM.** Reliable, but adds infrastructure overhead compared to Looker Studio.

**Setup Steps:**

1. Run Metabase via Docker on your droplet:
`docker run -d -p 3000:3000 --name metabase metabase/metabase`
2. Access `http://<YOUR-DROPLET-IP>:3000` and enter your NocoDB Postgres database connection details (`host.docker.internal` or local IP, DB name, user, password).
3. Use the Metabase visual query builder to create charts directly from your summary tables.
4. Turn on **Public Sharing** in Metabase settings to generate unlisted view links per client.
