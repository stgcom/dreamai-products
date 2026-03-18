# Dream AI — Centralized Logging System

> Agent infrastructure logging, storage, and alerting blueprint.

---

## 1. Log Specification

### 1.1 Log Levels

| Level    | Code | Meaning                                    | Action Required |
|----------|------|--------------------------------------------|-----------------|
| `DEBUG`  | 0    | Verbose tracing for development            | None            |
| `INFO`   | 1    | Normal operational events                  | None            |
| `WARN`   | 2    | Recoverable issues, degraded state         | Review within 1h |
| `ERROR`  | 3    | Failures requiring attention               | Investigate now |
| `CRITICAL` | 4 | System-threatening, data loss risk         | Immediate alert |

### 1.2 Log Format: JSON Structured Logging

Every log entry MUST be a single-line JSON object. No multi-line. No free-text fields.

**Required Fields:**

```json
{
  "timestamp": "2026-03-18T21:43:00.000Z",
  "level": "INFO",
  "agent_id": "sovereign",
  "session_id": "sess_abc123def456",
  "action": "message_received",
  "duration_ms": 0,
  "status": "success"
}
```

**Full Schema (all possible fields):**

```json
{
  "timestamp": "ISO-8601 UTC",
  "level": "DEBUG | INFO | WARN | ERROR | CRITICAL",
  "agent_id": "sovereign | vector | axiom | echo | helix | forge | bolt | prism | pixel | quill | scout | metric | maven | pulse | catalyst",
  "session_id": "uuid",
  "action": "string — what happened",
  "duration_ms": "number — processing time in ms",
  "status": "success | failure | partial",
  "category": "inbound | outbound | tool_calls | errors | performance | billing",
  "user_id": "string — if available",
  "channel": "webchat | telegram | discord | api",
  "model": "string — LLM model used",
  "tokens_input": "number",
  "tokens_output": "number",
  "cost_usd": "number — estimated cost",
  "error": {
    "type": "string — exception class",
    "message": "string — human-readable",
    "stack": "string — truncated stack trace"
  },
  "metadata": "object — arbitrary context, max 1KB"
}
```

### 1.3 Field Constraints

| Field          | Type   | Max Length | Required |
|----------------|--------|------------|----------|
| `timestamp`    | string | 30         | ✅ |
| `level`        | string | 10         | ✅ |
| `agent_id`     | string | 50         | ✅ |
| `session_id`   | string | 64         | ✅ |
| `action`       | string | 100        | ✅ |
| `duration_ms`  | int    | —          | ✅ |
| `status`       | string | 20         | ✅ |
| `category`     | string | 20         | ✅ |
| `user_id`      | string | 100        | ❌ |
| `channel`      | string | 20         | ❌ |
| `model`        | string | 50         | ❌ |
| `tokens_input` | int    | —          | ❌ |
| `tokens_output`| int    | —          | ❌ |
| `cost_usd`     | float  | —          | ❌ |
| `error`        | object | —          | ❌ |
| `metadata`     | object | 1024 bytes | ❌ |

---

## 2. Log Categories

### 2.1 `inbound` — User Messages Received

Logged when any user message enters the system.

```json
{
  "timestamp": "2026-03-18T21:43:00.000Z",
  "level": "INFO",
  "agent_id": "sovereign",
  "session_id": "sess_abc123",
  "action": "inbound_message",
  "duration_ms": 0,
  "status": "success",
  "category": "inbound",
  "user_id": "686076918",
  "channel": "webchat",
  "metadata": {
    "message_length": 42,
    "has_media": false,
    "thread_id": null
  }
}
```

**Triggered by:**
- Webhook receives message from OpenClaw gateway
- Any channel (webchat, Telegram, Discord) delivers a message
- API endpoint receives a request

---

### 2.2 `outbound` — Responses Sent

Logged when the agent produces and delivers a response.

```json
{
  "timestamp": "2026-03-18T21:43:02.150Z",
  "level": "INFO",
  "agent_id": "sovereign",
  "session_id": "sess_abc123",
  "action": "outbound_response",
  "duration_ms": 2150,
  "status": "success",
  "category": "outbound",
  "channel": "webchat",
  "model": "openrouter/openrouter/healer-alpha",
  "tokens_input": 1200,
  "tokens_output": 350,
  "cost_usd": 0.0021,
  "metadata": {
    "response_length": 890,
    "dual_delivery": true,
    "telegram_sent": true
  }
}
```

**Triggered by:**
- Agent completes inference and sends reply
- Response delivered to all channels

---

### 2.3 `tool_calls` — MCP/Tool Invocations

Logged for every tool call the agent makes.

```json
{
  "timestamp": "2026-03-18T21:43:01.200Z",
  "level": "INFO",
  "agent_id": "sovereign",
  "session_id": "sess_abc123",
  "action": "tool_call",
  "duration_ms": 480,
  "status": "success",
  "category": "tool_calls",
  "metadata": {
    "tool_name": "web_search",
    "tool_type": "mcp",
    "mcp_server": "n8n-mcp",
    "input_params": {"query": "latest AI trends"},
    "output_size_bytes": 12400
  }
}
```

**Triggered by:**
- Agent calls any tool (web_search, browser, exec, message, etc.)
- MCP server invocation (n8n-mcp, postgres-mcp)
- External API calls

**Special sub-categories:**
- `tool_call_mcp` — MCP server calls
- `tool_call_internal` — Built-in tools (read, write, exec)
- `tool_call_external` — External API calls

---

### 2.4 `errors` — Failures and Exceptions

Logged at ERROR or CRITICAL level for any failure.

```json
{
  "timestamp": "2026-03-18T21:43:02.100Z",
  "level": "ERROR",
  "agent_id": "sovereign",
  "session_id": "sess_abc123",
  "action": "error",
  "duration_ms": 0,
  "status": "failure",
  "category": "errors",
  "error": {
    "type": "APITimeoutError",
    "message": "OpenRouter API request timed out after 30s",
    "stack": "APITimeoutError: timeout\n  at openRouterCall (agent.js:142)\n  at processMessage (agent.js:89)"
  },
  "metadata": {
    "retry_attempt": 1,
    "max_retries": 3,
    "endpoint": "https://openrouter.ai/api/v1/chat/completions",
    "http_status": 504
  }
}
```

**Error Classifications:**

| Error Type           | Level     | Auto-Retry | Alert |
|---------------------|-----------|------------|-------|
| `APITimeoutError`   | ERROR     | Yes (3x)   | Yes   |
| `AuthenticationError` | ERROR   | No         | Yes   |
| `RateLimitError`    | WARN      | Yes (w/ backoff) | No |
| `DatabaseError`     | CRITICAL  | Yes (1x)   | Immediate |
| `SessionExpired`    | INFO      | No         | No    |
| `ToolExecutionError`| ERROR     | Yes (2x)   | Yes   |
| `PaymentError`      | CRITICAL  | No         | Immediate |

---

### 2.5 `performance` — Latency, Token Usage

Logged at the end of every session turn for performance tracking.

```json
{
  "timestamp": "2026-03-18T21:43:02.150Z",
  "level": "INFO",
  "agent_id": "sovereign",
  "session_id": "sess_abc123",
  "action": "performance_metrics",
  "duration_ms": 2150,
  "status": "success",
  "category": "performance",
  "model": "openrouter/openrouter/healer-alpha",
  "tokens_input": 1200,
  "tokens_output": 350,
  "cost_usd": 0.0021,
  "metadata": {
    "ttft_ms": 340,
    "ttft_ms": 340,
    "tpot_ms": 12.5,
    "inference_ms": 1810,
    "total_overhead_ms": 340,
    "context_window_used": 45000,
    "context_window_max": 200000
  }
}
```

**Metrics Definitions:**
- `ttft_ms` — Time to first token (perceived latency)
- `tpot_ms` — Time per output token (throughput)
- `inference_ms` — Pure LLM call duration
- `total_overhead_ms` — Everything else (tool calls, formatting, delivery)

---

### 2.6 `billing` — Cost Tracking Per Session

Logged at session end or every N turns for cost attribution.

```json
{
  "timestamp": "2026-03-18T21:43:02.150Z",
  "level": "INFO",
  "agent_id": "sovereign",
  "session_id": "sess_abc123",
  "action": "billing_snapshot",
  "duration_ms": 0,
  "status": "success",
  "category": "billing",
  "cost_usd": 0.0187,
  "metadata": {
    "session_turn_count": 8,
    "session_duration_minutes": 12.4,
    "total_tokens_input": 18500,
    "total_tokens_output": 4200,
    "model_costs": {
      "openrouter/openrouter/healer-alpha": {
        "input_cost": 0.0130,
        "output_cost": 0.0057,
        "calls": 8
      }
    },
    "external_api_costs": {
      "n8n_webhook_calls": 0.00,
      "postgres_queries": 0.00
    },
    "daily_total_usd": 1.24,
    "monthly_total_usd": 18.70,
    "monthly_budget_usd": 100.00,
    "budget_utilization_pct": 18.7
  }
}
```

**Billing Trigger Points:**
- Every 10 turns within a session
- Session end (timeout or explicit close)
- Daily rollover (midnight UTC)
- Budget threshold breach (80%, 90%, 95%, 100%)

---

## 3. Log Storage Schema (PostgreSQL)

### 3.1 Table: `agent_logs`

```sql
CREATE TABLE agent_logs (
    id              BIGSERIAL PRIMARY KEY,
    timestamp       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    level           VARCHAR(10) NOT NULL,
    agent_id        VARCHAR(50) NOT NULL,
    session_id      VARCHAR(64) NOT NULL,
    action          VARCHAR(100) NOT NULL,
    duration_ms     INTEGER NOT NULL DEFAULT 0,
    status          VARCHAR(20) NOT NULL,
    category        VARCHAR(20) NOT NULL,
    user_id         VARCHAR(100),
    channel         VARCHAR(20),
    model           VARCHAR(50),
    tokens_input    INTEGER,
    tokens_output   INTEGER,
    cost_usd        DECIMAL(10,6),
    error_type      VARCHAR(100),
    error_message   TEXT,
    error_stack     TEXT,
    metadata        JSONB,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes for common queries
CREATE INDEX idx_logs_timestamp ON agent_logs (timestamp DESC);
CREATE INDEX idx_logs_level ON agent_logs (level);
CREATE INDEX idx_logs_agent ON agent_logs (agent_id, timestamp DESC);
CREATE INDEX idx_logs_session ON agent_logs (session_id, timestamp DESC);
CREATE INDEX idx_logs_category ON agent_logs (category, timestamp DESC);
CREATE INDEX idx_logs_errors ON agent_logs (level, timestamp DESC) WHERE level IN ('ERROR', 'CRITICAL');

-- Partition by month for performance (optional, for high-volume)
-- CREATE TABLE agent_logs_2026_03 PARTITION OF agent_logs
--     FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');
```

### 3.2 Table: `agent_sessions`

```sql
CREATE TABLE agent_sessions (
    session_id          VARCHAR(64) PRIMARY KEY,
    agent_id            VARCHAR(50) NOT NULL,
    user_id             VARCHAR(100),
    channel             VARCHAR(20),
    started_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    ended_at            TIMESTAMPTZ,
    turn_count          INTEGER DEFAULT 0,
    total_tokens_input  INTEGER DEFAULT 0,
    total_tokens_output INTEGER DEFAULT 0,
    total_cost_usd      DECIMAL(10,6) DEFAULT 0,
    error_count         INTEGER DEFAULT 0,
    status              VARCHAR(20) DEFAULT 'active',
    metadata            JSONB
);
```

### 3.3 Table: `daily_metrics`

```sql
CREATE TABLE daily_metrics (
    date                DATE PRIMARY KEY,
    agent_id            VARCHAR(50),
    total_sessions      INTEGER DEFAULT 0,
    total_turns         INTEGER DEFAULT 0,
    total_tokens_input  BIGINT DEFAULT 0,
    total_tokens_output BIGINT DEFAULT 0,
    total_cost_usd      DECIMAL(10,4) DEFAULT 0,
    error_count         INTEGER DEFAULT 0,
    avg_response_ms     INTEGER,
    p95_response_ms     INTEGER,
    p99_response_ms     INTEGER,
    leads_captured      INTEGER DEFAULT 0,
    demos_booked        INTEGER DEFAULT 0,
    products_sold       INTEGER DEFAULT 0,
    revenue_usd         DECIMAL(10,2) DEFAULT 0
);
```

---

## 4. N8N Logging Workflow Blueprint

### 4.1 Workflow: `agent-log-ingest`

**Trigger:** Webhook at `https://dreamain8n.zeabur.app/webhook/agent-log`

**Node Graph:**

```
[Webhook] → [Validate Schema] → [Classify Level] → [Store in PostgreSQL]
                                              ↓ (ERROR/CRITICAL)
                                     [Alert → Telegram]
                                              ↓
                                     [Alert → Webchat]
```

#### Node 1: Webhook Trigger

```json
{
  "name": "Log Webhook",
  "type": "n8n-nodes-base.webhook",
  "parameters": {
    "httpMethod": "POST",
    "path": "agent-log",
    "responseMode": "responseNode",
    "options": {
      "rawBody": false
    }
  },
  "position": [250, 300]
}
```

#### Node 2: Validate Schema

```json
{
  "name": "Validate Log Schema",
  "type": "n8n-nodes-base.function",
  "parameters": {
    "functionCode": "const required = ['timestamp','level','agent_id','session_id','action','duration_ms','status'];\nconst log = items[0].json.body;\n\nconst missing = required.filter(f => !(f in log));\nif (missing.length > 0) {\n  throw new Error(`Missing required fields: ${missing.join(', ')}`);\n}\n\nconst validLevels = ['DEBUG','INFO','WARN','ERROR','CRITICAL'];\nif (!validLevels.includes(log.level)) {\n  throw new Error(`Invalid log level: ${log.level}`);\n}\n\nitems[0].json = log;\nreturn items;"
  },
  "position": [450, 300]
}
```

#### Node 3: Classify Level (Switch)

```json
{
  "name": "Classify Log Level",
  "type": "n8n-nodes-base.switch",
  "parameters": {
    "dataPropertyName": "level",
    "rules": {
      "rules": [
        { "value": "CRITICAL", "outputIndex": 0 },
        { "value": "ERROR", "outputIndex": 1 },
        { "value": "WARN", "outputIndex": 2 }
      ]
    },
    "fallbackOutput": 3
  },
  "position": [650, 300]
}
```

#### Node 4a: Store in PostgreSQL

```json
{
  "name": "Insert Log Record",
  "type": "n8n-nodes-base.postgres",
  "parameters": {
    "operation": "insert",
    "table": "agent_logs",
    "schema": "public",
    "columns": "timestamp,level,agent_id,session_id,action,duration_ms,status,category,user_id,channel,model,tokens_input,tokens_output,cost_usd,error_type,error_message,error_stack,metadata",
    "additionalFields": {
      "queryReplacement": "{{$json.timestamp}}, {{$json.level}}, {{$json.agent_id}}, {{$json.session_id}}, {{$json.action}}, {{$json.duration_ms}}, {{$json.status}}, {{$json.category}}, {{$json.user_id}}, {{$json.channel}}, {{$json.model}}, {{$json.tokens_input}}, {{$json.tokens_output}}, {{$json.cost_usd}}, {{$json.error?.type}}, {{$json.error?.message}}, {{$json.error?.stack}}, '{{JSON.stringify($json.metadata)}}'::jsonb"
    }
  },
  "position": [850, 500],
  "credentials": {
    "postgres": "Dream AI PostgreSQL"
  }
}
```

#### Node 4b: Alert on CRITICAL (Telegram)

```json
{
  "name": "Telegram Critical Alert",
  "type": "n8n-nodes-base.telegram",
  "parameters": {
    "operation": "sendMessage",
    "chatId": "686076918",
    "text": "🚨 CRITICAL LOG\n\nAgent: {{$json.agent_id}}\nSession: {{$json.session_id}}\nAction: {{$json.action}}\nError: {{$json.error?.message || 'N/A'}}\nTime: {{$json.timestamp}}",
    "additionalOptions": {
      "parseMode": "HTML"
    }
  },
  "position": [1050, 150],
  "credentials": {
    "telegramApi": "Dream AI Telegram"
  }
}
```

#### Node 4c: Alert on ERROR (Telegram + Webchat)

```json
{
  "name": "Telegram Error Alert",
  "type": "n8n-nodes-base.telegram",
  "parameters": {
    "operation": "sendMessage",
    "chatId": "686076918",
    "text": "⚠️ ERROR LOG\n\nAgent: {{$json.agent_id}}\nSession: {{$json.session_id}}\nAction: {{$json.action}}\nError: {{$json.error?.message || 'N/A'}}\nTime: {{$json.timestamp}}\n\nRetry: {{$json.metadata?.retry_attempt || 0}}/{{$json.metadata?.max_retries || 0}}"
  },
  "position": [1050, 300],
  "credentials": {
    "telegramApi": "Dream AI Telegram"
  }
}
```

#### Node 5: Respond

```json
{
  "name": "Respond OK",
  "type": "n8n-nodes-base.respondToWebhook",
  "parameters": {
    "respondWith": "json",
    "responseBody": "={{JSON.stringify({status: 'ok'})}}"
  },
  "position": [850, 700]
}
```

---

### 4.2 Workflow: `agent-daily-log-summary`

**Trigger:** Cron — every day at 08:00 UTC

**Node Graph:**

```
[Cron 08:00 UTC] → [Query Daily Metrics] → [Format Summary] → [Telegram Message] → [Webchat Message]
```

#### Node 1: Cron Trigger

```json
{
  "name": "Daily 8AM UTC",
  "type": "n8n-nodes-base.cron",
  "parameters": {
    "triggerTimes": {
      "item": [{ "hour": 8, "minute": 0 }]
    }
  }
}
```

#### Node 2: Query Yesterday's Metrics

```json
{
  "name": "Query Daily Metrics",
  "type": "n8n-nodes-base.postgres",
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT\n  agent_id,\n  COUNT(DISTINCT session_id) as sessions,\n  COUNT(*) as total_logs,\n  COUNT(*) FILTER (WHERE level = 'ERROR') as errors,\n  COUNT(*) FILTER (WHERE level = 'CRITICAL') as critical,\n  AVG(duration_ms) as avg_duration_ms,\n  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY duration_ms) as p95_duration_ms,\n  SUM(tokens_input) as total_input_tokens,\n  SUM(tokens_output) as total_output_tokens,\n  SUM(cost_usd) as total_cost_usd\nFROM agent_logs\nWHERE timestamp >= NOW() - INTERVAL '1 day'\nGROUP BY agent_id\nORDER BY total_cost_usd DESC;"
  },
  "position": [450, 300],
  "credentials": {
    "postgres": "Dream AI PostgreSQL"
  }
}
```

#### Node 3: Format Summary Message

```json
{
  "name": "Format Summary",
  "type": "n8n-nodes-base.function",
  "parameters": {
    "functionCode": "const rows = items.map(i => i.json);\nconst date = new Date(Date.now() - 86400000).toISOString().split('T')[0];\n\nlet msg = `📊 Daily Agent Report — ${date}\\n\\n`;\nlet totalCost = 0;\nlet totalSessions = 0;\nlet totalErrors = 0;\n\nfor (const row of rows) {\n  msg += `🤖 ${row.agent_id}\\n`;\n  msg += `   Sessions: ${row.sessions} | Errors: ${row.errors}/${row.critical} critical\\n`;\n  msg += `   Avg Response: ${Math.round(row.avg_duration_ms)}ms | P95: ${Math.round(row.p95_duration_ms)}ms\\n`;\n  msg += `   Tokens: ${(row.total_input_tokens || 0).toLocaleString()} in / ${(row.total_output_tokens || 0).toLocaleString()} out\\n`;\n  msg += `   Cost: $${(row.total_cost_usd || 0).toFixed(4)}\\n\\n`;\n  totalCost += parseFloat(row.total_cost_usd) || 0;\n  totalSessions += parseInt(row.sessions) || 0;\n  totalErrors += parseInt(row.errors) || 0;\n}\n\nmsg += `💰 Total Cost: $${totalCost.toFixed(4)}\\n`;\nmsg += `📈 Total Sessions: ${totalSessions} | Total Errors: ${totalErrors}`;\n\nreturn [{ json: { message: msg, date, totalCost, totalSessions, totalErrors } }];"
  },
  "position": [650, 300]
}
```

#### Node 4: Send to Telegram

```json
{
  "name": "Telegram Summary",
  "type": "n8n-nodes-base.telegram",
  "parameters": {
    "operation": "sendMessage",
    "chatId": "686076918",
    "text": "={{$json.message}}"
  },
  "position": [850, 250],
  "credentials": {
    "telegramApi": "Dream AI Telegram"
  }
}
```

---

## 5. Log Emission Helper (Agent-Side)

Every agent session should emit structured logs. Here's the pattern:

```javascript
// Log emission helper — wraps all logging
function emitLog(level, action, category, data = {}) {
  const log = {
    timestamp: new Date().toISOString(),
    level,
    agent_id: process.env.AGENT_ID || 'unknown',
    session_id: data.session_id || 'no-session',
    action,
    duration_ms: data.duration_ms || 0,
    status: data.status || 'success',
    category,
    ...data
  };

  // Send to N8N webhook (non-blocking)
  fetch('https://dreamain8n.zeabur.app/webhook/agent-log', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(log)
  }).catch(err => {
    // Fallback: write to local log file
    console.error('Log emit failed:', err.message);
    console.log(JSON.stringify(log)); // stdout fallback
  });

  return log;
}

// Usage examples:
// emitLog('INFO', 'inbound_message', 'inbound', { session_id, user_id, channel });
// emitLog('INFO', 'outbound_response', 'outbound', { session_id, duration_ms, model, tokens });
// emitLog('ERROR', 'api_timeout', 'errors', { session_id, error: { type: 'APITimeoutError', message: '...' } });
```

---

## 6. Retention Policy

| Log Level   | Hot Storage (PostgreSQL) | Warm Storage (S3/GCS) | Cold Archive |
|------------|--------------------------|----------------------|--------------|
| `DEBUG`    | 7 days                   | 30 days              | Delete       |
| `INFO`     | 30 days                  | 90 days              | 1 year       |
| `WARN`     | 90 days                  | 1 year               | 3 years      |
| `ERROR`    | 1 year                   | 3 years              | 7 years      |
| `CRITICAL` | 1 year                   | 3 years              | 7 years      |

---

## 7. Dashboard Integration Points

Logs feed directly into the [Observability Dashboard](./dashboard-spec.md):

- **Agent Health** ← `performance` category
- **Business Metrics** ← `billing` + custom business events
- **System Metrics** ← `tool_calls` + `errors` categories
- **Alert Rules** ← `ERROR` + `CRITICAL` level logs

---

*Last updated: 2026-03-18*
*Owner: SOVEREIGN (agent infrastructure)*
