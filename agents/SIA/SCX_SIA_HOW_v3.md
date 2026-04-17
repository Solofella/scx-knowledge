# SCX_SIA_HOW_v3.0

**Agent Name:** SIA (Signal Intensity Assessment)  
**Version:** 3.0  
**Last Updated:** Chat #74 · April 4, 2026  
**Model:** None (Pure JavaScript)  
**Status:** Verified operational - 11 nodes complete

---

## Purpose

SIA is a zero-cost aggregation agent. It reads EIP NocoDB records directly (no webhook), groups them by Domain + Signal Tier (T-NEGATIVE/T-AMBIGUOUS/T-POSITIVE), computes percentages and counts, and builds JSON breakdowns for downstream consumption.

**SIA makes ZERO AI calls.** All computation is deterministic JavaScript (`for` loops, `Object.keys()`, percentage calculations).

---

## Input Source

**Upstream Agent:** EIP (Emotion & Intelligence Preprocessor)

**SIA does NOT receive webhooks.** Instead, it queries the EIP NocoDB table directly.

**Reads from EIP table:**
- **Domain** ✓ (Pain Point Domain Confirmed)
- **Signal Type** ✓ (T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE)
- **Enriched Pain Point** ✓
- **Enriched Pain Point Breakdown JSON** ✓
- **Enriched Emotion Tag** ✓
- **Enriched Emotion Breakdown JSON** ✓
- Client ID ✓
- Date Posted ✓

**SIA does NOT query:**
- Emotion Dictionary ✗
- Pain Point Master ✗
- Any other agent tables ✗

**Why this works:** By the time EIP records exist in NocoDB, EIP has already resolved every dictionary field. SIA just reads and aggregates.

---

## Processing Logic

### Node Flow (11 Nodes Total)

**Trigger (Node 1):** Schedule (runs daily at 6am) OR Manual trigger

**INIT Nodes (2-3):** Define client ID, date range (last 7 days for weekly, last 30 days for monthly)

**EIP Query (Node 4):** NocoDB API query
```javascript
// Query EIP table for records in date range
const records = await nocodb.query({
  table: 'EIP',
  filter: `client_id = '${clientId}' AND date_posted >= '${startDate}'`
});
```

**Aggregation (Nodes 5-9):** Pure JavaScript
- **Node 5:** Group by Domain (Food Quality, Service Quality, etc.)
- **Node 6:** Group by Signal Tier (T-NEG, T-AMB, T-POS)
- **Node 7:** Count records per cluster (Domain + Tier combination)
- **Node 8:** Calculate percentages
- **Node 9:** Build JSON breakdowns (Enriched Pain Points, Enriched Emotions per cluster)

**Output (Nodes 10-11):** Write to SIA NocoDB table

---

## NocoDB Schema

**Table ID:** `mdn68l4lm609fve`

**Fields:**
- SIA Record ID (auto-increment, primary key)
- Client ID
- **Domain** (aggregation cluster key)
- **Signal Tier** (T-NEGATIVE / T-AMBIGUOUS / T-POSITIVE)
- **Count** (number of reviews in this cluster)
- **Percentage** (% of total reviews in date range)
- **Enriched Pain Points JSON** (array of pain points in this cluster)
- **Enriched Emotions JSON** (array of emotions in this cluster)
- **Trend Direction** (UP / DOWN / STABLE compared to prior period)
- Date Range Start
- Date Range End
- Created At (timestamp)

---

## Aggregation Logic (Pure JavaScript)

### Step 1: Group by Domain + Tier

```javascript
const clusters = {};

for (let i = 0; i < eipRecords.length; i++) {
  const record = eipRecords[i];
  const key = `${record.domain}|${record.signalType}`; // e.g., "Food Quality|T-NEGATIVE"
  
  if (!clusters[key]) {
    clusters[key] = {
      domain: record.domain,
      tier: record.signalType,
      count: 0,
      painPoints: [],
      emotions: []
    };
  }
  
  clusters[key].count += 1;
  clusters[key].painPoints.push(record.enrichedPainPoint);
  clusters[key].emotions.push(record.enrichedEmotionTag);
}
```

**No AI. No dictionary queries. Just object grouping.**

---

### Step 2: Calculate Percentages

```javascript
const totalRecords = eipRecords.length;

const clusterKeys = Object.keys(clusters);
for (let i = 0; i < clusterKeys.length; i++) {
  const key = clusterKeys[i];
  clusters[key].percentage = Math.round((clusters[key].count / totalRecords) * 100);
}
```

**Deterministic math. Zero variability.**

---

### Step 3: Build JSON Breakdowns

```javascript
for (let i = 0; i < clusterKeys.length; i++) {
  const key = clusterKeys[i];
  
  // Count frequency of each pain point in cluster
  const painPointCounts = {};
  for (let j = 0; j < clusters[key].painPoints.length; j++) {
    const pp = clusters[key].painPoints[j];
    painPointCounts[pp] = (painPointCounts[pp] || 0) + 1;
  }
  
  // Convert to sorted array
  const painPointArray = Object.keys(painPointCounts).map(pp => ({
    pain_point: pp,
    count: painPointCounts[pp]
  }));
  painPointArray.sort((a, b) => b.count - a.count); // Sort by frequency
  
  clusters[key].enrichedPainPointsJSON = JSON.stringify(painPointArray);
  
  // Repeat for emotions
  const emotionCounts = {};
  for (let j = 0; j < clusters[key].emotions.length; j++) {
    const emotion = clusters[key].emotions[j];
    emotionCounts[emotion] = (emotionCounts[emotion] || 0) + 1;
  }
  
  const emotionArray = Object.keys(emotionCounts).map(emotion => ({
    emotion: emotion,
    count: emotionCounts[emotion]
  }));
  emotionArray.sort((a, b) => b.count - a.count);
  
  clusters[key].enrichedEmotionsJSON = JSON.stringify(emotionArray);
}
```

**Frequency counting. Sorting. JSON serialization. No LLM.**

---

### Step 4: Trend Direction (Compare to Prior Period)

```javascript
// Query SIA table for prior period (same client, same domain+tier, previous date range)
const priorRecords = await nocodb.query({
  table: 'SIA',
  filter: `client_id = '${clientId}' AND date_range_end = '${priorEndDate}'`
});

for (let i = 0; i < clusterKeys.length; i++) {
  const key = clusterKeys[i];
  const currentCount = clusters[key].count;
  
  // Find matching prior cluster
  const priorCluster = priorRecords.find(r => 
    r.domain === clusters[key].domain && r.signalTier === clusters[key].tier
  );
  
  if (!priorCluster) {
    clusters[key].trendDirection = 'NEW'; // First appearance
  } else {
    const priorCount = priorCluster.count;
    const change = ((currentCount - priorCount) / priorCount) * 100;
    
    if (change > 10) {
      clusters[key].trendDirection = 'UP';
    } else if (change < -10) {
      clusters[key].trendDirection = 'DOWN';
    } else {
      clusters[key].trendDirection = 'STABLE';
    }
  }
}
```

**Deterministic comparison. No interpretation. Just math.**

---

## Token Budget

**Zero tokens per run.**

**SIA makes:**
- Zero OpenAI API calls
- Zero Anthropic API calls
- Zero dictionary queries

**Only operations:**
- NocoDB query (read EIP records)
- JavaScript loops and calculations
- NocoDB write (SIA output)

**Cost:** Infrastructure only (NocoDB queries, n8n execution time)

---

## Example Output

**SIA record for "Food Quality | T-NEGATIVE" cluster:**

```json
{
  "domain": "Food Quality",
  "signal_tier": "T-NEGATIVE",
  "count": 12,
  "percentage": 18,
  "enriched_pain_points_json": "[{\"pain_point\":\"Underseasoned protein\",\"count\":5},{\"pain_point\":\"Overcooked vegetables\",\"count\":4},{\"pain_point\":\"Cold entree\",\"count\":3}]",
  "enriched_emotions_json": "[{\"emotion\":\"Disappointment\",\"count\":7},{\"emotion\":\"Frustration\",\"count\":5}]",
  "trend_direction": "UP",
  "date_range_start": "2026-04-10",
  "date_range_end": "2026-04-17"
}
```

**Dashboard uses this to display:**
- 18% of reviews in T-NEGATIVE category
- Food Quality is trending UP (more negative signals than prior week)
- Top pain points: Underseasoned protein (5 mentions), Overcooked vegetables (4 mentions)
- Top emotions: Disappointment (7 mentions), Frustration (5 mentions)

---

## Key Design Decisions

### Why Pure JavaScript (No AI)?

**SIA's task is aggregation, not interpretation:**
- Group records by domain + tier (deterministic)
- Count frequencies (deterministic)
- Calculate percentages (deterministic)
- Compare to prior period (deterministic)

**No AI needed because:**
- EIP already classified everything (emotions, pain points, domains, signal types)
- SIA just counts and groups what EIP resolved
- No new insights generated, just statistical rollup

**Benefits:**
- **Zero incremental token cost per review** (scales infinitely at zero marginal AI cost)
- **Deterministic output** (same input = same output, no LLM variability)
- **Fast execution** (<1 second for 300 records vs ~30 seconds if using Claude)
- **No API rate limits** (pure computation)

---

### Why Read from EIP Table (Not Webhook)?

**SIA runs on schedule (daily 6am), not per-review:**
- Processes batch of reviews at once
- Aggregates across time window (7 days, 30 days)
- Compares to prior period

**Webhook approach would not work:**
- SIA needs entire dataset to calculate percentages
- Trend direction requires comparing current vs prior period
- Aggregation must happen after all reviews are processed

**Reading from EIP NocoDB table:**
- Query all records in date range
- Aggregate in single pass
- Write summary to SIA table
- Dashboard reads SIA table for display

---

### Why SIA Doesn't Use HSI Data?

**SIA aggregates at raw signal level (EIP output):**
- Counts domains + signal types before behavioral interpretation
- Provides baseline metrics (18% negative, 35% ambiguous, 47% positive)

**HSI adds behavioral interpretation:**
- Synthesizes emotion + pain point into narratives
- Generates operator insights
- Creates Hospitality Signal IDs

**SIA → HSI would add complexity without benefit:**
- SIA percentages are simpler for dashboard display
- Behavioral narratives are record-specific (not aggregated)
- Operator insights are qualitative (not quantitative)

**Architecture:** SIA and HSI are parallel outputs from ESS. SIA for metrics, HSI for narratives.

---

## Downstream Handoff

**SIA → Dashboard (read-only display):**
- Signal Pulse chart (T-NEG/T-AMB/T-POS percentages)
- Signal Distribution pie chart (domain breakdown)
- Trend Signals (week-over-week changes)

**SIA → MRA (future, when built):**
- Weekly Monday 7am intelligence brief includes SIA aggregations
- Monthly 1st-day report includes trend analysis from SIA

**SIA does NOT pass to BRA or RDA** (those agents operate on individual records, not aggregations)

---

## Related Documents

- **Changelog:** [SCX_SIA_CHANGELOG.md](SCX_SIA_CHANGELOG.md)
- **Schema:** [SIA_Schema.md](SIA_Schema.md)
- **Design Rationale:** [SIA_Design_Rationale.md](SIA_Design_Rationale.md)
- **Upstream Agent:** [../EIP/SCX_EIP_HOW_v4.md](../EIP/SCX_EIP_HOW_v4.md)
- **Dashboard Spec:** [../../commercial/Dashboard_Freelancer_Brief.md](../../commercial/Dashboard_Freelancer_Brief.md)

---

## n8n Workflow Details

**Workflow Name:** SCX-SIA  
**Trigger:** Schedule (daily 6am UTC) OR Manual trigger  
**Credentials:** NocoDB xc-token only (no AI API keys)

**Critical Rules:**
- All aggregation logic in Code Nodes (pure JavaScript)
- No AI API calls anywhere in workflow
- No spread operator (key-by-key object assignment only)
- Use index-based `for` loops (no `for...of` in Code Nodes)
- `pageInfo.totalRows` for record counts (not `list.length`)

---

**End of SIA HOW v3**
