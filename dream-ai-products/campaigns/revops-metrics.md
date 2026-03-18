# Dream AI — Pipeline Metrics & Analytics

> **Database:** PostgreSQL (dream_ai_crm)
> **Reporting cadence:** Weekly pipeline review, monthly deep-dive
> **Dashboard tools:** N8N → PostgreSQL views → API → dashboard (Metabase/Grafic/Looker)

---

## 1. Stage Conversion Rates (Targets)

### Conversion Funnel

```
Unknown → Aware:           30%    (form fills / total traffic)
Aware → Engaged:           25%    (2+ touchpoints)
Engaged → MQL:             20%    (meets scoring threshold)
MQL → SQL:                 40%    (sales-accepted)
SQL → Demo:                60%    (demo completed)
Demo → Proposal:           50%    (proposal delivered)
Proposal → Won:            35%    (closed-won)
```

### Target Conversion Rates by Niche

| Stage Transition | Real Estate | MedSpa | HVAC | Overall Target |
|-----------------|-------------|--------|------|----------------|
| Unknown → Aware | 30% | 28% | 32% | 30% |
| Aware → Engaged | 28% | 22% | 25% | 25% |
| Engaged → MQL | 22% | 18% | 20% | 20% |
| MQL → SQL | 45% | 35% | 40% | 40% |
| SQL → Demo | 65% | 55% | 60% | 60% |
| Demo → Proposal | 55% | 45% | 50% | 50% |
| Proposal → Won | 40% | 30% | 35% | 35% |
| **Overall (Aware → Won)** | **~2.6%** | **~1.1%** | **~1.8%** | **~1.8%** |

### Conversion Rate SQL Views

```sql
-- Funnel conversion rates (last 30 days)
WITH stage_counts AS (
  SELECT
    lifecycle_stage,
    COUNT(*) as lead_count,
    COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '30 days') as recent_count
  FROM contacts
  GROUP BY lifecycle_stage
),
conversion AS (
  SELECT
    'aware → engaged' as transition,
    (SELECT COUNT(*) FROM contacts WHERE lifecycle_stage IN ('engaged','mql','sql','demo','proposal','won'))::DECIMAL
    / NULLIF((SELECT COUNT(*) FROM contacts WHERE lifecycle_stage IN ('engaged','mql','sql','demo','proposal','won','aware')), 0) * 100
    as conversion_pct
  UNION ALL
  SELECT
    'engaged → mql',
    (SELECT COUNT(*) FROM contacts WHERE lifecycle_stage IN ('mql','sql','demo','proposal','won'))::DECIMAL
    / NULLIF((SELECT COUNT(*) FROM contacts WHERE lifecycle_stage IN ('engaged','mql','sql','demo','proposal','won')), 0) * 100
  UNION ALL
  SELECT
    'mql → sql',
    (SELECT COUNT(*) FROM contacts WHERE lifecycle_stage IN ('sql','demo','proposal','won'))::DECIMAL
    / NULLIF((SELECT COUNT(*) FROM contacts WHERE lifecycle_stage IN ('mql','sql','demo','proposal','won')), 0) * 100
  UNION ALL
  SELECT
    'sql → won',
    (SELECT COUNT(*) FROM contacts WHERE lifecycle_stage = 'won')::DECIMAL
    / NULLIF((SELECT COUNT(*) FROM contacts WHERE lifecycle_stage IN ('sql','demo','proposal','won')), 0) * 100
)
SELECT * FROM conversion ORDER BY conversion_pct DESC;
```

---

## 2. Velocity Metrics (Days per Stage)

### Target Velocity by Stage

| Stage | Target Days | Alert Threshold (2x avg) | Notes |
|-------|-------------|-------------------------|-------|
| Unknown → Aware | < 1 second (auto) | N/A | Instant on form fill |
| Aware → Engaged | 7 days | 14 days | Nurture engagement |
| Engaged → MQL | 5 days | 10 days | Score accumulation |
| MQL → SQL | 2 days | 4 days | Sales response SLA |
| SQL → Demo | 5 days | 10 days | Discovery + scheduling |
| Demo → Proposal | 5 days | 10 days | Proposal creation |
| Proposal → Won | 7 days | 14 days | Negotiation + signing |
| **Total cycle (MQL → Won)** | **19 days** | — | Velocity benchmark |

### Pipeline Velocity Formula

```
Pipeline Velocity = (# Active Deals × Avg ACV × Win Rate) / Avg Sales Cycle (days)

Example:
  50 active deals × $18,000 ACV × 35% win rate / 19 days
  = $315,000 / 19 = $16,579/day revenue potential
```

### Velocity SQL Queries

```sql
-- Average days per stage (last 90 days)
WITH stage_transitions AS (
  SELECT
    contact_id,
    previous_stage,
    new_stage,
    created_at,
    LAG(created_at) OVER (PARTITION BY contact_id ORDER BY created_at) as prev_stage_at,
    created_at - LAG(created_at) OVER (PARTITION BY contact_id ORDER BY created_at) as days_in_stage
  FROM activity_log
  WHERE activity_type = 'stage_change'
    AND created_at >= NOW() - INTERVAL '90 days'
)
SELECT
  previous_stage,
  new_stage,
  ROUND(AVG(EXTRACT(EPOCH FROM days_in_stage) / 86400), 1) as avg_days,
  ROUND(MIN(EXTRACT(EPOCH FROM days_in_stage) / 86400), 1) as min_days,
  ROUND(MAX(EXTRACT(EPOCH FROM days_in_stage) / 86400), 1) as max_days,
  COUNT(*) as transition_count
FROM stage_transitions
WHERE days_in_stage IS NOT NULL
GROUP BY previous_stage, new_stage
ORDER BY avg_days DESC;

-- Stale deals alert (deals stuck > 2x average)
SELECT
  d.id,
  c.first_name,
  c.last_name,
  a.name as company,
  d.stage,
  d.estimated_acv,
  d.updated_at,
  NOW() - d.updated_at as days_stuck,
  (SELECT ROUND(AVG(EXTRACT(EPOCH FROM days_in_stage) / 86400))
   FROM activity_log al
   WHERE al.deal_id = d.id AND al.activity_type = 'stage_change'
  ) as avg_days_per_stage
FROM deals d
JOIN contacts c ON d.contact_id = c.id
JOIN accounts a ON d.account_id = a.id
WHERE d.stage IN ('sql', 'demo', 'proposal')
  AND d.updated_at < NOW() - INTERVAL '14 days'
  AND d.stage NOT IN ('won', 'lost')
ORDER BY days_stuck DESC;

-- Pipeline velocity (weekly)
SELECT
  DATE_TRUNC('week', d.created_at) as week,
  COUNT(DISTINCT d.id) as active_deals,
  AVG(d.estimated_acv) as avg_acv,
  COUNT(*) FILTER (WHERE d.stage = 'won')::DECIMAL / NULLIF(COUNT(*), 0) * 100 as win_rate,
  AVG(EXTRACT(EPOCH FROM (COALESCE(d.closed_at, NOW()) - d.created_at)) / 86400) as avg_cycle_days,
  -- Pipeline velocity
  ROUND(
    (COUNT(DISTINCT d.id)::DECIMAL * AVG(d.estimated_acv) *
     (COUNT(*) FILTER (WHERE d.stage = 'won')::DECIMAL / NULLIF(COUNT(*), 0)))
    / NULLIF(AVG(EXTRACT(EPOCH FROM (COALESCE(d.closed_at, NOW()) - d.created_at)) / 86400), 0),
  2) as daily_revenue_velocity
FROM deals d
WHERE d.created_at >= NOW() - INTERVAL '90 days'
GROUP BY DATE_TRUNC('week', d.created_at)
ORDER BY week DESC;
```

---

## 3. Lead Source Attribution

### Attribution Model

Dream AI uses a **primary-source attribution** model with assisted-touch tracking:
- **First-touch** → credits the channel that first captured the lead (for top-funnel ROI)
- **Last-touch** → credits the channel immediately before MQL (for conversion ROI)
- **Assisted** → tracks all touchpoints that contributed (for multi-touch analysis)

### Source Performance Table

```sql
-- Lead source performance (last 90 days)
SELECT
  c.source,
  c.utm_source,
  c.utm_medium,
  COUNT(DISTINCT c.id) as total_leads,
  COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'engaged') as engaged,
  COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'mql') as mqls,
  COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'sql') as sqls,
  COUNT(*) FILTER (WHERE c.lifecycle_stage = 'won') as wins,
  -- Conversion rates
  ROUND(COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'mql')::DECIMAL / NULLIF(COUNT(*), 0) * 100, 1) as mql_rate,
  ROUND(COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'sql')::DECIMAL / NULLIF(COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'mql'), 0) * 100, 1) as mql_to_sql_rate,
  ROUND(COUNT(*) FILTER (WHERE c.lifecycle_stage = 'won')::DECIMAL / NULLIF(COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'sql'), 0) * 100, 1) as sql_to_won_rate,
  -- Revenue
  COALESCE(SUM(d.actual_acv) FILTER (WHERE d.stage = 'won'), 0) as total_revenue,
  ROUND(AVG(d.actual_acv) FILTER (WHERE d.stage = 'won'), 2) as avg_acv,
  -- Velocity
  ROUND(AVG(EXTRACT(EPOCH FROM (COALESCE(d.closed_at, NOW()) - c.created_at)) / 86400) FILTER (WHERE d.stage = 'won'), 1) as avg_days_to_close
FROM contacts c
LEFT JOIN deals d ON c.id = d.contact_id
WHERE c.created_at >= NOW() - INTERVAL '90 days'
GROUP BY c.source, c.utm_source, c.utm_medium
ORDER BY total_revenue DESC;
```

### Source Attribution Targets

| Source | Target Cost/MQL | Target CAC | Target LTV:CAC |
|--------|----------------|------------|----------------|
| Organic Search | $50 | $200 | 10:1 |
| Paid Google | $150 | $600 | 5:1 |
| Paid Meta | $200 | $800 | 4:1 |
| Paid LinkedIn | $300 | $1,200 | 3:1 |
| Referral | $25 | $100 | 15:1 |
| Outbound Email | $75 | $300 | 8:1 |
| Outbound LinkedIn | $100 | $400 | 6:1 |
| Webinar | $125 | $500 | 5:1 |
| Content Download | $60 | $250 | 8:1 |
| Free Tool | $40 | $160 | 12:1 |

### UTM Tracking Schema

```sql
CREATE TABLE attribution_touchpoints (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID REFERENCES contacts(id),
  touch_number INT NOT NULL,  -- 1 = first touch, N = latest
  touch_type TEXT NOT NULL,   -- 'first', 'assisted', 'last'
  source deal_source,
  utm_source TEXT,
  utm_medium TEXT,
  utm_campaign TEXT,
  utm_content TEXT,
  landing_page_url TEXT,
  touch_at TIMESTAMPTZ DEFAULT now(),
  -- Metadata
  session_duration_seconds INT,
  page_views_in_session INT,
  is_converting_touch BOOLEAN DEFAULT false  -- true if this touch led to MQL
);
```

---

## 4. Revenue Pipeline Forecasting Model

### Forecast SQL View

```sql
-- Monthly pipeline forecast
WITH pipeline_by_stage AS (
  SELECT
    d.stage,
    d.niche,
    COUNT(*) as deal_count,
    SUM(d.estimated_acv) as pipeline_value,
    AVG(d.estimated_acv) as avg_deal_size
  FROM deals d
  WHERE d.stage NOT IN ('won', 'lost')
  GROUP BY d.stage, d.niche
),
win_probabilities AS (
  SELECT * FROM (VALUES
    ('sql', 0.15),
    ('demo', 0.35),
    ('proposal', 0.60),
    ('won', 1.0),
    ('lost', 0.0)
  ) AS t(stage, probability)
)
SELECT
  pb.stage,
  pb.niche,
  pb.deal_count,
  pb.pipeline_value,
  pb.avg_deal_size,
  wp.probability,
  ROUND(pb.pipeline_value * wp.probability, 2) as weighted_forecast,
  -- Time-based: expected close within 30 days
  COUNT(*) FILTER (
    WHERE d.updated_at >= NOW() - INTERVAL '7 days'
  ) as deals_updated_this_week,
  ROUND(SUM(d.estimated_acv) FILTER (
    WHERE d.updated_at >= NOW() - INTERVAL '7 days'
  ), 2) as recently_active_value
FROM pipeline_by_stage pb
JOIN win_probabilities wp ON pb.stage = wp.stage
JOIN deals d ON d.stage = pb.stage AND d.niche = pb.niche
WHERE pb.stage NOT IN ('won', 'lost')
GROUP BY pb.stage, pb.niche, pb.deal_count, pb.pipeline_value, pb.avg_deal_size, wp.probability
ORDER BY weighted_forecast DESC;

-- Total forecast summary
SELECT
  SUM(pipeline_value) as total_pipeline,
  SUM(weighted_forecast) as weighted_forecast,
  SUM(deal_count) as total_deals,
  AVG(avg_deal_size) as avg_deal_size,
  -- By niche breakdown
  SUM(pipeline_value) FILTER (WHERE niche = 'real_estate') as real_estate_pipeline,
  SUM(pipeline_value) FILTER (WHERE niche = 'medspa') as medspa_pipeline,
  SUM(pipeline_value) FILTER (WHERE niche = 'hvac') as hvac_pipeline
FROM (
  -- (subquery from above)
) f;
```

### Pipeline Coverage Ratio

```
Pipeline Coverage = Total Pipeline Value / Monthly Revenue Target

Target: 3-4x coverage

Example:
  Monthly target: $50,000 ARR (new)
  Total pipeline: $180,000
  Coverage: 3.6x ✓

If coverage < 3x: Marketing needs to generate more leads
If coverage > 5x: Qualify harder, pipeline is bloated
```

### Forecast Accuracy Tracking

```sql
-- Forecast accuracy (monthly)
SELECT
  DATE_TRUNC('month', forecast_month) as month,
  predicted_revenue,
  actual_revenue,
  ROUND((actual_revenue / NULLIF(predicted_revenue, 0) - 1) * 100, 1) as forecast_variance_pct,
  CASE
    WHEN ABS(actual_revenue / NULLIF(predicted_revenue, 0) - 1) <= 0.1 THEN 'on_target'
    WHEN actual_revenue / NULLIF(predicted_revenue, 0) - 1 > 0.1 THEN 'over_performed'
    ELSE 'under_performed'
  END as accuracy_rating
FROM pipeline_forecasts
ORDER BY month DESC
LIMIT 12;
```

---

## 5. Weekly Pipeline Review Template

### Automated Weekly Report (N8N → Slack/Email)

**Generated every Monday at 9:00 AM UTC**

```sql
-- Weekly pipeline summary query
WITH this_week AS (
  SELECT * FROM contacts WHERE created_at >= DATE_TRUNC('week', NOW())
),
last_week AS (
  SELECT * FROM contacts WHERE created_at >= DATE_TRUNC('week', NOW()) - INTERVAL '7 days'
                            AND created_at < DATE_TRUNC('week', NOW())
),
deals_this_week AS (
  SELECT * FROM deals WHERE updated_at >= DATE_TRUNC('week', NOW())
)
SELECT
  -- NEW LEADS
  (SELECT COUNT(*) FROM this_week) as new_leads_this_week,
  (SELECT COUNT(*) FROM last_week) as new_leads_last_week,
  ROUND(((SELECT COUNT(*) FROM this_week)::DECIMAL / NULLIF((SELECT COUNT(*) FROM last_week), 0) - 1) * 100, 1) as lead_growth_pct,

  -- MQLS
  (SELECT COUNT(*) FROM this_week WHERE lifecycle_stage >= 'mql') as mqls_this_week,

  -- SQLS CREATED
  (SELECT COUNT(*) FROM deals_this_week WHERE stage = 'sql') as new_sqls,

  -- DEMOS BOOKED
  (SELECT COUNT(*) FROM deals_this_week WHERE stage = 'demo') as demos_booked,

  -- WINS
  (SELECT COUNT(*) FROM deals_this_week WHERE stage = 'won') as deals_won,
  (SELECT COALESCE(SUM(actual_acv), 0) FROM deals_this_week WHERE stage = 'won') as revenue_closed,

  -- PIPELINE HEALTH
  (SELECT COUNT(*) FROM deals WHERE stage NOT IN ('won', 'lost')) as active_deals,
  (SELECT COALESCE(SUM(estimated_acv), 0) FROM deals WHERE stage NOT IN ('won', 'lost')) as total_pipeline,
  (SELECT COALESCE(SUM(estimated_acv), 0) FROM deals WHERE stage NOT IN ('won', 'lost')) / 50000 as pipeline_coverage,  -- assuming $50K/mo target

  -- SLA COMPLIANCE
  (SELECT COUNT(*) FROM contacts WHERE lifecycle_stage = 'mql' AND assigned_at IS NOT NULL
   AND EXTRACT(EPOCH FROM (assigned_at - mql_at)) / 3600 <= 4)::DECIMAL
   / NULLIF((SELECT COUNT(*) FROM contacts WHERE lifecycle_stage = 'mql' AND assigned_at IS NOT NULL), 0) * 100
   as sla_compliance_pct,

  -- STALE DEALS
  (SELECT COUNT(*) FROM deals WHERE stage IN ('sql', 'demo', 'proposal') AND updated_at < NOW() - INTERVAL '14 days') as stale_deals;

```

### Weekly Review Agenda

| # | Topic | Data Source | Owner | Time |
|---|-------|------------|-------|------|
| 1 | **Lead volume & quality** | This week vs. last: new leads, MQLs, conversion rates | Marketing | 5 min |
| 2 | **Pipeline health** | Total pipeline value, coverage ratio, deal count | RevOps | 5 min |
| 3 | **Stage conversions** | Week-over-week conversion rate changes by stage | RevOps | 5 min |
| 4 | **Slippage review** | Deals pushed past close date — why? Action items | Sales | 10 min |
| 5 | **Stale deal review** | Deals stuck > 14 days — re-engage or recycle | Sales | 10 min |
| 6 | **SLA compliance** | Response time adherence, escalation count | Sales Mgr | 5 min |
| 7 | **Win/loss review** | 2-3 recent deals — what won, what lost, why | Sales | 10 min |
| 8 | **Forecast** | Weighted pipeline forecast vs. target | RevOps | 5 min |
| 9 | **Action items** | Decisions, owners, deadlines | All | 5 min |

**Total: 60 minutes**

### Slack Report Format

```
📊 WEEKLY PIPELINE REPORT — Week of {{ week_start }}

── LEADS ─────────────────────────
New leads:       {{ new_leads }} ({{ lead_growth }} vs last week)
MQLs created:    {{ mqls }}
SQLs created:    {{ sqls }}
Demos booked:    {{ demos }}

── REVENUE ───────────────────────
Deals won:       {{ deals_won }}
Revenue closed:  ${{ revenue_closed }}
Pipeline value:  ${{ pipeline_value }}
Coverage ratio:  {{ coverage }}x target

── HEALTH ────────────────────────
SLA compliance:  {{ sla_pct }}%
Stale deals:     {{ stale_count }}
Hot leads:       {{ hot_lead_count }} (uncontacted: {{ hot_uncontacted }})

── TOP RISKS ─────────────────────
{{ #each stale_deals }}
⚠️ {{ company }} — stuck in {{ stage }} for {{ days }} days
{{ /each }}
```

---

## 6. PostgreSQL Materialized Views for Dashboarding

```sql
-- Daily snapshot for trend analysis
CREATE MATERIALIZED VIEW daily_pipeline_snapshot AS
SELECT
  DATE_TRUNC('day', NOW()) as snapshot_date,
  COUNT(*) as total_contacts,
  COUNT(*) FILTER (WHERE lifecycle_stage = 'aware') as aware_count,
  COUNT(*) FILTER (WHERE lifecycle_stage = 'engaged') as engaged_count,
  COUNT(*) FILTER (WHERE lifecycle_stage = 'mql') as mql_count,
  COUNT(*) FILTER (WHERE lifecycle_stage = 'sql') as sql_count,
  COUNT(*) FILTER (WHERE lifecycle_stage = 'demo') as demo_count,
  COUNT(*) FILTER (WHERE lifecycle_stage = 'proposal') as proposal_count,
  COUNT(*) FILTER (WHERE lifecycle_stage = 'won') as won_count,
  COUNT(*) FILTER (WHERE lifecycle_stage = 'lost') as lost_count,
  COALESCE(SUM(estimated_acv) FILTER (WHERE stage NOT IN ('won', 'lost')), 0) as total_pipeline,
  COALESCE(SUM(actual_acv) FILTER (WHERE stage = 'won'), 0) as total_won_revenue
FROM contacts c
LEFT JOIN deals d ON c.id = d.contact_id;

-- Refresh daily via N8N cron
-- REFRESH MATERIALIZED VIEW daily_pipeline_snapshot;

-- Niche breakdown view
CREATE MATERIALIZED VIEW niche_pipeline_summary AS
SELECT
  a.niche,
  COUNT(DISTINCT c.id) as total_contacts,
  COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'mql') as mqls,
  COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'sql') as sqls,
  COUNT(*) FILTER (WHERE c.lifecycle_stage = 'won') as wins,
  COALESCE(SUM(d.estimated_acv) FILTER (WHERE d.stage NOT IN ('won', 'lost')), 0) as open_pipeline,
  COALESCE(SUM(d.actual_acv) FILTER (WHERE d.stage = 'won'), 0) as won_revenue,
  ROUND(AVG(d.actual_acv) FILTER (WHERE d.stage = 'won'), 2) as avg_won_acv,
  ROUND(AVG(EXTRACT(EPOCH FROM (d.closed_at - c.created_at)) / 86400) FILTER (WHERE d.stage = 'won'), 1) as avg_days_to_close
FROM contacts c
LEFT JOIN accounts a ON c.account_id = a.id
LEFT JOIN deals d ON c.id = d.contact_id
GROUP BY a.niche;

-- Source performance view
CREATE MATERIALIZED VIEW source_performance_summary AS
SELECT
  c.source,
  COUNT(*) as total_leads,
  COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'mql') as mqls,
  COUNT(*) FILTER (WHERE c.lifecycle_stage = 'won') as wins,
  ROUND(COUNT(*) FILTER (WHERE c.lifecycle_stage >= 'mql')::DECIMAL / NULLIF(COUNT(*), 0) * 100, 1) as mql_rate,
  COALESCE(SUM(d.actual_acv) FILTER (WHERE d.stage = 'won'), 0) as won_revenue
FROM contacts c
LEFT JOIN deals d ON c.id = d.contact_id
GROUP BY c.source;
```

### N8N Cron Job for Materialized View Refresh

```
Schedule: Daily at 00:00 UTC
Node: PostgreSQL Execute Query

Queries:
  REFRESH MATERIALIZED VIEW daily_pipeline_snapshot;
  REFRESH MATERIALIZED VIEW niche_pipeline_summary;
  REFRESH MATERIALIZED VIEW source_performance_summary;
```

---

## 7. KPI Targets Summary

| Metric | Target | Red Flag |
|--------|--------|----------|
| Leads/month | 200+ | < 100 |
| MQL rate | 5-15% of leads | < 3% |
| MQL → SQL | 40% | < 25% |
| SQL → Won | 20% | < 12% |
| Avg sales cycle (MQL → Won) | 19 days | > 35 days |
| Pipeline coverage | 3-4x monthly target | < 2.5x |
| Speed-to-lead (MQL contact) | < 4 hours | > 8 hours |
| Win rate (SQL → Won) | 20-30% | < 15% |
| Avg ACV | $18,000 | < $12,000 |
| LTV:CAC | > 3:1 | < 2:1 |
| Monthly revenue (new biz) | $50,000 | < $30,000 |
| SLA compliance | > 90% | < 75% |
| Forecast accuracy | ±15% | > ±25% |
