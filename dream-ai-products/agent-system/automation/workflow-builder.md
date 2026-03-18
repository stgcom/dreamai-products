# Dream AI — AI Workflow Builder

## Natural Language → N8N Workflow System

Convert plain English instructions into deployed N8N automation workflows.

**Example:** "When a lead scores above 70, send me a Telegram alert" → Auto-generated N8N workflow with webhook trigger, scoring condition, and Telegram notification node.

---

## Architecture Overview

```
User Input (NL) → Intent Parser → Template Matcher → Parameter Extractor → N8N Workflow JSON → Deploy/Test
```

### Pipeline

1. **Parse Intent** — Extract trigger, conditions, actions from natural language
2. **Match Template** — Find best-fit workflow template from library
3. **Extract Parameters** — Pull user-specific values (thresholds, channels, timing)
4. **Generate N8N JSON** — Build deployable workflow from template
5. **Test Mode** — Dry-run with sample data before live deployment
6. **Deploy** — Push to N8N instance via API

---

## Workflow Templates Library

### Template 1: Lead Capture → Score → Alert

**Trigger:** Lead form submission (webhook)  
**Conditions:** Lead score threshold  
**Actions:** Score calculation, notification  

```json
{
  "name": "lead-capture-score-alert",
  "version": "1.0.0",
  "trigger": {
    "type": "webhook",
    "endpoint": "/hooks/lead-capture",
    "method": "POST"
  },
  "conditions": [
    {
      "field": "lead_score",
      "operator": "greater_than",
      "value": "{{score_threshold}}",
      "default": 70
    }
  ],
  "actions": [
    {
      "type": "function",
      "name": "calculate-lead-score",
      "config": {
        "inputs": ["email", "company", "role", "engagement_level"],
        "formula": "weighted_score",
        "weights": {
          "role": {"C-suite": 30, "VP": 25, "Director": 20, "Manager": 15, "Individual": 10},
          "company_size": {"enterprise": 25, "mid-market": 20, "smb": 15},
          "engagement": {"demo_request": 30, "pricing_page": 20, "blog_sub": 10}
        }
      }
    },
    {
      "type": "notify",
      "name": "alert-owner",
      "config": {
        "channel": "{{notification_channel}}",
        "default": "telegram",
        "target": "{{notification_target}}",
        "message": "🔥 High-scoring lead: {{lead_name}} ({{company}}) — Score: {{lead_score}}/{{max_score}}"
      }
    }
  ],
  "outputs": [
    {"field": "lead_id", "type": "string"},
    {"field": "lead_score", "type": "number"},
    {"field": "notification_sent", "type": "boolean"}
  ],
  "parameters": {
    "score_threshold": {"type": "number", "required": true, "default": 70, "description": "Minimum score to trigger alert"},
    "notification_channel": {"type": "string", "required": true, "default": "telegram", "options": ["telegram", "slack", "email"]},
    "notification_target": {"type": "string", "required": true, "description": "Chat ID or email address"},
    "scoring_model": {"type": "string", "required": false, "default": "weighted_score", "options": ["weighted_score", "linear", "custom"]}
  }
}
```

**N8N Node Mapping:**
- `Webhook` → Receives form submission
- `Function` → Calculates lead score using weighted model
- `IF` → Checks score against threshold
- `Telegram/SMS/Email` → Sends alert if condition met
- `PostgreSQL Insert` → Logs lead and score for analytics

---

### Template 2: Purchase → Deliver Product → Follow-up

**Trigger:** Purchase webhook (Stripe/Gumroad/etc.)  
**Conditions:** Payment verified  
**Actions:** Product delivery, thank-you sequence  

```json
{
  "name": "purchase-delivery-followup",
  "version": "1.0.0",
  "trigger": {
    "type": "webhook",
    "endpoint": "/hooks/purchase",
    "method": "POST",
    "source": "payment_processor"
  },
  "conditions": [
    {
      "field": "payment_status",
      "operator": "equals",
      "value": "completed"
    },
    {
      "field": "amount",
      "operator": "greater_than",
      "value": 0
    }
  ],
  "actions": [
    {
      "type": "deliver",
      "name": "send-product-access",
      "config": {
        "method": "{{delivery_method}}",
        "default": "email",
        "template": "product-delivery",
        "includes": ["access_link", "login_credentials", "getting_started_guide"]
      }
    },
    {
      "type": "notify",
      "name": "internal-sale-alert",
      "config": {
        "channel": "telegram",
        "message": "💰 New sale: {{product_name}} — ${{amount}} — {{customer_email}}"
      }
    },
    {
      "type": "sequence",
      "name": "followup-drip",
      "config": {
        "sequence": [
          {"delay": "0h", "action": "thank_you", "template": "purchase-thankyou"},
          {"delay": "24h", "action": "check_in", "template": "getting-started"},
          {"delay": "72h", "action": "upsell", "template": "related-product"},
          {"delay": "7d", "action": "review_request", "template": "ask-review"}
        ]
      }
    }
  ],
  "outputs": [
    {"field": "order_id", "type": "string"},
    {"field": "customer_email", "type": "string"},
    {"field": "delivery_status", "type": "string"},
    {"field": "sequence_scheduled", "type": "boolean"}
  ],
  "parameters": {
    "delivery_method": {"type": "string", "default": "email", "options": ["email", "telegram_bot", "gumroad", "custom_webhook"]},
    "followup_enabled": {"type": "boolean", "default": true},
    "upsell_product_id": {"type": "string", "required": false, "description": "Product ID for upsell in follow-up sequence"},
    "review_delay_hours": {"type": "number", "default": 168, "description": "Hours before requesting review (default: 7 days)"}
  }
}
```

**N8N Node Mapping:**
- `Webhook` → Receives payment notification
- `Function` → Validates payment and extracts order details
- `IF` → Confirms payment status
- `HTTP Request` → Delivers product (API call to delivery platform)
- `Send Email` → Customer delivery email
- `Telegram` → Internal sale notification
- `Wait` nodes → Delayed follow-up sequence
- `PostgreSQL` → Order logging

---

### Template 3: Demo Booking → Reminder → Confirm

**Trigger:** Calendly/Cal.com webhook  
**Conditions:** Booking confirmed  
**Actions:** Calendar invite, reminders, confirmation flow  

```json
{
  "name": "demo-booking-reminder-confirm",
  "version": "1.0.0",
  "trigger": {
    "type": "webhook",
    "endpoint": "/hooks/demo-booking",
    "method": "POST",
    "source": "calendly"
  },
  "conditions": [
    {
      "field": "event_type",
      "operator": "equals",
      "value": "demo_call"
    },
    {
      "field": "status",
      "operator": "equals",
      "value": "active"
    }
  ],
  "actions": [
    {
      "type": "enrich",
      "name": "lookup-attendee",
      "config": {
        "sources": ["crm_lookup", "linkedin_enrichment"],
        "fields": ["company", "role", "linkedin_url", "company_size"]
      }
    },
    {
      "type": "notify",
      "name": "internal-prep-alert",
      "config": {
        "channel": "telegram",
        "message": "📅 Demo booked: {{attendee_name}} ({{company}}) — {{date}} {{time}}"
      }
    },
    {
      "type": "sequence",
      "name": "reminder-flow",
      "config": {
        "sequence": [
          {"delay": "-24h", "action": "email_reminder", "template": "demo-24h-reminder"},
          {"delay": "-1h", "action": "sms_reminder", "template": "demo-1h-reminder"},
          {"delay": "0", "action": "confirmation_sms", "template": "demo-starting-now"},
          {"delay": "+15m", "action": "no_show_check", "condition": "attendee_joined"}
        ]
      }
    },
    {
      "type": "post_action",
      "name": "post-demo-followup",
      "config": {
        "condition": "demo_completed",
        "actions": [
          "send_recording",
          "send_proposal_link",
          "log_to_crm"
        ]
      }
    }
  ],
  "outputs": [
    {"field": "booking_id", "type": "string"},
    {"field": "attendee_name", "type": "string"},
    {"field": "demo_datetime", "type": "datetime"},
    {"field": "reminders_sent", "type": "number"},
    {"field": "attended", "type": "boolean"}
  ],
  "parameters": {
    "reminder_times": {"type": "array", "default": ["-24h", "-1h"], "description": "When to send reminders relative to demo time"},
    "include_sms": {"type": "boolean", "default": true},
    "no_show_threshold_minutes": {"type": "number", "default": 15, "description": "Minutes after start to flag as no-show"},
    "enrich_attendee": {"type": "boolean", "default": true, "description": "Look up attendee info before demo"}
  }
}
```

**N8N Node Mapping:**
- `Webhook` → Calendly/Cal.com event
- `Function` → Parse and validate booking data
- `HTTP Request` → CRM lookup for attendee enrichment
- `Telegram` → Internal prep alert
- `Wait` → Countdown to reminder times
- `Send Email` → Reminder and follow-up emails
- `Send SMS` → Same-day reminders
- `PostgreSQL` → Booking and attendance tracking

---

### Template 4: Review Request → Collect → Publish

**Trigger:** Post-purchase or post-service timer  
**Conditions:** Eligible customer, time since delivery  
**Actions:** Review request, collection, social proof publishing  

```json
{
  "name": "review-request-collect-publish",
  "version": "1.0.0",
  "trigger": {
    "type": "schedule",
    "endpoint": null,
    "cron": "0 10 * * *",
    "description": "Daily check for eligible review requests"
  },
  "conditions": [
    {
      "field": "days_since_delivery",
      "operator": "greater_than_or_equal",
      "value": "{{review_delay_days}}",
      "default": 7
    },
    {
      "field": "review_requested",
      "operator": "not_equals",
      "value": true
    },
    {
      "field": "customer_status",
      "operator": "equals",
      "value": "delivered"
    }
  ],
  "actions": [
    {
      "type": "notify",
      "name": "request-review",
      "config": {
        "channel": "email",
        "template": "review-request",
        "personalization": ["first_name", "product_name"],
        "include_direct_link": true
      }
    },
    {
      "type": "wait",
      "name": "wait-for-response",
      "config": {
        "duration": "72h",
        "check_interval": "6h"
      }
    },
    {
      "type": "conditional",
      "name": "handle-response",
      "config": {
        "if_reviewed": {
          "actions": [
            {"type": "publish", "target": ["website", "social", "sales_page"], "template": "social-proof-card"},
            {"type": "notify", "channel": "telegram", "message": "⭐ New review from {{reviewer_name}} — {{rating}}/5"}
          ]
        },
        "if_no_response": {
          "actions": [
            {"type": "followup", "delay": "48h", "template": "review-reminder"},
            {"type": "incentivize", "offer": "discount_code", "value": "10%"}
          ]
        }
      }
    }
  ],
  "outputs": [
    {"field": "review_request_sent", "type": "boolean"},
    {"field": "review_received", "type": "boolean"},
    {"field": "review_rating", "type": "number"},
    {"field": "review_published_to", "type": "array"},
    {"field": "incentive_offered", "type": "boolean"}
  ],
  "parameters": {
    "review_delay_days": {"type": "number", "default": 7, "description": "Days after delivery to request review"},
    "max_reminders": {"type": "number", "default": 2},
    "publish_targets": {"type": "array", "default": ["website"], "options": ["website", "twitter", "linkedin", "sales_page", "gumroad"]},
    "incentivize_unresponsive": {"type": "boolean", "default": true},
    "minimum_rating_to_publish": {"type": "number", "default": 4}
  }
}
```

**N8N Node Mapping:**
- `Schedule Trigger` → Daily cron job
- `PostgreSQL Query` → Find eligible customers
- `IF` → Check conditions met
- `Send Email` → Review request with personalized link
- `Wait` → Response window
- `IF/ELSE` → Handle reviewed vs. no response
- `HTTP Request` → Publish to website/social platforms
- `Telegram` → Internal notification of new review

---

### Template 5: Webinar Registration → Reminder → Recording

**Trigger:** Registration webhook  
**Conditions:** Valid registration  
**Actions:** Confirmation, reminder sequence, post-webinar delivery  

```json
{
  "name": "webinar-registration-reminder-recording",
  "version": "1.0.0",
  "trigger": {
    "type": "webhook",
    "endpoint": "/hooks/webinar-registration",
    "method": "POST"
  },
  "conditions": [
    {
      "field": "email",
      "operator": "is_valid_email"
    },
    {
      "field": "webinar_id",
      "operator": "exists"
    }
  ],
  "actions": [
    {
      "type": "notify",
      "name": "internal-registration",
      "config": {
        "channel": "telegram",
        "message": "🎯 New webinar registration: {{name}} — {{webinar_title}} ({{total_registrants}} total)"
      }
    },
    {
      "type": "sequence",
      "name": "pre-webinar",
      "config": {
        "sequence": [
          {"delay": "0", "action": "confirmation", "template": "webinar-confirmed", "includes": ["calendar_invite", "add_to_wallet"]},
          {"delay": "-24h", "action": "reminder", "template": "webinar-24h"},
          {"delay": "-1h", "action": "reminder", "template": "webinar-1h", "includes": ["join_link"]},
          {"delay": "0", "action": "start_notification", "template": "webinar-live"}
        ]
      }
    },
    {
      "type": "conditional",
      "name": "post-webinar",
      "config": {
        "if_attended": {
          "actions": [
            {"type": "deliver", "template": "webinar-recording", "includes": ["recording_link", "slides", "bonus_offer"]},
            {"type": "sequence", "followup": [
              {"delay": "+1h", "template": "thankyou-attendee"},
              {"delay": "+24h", "template": "replay-reminder"},
              {"delay": "+72h", "template": "special-offer-attendee"}
            ]}
          ]
        },
        "if_registered_no_show": {
          "actions": [
            {"type": "deliver", "template": "webinar-recording-noshow", "includes": ["recording_link", "replay_available_48h"]},
            {"type": "sequence", "followup": [
              {"delay": "+2h", "template": "sorry-you-missed"},
              {"delay": "+48h", "template": "replay-expiring"}
            ]}
          ]
        }
      }
    }
  ],
  "outputs": [
    {"field": "registration_id", "type": "string"},
    {"field": "webinar_id", "type": "string"},
    {"field": "attendee_joined", "type": "boolean"},
    {"field": "watch_duration_minutes", "type": "number"},
    {"field": "recording_sent", "type": "boolean"}
  ],
  "parameters": {
    "webinar_platform": {"type": "string", "default": "zoom", "options": ["zoom", "streamyard", " Riverside", "custom"]},
    "recording_expiry_hours": {"type": "number", "default": 48, "description": "Hours before recording link expires"},
    "include_slides": {"type": "boolean", "default": true},
    "post_webinar_offer": {"type": "boolean", "default": true, "description": "Send special offer to attendees"},
    "track_attendance_duration": {"type": "boolean", "default": true}
  }
}
```

**N8N Node Mapping:**
- `Webhook` → Registration submission
- `Function` → Validate and deduplicate
- `Telegram` → Internal registration count
- `Google Calendar` → Add calendar invite
- `Wait` → Pre-webinar reminder sequence
- `Send Email` → Confirmation and reminders
- `Zoom/Platform API` → Track attendance
- `HTTP Request` → Send recording post-webinar
- `PostgreSQL` → Registration and attendance data

---

## N8N Implementation Guide

### Workflow Generation Process

**Step 1: Parse Natural Language**
```
Input: "When a lead scores above 70, send me a Telegram alert"

Parsed Intent:
{
  "trigger_type": "event",
  "trigger_event": "lead_score_change",
  "condition": {"field": "score", "operator": ">", "value": 70},
  "action": {"type": "notify", "channel": "telegram"}
}
```

**Step 2: Match Template**
```
Best match: "lead-capture-score-alert" (confidence: 0.95)
```

**Step 3: Extract Parameters**
```
score_threshold = 70
notification_channel = telegram
notification_target = (from system config)
```

**Step 4: Generate N8N Workflow**
```json
{
  "name": "Lead Score Alert - Auto Generated",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "lead-score-webhook",
        "responseMode": "onReceived"
      },
      "name": "Lead Score Webhook",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300]
    },
    {
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{$json.lead_score}}",
              "operation": "larger",
              "value2": 70
            }
          ]
        }
      },
      "name": "Score Threshold Check",
      "type": "n8n-nodes-base.if",
      "position": [450, 300]
    },
    {
      "parameters": {
        "chatId": "686076918",
        "text": "🔥 High-scoring lead: {{$json.lead_name}} ({{$json.company}}) — Score: {{$json.lead_score}}"
      },
      "name": "Telegram Alert",
      "type": "n8n-nodes-base.telegram",
      "position": [650, 200]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO leads (name, email, company, score, created_at) VALUES ('{{$json.lead_name}}', '{{$json.email}}', '{{$json.company}}', {{$json.lead_score}}, NOW())"
      },
      "name": "Log Lead",
      "type": "n8n-nodes-base.postgres",
      "position": [650, 400]
    }
  ],
  "connections": {
    "Lead Score Webhook": {"main": [[{"node": "Score Threshold Check", "type": "main", "index": 0}]]},
    "Score Threshold Check": {
      "main": [
        [{"node": "Telegram Alert", "type": "main", "index": 0}],
        [{"node": "Log Lead", "type": "main", "index": 0}]
      ]
    }
  }
}
```

### Customizable Parameters Per Template

| Template | Key Parameters | Defaults |
|----------|---------------|----------|
| Lead Capture | score_threshold, notification_channel, scoring_model | 70, telegram, weighted |
| Purchase | delivery_method, followup_enabled, upsell_product_id | email, true, none |
| Demo Booking | reminder_times, include_sms, enrich_attendee | [-24h,-1h], true, true |
| Review | review_delay_days, publish_targets, min_rating | 7, [website], 4 |
| Webinar | webinar_platform, recording_expiry, post_webinar_offer | zoom, 48h, true |

### Testing Mode

Every workflow deploys with testing enabled by default:

```json
{
  "testing": {
    "enabled": true,
    "mode": "dry_run",
    "sample_data": {
      "lead_name": "Test User",
      "company": "Acme Corp",
      "email": "test@example.com",
      "lead_score": 85
    },
    "notify_on_test": true,
    "test_message_prefix": "[TEST]"
  }
}
```

**Test workflow:**
1. Webhook receives sample data
2. Conditions evaluate against sample
3. Actions execute in dry-run mode (log but don't send)
4. Output shows what would happen
5. User approves to go live

### Version Control

```json
{
  "workflow_id": "wf_lead_capture_v1",
  "version": "1.2.3",
  "changelog": [
    {"version": "1.2.3", "date": "2026-03-18", "change": "Added SMS reminder option to demo workflow"},
    {"version": "1.2.2", "date": "2026-03-15", "change": "Fixed Telegram notification formatting"},
    {"version": "1.0.0", "date": "2026-03-01", "change": "Initial template creation"}
  ],
  "rollback_available": true,
  "active_version": "1.2.3"
}
```

---

## Usage Examples

### Natural Language Inputs → Outputs

| Input | Template | Key Params |
|-------|----------|------------|
| "Alert me when a lead scores above 80 on Telegram" | lead-capture-score-alert | score_threshold=80, channel=telegram |
| "Send the product download link after payment" | purchase-delivery-followup | delivery_method=email |
| "Remind prospects 2 hours before their demo" | demo-booking-reminder-confirm | reminder_times=["-2h"] |
| "Ask for a review 10 days after delivery" | review-request-collect-publish | review_delay_days=10 |
| "Send the webinar replay to no-shows" | webinar-registration-reminder-recording | platform=zoom |

### Multi-Step Example

**Input:** "When someone buys the AI Toolkit, deliver it via email, send a Telegram alert to me, then 3 days later ask them to leave a review. If they give 5 stars, post it to my website."

**Generated Workflow Chain:**
1. Purchase webhook → Validate payment
2. Email delivery → Send product access
3. Telegram alert → "💰 AI Toolkit sold to {{email}}"
4. Wait 3 days
5. Send review request email
6. If review received with 5 stars → Publish to website widget
7. Telegram alert → "⭐ New 5-star review from {{name}}"

---

## Integration with Dream AI Agent System

### Agent-Triggered Workflows

The agent system can trigger workflows via N8N API:

```bash
# SOVEREIGN triggers workflow for high-priority lead
curl -X POST https://dreamain8n.zeabur.app/webhook/trigger-workflow \
  -H "Content-Type: application/json" \
  -d '{
    "workflow": "lead-capture-score-alert",
    "data": {
      "lead_name": "Jane Smith",
      "company": "TechCorp",
      "lead_score": 92,
      "priority": "high"
    }
  }'
```

### Agent-Readable Outputs

After workflow execution, agents can query status:

```json
{
  "workflow_id": "wf_001",
  "status": "completed",
  "duration_ms": 1250,
  "nodes_executed": 5,
  "output": {
    "alert_sent": true,
    "lead_logged": true,
    "next_action": "assign_to_sales_agent"
  }
}
```

---

## Maintenance

- **Template updates:** Push new version via `workflow-builder update <name>`
- **Health checks:** Daily cron validates all active workflows
- **Error alerts:** Failed workflows trigger Telegram notification within 60s
- **Usage logs:** All workflow executions logged to PostgreSQL `workflow_runs` table
