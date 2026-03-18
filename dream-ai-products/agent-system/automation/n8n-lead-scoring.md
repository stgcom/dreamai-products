# AI Lead Scoring System — N8N Workflow Blueprint

> **Workflow Name:** AI Lead Scoring System
> **Purpose:** Real-time lead scoring via webhook events + daily recalculation with engagement decay
> **Created:** 2026-03-18
> **Status:** Blueprint (import-ready JSON below)

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Database Schema](#database-schema)
3. [Workflow 1: Real-Time Scoring (Webhook)](#workflow-1-real-time-scoring-webhook)
4. [Workflow 2: Daily Recalculation (Scheduled)](#workflow-2-daily-recalculation-scheduled)
5. [N8N Node JSON — Complete Export](#n8n-node-json--complete-export)
6. [Lead Scoring Dashboard Queries](#lead-scoring-dashboard-queries)
7. [Environment Variables](#environment-variables)
8. [Deployment Checklist](#deployment-checklist)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     WEBSITE / APP EVENTS                        │
│  calculator_complete · scorecard_complete · email_open · etc.   │
└───────────────────────────┬─────────────────────────────────────┘
                            │ POST /webhook/lead-event
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  WORKFLOW 1: REAL-TIME SCORING                                  │
│                                                                 │
│  Webhook ──► Get Contact ──► Calculate Score ──► Apply Decay   │
│                                              │                  │
│                                              ▼                  │
│                                        Update Contact           │
│                                              │                  │
│                                         Score >= 70?            │
│                                         YES ──► Telegram Alert  │
│                                         NO  ──► (end)           │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  WORKFLOW 2: DAILY RECALCULATION                                │
│                                                                 │
│  Schedule (09:00 UTC) ──► Get All Contacts ──► Loop:           │
│    ► Apply Decay ──► Update Score ──► Check Threshold           │
│                                    ──► Telegram Alert (if hot)  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Database Schema

```sql
-- PostgreSQL tables for lead scoring

CREATE TABLE IF NOT EXISTS contacts (
    id              SERIAL PRIMARY KEY,
    email           VARCHAR(255) UNIQUE NOT NULL,
    name            VARCHAR(255),
    vertical        VARCHAR(100),           -- industry
    company         VARCHAR(255),
    phone           VARCHAR(50),
    source          VARCHAR(100),           -- where they came from
    lead_score      INTEGER DEFAULT 0,
    temperature     VARCHAR(20) DEFAULT 'cold', -- cold / warm / hot
    last_engagement TIMESTAMP,
    created_at      TIMESTAMP DEFAULT NOW(),
    updated_at      TIMESTAMP DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS lead_events (
    id              SERIAL PRIMARY KEY,
    contact_id      INTEGER REFERENCES contacts(id),
    email           VARCHAR(255) NOT NULL,  -- denormalized for webhook lookups
    event_type      VARCHAR(100) NOT NULL,
    points_awarded  INTEGER NOT NULL,
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_contacts_email ON contacts(email);
CREATE INDEX idx_contacts_score ON contacts(lead_score DESC);
CREATE INDEX idx_contacts_temperature ON contacts(temperature);
CREATE INDEX idx_lead_events_contact ON lead_events(contact_id);
CREATE INDEX idx_lead_events_type ON lead_events(event_type);
CREATE INDEX idx_contacts_last_engagement ON contacts(last_engagement);
```

---

## Scoring Rules

| Event Type           | Points | Category       |
|----------------------|--------|----------------|
| `calculator_complete`| +20    | High intent    |
| `scorecard_complete` | +25    | High intent    |
| `email_open`         | +5     | Low intent     |
| `link_click`         | +10    | Medium intent  |
| `page_visit`         | +5     | Low intent     |
| `pricing_page`       | +15    | Medium intent  |
| `demo_booked`        | +30    | Highest intent |
| `email_reply`        | +20    | High intent    |

### Temperature Thresholds

| Temperature | Score Range | Action |
|-------------|-------------|--------|
| 🔥 Hot      | ≥ 70        | Alert sales immediately |
| 🟡 Warm     | 30–69       | Nurture sequence |
| 🔵 Cold     | 0–29        | General list |

### Decay Rule

- If `last_engagement` > 7 days ago: subtract **5 points per week** (rounded down)
- Minimum score: 0 (never goes negative)
- Decay is applied during daily recalculation and on each new event

---

## Workflow 1: Real-Time Scoring (Webhook)

### Node 1: Webhook Input

**Type:** `n8n-nodes-base.webhook`
**Method:** POST
**Path:** `/lead-event`

```json
{
  "parameters": {
    "httpMethod": "POST",
    "path": "lead-event",
    "responseMode": "onReceived",
    "responseData": "allEntries",
    "options": {
      "rawBody": false
    }
  },
  "id": "webhook-lead-event",
  "name": "Webhook - Lead Event",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 1,
  "position": [240, 300],
  "webhookId": "lead-event-001",
  "notes": "Receives scoring events from website. Expected payload: {email, event_type, timestamp, metadata}"
}
```

**Expected Input Payload:**
```json
{
  "email": "prospect@acmecorp.com",
  "event_type": "calculator_complete",
  "timestamp": "2026-03-18T14:30:00Z",
  "metadata": {
    "calculator_type": "roi",
    "company_size": "50-200",
    "page_url": "/tools/roi-calculator"
  }
}
```

---

### Node 2: Get Contact

**Type:** `n8n-nodes-base.postgres` (or HubSpot CRM node)

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT id, email, name, vertical, company, lead_score, last_engagement, created_at FROM contacts WHERE email = '{{ $json.email }}' LIMIT 1",
    "options": {}
  },
  "id": "get-contact",
  "name": "Get Contact",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [460, 300],
  "credentials": {
    "postgres": {
      "id": "REPLACE_WITH_CREDENTIAL_ID",
      "name": "Dream AI PostgreSQL"
    }
  },
  "notes": "Looks up existing contact by email. If no row returned, contact will be created in Update Contact step."
}
```

**If contact not found** (empty result): Set a flag `contact_exists = false` via a subsequent IF node so Update Contact can INSERT instead of UPDATE.

---

### Node 3: Set Score Points (Code Node)

**Type:** `n8n-nodes-base.code`

```json
{
  "parameters": {
    "jsCode": "// Score mapping\nconst SCORE_MAP = {\n  'calculator_complete': 20,\n  'scorecard_complete': 25,\n  'email_open': 5,\n  'link_click': 10,\n  'page_visit': 5,\n  'pricing_page': 15,\n  'demo_booked': 30,\n  'email_reply': 20\n};\n\nconst items = $input.all();\nconst results = [];\n\nfor (const item of items) {\n  const eventType = item.json.event_type;\n  const points = SCORE_MAP[eventType] || 0;\n  \n  if (points === 0) {\n    // Unknown event type — skip with warning\n    console.warn(`Unknown event type: ${eventType}`);\n  }\n  \n  results.push({\n    json: {\n      ...item.json,\n      points_awarded: points,\n      scoring_applied: points > 0\n    }\n  });\n}\n\nreturn results;"
  },
  "id": "calculate-score",
  "name": "Calculate Score",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [680, 300],
  "notes": "Looks up points for the event_type. Returns points_awarded for downstream use."
}
```

---

### Node 4: Apply Decay (Code Node)

**Type:** `n8n-nodes-base.code`

```json
{
  "parameters": {
    "jsCode": "// Apply engagement decay\n// If last_engagement > 7 days ago, subtract 5 points per week\n\nconst items = $input.all();\nconst results = [];\n\nfor (const item of items) {\n  const data = item.json;\n  let currentScore = data.contact_exists !== false && data.lead_score\n    ? data.lead_score\n    : 0;\n  \n  let decayApplied = 0;\n  \n  if (data.last_engagement) {\n    const lastEngagement = new Date(data.last_engagement);\n    const now = new Date();\n    const daysSince = Math.floor((now - lastEngagement) / (1000 * 60 * 60 * 24));\n    \n    if (daysSince > 7) {\n      const weeksSince = Math.floor((daysSince - 7) / 7);\n      decayApplied = weeksSince * 5;\n    }\n  }\n  \n  const newScore = Math.max(0, currentScore + (data.points_awarded || 0) - decayApplied);\n  const newTemperature = newScore >= 70 ? 'hot' : (newScore >= 30 ? 'warm' : 'cold');\n  \n  results.push({\n    json: {\n      ...data,\n      current_score: currentScore,\n      decay_applied: decayApplied,\n      new_score: newScore,\n      new_temperature: newTemperature,\n      threshold_triggered: newScore >= 70\n    }\n  });\n}\n\nreturn results;"
  },
  "id": "apply-decay",
  "name": "Apply Decay",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [900, 300],
  "notes": "Applies score decay: -5 points per week since last engagement (only if > 7 days). Never goes below 0."
}
```

---

### Node 5: Update Contact

**Type:** `n8n-nodes-base.postgres`

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "INSERT INTO contacts (email, name, vertical, company, lead_score, temperature, last_engagement, created_at, updated_at) VALUES ('{{ $json.email }}', '{{ $json.name || '' }}', '{{ $json.vertical || '' }}', '{{ $json.company || '' }}', {{ $json.new_score }}, '{{ $json.new_temperature }}', '{{ $json.timestamp }}', NOW(), NOW()) ON CONFLICT (email) DO UPDATE SET lead_score = {{ $json.new_score }}, temperature = '{{ $json.new_temperature }}', last_engagement = '{{ $json.timestamp }}', updated_at = NOW() RETURNING id, email, name, vertical, lead_score, temperature",
    "options": {}
  },
  "id": "update-contact",
  "name": "Update Contact",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [1120, 300],
  "credentials": {
    "postgres": {
      "id": "REPLACE_WITH_CREDENTIAL_ID",
      "name": "Dream AI PostgreSQL"
    }
  },
  "notes": "UPSERT: creates contact if new, updates score/temperature if existing. Also logs event to lead_events table."
}
```

**Additional query to log the event** (Add as a second Execute Query node or use a Merge):

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "INSERT INTO lead_events (contact_id, email, event_type, points_awarded, metadata, created_at) VALUES ((SELECT id FROM contacts WHERE email = '{{ $json.email }}'), '{{ $json.email }}', '{{ $json.event_type }}', {{ $json.points_awarded }}, '{{ JSON.stringify($json.metadata || {}) }}'::jsonb, '{{ $json.timestamp }}')"
  },
  "id": "log-event",
  "name": "Log Event",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [1120, 460],
  "credentials": {
    "postgres": {
      "id": "REPLACE_WITH_CREDENTIAL_ID",
      "name": "Dream AI PostgreSQL"
    }
  }
}
```

---

### Node 6: Check Threshold

**Type:** `n8n-nodes-base.if`

```json
{
  "parameters": {
    "conditions": {
      "boolean": [
        {
          "value1": "={{ $json.threshold_triggered }}",
          "value2": true
        }
      ]
    }
  },
  "id": "check-threshold",
  "name": "Score >= 70?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1,
  "position": [1340, 300],
  "notes": "Routes hot leads (score >= 70) to Telegram alert. Others end here."
}
```

---

### Node 7: Telegram Alert (Hot Lead)

**Type:** `n8n-nodes-base.telegram`

```json
{
  "parameters": {
    "chatId": "686076918",
    "text": "🔥 HOT LEAD ALERT\n\nName: {{ $json.name || 'Unknown' }}\nEmail: {{ $json.email }}\nScore: {{ $json.new_score }}\nIndustry: {{ $json.vertical || 'Not specified' }}\nLast Event: {{ $json.event_type }} at {{ $json.timestamp }}\n\nAction: Call/email within 2 hours",
    "additionalFields": {
      "parse_mode": ""
    }
  },
  "id": "telegram-alert",
  "name": "Telegram - Hot Lead Alert",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1,
  "position": [1560, 220],
  "credentials": {
    "telegramApi": {
      "id": "REPLACE_WITH_CREDENTIAL_ID",
      "name": "Dream AI Telegram Bot"
    }
  },
  "notes": "Sends alert to Stephen's Telegram when a lead crosses the hot threshold (≥ 70)."
}
```

---

## Workflow 2: Daily Recalculation (Scheduled)

### Trigger: Schedule

**Type:** `n8n-nodes-base.scheduleTrigger`

```json
{
  "parameters": {
    "rule": {
      "interval": [
        {
          "field": "cronExpression",
          "expression": "0 9 * * *"
        }
      ]
    }
  },
  "id": "daily-schedule",
  "name": "Daily 09:00 UTC",
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1,
  "position": [240, 300],
  "notes": "Triggers daily at 09:00 UTC to recalculate all lead scores with decay."
}
```

### Get All Contacts

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT id, email, name, vertical, company, lead_score, last_engagement FROM contacts WHERE lead_score > 0 ORDER BY lead_score DESC",
    "options": {}
  },
  "id": "get-all-contacts",
  "name": "Get All Contacts",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [460, 300],
  "credentials": {
    "postgres": {
      "id": "REPLACE_WITH_CREDENTIAL_ID",
      "name": "Dream AI PostgreSQL"
    }
  }
}
```

### Loop + Apply Decay (Split In Batches)

```json
{
  "parameters": {
    "batchSize": 50,
    "options": {}
  },
  "id": "split-batches",
  "name": "Process in Batches",
  "type": "n8n-nodes-base.splitInBatches",
  "typeVersion": 3,
  "position": [680, 300]
}
```

### Calculate Decay (Code Node)

```json
{
  "parameters": {
    "jsCode": "// Daily decay recalculation for all contacts\n\nconst items = $input.all();\nconst results = [];\n\nfor (const item of items) {\n  const contact = item.json;\n  let decayApplied = 0;\n  \n  if (contact.last_engagement) {\n    const lastEngagement = new Date(contact.last_engagement);\n    const now = new Date();\n    const daysSince = Math.floor((now - lastEngagement) / (1000 * 60 * 60 * 24));\n    \n    if (daysSince > 7) {\n      const weeksSince = Math.floor((daysSince - 7) / 7);\n      decayApplied = Math.min(weeksSince * 5, contact.lead_score); // Don't go below 0\n    }\n  }\n  \n  const newScore = Math.max(0, contact.lead_score - decayApplied);\n  const newTemperature = newScore >= 70 ? 'hot' : (newScore >= 30 ? 'warm' : 'cold');\n  \n  results.push({\n    json: {\n      id: contact.id,\n      email: contact.email,\n      name: contact.name,\n      vertical: contact.vertical,\n      old_score: contact.lead_score,\n      decay_applied: decayApplied,\n      new_score: newScore,\n      new_temperature: newTemperature,\n      was_hot: contact.lead_score >= 70,\n      is_hot: newScore >= 70,\n      just_crossed_threshold: (newScore >= 70) && (contact.lead_score < 70)\n    }\n  });\n}\n\nreturn results;"
  },
  "id": "daily-decay",
  "name": "Calculate Daily Decay",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [900, 300]
}
```

### Update Each Contact

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "UPDATE contacts SET lead_score = {{ $json.new_score }}, temperature = '{{ $json.new_temperature }}', updated_at = NOW() WHERE id = {{ $json.id }}",
    "options": {}
  },
  "id": "update-daily-contact",
  "name": "Update Contact Score",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [1120, 300],
  "credentials": {
    "postgres": {
      "id": "REPLACE_WITH_CREDENTIAL_ID",
      "name": "Dream AI PostgreSQL"
    }
  }
}
```

### Check: Just Crossed Hot Threshold?

```json
{
  "parameters": {
    "conditions": {
      "boolean": [
        {
          "value1": "={{ $json.just_crossed_threshold }}",
          "value2": true
        }
      ]
    }
  },
  "id": "daily-check-threshold",
  "name": "Just Became Hot?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1,
  "position": [1340, 300]
}
```

### Daily Telegram Digest

```json
{
  "parameters": {
    "chatId": "686076918",
    "text": "📊 DAILY LEAD SCORING UPDATE\n\nLeads processed: {{ $json.total_processed }}\nDecay applied to: {{ $json.decayed_count }} contacts\nNew hot leads: {{ $json.new_hot_leads }}\n\n---\n🔥 Hot Leads (≥70): {{ $json.hot_count }}\n🟡 Warm Leads (30-69): {{ $json.warm_count }}\n🔵 Cold Leads (<30): {{ $json.cold_count }}"
  },
  "id": "daily-telegram-digest",
  "name": "Telegram - Daily Digest",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1,
  "position": [1560, 220],
  "credentials": {
    "telegramApi": {
      "id": "REPLACE_WITH_CREDENTIAL_ID",
      "name": "Dream AI Telegram Bot"
    }
  },
  "notes": "Sends a daily summary to Telegram with lead distribution counts."
}
```

### Aggregate Stats (Before Digest)

```json
{
  "parameters": {
    "jsCode": "// After processing all contacts, aggregate stats\n// This node would be placed after the batch loop completes\n\nconst items = $input.all();\nlet hotCount = 0, warmCount = 0, coldCount = 0;\nlet decayedCount = 0, newHotLeads = 0;\n\nfor (const item of items) {\n  if (item.json.new_temperature === 'hot') hotCount++;\n  else if (item.json.new_temperature === 'warm') warmCount++;\n  else coldCount++;\n  \n  if (item.json.decay_applied > 0) decayedCount++;\n  if (item.json.just_crossed_threshold) newHotLeads++;\n}\n\nreturn [{\n  json: {\n    total_processed: items.length,\n    hot_count: hotCount,\n    warm_count: warmCount,\n    cold_count: coldCount,\n    decayed_count: decayedCount,\n    new_hot_leads: newHotLeads\n  }\n}];"
  },
  "id": "aggregate-stats",
  "name": "Aggregate Stats",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [1340, 500]
}
```

---

## N8N Node JSON — Complete Export

Below is the complete N8N workflow JSON for **Workflow 1 (Real-Time Scoring)** that can be directly imported.

### Import Instructions

1. Open N8N → Workflows → Import from File
2. Paste the JSON below
3. Replace `REPLACE_WITH_CREDENTIAL_ID` placeholders with your actual credential IDs
4. Activate the workflow

```json
{
  "name": "AI Lead Scoring - Real-Time",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "lead-event",
        "responseMode": "onReceived",
        "responseData": "allEntries",
        "options": {}
      },
      "id": "webhook-lead-event",
      "name": "Webhook - Lead Event",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [240, 300],
      "webhookId": "lead-event-001"
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "SELECT id, email, name, vertical, company, lead_score, last_engagement, created_at FROM contacts WHERE email = '{{ $json.email }}' LIMIT 1",
        "options": {}
      },
      "id": "get-contact",
      "name": "Get Contact",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [460, 300],
      "credentials": {
        "postgres": {
          "id": "REPLACE_WITH_CREDENTIAL_ID",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Score mapping\nconst SCORE_MAP = {\n  'calculator_complete': 20,\n  'scorecard_complete': 25,\n  'email_open': 5,\n  'link_click': 10,\n  'page_visit': 5,\n  'pricing_page': 15,\n  'demo_booked': 30,\n  'email_reply': 20\n};\n\nconst items = $input.all();\nconst results = [];\n\nfor (const item of items) {\n  const eventType = item.json.event_type;\n  const points = SCORE_MAP[eventType] || 0;\n  const contactExists = item.json.id != null;\n  const currentScore = item.json.lead_score || 0;\n  const lastEngagement = item.json.last_engagement || item.json.timestamp;\n  \n  results.push({\n    json: {\n      ...item.json,\n      points_awarded: points,\n      scoring_applied: points > 0,\n      contact_exists: contactExists,\n      current_score: currentScore,\n      last_engagement_for_decay: lastEngagement\n    }\n  });\n}\n\nreturn results;"
      },
      "id": "calculate-score",
      "name": "Calculate Score",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [680, 300]
    },
    {
      "parameters": {
        "jsCode": "// Apply engagement decay\nconst items = $input.all();\nconst results = [];\n\nfor (const item of items) {\n  const data = item.json;\n  let currentScore = data.current_score || 0;\n  let decayApplied = 0;\n  \n  if (data.last_engagement_for_decay) {\n    const lastEngagement = new Date(data.last_engagement_for_decay);\n    const now = new Date();\n    const daysSince = Math.floor((now - lastEngagement) / (1000 * 60 * 60 * 24));\n    \n    if (daysSince > 7) {\n      const weeksSince = Math.floor((daysSince - 7) / 7);\n      decayApplied = Math.min(weeksSince * 5, currentScore);\n    }\n  }\n  \n  const newScore = Math.max(0, currentScore + (data.points_awarded || 0) - decayApplied);\n  const newTemperature = newScore >= 70 ? 'hot' : (newScore >= 30 ? 'warm' : 'cold');\n  \n  results.push({\n    json: {\n      email: data.email,\n      name: data.name,\n      vertical: data.vertical,\n      company: data.company,\n      event_type: data.event_type,\n      timestamp: data.timestamp,\n      metadata: data.metadata,\n      points_awarded: data.points_awarded,\n      current_score: currentScore,\n      decay_applied: decayApplied,\n      new_score: newScore,\n      new_temperature: newTemperature,\n      threshold_triggered: newScore >= 70\n    }\n  });\n}\n\nreturn results;"
      },
      "id": "apply-decay",
      "name": "Apply Decay",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [900, 300]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO contacts (email, name, vertical, company, lead_score, temperature, last_engagement, created_at, updated_at) VALUES ('{{ $json.email }}', '{{ $json.name || '' }}', '{{ $json.vertical || '' }}', '{{ $json.company || '' }}', {{ $json.new_score }}, '{{ $json.new_temperature }}', '{{ $json.timestamp }}', NOW(), NOW()) ON CONFLICT (email) DO UPDATE SET lead_score = {{ $json.new_score }}, temperature = '{{ $json.new_temperature }}', last_engagement = '{{ $json.timestamp }}', updated_at = NOW() RETURNING id, email, name, vertical, lead_score, temperature",
        "options": {}
      },
      "id": "update-contact",
      "name": "Update Contact",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1120, 300],
      "credentials": {
        "postgres": {
          "id": "REPLACE_WITH_CREDENTIAL_ID",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO lead_events (contact_id, email, event_type, points_awarded, metadata, created_at) VALUES ((SELECT id FROM contacts WHERE email = '{{ $json.email }}'), '{{ $json.email }}', '{{ $json.event_type }}', {{ $json.points_awarded }}, '{{ JSON.stringify($json.metadata || {}) }}'::jsonb, '{{ $json.timestamp }}')",
        "options": {}
      },
      "id": "log-event",
      "name": "Log Event",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1120, 460],
      "credentials": {
        "postgres": {
          "id": "REPLACE_WITH_CREDENTIAL_ID",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{ $json.threshold_triggered }}",
              "value2": true
            }
          ]
        }
      },
      "id": "check-threshold",
      "name": "Score >= 70?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [1340, 300]
    },
    {
      "parameters": {
        "chatId": "686076918",
        "text": "🔥 HOT LEAD ALERT\n\nName: {{ $json.name || 'Unknown' }}\nEmail: {{ $json.email }}\nScore: {{ $json.new_score }}\nIndustry: {{ $json.vertical || 'Not specified' }}\nLast Event: {{ $json.event_type }} at {{ $json.timestamp }}\n\nAction: Call/email within 2 hours",
        "additionalFields": {
          "parse_mode": ""
        }
      },
      "id": "telegram-alert",
      "name": "Telegram - Hot Lead Alert",
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1,
      "position": [1560, 220],
      "credentials": {
        "telegramApi": {
          "id": "REPLACE_WITH_CREDENTIAL_ID",
          "name": "Dream AI Telegram Bot"
        }
      }
    }
  ],
  "connections": {
    "Webhook - Lead Event": {
      "main": [
        [{ "node": "Get Contact", "type": "main", "index": 0 }]
      ]
    },
    "Get Contact": {
      "main": [
        [{ "node": "Calculate Score", "type": "main", "index": 0 }]
      ]
    },
    "Calculate Score": {
      "main": [
        [{ "node": "Apply Decay", "type": "main", "index": 0 }]
      ]
    },
    "Apply Decay": {
      "main": [
        [
          { "node": "Update Contact", "type": "main", "index": 0 },
          { "node": "Log Event", "type": "main", "index": 0 }
        ]
      ]
    },
    "Update Contact": {
      "main": [
        [{ "node": "Score >= 70?", "type": "main", "index": 0 }]
      ]
    },
    "Score >= 70?": {
      "main": [
        [{ "node": "Telegram - Hot Lead Alert", "type": "main", "index": 0 }],
        []
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1"
  },
  "versionId": "1",
  "tags": [
    { "name": "lead-scoring" },
    { "name": "sales-automation" },
    { "name": "dream-ai" }
  ]
}
```

---

## Lead Scoring Dashboard Queries

### 1. Lead Temperature Summary

```sql
SELECT
    temperature,
    COUNT(*) AS lead_count,
    ROUND(AVG(lead_score), 1) AS avg_score,
    MAX(lead_score) AS max_score,
    MIN(lead_score) AS min_score
FROM contacts
WHERE lead_score > 0
GROUP BY temperature
ORDER BY
    CASE temperature
        WHEN 'hot' THEN 1
        WHEN 'warm' THEN 2
        WHEN 'cold' THEN 3
    END;
```

**Expected Output:**
```
temperature | lead_count | avg_score | max_score | min_score
------------|------------|-----------|-----------|----------
hot         | 23         | 82.4      | 120       | 70
warm        | 89         | 45.2      | 68        | 30
cold        | 347        | 12.8      | 29        | 1
```

---

### 2. Hot Leads — Action Required

```sql
SELECT
    c.email,
    c.name,
    c.vertical,
    c.company,
    c.lead_score,
    c.last_engagement,
    le.event_type AS last_event_type,
    le.created_at AS last_event_time,
    EXTRACT(EPOCH FROM (NOW() - c.last_engagement)) / 3600 AS hours_since_engagement
FROM contacts c
LEFT JOIN lead_events le ON le.contact_id = c.id
    AND le.created_at = (
        SELECT MAX(created_at)
        FROM lead_events
        WHERE contact_id = c.id
    )
WHERE c.temperature = 'hot'
ORDER BY c.lead_score DESC, c.last_engagement DESC;
```

---

### 3. Warm Leads — Most Likely to Convert

```sql
SELECT
    c.email,
    c.name,
    c.vertical,
    c.lead_score,
    c.last_engagement,
    COUNT(le.id) AS total_events,
    ARRAY_AGG(DISTINCT le.event_type) AS event_types
FROM contacts c
JOIN lead_events le ON le.contact_id = c.id
WHERE c.temperature = 'warm'
    AND c.lead_score >= 50  -- top half of warm
GROUP BY c.id
ORDER BY c.lead_score DESC
LIMIT 25;
```

---

### 4. Lead Velocity (Score Changes Over Last 7 Days)

```sql
WITH daily_scores AS (
    SELECT
        email,
        DATE(created_at) AS event_date,
        SUM(points_awarded) AS daily_points
    FROM lead_events
    WHERE created_at >= NOW() - INTERVAL '7 days'
    GROUP BY email, DATE(created_at)
)
SELECT
    email,
    STRING_AGG(event_date::text || ': +' || daily_points, ', ' ORDER BY event_date) AS score_timeline,
    SUM(daily_points) AS total_points_7d
FROM daily_scores
GROUP BY email
ORDER BY total_points_7d DESC
LIMIT 20;
```

---

### 5. Score Distribution Histogram

```sql
SELECT
    CASE
        WHEN lead_score = 0 THEN '0'
        WHEN lead_score BETWEEN 1 AND 10 THEN '1-10'
        WHEN lead_score BETWEEN 11 AND 20 THEN '11-20'
        WHEN lead_score BETWEEN 21 AND 30 THEN '21-30'
        WHEN lead_score BETWEEN 31 AND 50 THEN '31-50'
        WHEN lead_score BETWEEN 51 AND 70 THEN '51-70'
        WHEN lead_score BETWEEN 71 AND 100 THEN '71-100'
        ELSE '100+'
    END AS score_bucket,
    COUNT(*) AS contacts,
    ROUND(COUNT(*)::numeric / SUM(COUNT(*)) OVER () * 100, 1) AS percentage
FROM contacts
GROUP BY score_bucket
ORDER BY MIN(lead_score);
```

---

### 6. Event Type Performance

```sql
SELECT
    event_type,
    COUNT(*) AS total_events,
    COUNT(DISTINCT email) AS unique_contacts,
    AVG(points_awarded) AS avg_points,
    MAX(created_at) AS last_occurrence
FROM lead_events
GROUP BY event_type
ORDER BY total_events DESC;
```

---

### 7. Stale Leads (Needs Re-engagement)

```sql
SELECT
    c.email,
    c.name,
    c.lead_score,
    c.temperature AS old_temperature,
    c.last_engagement,
    EXTRACT(DAY FROM NOW() - c.last_engagement) AS days_inactive,
    ROUND(c.lead_score - (EXTRACT(DAY FROM NOW() - c.last_engagement) / 7.0 * 5), 0) AS projected_score
FROM contacts c
WHERE c.lead_score > 20
    AND c.last_engagement < NOW() - INTERVAL '7 days'
ORDER BY c.last_engagement ASC
LIMIT 20;
```

---

## Environment Variables

Set these in N8N (Settings → Variables) or in `.env`:

```bash
# PostgreSQL
DB_HOST=sjc1.clusters.zeabur.com
DB_PORT=22417
DB_NAME=zeabur
DB_USER=root
DB_PASSWORD=<from .secrets/zeabur-postgres.env>

# Telegram Bot
TELEGRAM_BOT_TOKEN=<your-bot-token>
TELEGRAM_CHAT_ID=686076918

# Webhook base URL (for N8N public URL)
N8N_BASE_URL=https://dreamain8n.zeabur.app
WEBHOOK_PATH=/webhook/lead-event
```

---

## Deployment Checklist

- [ ] **Database tables created** — Run the schema SQL above on your Zeabur PostgreSQL
- [ ] **N8N credentials configured** — PostgreSQL + Telegram Bot
- [ ] **Webhook URL published** — `https://dreamain8n.zeabur.app/webhook/lead-event`
- [ ] **Website event integration** — Add tracking code to fire webhook on each event type
- [ ] **Schedule activated** — Daily 09:00 UTC workflow activated
- [ ] **Telegram bot added** — Bot can message Stephen's chat
- [ ] **Test with sample event** — `curl -X POST https://dreamain8n.zeabur.app/webhook/lead-event -H "Content-Type: application/json" -d '{"email":"test@example.com","event_type":"demo_booked","timestamp":"2026-03-18T22:00:00Z","metadata":{"source":"website"}}'`
- [ ] **Monitor first 24h** — Check N8N execution logs for errors
- [ ] **Dashboard connected** — Optionally connect to a Metabase/Grafana instance for visualization

---

## Website Tracking Snippet

Add this to your website to fire scoring events:

```html
<script>
(function() {
  const LEAD_SCORER = {
    webhookUrl: 'https://dreamain8n.zeabur.app/webhook/lead-event',
    
    track: function(eventType, metadata = {}) {
      const email = this.getEmail();
      if (!email) {
        console.warn('Lead Scorer: No email found. Store email first.');
        return;
      }
      
      fetch(this.webhookUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: email,
          event_type: eventType,
          timestamp: new Date().toISOString(),
          metadata: {
            page_url: window.location.href,
            referrer: document.referrer,
            ...metadata
          }
        })
      }).catch(err => console.error('Lead Scorer error:', err));
    },
    
    getEmail: function() {
      // Check localStorage, cookie, or data attribute
      return localStorage.getItem('lead_email') ||
             document.querySelector('[data-lead-email]')?.dataset?.leadEmail ||
             null;
    },
    
    setEmail: function(email) {
      localStorage.setItem('lead_email', email);
    }
  };
  
  window.LEAD_SCORER = LEAD_SCORER;
  
  // Auto-track page visits
  LEAD_SCORER.track('page_visit', { title: document.title });
})();
</script>

<!-- Usage examples:
  LEAD_SCORER.setEmail('prospect@acme.com');
  LEAD_SCORER.track('calculator_complete', { calculator_type: 'roi', result: '$50k' });
  LEAD_SCORER.track('scorecard_complete', { score: 85, category: 'high-growth' });
  LEAD_SCORER.track('pricing_page', { plan_viewed: 'enterprise' });
  LEAD_SCORER.track('demo_booked', { date: '2026-03-20', time: '14:00' });
-->
```

---

## Customization Notes

| Item | How to Change |
|------|---------------|
| **Score values** | Edit `SCORE_MAP` in Calculate Score node |
| **Temperature thresholds** | Edit in Apply Decay and daily recalculation nodes (currently 70/30) |
| **Decay rate** | Change `5` (points per week) and `7` (days threshold) in decay logic |
| **Alert recipient** | Change `chatId` in Telegram nodes |
| **Schedule time** | Change cron expression `0 9 * * *` in Schedule node |
| **Add new event types** | Add to `SCORE_MAP` + website tracking snippet |
| **CRM integration** | Replace PostgreSQL nodes with HubSpot/GoHighLevel nodes |

---

*Blueprint created 2026-03-18. Import into N8N and configure credentials to activate.*
