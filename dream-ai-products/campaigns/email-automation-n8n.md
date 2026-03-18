# Dream AI — N8N Workflow Specifications

## Overview

This document specifies the complete N8N workflow implementations for all three Dream AI email sequences.

| Workflow | Trigger | Emails | Database |
|----------|---------|--------|----------|
| Welcome Sequence | Quiz completion / opt-in | 4 | PostgreSQL + HubSpot |
| Nurture Sequence | Product page view | 4 | PostgreSQL + HubSpot |
| Re-engagement Sequence | 30 days inactive | 3 | PostgreSQL + HubSpot |

### Prerequisites

- N8N instance: `dreamain8n.zeabur.app`
- PostgreSQL: `zeabur` database on Zeabur
- HubSpot: API key in N8N credentials
- Email provider: HubSpot Marketing Hub (or SendGrid/Mailgun via HTTP)

---

## Database Schema (PostgreSQL)

### contacts table

```sql
CREATE TABLE contacts (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    business_name VARCHAR(255),
    industry VARCHAR(100),
    phone VARCHAR(20),
    source VARCHAR(50),                    -- 'quiz', 'opt-in', 'product_view', 'manual'
    status VARCHAR(20) DEFAULT 'active',   -- 'active', 'inactive', 'unsubscribed', 'bounced'
    quiz_score INTEGER,
    quiz_results JSONB,
    calculator_data JSONB,
    active_sequence VARCHAR(50),           -- 'welcome', 'nurture', 'reengagement', NULL
    sequence_day INTEGER DEFAULT 0,
    sequence_started_at TIMESTAMP,
    last_email_sent_at TIMESTAMP,
    last_open_at TIMESTAMP,
    last_click_at TIMESTAMP,
    last_site_visit_at TIMESTAMP,
    last_activity_at TIMESTAMP,
    engagement_score INTEGER DEFAULT 0,
    tags JSONB DEFAULT '[]',
    subscribed_at TIMESTAMP DEFAULT NOW(),
    unsubscribed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_contacts_email ON contacts(email);
CREATE INDEX idx_contacts_status ON contacts(status);
CREATE INDEX idx_contacts_active_sequence ON contacts(active_sequence);
CREATE INDEX idx_contacts_last_activity ON contacts(last_activity_at);
```

### email_events table

```sql
CREATE TABLE email_events (
    id SERIAL PRIMARY KEY,
    contact_id INTEGER REFERENCES contacts(id),
    email VARCHAR(255) NOT NULL,
    sequence VARCHAR(50),                   -- 'welcome', 'nurture', 'reengagement'
    email_number INTEGER,                   -- 1, 2, 3, 4
    subject VARCHAR(500),
    event_type VARCHAR(20),                -- 'sent', 'delivered', 'opened', 'clicked', 'bounced', 'unsubscribed', 'replied'
    event_data JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_email_events_contact ON email_events(contact_id);
CREATE INDEX idx_email_events_type ON email_events(event_type);
```

### email_sequences table

```sql
CREATE TABLE email_sequences (
    id SERIAL PRIMARY KEY,
    contact_id INTEGER REFERENCES contacts(id),
    sequence VARCHAR(50) NOT NULL,         -- 'welcome', 'nurture', 'reengagement'
    status VARCHAR(20) DEFAULT 'active',   -- 'active', 'paused', 'completed', 'exited'
    current_email INTEGER DEFAULT 1,
    next_send_at TIMESTAMP,
    started_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP,
    exit_reason VARCHAR(100),              -- 'purchased', 'unsubscribed', 'reengaged', 'completed', 'manual'
    exit_data JSONB
);
```

### unsubscribes table

```sql
CREATE TABLE unsubscribes (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    reason VARCHAR(255),
    source VARCHAR(50),                    -- 'email_link', 'manual', 'complaint'
    sequence VARCHAR(50),
    unsubscribed_at TIMESTAMP DEFAULT NOW()
);
```

---

## Workflow 1: Welcome Sequence

### Workflow JSON (N8N Import)

```json
{
  "name": "Dream AI — Welcome Sequence",
  "nodes": [
    {
      "parameters": {
        "rules": {
          "values": [
            {
              "conditions": {
                "string": [
                  {
                    "value1": "={{$json.source}}",
                    "operation": "equals",
                    "value2": "quiz_completion"
                  }
                ]
              }
            },
            {
              "conditions": {
                "string": [
                  {
                    "value1": "={{$json.source}}",
                    "operation": "equals",
                    "value2": "email_optin"
                  }
                ]
              }
            }
          ]
        }
      },
      "id": "trigger-welcome",
      "name": "Trigger: Quiz/Opt-in",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [250, 300],
      "webhook": "dream-ai-welcome"
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO contacts (email, first_name, last_name, business_name, industry, source, quiz_score, quiz_results, calculator_data, active_sequence, sequence_started_at, last_activity_at) VALUES ($1, $2, $3, $4, $5, 'quiz', $6, $7, $8, 'welcome', NOW(), NOW()) RETURNING id",
        "additionalFields": {}
      },
      "id": "insert-contact",
      "name": "Insert Contact",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [450, 300],
      "credentials": {
        "postgres": {
          "id": "zeabur-postgres",
          "name": "Zeabur PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO email_sequences (contact_id, sequence, status, current_email, next_send_at, started_at) VALUES ($1, 'welcome', 'active', 1, NOW(), NOW()) RETURNING id",
        "additionalFields": {}
      },
      "id": "create-sequence",
      "name": "Create Sequence Record",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [650, 300]
    },
    {
      "parameters": {
        "subject": "Your AI Revenue Report is ready, {{$json.first_name}}",
        "fromEmail": "stephen@dreamai.co",
        "fromName": "Stephen @ Dream AI",
        "toEmail": "={{$json.email}}",
        "text": "={{$json.email1_body}}",
        "html": "={{$json.email1_html}}"
      },
      "id": "send-email-1",
      "name": "Send Email 1: Report Ready",
      "type": "n8n-nodes-base.hubspot",
      "typeVersion": 2,
      "position": [850, 300]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO email_events (contact_id, email, sequence, email_number, subject, event_type) VALUES ($1, $2, 'welcome', 1, $3, 'sent')",
        "additionalFields": {}
      },
      "id": "log-email-1-sent",
      "name": "Log Email 1 Sent",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1050, 300]
    },
    {
      "parameters": {
        "amount": 2,
        "unit": "days"
      },
      "id": "wait-2-days",
      "name": "Wait 2 Days",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [1250, 300]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{$json.active_sequence}}",
              "operation": "equals",
              "value2": "welcome"
            }
          ]
        }
      },
      "id": "check-still-active",
      "name": "Still Active?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [1450, 300]
    },
    {
      "parameters": {
        "subject": "The $6,200 mistake {{$json.business_name}} is making",
        "fromEmail": "stephen@dreamai.co",
        "fromName": "Stephen @ Dream AI",
        "toEmail": "={{$json.email}}",
        "text": "={{$json.email2_body}}",
        "html": "={{$json.email2_html}}"
      },
      "id": "send-email-2",
      "name": "Send Email 2: $6,200 Mistake",
      "type": "n8n-nodes-base.hubspot",
      "typeVersion": 2,
      "position": [1650, 200]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "UPDATE email_sequences SET current_email = 2, next_send_at = NOW() WHERE contact_id = $1 AND sequence = 'welcome'",
        "additionalFields": {}
      },
      "id": "update-seq-2",
      "name": "Update Sequence → Email 2 Sent",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1850, 200]
    },
    {
      "parameters": {
        "amount": 2,
        "unit": "days"
      },
      "id": "wait-2-more-days",
      "name": "Wait 2 More Days",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [2050, 200]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{$json.active_sequence}}",
              "operation": "equals",
              "value2": "welcome"
            }
          ]
        }
      },
      "id": "check-active-3",
      "name": "Still Active?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [2250, 200]
    },
    {
      "parameters": {
        "subject": "Quick question, {{$json.first_name}}",
        "fromEmail": "stephen@dreamai.co",
        "fromName": "Stephen @ Dream AI",
        "toEmail": "={{$json.email}}",
        "text": "={{$json.email3_body}}",
        "html": "={{$json.email3_html}}"
      },
      "id": "send-email-3",
      "name": "Send Email 3: Quick Question",
      "type": "n8n-nodes-base.hubspot",
      "typeVersion": 2,
      "position": [2450, 100]
    },
    {
      "parameters": {
        "amount": 3,
        "unit": "days"
      },
      "id": "wait-3-days",
      "name": "Wait 3 Days",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [2650, 100]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{$json.active_sequence}}",
              "operation": "equals",
              "value2": "welcome"
            }
          ]
        }
      },
      "id": "check-active-4",
      "name": "Still Active?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [2850, 100]
    },
    {
      "parameters": {
        "subject": "Case study: +$45,000/mo from one AI employee",
        "fromEmail": "stephen@dreamai.co",
        "fromName": "Stephen @ Dream AI",
        "toEmail": "={{$json.email}}",
        "text": "={{$json.email4_body}}",
        "html": "={{$json.email4_html}}"
      },
      "id": "send-email-4",
      "name": "Send Email 4: Case Study + Offer",
      "type": "n8n-nodes-base.hubspot",
      "typeVersion": 2,
      "position": [3050, 100]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "UPDATE email_sequences SET current_email = 4, status = 'completed', completed_at = NOW(), exit_reason = 'completed' WHERE contact_id = $1 AND sequence = 'welcome'",
        "additionalFields": {}
      },
      "id": "complete-welcome",
      "name": "Complete Welcome Sequence",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [3250, 100]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "UPDATE contacts SET active_sequence = NULL, sequence_day = 7, last_activity_at = NOW() WHERE id = $1",
        "additionalFields": {}
      },
      "id": "clear-sequence",
      "name": "Clear Active Sequence",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [3450, 100]
    }
  ],
  "connections": {
    "Trigger: Quiz/Opt-in": {
      "main": [
        [{ "node": "Insert Contact", "type": "main", "index": 0 }]
      ]
    },
    "Insert Contact": {
      "main": [
        [{ "node": "Create Sequence Record", "type": "main", "index": 0 }]
      ]
    },
    "Create Sequence Record": {
      "main": [
        [{ "node": "Send Email 1: Report Ready", "type": "main", "index": 0 }]
      ]
    },
    "Send Email 1: Report Ready": {
      "main": [
        [{ "node": "Log Email 1 Sent", "type": "main", "index": 0 }]
      ]
    },
    "Log Email 1 Sent": {
      "main": [
        [{ "node": "Wait 2 Days", "type": "main", "index": 0 }]
      ]
    },
    "Wait 2 Days": {
      "main": [
        [{ "node": "Still Active?", "type": "main", "index": 0 }]
      ]
    },
    "Still Active?": {
      "main": [
        [{ "node": "Send Email 2: $6,200 Mistake", "type": "main", "index": 0 }]
      ]
    },
    "Send Email 2: $6,200 Mistake": {
      "main": [
        [{ "node": "Update Sequence → Email 2 Sent", "type": "main", "index": 0 }]
      ]
    },
    "Update Sequence → Email 2 Sent": {
      "main": [
        [{ "node": "Wait 2 More Days", "type": "main", "index": 0 }]
      ]
    },
    "Wait 2 More Days": {
      "main": [
        [{ "node": "Still Active? (3)", "type": "main", "index": 0 }]
      ]
    },
    "Still Active? (3)": {
      "main": [
        [{ "node": "Send Email 3: Quick Question", "type": "main", "index": 0 }]
      ]
    },
    "Send Email 3: Quick Question": {
      "main": [
        [{ "node": "Wait 3 Days", "type": "main", "index": 0 }]
      ]
    },
    "Wait 3 Days": {
      "main": [
        [{ "node": "Still Active? (4)", "type": "main", "index": 0 }]
      ]
    },
    "Still Active? (4)": {
      "main": [
        [{ "node": "Send Email 4: Case Study + Offer", "type": "main", "index": 0 }]
      ]
    },
    "Send Email 4: Case Study + Offer": {
      "main": [
        [{ "node": "Complete Welcome Sequence", "type": "main", "index": 0 }]
      ]
    },
    "Complete Welcome Sequence": {
      "main": [
        [{ "node": "Clear Active Sequence", "type": "main", "index": 0 }]
      ]
    }
  }
}
```

---

## Workflow 2: Nurture Sequence

### Trigger Node — Product Page Webhook

```json
{
  "parameters": {
    "rules": {
      "values": [
        {
          "conditions": {
            "string": [
              {
                "value1": "={{$json.event_type}}",
                "operation": "equals",
                "value2": "product_page_view"
              }
            ]
          }
        }
      ]
    }
  },
  "id": "trigger-nurture",
  "name": "Trigger: Product Page View",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 1,
  "position": [250, 300],
  "webhook": "dream-ai-product-view"
}
```

### Check Existing Contact Node

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT c.*, es.status as seq_status FROM contacts c LEFT JOIN email_sequences es ON c.id = es.contact_id AND es.sequence = 'nurture' AND es.status = 'active' WHERE c.email = $1",
    "additionalFields": {}
  },
  "id": "check-existing",
  "name": "Check Existing Contact",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [450, 300]
}
```

### IF Node — Has Active Nurture?

```json
{
  "parameters": {
    "conditions": {
      "string": [
        {
          "value1": "={{$json.seq_status}}",
          "operation": "isEmpty"
        }
      ]
    }
  },
  "id": "has-active-nurture",
  "name": "Already in Nurture?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1,
  "position": [650, 300]
}
```

### Update Product View Node

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "UPDATE contacts SET last_site_visit_at = NOW(), last_activity_at = NOW(), active_sequence = 'nurture', sequence_started_at = NOW() WHERE email = $1",
    "additionalFields": {}
  },
  "id": "start-nurture",
  "name": "Start Nurture Sequence",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [850, 200]
}
```

### Nurture Sequence Timeline

```
Day 0:  Email 1 → Wait 3 days
Day 3:  Email 2 → Wait 4 days
Day 7:  Email 3 → Wait 7 days
Day 14: Email 4 → Complete
```

### Full Nurture Workflow Nodes (abbreviated — follows same pattern as Welcome)

```json
{
  "name": "Dream AI — Nurture Sequence",
  "nodes": [
    {
      "name": "Trigger: Product Page View",
      "type": "n8n-nodes-base.webhook",
      "webhook": "dream-ai-product-view"
    },
    {
      "name": "Check Existing Contact",
      "type": "n8n-nodes-base.postgres",
      "query": "SELECT c.*, es.status as seq_status FROM contacts c LEFT JOIN email_sequences es ON c.id = es.contact_id AND es.sequence = 'nurture' AND es.status = 'active' WHERE c.email = $1"
    },
    {
      "name": "Already in Nurture?",
      "type": "n8n-nodes-base.if"
    },
    {
      "name": "Start Nurture Sequence",
      "type": "n8n-nodes-base.postgres",
      "query": "UPDATE contacts SET active_sequence = 'nurture', sequence_started_at = NOW(), last_activity_at = NOW() WHERE email = $1"
    },
    {
      "name": "Create Sequence Record",
      "type": "n8n-nodes-base.postgres",
      "query": "INSERT INTO email_sequences (contact_id, sequence, status, current_email, next_send_at) VALUES ($1, 'nurture', 'active', 1, NOW())"
    },
    {
      "name": "Send Email 1: Still Thinking?",
      "type": "n8n-nodes-base.hubspot"
    },
    {
      "name": "Wait 3 Days",
      "type": "n8n-nodes-base.wait",
      "amount": 3,
      "unit": "days"
    },
    {
      "name": "Check: Purchased?",
      "type": "n8n-nodes-base.if",
      "query": "SELECT status FROM contacts WHERE id = $1 AND status = 'purchased'"
    },
    {
      "name": "Send Email 2: What's Holding You Back?",
      "type": "n8n-nodes-base.hubspot"
    },
    {
      "name": "Wait 4 Days",
      "type": "n8n-nodes-base.wait",
      "amount": 4,
      "unit": "days"
    },
    {
      "name": "Check: Purchased?",
      "type": "n8n-nodes-base.if"
    },
    {
      "name": "Send Email 3: Free Guide",
      "type": "n8n-nodes-base.hubspot"
    },
    {
      "name": "Wait 7 Days",
      "type": "n8n-nodes-base.wait",
      "amount": 7,
      "unit": "days"
    },
    {
      "name": "Check: Purchased?",
      "type": "n8n-nodes-base.if"
    },
    {
      "name": "Send Email 4: 20% Discount",
      "type": "n8n-nodes-base.hubspot"
    },
    {
      "name": "Complete Nurture Sequence",
      "type": "n8n-nodes-base.postgres",
      "query": "UPDATE email_sequences SET status = 'completed', completed_at = NOW() WHERE contact_id = $1 AND sequence = 'nurture'"
    }
  ]
}
```

### IF Node — Purchase Check (before each email)

```json
{
  "parameters": {
    "conditions": {
      "string": [
        {
          "value1": "={{$json.status}}",
          "operation": "notEquals",
          "value2": "purchased"
        }
      ]
    }
  },
  "id": "check-purchased",
  "name": "Not Purchased Yet?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1
}
```

---

## Workflow 3: Re-engagement Sequence

### Cron Trigger Node

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
  "id": "trigger-daily-check",
  "name": "Daily: Check Inactive Contacts",
  "type": "n8n-nodes-base.cron",
  "typeVersion": 1,
  "position": [250, 300]
}
```

### Find Inactive Contacts Node

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT c.* FROM contacts c LEFT JOIN email_sequences es ON c.id = es.contact_id AND es.status = 'active' WHERE c.status = 'active' AND c.last_activity_at < NOW() - INTERVAL '30 days' AND c.last_activity_at > NOW() - INTERVAL '31 days' AND es.id IS NULL AND c.unsubscribed_at IS NULL AND NOT EXISTS (SELECT 1 FROM email_sequences es2 WHERE es2.contact_id = c.id AND es2.sequence = 'reengagement' AND es2.status = 'completed' AND es2.completed_at > NOW() - INTERVAL '180 days')",
    "additionalFields": {}
  },
  "id": "find-inactive",
  "name": "Find Newly Inactive (30 days)",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [450, 300]
}
```

### Split Into Batches Node

```json
{
  "parameters": {
    "batchSize": 100
  },
  "id": "batch-contacts",
  "name": "Batch: 100 Contacts",
  "type": "n8n-nodes-base.splitInBatches",
  "typeVersion": 2,
  "position": [650, 300]
}
```

### Create Re-engagement Sequence Records

```json
{
  "parameters": {
    "operation": "executeQuery",
    "query": "INSERT INTO email_sequences (contact_id, sequence, status, current_email, next_send_at, started_at) VALUES ($1, 'reengagement', 'active', 1, NOW(), NOW()) ON CONFLICT DO NOTHING",
    "additionalFields": {}
  },
  "id": "create-reengagement-seq",
  "name": "Start Re-engagement Sequence",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [850, 300]
}
```

### Re-engagement Sequence Timeline

```
Day 0 (when detected):  Email 1 → Wait 15 days
Day 15:                 Email 2 → Wait 15 days
Day 30:                 Email 3 → Wait 15 days (grace period)
Day 45:                 Move to inactive
```

### Re-engagement Workflow JSON (abbreviated)

```json
{
  "name": "Dream AI — Re-engagement Sequence",
  "nodes": [
    {
      "name": "Daily: Check Inactive Contacts",
      "type": "n8n-nodes-base.cron",
      "cronExpression": "0 9 * * *"
    },
    {
      "name": "Find Newly Inactive (30 days)",
      "type": "n8n-nodes-base.postgres",
      "query": "SELECT c.* FROM contacts c WHERE c.status = 'active' AND c.last_activity_at < NOW() - INTERVAL '30 days' AND c.last_activity_at >= NOW() - INTERVAL '31 days' AND c.active_sequence IS NULL AND c.unsubscribed_at IS NULL"
    },
    {
      "name": "Start Re-engagement Sequence",
      "type": "n8n-nodes-base.postgres",
      "query": "INSERT INTO email_sequences (contact_id, sequence, status, current_email, next_send_at) VALUES ($1, 'reengagement', 'active', 1, NOW())"
    },
    {
      "name": "Send Email 1: We Miss You",
      "type": "n8n-nodes-base.hubspot"
    },
    {
      "name": "Wait 15 Days",
      "type": "n8n-nodes-base.wait",
      "amount": 15,
      "unit": "days"
    },
    {
      "name": "Check: Re-engaged?",
      "type": "n8n-nodes-base.if",
      "query": "SELECT last_open_at, last_click_at FROM contacts WHERE id = $1 AND last_activity_at > $sequence_started_at"
    },
    {
      "name": "Send Email 2: Is This Goodbye?",
      "type": "n8n-nodes-base.hubspot"
    },
    {
      "name": "Wait 15 Days",
      "type": "n8n-nodes-base.wait",
      "amount": 15,
      "unit": "days"
    },
    {
      "name": "Check: Re-engaged?",
      "type": "n8n-nodes-base.if"
    },
    {
      "name": "Send Email 3: Final Gift",
      "type": "n8n-nodes-base.hubspot"
    },
    {
      "name": "Wait 15 Days (Grace)",
      "type": "n8n-nodes-base.wait",
      "amount": 15,
      "unit": "days"
    },
    {
      "name": "Final Check: Re-engaged?",
      "type": "n8n-nodes-base.if"
    },
    {
      "name": "Move to Inactive",
      "type": "n8n-nodes-base.postgres",
      "queries": [
        "UPDATE contacts SET status = 'inactive', active_sequence = NULL WHERE id = $1",
        "INSERT INTO inactive_contacts SELECT * FROM contacts WHERE id = $1",
        "UPDATE email_sequences SET status = 'completed', exit_reason = 'moved_to_inactive' WHERE contact_id = $1 AND sequence = 'reengagement'"
      ]
    }
  ]
}
```

---

## Shared: Unsubscribe Handling Workflow

### Webhook Trigger

```json
{
  "parameters": {
    "rules": {
      "values": [
        {
          "conditions": {
            "string": [
              {
                "value1": "={{$json.event_type}}",
                "operation": "equals",
                "value2": "unsubscribe"
              }
            ]
          }
        }
      ]
    }
  },
  "id": "trigger-unsubscribe",
  "name": "Trigger: Unsubscribe",
  "type": "n8n-nodes-base.webhook",
  "webhook": "dream-ai-unsubscribe"
}
```

### Full Unsubscribe Flow

```json
{
  "name": "Dream AI — Unsubscribe Handler",
  "nodes": [
    {
      "name": "Trigger: Unsubscribe",
      "type": "n8n-nodes-base.webhook",
      "webhook": "dream-ai-unsubscribe"
    },
    {
      "name": "Log Unsubscribe",
      "type": "n8n-nodes-base.postgres",
      "query": "INSERT INTO unsubscribes (email, reason, source, sequence) VALUES ($1, $2, 'email_link', (SELECT active_sequence FROM contacts WHERE email = $1))"
    },
    {
      "name": "Update Contact Status",
      "type": "n8n-nodes-base.postgres",
      "query": "UPDATE contacts SET status = 'unsubscribed', active_sequence = NULL, unsubscribed_at = NOW(), updated_at = NOW() WHERE email = $1"
    },
    {
      "name": "Cancel Active Sequences",
      "type": "n8n-nodes-base.postgres",
      "query": "UPDATE email_sequences SET status = 'exited', exit_reason = 'unsubscribed', exit_data = '{\"reason\": $2}', completed_at = NOW() WHERE contact_id = (SELECT id FROM contacts WHERE email = $1) AND status = 'active'"
    },
    {
      "name": "Remove from HubSpot Lists",
      "type": "n8n-nodes-base.hubspot",
      "operation": "removeFromList"
    },
    {
      "name": "Send Internal Alert",
      "type": "n8n-nodes-base.discord",
      "channel": "alerts",
      "message": "🚫 Unsubscribed: {{$json.email}} from sequence: {{$json.sequence}} | Reason: {{$json.reason}}"
    },
    {
      "name": "Return Confirmation",
      "type": "n8n-nodes-base.respondToWebhook",
      "respondWith": "json",
      "responseBody": "={{JSON.stringify({success: true, message: 'You have been unsubscribed. You will no longer receive emails from Dream AI.'})}}"
    }
  ]
}
```

---

## Shared: Webhook URLs & Endpoints

### Inbound Webhooks (receive data INTO N8N)

| Webhook URL | Purpose | Source |
|-------------|---------|--------|
| `https://dreamain8n.zeabur.app/webhook/dream-ai-welcome` | Quiz completion / opt-in | Scorecard landing page |
| `https://dreamain8n.zeabur.app/webhook/dream-ai-product-view` | Product page view | Website tracker |
| `https://dreamain8n.zeabur.app/webhook/dream-ai-unsubscribe` | Unsubscribe request | Email footer link |
| `https://dreamain8n.zeabur.app/webhook/dream-ai-reply` | Email reply received | Email webhook |

### Outbound Actions (N8N sends data out)

| Action | Target | Method |
|--------|--------|--------|
| Send email | HubSpot Marketing | HubSpot node |
| Update contact | PostgreSQL | Postgres node |
| Log event | PostgreSQL | Postgres node |
| Alert on unsubscribe | Discord | Discord node |
| Alert on re-engagement | Discord | Discord node |

---

## Shared: Engagement Webhook (HubSpot → N8N)

### Email Open Webhook

```json
{
  "parameters": {
    "conditions": {
      "string": [
        {
          "value1": "={{$json.event_type}}",
          "operation": "equals",
          "value2": "email_open"
        }
      ]
    }
  },
  "id": "trigger-open",
  "name": "Trigger: Email Opened",
  "type": "n8n-nodes-base.webhook",
  "webhook": "dream-ai-email-open"
}
```

### Update Last Activity on Open

```json
{
  "name": "Update Last Open",
  "type": "n8n-nodes-base.postgres",
  "query": "UPDATE contacts SET last_open_at = NOW(), last_activity_at = NOW(), engagement_score = engagement_score + 1 WHERE email = $1"
}
```

### Email Click Webhook

```json
{
  "name": "Update Last Click",
  "type": "n8n-nodes-base.postgres",
  "query": "UPDATE contacts SET last_click_at = NOW(), last_activity_at = NOW(), engagement_score = engagement_score + 3 WHERE email = $1"
}
```

### Reply Webhook

```json
{
  "name": "Handle Reply",
  "type": "n8n-nodes-base.postgres",
  "query": "UPDATE contacts SET last_activity_at = NOW(), engagement_score = engagement_score + 5 WHERE email = $1"
}
```

---

## Shared: IF Node Patterns

### Pattern 1: Check Active Sequence

```json
{
  "parameters": {
    "conditions": {
      "string": [
        {
          "value1": "={{$json.active_sequence}}",
          "operation": "equals",
          "value2": "welcome"
        }
      ]
    }
  },
  "id": "check-welcome-active",
  "name": "Still in Welcome Sequence?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1
}
```

### Pattern 2: Check Purchase Status

```json
{
  "parameters": {
    "conditions": {
      "string": [
        {
          "value1": "={{$json.status}}",
          "operation": "notEquals",
          "value2": "purchased"
        }
      ]
    }
  },
  "id": "check-not-purchased",
  "name": "Not Purchased?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1
}
```

### Pattern 3: Check Email Opened

```json
{
  "parameters": {
    "conditions": {
      "dateTime": [
        {
          "value1": "={{$json.last_open_at}}",
          "operation": "after",
          "value2": "={{$json.sequence_started_at}}"
        }
      ]
    }
  },
  "id": "check-email-opened",
  "name": "Email Was Opened?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1
}
```

### Pattern 4: Check Unsubscribed

```json
{
  "parameters": {
    "conditions": {
      "string": [
        {
          "value1": "={{$json.status}}",
          "operation": "notEquals",
          "value2": "unsubscribed"
        }
      ]
    }
  },
  "id": "check-not-unsubscribed",
  "name": "Not Unsubscribed?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1
}
```

---

## Implementation Checklist

### Phase 1: Database Setup
- [ ] Run CREATE TABLE scripts for all 4 tables
- [ ] Create indexes
- [ ] Verify connection from N8N

### Phase 2: Workflow Creation
- [ ] Import Welcome Sequence workflow
- [ ] Import Nurture Sequence workflow
- [ ] Import Re-engagement Sequence workflow
- [ ] Import Unsubscribe Handler workflow
- [ ] Configure cron for re-engagement daily check

### Phase 3: Webhook Configuration
- [ ] Set up HubSpot email event webhooks (open, click, bounce)
- [ ] Configure website tracker script for product page views
- [ ] Set up unsubscribe link in email templates
- [ ] Test all webhook endpoints

### Phase 4: Email Templates
- [ ] Create all 4 Welcome email templates in HubSpot
- [ ] Create all 4 Nurture email templates in HubSpot
- [ ] Create all 3 Re-engagement email templates in HubSpot
- [ ] Add personalization tokens to all templates
- [ ] Test rendering across email clients

### Phase 5: Testing
- [ ] Test complete Welcome flow with test contact
- [ ] Test Nurture trigger with simulated product view
- [ ] Test Re-engagement cron detection
- [ ] Test unsubscribe flow end-to-end
- [ ] Test engagement tracking (open/click webhooks)
- [ ] Verify all IF node branches (purchased, unsubscribed, inactive)

### Phase 6: Monitoring
- [ ] Set up Discord alerts for unsubscribes
- [ ] Set up Discord alerts for sequence completions
- [ ] Set up Discord alerts for re-engagements
- [ ] Create dashboard query for sequence performance
