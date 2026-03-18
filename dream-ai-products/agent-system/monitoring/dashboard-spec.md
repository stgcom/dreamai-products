# Dream AI — Observability Dashboard Specification

> Monitoring dashboard for agent infrastructure, business metrics, and system health.

---

## 1. Dashboard Overview

**Platform:** N8N Dashboard Nodes + Embedded Grafana (optional) or custom web widget
**Data Source:** PostgreSQL (agent_logs, agent_sessions, daily_metrics tables)
**Refresh Rate:** 30 seconds (real-time widgets), 5 minutes (aggregate widgets)
**Access:** Internal only — Dream AI team

---

## 2. Agent Health Dashboard

### 2.1 Widget: Uptime %

**Type:** Gauge
**Query:**
```sql
SELECT
  agent_id,
  ROUND(
    100.0 * COUNT(*) FILTER (WHERE status != 'failure') / NULLIF(COUNT(*), 0),
    2
  ) as uptime_pct
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '24 hours'
GROUP BY agent_id;
```

**Widget Config:**
```json
{
  "id": "agent-uptime",
  "type": "gauge",
  "title": "Agent Uptime (24h)",
  "data_source": "postgres",
  "query": "uptime_query_above",
  "thresholds": {
    "green": { "min": 99.5, "max": 100 },
    "yellow": { "min": 98, "max": 99.5 },
    "red": { "min": 0, "max": 98 }
  },
  "format": "percent",
  "refresh_seconds": 300
}
```

---

### 2.2 Widget: Average Response Time

**Type:** Time-series line chart
**Query:**
```sql
SELECT
  date_trunc('minute', timestamp) as bucket,
  agent_id,
  AVG(duration_ms) as avg_response_ms,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY duration_ms) as p95_ms
FROM agent_logs
WHERE category = 'outbound'
  AND timestamp >= NOW() - INTERVAL '1 hour'
GROUP BY bucket, agent_id
ORDER BY bucket;
```

**Widget Config:**
```json
{
  "id": "response-time",
  "type": "line-chart",
  "title": "Response Time (1h)",
  "data_source": "postgres",
  "query": "response_time_query_above",
  "series": ["agent_id"],
  "y_axis": {
    "label": "ms",
    "min": 0
  },
  "thresholds": [
    { "value": 5000, "color": "yellow", "label": "Warning (5s)" },
    { "value": 10000, "color": "red", "label": "Critical (10s)" }
  ],
  "refresh_seconds": 30
}
```

---

### 2.3 Widget: Error Rate

**Type:** Single stat + sparkline
**Query:**
```sql
SELECT
  agent_id,
  ROUND(
    100.0 * COUNT(*) FILTER (WHERE level IN ('ERROR', 'CRITICAL'))
    / NULLIF(COUNT(*), 0),
    2
  ) as error_rate_pct,
  COUNT(*) FILTER (WHERE level IN ('ERROR', 'CRITICAL')) as error_count,
  COUNT(*) as total_count
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '1 hour'
GROUP BY agent_id;
```

**Widget Config:**
```json
{
  "id": "error-rate",
  "type": "stat-panel",
  "title": "Error Rate (1h)",
  "data_source": "postgres",
  "query": "error_rate_query_above",
  "display_field": "error_rate_pct",
  "format": "percent",
  "thresholds": {
    "green": { "max": 2 },
    "yellow": { "min": 2, "max": 5 },
    "red": { "min": 5 }
  },
  "sparkline": true,
  "refresh_seconds": 60
}
```

---

### 2.4 Widget: Tokens Used (Hourly/Daily)

**Type:** Bar chart (grouped by agent)
**Query:**
```sql
-- Hourly
SELECT
  date_trunc('hour', timestamp) as hour,
  agent_id,
  SUM(tokens_input) as input_tokens,
  SUM(tokens_output) as output_tokens,
  SUM(cost_usd) as cost
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '24 hours'
  AND tokens_input IS NOT NULL
GROUP BY hour, agent_id
ORDER BY hour;
```

**Widget Config:**
```json
{
  "id": "token-usage",
  "type": "bar-chart",
  "title": "Token Usage (24h)",
  "data_source": "postgres",
  "query": "token_usage_query_above",
  "series": [
    { "field": "input_tokens", "label": "Input Tokens", "color": "#4CAF50" },
    { "field": "output_tokens", "label": "Output Tokens", "color": "#2196F3" }
  ],
  "x_axis": "hour",
  "y_axis": { "label": "Tokens", "stacked": false },
  "refresh_seconds": 300
}
```

---

## 3. Business Metrics Dashboard

### 3.1 Widget: Leads Captured

**Type:** Counter + trend
**Query:**
```sql
SELECT
  DATE(timestamp) as date,
  COUNT(DISTINCT session_id) FILTER (
    WHERE action = 'lead_captured'
  ) as leads_captured
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '30 days'
  AND action IN ('lead_captured', 'lead_form_submitted', 'contact_info_collected')
GROUP BY date
ORDER BY date;
```

**Widget Config:**
```json
{
  "id": "leads-captured",
  "type": "counter-panel",
  "title": "Leads Captured (30d)",
  "data_source": "postgres",
  "query": "leads_query_above",
  "current_value_field": "leads_captured_today",
  "trend": "daily_comparison",
  "refresh_seconds": 300
}
```

---

### 3.2 Widget: Demos Booked

**Type:** Counter + calendar heatmap
**Query:**
```sql
SELECT
  DATE(timestamp) as date,
  COUNT(*) FILTER (
    WHERE action = 'demo_booked'
  ) as demos_booked,
  COUNT(*) FILTER (
    WHERE action = 'demo_confirmed'
  ) as demos_confirmed
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '30 days'
GROUP BY date
ORDER BY date;
```

**Widget Config:**
```json
{
  "id": "demos-booked",
  "type": "counter-panel",
  "title": "Demos Booked (30d)",
  "data_source": "postgres",
  "query": "demos_query_above",
  "current_value_field": "demos_booked_today",
  "trend": "weekly_comparison",
  "calendar_heatmap": true,
  "refresh_seconds": 300
}
```

---

### 3.3 Widget: Products Sold

**Type:** Counter + pie chart (by product)
**Query:**
```sql
SELECT
  DATE(timestamp) as date,
  metadata->>'product_name' as product,
  COUNT(*) as units_sold,
  SUM((metadata->>'price_usd')::DECIMAL) as revenue
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '30 days'
  AND action = 'product_purchased'
GROUP BY date, product
ORDER BY date DESC;
```

**Widget Config:**
```json
{
  "id": "products-sold",
  "type": "combined-panel",
  "title": "Products Sold (30d)",
  "data_source": "postgres",
  "query": "products_query_above",
  "panels": [
    { "type": "counter", "field": "units_sold", "label": "Total Units" },
    { "type": "pie", "group_by": "product", "value": "revenue", "label": "Revenue by Product" }
  ],
  "refresh_seconds": 300
}
```

---

### 3.4 Widget: Revenue Generated

**Type:** Area chart + KPI card
**Query:**
```sql
SELECT
  DATE(timestamp) as date,
  SUM((metadata->>'amount_usd')::DECIMAL) as daily_revenue,
  COUNT(*) FILTER (WHERE action = 'payment_completed') as transactions
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '90 days'
  AND action IN ('payment_completed', 'product_purchased', 'subscription_started')
GROUP BY date
ORDER BY date;
```

**Widget Config:**
```json
{
  "id": "revenue",
  "type": "area-chart",
  "title": "Revenue (90d)",
  "data_source": "postgres",
  "query": "revenue_query_above",
  "series": [
    { "field": "daily_revenue", "label": "Revenue ($)", "color": "#4CAF50" }
  ],
  "x_axis": "date",
  "y_axis": { "label": "USD", "format": "currency" },
  "kpi_card": {
    "field": "monthly_total",
    "compare": "previous_month",
    "format": "currency"
  },
  "refresh_seconds": 300
}
```

---

## 4. System Metrics Dashboard

### 4.1 Widget: API Calls Volume

**Type:** Stacked bar chart
**Query:**
```sql
SELECT
  date_trunc('hour', timestamp) as hour,
  COUNT(*) FILTER (WHERE metadata->>'api' = 'openrouter') as openrouter_calls,
  COUNT(*) FILTER (WHERE metadata->>'api' = 'n8n') as n8n_calls,
  COUNT(*) FILTER (WHERE metadata->>'api' = 'postgres') as postgres_calls
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '24 hours'
  AND category = 'tool_calls'
GROUP BY hour
ORDER BY hour;
```

**Widget Config:**
```json
{
  "id": "api-calls",
  "type": "stacked-bar",
  "title": "API Calls (24h)",
  "data_source": "postgres",
  "query": "api_calls_query_above",
  "series": [
    { "field": "openrouter_calls", "label": "OpenRouter", "color": "#2196F3" },
    { "field": "n8n_calls", "label": "N8N", "color": "#FF9800" },
    { "field": "postgres_calls", "label": "PostgreSQL", "color": "#9C27B0" }
  ],
  "x_axis": "hour",
  "refresh_seconds": 300
}
```

---

### 4.2 Widget: MCP Server Status

**Type:** Status grid
**Query:**
```sql
SELECT
  metadata->>'mcp_server' as server_name,
  COUNT(*) FILTER (WHERE status = 'success') as success_count,
  COUNT(*) FILTER (WHERE status = 'failure') as failure_count,
  MAX(timestamp) as last_call,
  AVG(duration_ms) as avg_latency_ms
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '1 hour'
  AND category = 'tool_calls'
  AND metadata->>'mcp_server' IS NOT NULL
GROUP BY server_name;
```

**Widget Config:**
```json
{
  "id": "mcp-status",
  "type": "status-grid",
  "title": "MCP Server Status",
  "data_source": "postgres",
  "query": "mcp_status_query_above",
  "status_logic": {
    "green": "failure_count = 0 AND last_call < 5 minutes ago",
    "yellow": "failure_rate < 5% OR last_call > 5 minutes ago",
    "red": "failure_rate >= 5% OR last_call > 30 minutes ago"
  },
  "display_fields": ["server_name", "avg_latency_ms", "last_call"],
  "refresh_seconds": 60
}
```

**Expected Servers:**
| Server      | Type  | Purpose                    |
|-------------|-------|----------------------------|
| `n8n-mcp`   | HTTP  | N8N workflow management    |
| `postgres-mcp` | stdio | PostgreSQL database tools |

---

### 4.3 Widget: Cron Job Health

**Type:** Status list
**Query:**
```sql
SELECT
  agent_id,
  action,
  MAX(timestamp) as last_run,
  COUNT(*) FILTER (WHERE status = 'success') as success_count,
  COUNT(*) FILTER (WHERE status = 'failure') as failure_count
FROM agent_logs
WHERE action LIKE '%cron%'
  OR action LIKE '%scheduled%'
  OR action LIKE '%heartbeat%'
  OR action LIKE '%daily%'
GROUP BY agent_id, action;
```

**Widget Config:**
```json
{
  "id": "cron-health",
  "type": "status-list",
  "title": "Cron Job Health",
  "data_source": "postgres",
  "query": "cron_health_query_above",
  "status_logic": {
    "green": "last_run < expected_interval * 1.5 AND failure_count = 0",
    "yellow": "last_run < expected_interval * 3 OR failure_count > 0",
    "red": "last_run > expected_interval * 5"
  },
  "expected_intervals": {
    "daily-log-summary": 86400,
    "heartbeat-check": 300,
    "budget-monitor": 3600
  },
  "refresh_seconds": 300
}
```

---

### 4.4 Widget: Active Session Count

**Type:** Live counter + table
**Query:**
```sql
-- Active sessions (had activity in last 30 min)
SELECT
  COUNT(DISTINCT session_id) as active_sessions,
  COUNT(DISTINCT agent_id) as active_agents,
  array_agg(DISTINCT agent_id) as agent_list
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '30 minutes';

-- Recent sessions table
SELECT
  session_id,
  agent_id,
  user_id,
  channel,
  MAX(timestamp) as last_activity,
  COUNT(*) as turn_count
FROM agent_logs
WHERE timestamp >= NOW() - INTERVAL '24 hours'
GROUP BY session_id, agent_id, user_id, channel
ORDER BY last_activity DESC
LIMIT 20;
```

**Widget Config:**
```json
{
  "id": "session-count",
  "type": "combined-panel",
  "title": "Sessions",
  "data_source": "postgres",
  "panels": [
    {
      "type": "stat-panel",
      "title": "Active (30m)",
      "field": "active_sessions",
      "refresh_seconds": 30
    },
    {
      "type": "table",
      "title": "Recent Sessions (24h)",
      "columns": ["session_id", "agent_id", "user_id", "channel", "last_activity", "turn_count"],
      "refresh_seconds": 60
    }
  ]
}
```

---

## 5. Alert Rules

### 5.1 Alert Definitions

```json
{
  "alerts": [
    {
      "id": "alert-error-rate",
      "name": "High Error Rate",
      "description": "Agent error rate exceeds 5% in a 15-minute window",
      "severity": "warning",
      "condition": {
        "query": "SELECT 100.0 * COUNT(*) FILTER (WHERE level IN ('ERROR','CRITICAL')) / NULLIF(COUNT(*), 0) as rate FROM agent_logs WHERE timestamp >= NOW() - INTERVAL '15 minutes'",
        "operator": ">",
        "threshold": 5,
        "field": "rate"
      },
      "actions": [
        {
          "type": "telegram",
          "target": "686076918",
          "message": "⚠️ High Error Rate: {{rate}}% in last 15min"
        }
      ],
      "cooldown_minutes": 30
    },
    {
      "id": "alert-response-time",
      "name": "Slow Response Time",
      "description": "Average response time exceeds 5 seconds",
      "severity": "warning",
      "condition": {
        "query": "SELECT AVG(duration_ms) as avg_ms FROM agent_logs WHERE category = 'outbound' AND timestamp >= NOW() - INTERVAL '10 minutes'",
        "operator": ">",
        "threshold": 5000,
        "field": "avg_ms"
      },
      "actions": [
        {
          "type": "telegram",
          "target": "686076918",
          "message": "⚠️ Slow Response: avg {{avg_ms}}ms in last 10min"
        }
      ],
      "cooldown_minutes": 15
    },
    {
      "id": "alert-downtime",
      "name": "Agent Downtime",
      "description": "No logs received for any agent in > 1 minute",
      "severity": "critical",
      "condition": {
        "query": "SELECT EXTRACT(EPOCH FROM (NOW() - MAX(timestamp))) as seconds_since_last FROM agent_logs",
        "operator": ">",
        "threshold": 60,
        "field": "seconds_since_last"
      },
      "actions": [
        {
          "type": "telegram",
          "target": "686076918",
          "message": "🚨 CRITICAL: No agent activity for {{seconds_since_last}}s — system may be down"
        }
      ],
      "cooldown_minutes": 5
    },
    {
      "id": "alert-revenue-anomaly",
      "name": "Revenue Anomaly",
      "description": "Revenue deviation > 2 standard deviations from 30-day rolling average",
      "severity": "critical",
      "condition": {
        "query": "WITH stats AS (SELECT AVG(daily_rev) as mean, STDDEV(daily_rev) as stddev FROM (SELECT DATE(timestamp) as d, SUM((metadata->>'amount_usd')::DECIMAL) as daily_rev FROM agent_logs WHERE action IN ('payment_completed','product_purchased') AND timestamp >= NOW() - INTERVAL '30 days' GROUP BY d) sub), today AS (SELECT SUM((metadata->>'amount_usd')::DECIMAL) as today_rev FROM agent_logs WHERE action IN ('payment_completed','product_purchased') AND DATE(timestamp) = CURRENT_DATE) SELECT today.today_rev, stats.mean, stats.stddev, ABS(today.today_rev - stats.mean) / NULLIF(stats.stddev, 0) as z_score FROM stats, today",
        "operator": ">",
        "threshold": 2,
        "field": "z_score"
      },
      "actions": [
        {
          "type": "telegram",
          "target": "686076918",
          "message": "🚨 Revenue Anomaly: Today ${{today_rev}} vs avg ${{mean}} (σ={{stddev}}, z={{z_score}})"
        }
      ],
      "cooldown_minutes": 60
    },
    {
      "id": "alert-budget-threshold",
      "name": "Budget Threshold Breach",
      "description": "Monthly cost exceeds budget threshold",
      "severity": "warning",
      "thresholds": [
        { "value": 80, "severity": "warning", "message": "💰 Budget at 80%: ${{current_cost}} / ${{budget}}" },
        { "value": 90, "severity": "warning", "message": "💰 Budget at 90%: ${{current_cost}} / ${{budget}}" },
        { "value": 95, "severity": "critical", "message": "🚨 Budget at 95%: ${{current_cost}} / ${{budget}}" },
        { "value": 100, "severity": "critical", "message": "🚨 BUDGET EXCEEDED: ${{current_cost}} / ${{budget}}" }
      ],
      "cooldown_minutes": 60
    }
  ]
}
```

### 5.2 Alert Evaluation N8N Workflow

**Cron:** Every 1 minute

```
[Cron 1min] → [Run Alert Queries] → [Compare Thresholds] → [Fire Alerts (deduplicated)]
```

#### Node: Alert Evaluator

```json
{
  "name": "Evaluate Alerts",
  "type": "n8n-nodes-base.function",
  "parameters": {
    "functionCode": "const alerts = $node['Alert Config'].json.alerts;\nconst results = [];\n\nfor (const alert of alerts) {\n  // Query would be executed by a prior PostgreSQL node\n  const queryResult = items[0].json;\n  const value = queryResult[alert.condition.field];\n  \n  let triggered = false;\n  switch (alert.condition.operator) {\n    case '>': triggered = value > alert.condition.threshold; break;\n    case '<': triggered = value < alert.condition.threshold; break;\n    case '>=': triggered = value >= alert.condition.threshold; break;\n    case '<=': triggered = value <= alert.condition.threshold; break;\n  }\n  \n  if (triggered) {\n    results.push({\n      json: {\n        alert_id: alert.id,\n        alert_name: alert.name,\n        severity: alert.severity,\n        value,\n        threshold: alert.condition.threshold,\n        actions: alert.actions,\n        message: alert.actions[0].message\n          .replace('{{rate}}', value.toFixed(2))\n          .replace('{{avg_ms}}', Math.round(value))\n      }\n    });\n  }\n}\n\nreturn results;"
  }
}
```

---

## 6. Dashboard Layout

### 6.1 Page 1: Agent Health (Primary)

```
┌─────────────────────────────────────────────────────────┐
│                    AGENT HEALTH                         │
├────────────────┬────────────────┬────────────────────────┤
│   Uptime %     │  Error Rate    │   Response Time        │
│   [Gauge]      │  [Stat+Spark]  │   [Line Chart 1h]     │
├────────────────┴────────────────┴────────────────────────┤
│              Token Usage (24h)                           │
│              [Bar Chart by Agent]                        │
├─────────────────────────────────────────────────────────┤
│              Active Sessions [Table]                     │
└─────────────────────────────────────────────────────────┘
```

### 6.2 Page 2: Business Metrics

```
┌─────────────────────────────────────────────────────────┐
│                  BUSINESS METRICS                       │
├────────────────┬────────────────┬────────────────────────┤
│ Leads (30d)    │ Demos (30d)    │ Products (30d)         │
│ [Counter]      │ [Counter]      │ [Counter + Pie]        │
├────────────────┴────────────────┴────────────────────────┤
│              Revenue (90d)                               │
│              [Area Chart + KPI Card]                     │
└─────────────────────────────────────────────────────────┘
```

### 6.3 Page 3: System Metrics

```
┌─────────────────────────────────────────────────────────┐
│                   SYSTEM METRICS                        │
├─────────────────────────────────────────────────────────┤
│              API Calls Volume (24h)                     │
│              [Stacked Bar Chart]                        │
├────────────────┬────────────────┬────────────────────────┤
│ MCP Servers    │ Cron Jobs      │ Session Count          │
│ [Status Grid]  │ [Status List]  │ [Stat + Table]         │
├────────────────┴────────────────┴────────────────────────┤
│              Active Alerts                              │
│              [Alert List Panel]                         │
└─────────────────────────────────────────────────────────┘
```

---

## 7. Implementation Notes

### 7.1 N8N Dashboard Nodes

N8N supports dashboard widgets through:
- **`n8n-nodes-base.httpRequest`** — Query PostgreSQL API
- **`n8n-nodes-base.function`** — Transform data for display
- **Custom HTML/CSS** — Embed in N8N's built-in dashboard view
- **Webhook → Grafana** — For advanced dashboards, push to Grafana via webhook

### 7.2 Grafana Integration (Optional)

For production-grade dashboards, N8N can feed Grafana:

```json
{
  "datasource": {
    "type": "postgresql",
    "host": "sjc1.clusters.zeabur.com:22417",
    "database": "zeabur",
    "user": "root",
    "ssl": "require"
  },
  "dashboards": [
    {
      "uid": "dream-ai-agents",
      "title": "Dream AI Agent Monitoring",
      "tags": ["agents", "monitoring"],
      "refresh": "30s"
    }
  ]
}
```

### 7.3 Webhook → OpenClaw Integration

For real-time alerts to SOVEREIGN's webchat:

```json
{
  "name": "Webhook Alert to OpenClaw",
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "url": "https://YOUR_GATEWAY/webhook/alert",
    "method": "POST",
    "body": {
      "message": "={{$json.alert_message}}",
      "severity": "={{$json.severity}}",
      "timestamp": "={{$json.timestamp}}"
    }
  }
}
```

---

## 8. Data Flow Summary

```
[OpenClaw Agents]
       │
       ├── emit structured JSON logs
       ▼
[N8N Webhook: /webhook/agent-log]
       │
       ├── validate schema
       ├── classify level
       ├── store in PostgreSQL
       ├── alert on ERROR/CRITICAL → Telegram
       ▼
[PostgreSQL: agent_logs table]
       │
       ├── daily_metrics aggregation (cron)
       ├── alert evaluation (cron, every 1min)
       ├── dashboard queries (30s refresh)
       ▼
[Dashboard Widgets]
       │
       ├── Agent Health page
       ├── Business Metrics page
       └── System Metrics page
```

---

*Last updated: 2026-03-18*
*Owner: SOVEREIGN (agent infrastructure)*
*Related: [Logging System](./logging-system.md)*
