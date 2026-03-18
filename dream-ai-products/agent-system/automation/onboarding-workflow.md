# Dream AI — Automated Customer Onboarding Workflow (N8N Blueprint)

> **Version:** 1.0  
> **Created:** 2026-03-18  
> **Owner:** SOVEREIGN / Stephen  
> **N8N Instance:** dreamain8n.zeabur.app  

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [PostgreSQL Tables](#postgresql-tables)
3. [PHASE 1: Immediate Post-Purchase (0–5 min)](#phase-1)
4. [PHASE 2: Onboarding Questionnaire](#phase-2)
5. [PHASE 3: AI Agent Setup (Automated)](#phase-3)
6. [PHASE 4: Client Handoff](#phase-4)
7. [Email Templates](#email-templates)
8. [Telegram Alert Formats](#telegram-alerts)
9. [Full N8N Workflow JSON](#n8n-workflow-json)
10. [Timeline: Day 0 → Day 4](#timeline)

---

## <a id="architecture-overview"></a>Architecture Overview

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│   Stripe     │────▶│  N8N Workflow     │────▶│  HubSpot CRM        │
│   Webhook    │     │  (Orchestrator)   │     │  (Contact/Deal)     │
└─────────────┘     └────────┬─────────┘     └─────────────────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌──────────────┐ ┌───────────┐ ┌────────────────┐
     │ Email System │ │ Telegram  │ │ PostgreSQL     │
     │ (SendGrid)   │ │ Bot API   │ │ (Tracking DB)  │
     └──────────────┘ └───────────┘ └────────────────┘
              │              │              │
              ▼              ▼              ▼
     ┌──────────────┐ ┌───────────┐ ┌────────────────┐
     │ Questionnaire│ │ Slack/    │ │ AI Agent       │
     │ (Typeform)   │ │ Telegram  │ │ Config Gen     │
     └──────────────┘ └───────────┘ └────────────────┘
```

### Trigger Products

| Product | Price | Stripe Price ID | Agent Type |
|---------|-------|-----------------|------------|
| Starter Kit | $297 | `price_starter_kit` | Chat agent |
| Managed Service (Basic) | $597/mo | `price_managed_basic` | Chat + Voice |
| Managed Service (Pro) | $997/mo | `price_managed_pro` | Chat + Voice + Custom |

---

## <a id="postgresql-tables"></a>PostgreSQL Tables

```sql
-- ============================================================
-- TABLE: onboarding_customers
-- Core tracking table for every purchased customer
-- ============================================================
CREATE TABLE onboarding_customers (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    stripe_customer_id  VARCHAR(255) NOT NULL,
    stripe_session_id   VARCHAR(255),
    email               VARCHAR(255) NOT NULL,
    first_name          VARCHAR(100),
    last_name           VARCHAR(100),
    product_name        VARCHAR(100) NOT NULL,          -- 'Starter Kit', 'Managed Basic', 'Managed Pro'
    product_price       INTEGER NOT NULL,                -- in cents
    payment_status      VARCHAR(50) DEFAULT 'succeeded',
    
    -- Phase tracking
    phase               INTEGER DEFAULT 1,              -- 1, 2, 3, 4
    status              VARCHAR(50) DEFAULT 'new',      -- new, questionnaire_sent, questionnaire_completed, 
                                                        -- agent_building, testing, handoff_scheduled, active
    
    -- HubSpot
    hubspot_contact_id  VARCHAR(255),
    hubspot_deal_id     VARCHAR(255),
    
    -- Questionnaire
    questionnaire_sent_at   TIMESTAMPTZ,
    questionnaire_completed_at  TIMESTAMPTZ,
    questionnaire_response_id   VARCHAR(255),
    
    -- Agent config
    agent_config_generated_at  TIMESTAMPTZ,
    agent_id                    VARCHAR(255),
    knowledge_base_id           VARCHAR(255),
    
    -- Handoff
    demo_scheduled_at       TIMESTAMPTZ,
    demo_call_link          VARCHAR(500),
    training_sent_at        TIMESTAMPTZ,
    launched_at             TIMESTAMPTZ,
    
    -- Metadata
    telegram_alert_sent     BOOLEAN DEFAULT FALSE,
    support_channel_id      VARCHAR(255),
    
    created_at          TIMESTAMPTZ DEFAULT NOW(),
    updated_at          TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_onboarding_email ON onboarding_customers(email);
CREATE INDEX idx_onboarding_stripe ON onboarding_customers(stripe_customer_id);
CREATE INDEX idx_onboarding_status ON onboarding_customers(status);
CREATE INDEX idx_onboarding_phase ON onboarding_customers(phase);


-- ============================================================
-- TABLE: questionnaire_responses
-- Stores full questionnaire data per customer
-- ============================================================
CREATE TABLE questionnaire_responses (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id             UUID REFERENCES onboarding_customers(id),
    
    business_name           VARCHAR(255),
    industry                VARCHAR(50),                 -- 'real_estate', 'medspa', 'hvac', 'other'
    phone_number            VARCHAR(50),
    
    goals                   TEXT[],                      -- Array of selected goals
    current_tech_stack      TEXT,
    existing_crm            VARCHAR(100),
    calendar_link           VARCHAR(500),
    
    business_hours          JSONB,                       -- {"monday": {"open": "09:00", "close": "17:00"}, ...}
    timezone                VARCHAR(50),
    
    faq_document_url        VARCHAR(500),
    additional_notes        TEXT,
    
    -- Voice agent preference
    wants_voice_agent       BOOLEAN DEFAULT FALSE,
    voice_agent_phone       VARCHAR(50),
    
    submitted_at            TIMESTAMPTZ DEFAULT NOW()
);


-- ============================================================
-- TABLE: onboarding_tasks
-- Internal task tracking per customer
-- ============================================================
CREATE TABLE onboarding_tasks (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id         UUID REFERENCES onboarding_customers(id),
    
    task_type           VARCHAR(100) NOT NULL,          -- 'generate_agent_config', 'setup_kb', 'configure_voice',
                                                        -- 'create_test_scenarios', 'schedule_demo', 'send_training'
    task_status         VARCHAR(50) DEFAULT 'pending',  -- pending, in_progress, completed, failed
    
    assigned_to         VARCHAR(100),                   -- 'AI_SYSTEM', 'stephen', 'support_agent'
    
    started_at          TIMESTAMPTZ,
    completed_at        TIMESTAMPTZ,
    error_message       TEXT,
    
    metadata            JSONB DEFAULT '{}',
    
    created_at          TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_tasks_customer ON onboarding_tasks(customer_id);
CREATE INDEX idx_tasks_status ON onboarding_tasks(task_status);


-- ============================================================
-- TABLE: agent_configs
-- Generated AI agent configurations
-- ============================================================
CREATE TABLE agent_configs (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id             UUID REFERENCES onboarding_customers(id),
    
    agent_name              VARCHAR(255),
    agent_type              VARCHAR(50),                 -- 'chat', 'voice', 'hybrid'
    
    llm_model               VARCHAR(100) DEFAULT 'gpt-4o',
    system_prompt           TEXT,
    knowledge_base_urls     TEXT[],
    knowledge_base_files    TEXT[],
    
    voice_config            JSONB,                       -- ElevenLabs voice settings (if voice)
    
    greeting_message        TEXT,
    business_hours          JSONB,
    fallback_behavior       TEXT,                        -- What to do when AI can't answer
    
    test_scenarios          JSONB DEFAULT '[]',
    test_results            JSONB DEFAULT '{}',
    
    deployed_at             TIMESTAMPTZ,
    is_active               BOOLEAN DEFAULT FALSE,
    
    created_at              TIMESTAMPTZ DEFAULT NOW(),
    updated_at              TIMESTAMPTZ DEFAULT NOW()
);


-- ============================================================
-- TABLE: onboarding_timeline
-- Audit log of all onboarding events
-- ============================================================
CREATE TABLE onboarding_timeline (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id     UUID REFERENCES onboarding_customers(id),
    
    event_type      VARCHAR(100) NOT NULL,
    event_data      JSONB DEFAULT '{}',
    
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_timeline_customer ON onboarding_timeline(customer_id);
```

---

## <a id="phase-1"></a>PHASE 1: Immediate Post-Purchase (0–5 min)

### N8N Workflow Nodes

#### Node 1: Stripe Webhook Trigger

```json
{
  "name": "Stripe Webhook",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 1,
  "position": [250, 300],
  "webhook": "stripe-payment",
  "httpMethod": "POST",
  "responseMode": "responseNode",
  "options": {}
}
```

**Expected Stripe Payload:**
```json
{
  "type": "checkout.session.completed",
  "data": {
    "object": {
      "id": "cs_live_...",
      "customer": "cus_...",
      "customer_details": {
        "email": "client@example.com",
        "name": "John Smith"
      },
      "metadata": {
        "product_name": "Starter Kit",
        "product_slug": "starter-kit"
      },
      "amount_total": 29700,
      "payment_status": "paid"
    }
  }
}
```

#### Node 2: Validate Webhook & Extract Data (Code Node)

```json
{
  "name": "Parse Payment",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [470, 300],
  "code": "// Verify Stripe signature (in production, use crypto module)\nconst event = $input.first().json;\n\nif (event.type !== 'checkout.session.completed') {\n  return [{ json: { skip: true, reason: 'Not a checkout completion' } }];\n}\n\nconst session = event.data.object;\nconst nameParts = (session.customer_details.name || '').split(' ');\n\nreturn [{\n  json: {\n    stripeCustomerId: session.customer,\n    stripeSessionId: session.id,\n    email: session.customer_details.email,\n    firstName: nameParts[0] || '',\n    lastName: nameParts.slice(1).join(' ') || '',\n    productName: session.metadata.product_name || 'Unknown Product',\n    productSlug: session.metadata.product_slug || 'unknown',\n    amountTotal: session.amount_total,\n    amountFormatted: `$${(session.amount_total / 100).toFixed(2)}`,\n    paymentStatus: session.payment_status,\n    createdAt: new Date().toISOString()\n  }\n}];"
}
```

#### Node 3: Insert into PostgreSQL (Onboarding Record)

```json
{
  "name": "Create Onboarding Record",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [690, 300],
  "operation": "insert",
  "table": "onboarding_customers",
  "columns": "stripe_customer_id, stripe_session_id, email, first_name, last_name, product_name, product_price, payment_status, phase, status",
  "additionalFields": {
    "values": {
      "stripe_customer_id": "={{ $json.stripeCustomerId }}",
      "stripe_session_id": "={{ $json.stripeSessionId }}",
      "email": "={{ $json.email }}",
      "first_name": "={{ $json.firstName }}",
      "last_name": "={{ $json.lastName }}",
      "product_name": "={{ $json.productName }}",
      "product_price": "={{ $json.amountTotal }}",
      "payment_status": "={{ $json.paymentStatus }}",
      "phase": 1,
      "status": "new"
    }
  }
}
```

#### Node 4: Create HubSpot Contact (HubSpot Node)

```json
{
  "name": "Create HubSpot Contact",
  "type": "n8n-nodes-base.hubspot",
  "typeVersion": 2,
  "position": [910, 200],
  "resource": "contact",
  "operation": "create",
  "additionalFields": {
    "email": "={{ $('Parse Payment').item.json.email }}",
    "firstName": "={{ $('Parse Payment').item.json.firstName }}",
    "lastName": "={{ $('Parse Payment').item.json.lastName }}",
    "lifeCycleStage": "customer",
    "leadStatus": "new-customer",
    "properties": [
      {
        "property": "product_purchased",
        "value": "={{ $('Parse Payment').item.json.productName }}"
      },
      {
        "property": "deal_amount",
        "value": "={{ $('Parse Payment').item.json.amountTotal / 100 }}"
      },
      {
        "property": "onboarding_stage",
        "value": "Phase 1 — Welcome Sent"
      }
    ]
  }
}
```

#### Node 5: Create HubSpot Deal

```json
{
  "name": "Create HubSpot Deal",
  "type": "n8n-nodes-base.hubspot",
  "typeVersion": 2,
  "position": [910, 420],
  "resource": "deal",
  "operation": "create",
  "additionalFields": {
    "dealName": "={{ $('Parse Payment').item.json.productName }} — {{ $('Parse Payment').item.json.firstName }} {{ $('Parse Payment').item.json.lastName }}",
    "dealStage": "closedwon",
    "pipelineId": "default",
    "amount": "={{ $('Parse Payment').item.json.amountTotal / 100 }}",
    "closeDate": "={{ new Date().toISOString() }}",
    "properties": [
      {
        "property": "onboarding_phase",
        "value": "1"
      }
    ]
  }
}
```

#### Node 6: Send Welcome Email (SendGrid / Gmail)

```json
{
  "name": "Send Welcome Email",
  "type": "n8n-nodes-base.emailSend",
  "typeVersion": 2,
  "position": [1130, 300],
  "fromEmail": "hello@dream-ai.co",
  "fromName": "Dream AI",
  "toEmail": "={{ $('Parse Payment').item.json.email }}",
  "subject": "=Welcome to Dream AI, {{ $('Parse Payment').item.json.firstName }}! Here's your next step 🚀",
  "emailType": "html",
  "html": "={{ $json.htmlBody }}",
  "options": {}
}
```

> **See [Email Templates](#email-templates) below for full HTML.**

#### Node 7: Alert Stephen on Telegram

```json
{
  "name": "Telegram Alert — New Customer",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1,
  "position": [1130, 500],
  "resource": "message",
  "operation": "send",
  "chatId": "686076918",
  "text": "=🟢 *NEW CUSTOMER — Dream AI*\n\n👤 *Name:* {{ $('Parse Payment').item.json.firstName }} {{ $('Parse Payment').item.json.lastName }}\n📧 *Email:* {{ $('Parse Payment').item.json.email }}\n📦 *Product:* {{ $('Parse Payment').item.json.productName }}\n💰 *Revenue:* {{ $('Parse Payment').item.json.amountFormatted }}\n⏰ *Time:* {{ new Date().toLocaleString('en-US', { timeZone: 'UTC' }) }}\n\n✅ Welcome email sent\n✅ HubSpot contact & deal created\n\nNext: Questionnaire link goes out Day 1.",
  "parseMode": "Markdown"
}
```

**Alert format preview:**
```
🟢 NEW CUSTOMER — Dream AI

👤 Name: John Smith
📧 Email: john@example.com
📦 Product: Starter Kit
💰 Revenue: $297.00
⏰ Time: 3/18/2026, 10:00:00 PM

✅ Welcome email sent
✅ HubSpot contact & deal created

Next: Questionnaire link goes out Day 1.
```

#### Node 8: Respond to Stripe Webhook

```json
{
  "name": "Respond to Stripe",
  "type": "n8n-nodes-base.respondToWebhook",
  "typeVersion": 1,
  "position": [1350, 300],
  "options": {
    "responseCode": 200,
    "responseData": "{ \"received\": true }"
  }
}
```

### Phase 1 Flow Diagram

```
Stripe Webhook ──▶ Parse Payment ──┬──▶ Create Onboarding Record (PostgreSQL)
                                    ├──▶ Create HubSpot Contact
                                    ├──▶ Create HubSpot Deal
                                    ├──▶ Send Welcome Email
                                    └──▶ Telegram Alert ──▶ Respond to Stripe (200 OK)
```

---

## <a id="phase-2"></a>PHASE 2: Onboarding Questionnaire (Day 1)

### Trigger: N8N Cron / Wait Node

**Option A:** Wait node in main workflow (recommended)
**Option B:** Separate workflow triggered by cron (checks `status = 'new'` and `created_at < NOW() - INTERVAL '1 day'`)

#### Node 1: Trigger — Check for Pending Questionnaires

```json
{
  "name": "Check Pending Questionnaires",
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1,
  "position": [250, 300],
  "rule": {
    "interval": [
      {
        "field": "cronExpression",
        "expression": "0 9 * * *"  // 9 AM UTC daily
      }
    ]
  }
}
```

#### Node 2: Query Customers Needing Questionnaire

```json
{
  "name": "Get Pending Customers",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [470, 300],
  "operation": "executeQuery",
  "query": "SELECT id, email, first_name, last_name, product_name FROM onboarding_customers WHERE status = 'new' AND created_at < NOW() - INTERVAL '20 hours' AND created_at > NOW() - INTERVAL '3 days' AND questionnaire_sent_at IS NULL;"
}
```

#### Node 3: Send Questionnaire Email

```json
{
  "name": "Send Questionnaire Email",
  "type": "n8n-nodes-base.emailSend",
  "typeVersion": 2,
  "position": [690, 300],
  "toEmail": "={{ $json.email }}",
  "subject": "={{ $json.first_name }}, let's set up your AI agent — quick questionnaire inside",
  "html": "={{ /* See questionnaire email template below */ }}",
  "fromEmail": "hello@dream-ai.co",
  "fromName": "Dream AI — Onboarding"
}
```

#### Node 4: Update Status

```json
{
  "name": "Mark Questionnaire Sent",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [910, 300],
  "operation": "update",
  "table": "onboarding_customers",
  "updateKey": "id",
  "additionalFields": {
    "values": {
      "status": "questionnaire_sent",
      "questionnaire_sent_at": "={{ new Date().toISOString() }}"
    }
  }
}
```

#### Node 5: Wait for Questionnaire Completion

```json
{
  "name": "Wait for Questionnaire",
  "type": "n8n-nodes-base.wait",
  "typeVersion": 1,
  "position": [1130, 300],
  "resume": "webhook",
  "webhook": {
    "path": "questionnaire-complete"
  }
}
```

### Questionnaire Fields (Typeform / N8N Form)

**Form Configuration:**

| # | Field | Type | Required | Notes |
|---|-------|------|----------|-------|
| 1 | Business Name | Text | ✅ | — |
| 2 | Industry | Dropdown | ✅ | Real Estate, MedSpa, HVAC, Other |
| 3 | Business Phone for AI | Phone | ✅ | Where AI calls route to |
| 4 | Main Goals | Multi-checkbox | ✅ | Lead capture, appointment booking, FAQ handling, follow-ups, reviews, other |
| 5 | Current Tech Stack | Text | ❌ | Freeform |
| 6 | Current CRM | Dropdown | ❌ | HubSpot, Salesforce, GoHighLevel, None, Other |
| 7 | Calendar Link | URL | ✅ | Calendly or Google Calendar link |
| 8 | Business Hours | Schedule picker | ✅ | Per-day open/close times |
| 9 | Business Timezone | Dropdown | ✅ | — |
| 10 | FAQ Document | File Upload | ❌ | PDF, DOCX, TXT |
| 11 | Additional Notes | Textarea | ❌ | Anything else we should know |
| 12 | Voice Agent (Managed only) | Yes/No | ❌ | Do you want a voice phone agent? |

### Webhook Handler — Questionnaire Complete

```json
{
  "name": "Questionnaire Webhook",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 1,
  "position": [250, 600],
  "webhook": "questionnaire-complete",
  "httpMethod": "POST"
}
```

**Code Node — Parse & Store Questionnaire:**

```javascript
const data = $input.first().json.body;

const response = {
  customerEmail: data.email,
  businessName: data.business_name,
  industry: data.industry,                    // 'real_estate' | 'medspa' | 'hvac' | 'other'
  phoneNumber: data.phone_for_ai,
  goals: data.goals,                          // ['lead_capture', 'appointment_booking', ...]
  currentTechStack: data.tech_stack,
  existingCrm: data.crm,
  calendarLink: data.calendar_link,
  businessHours: data.business_hours,         // JSON object
  timezone: data.timezone,
  faqDocumentUrl: data.faq_url,
  additionalNotes: data.notes,
  wantsVoiceAgent: data.wants_voice_agent || false,
  voiceAgentPhone: data.voice_phone
};

return [{ json: response }];
```

#### Node: Store Questionnaire Response

```json
{
  "name": "Store Questionnaire Response",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "operation": "insert",
  "table": "questionnaire_responses",
  "columns": "customer_id, business_name, industry, phone_number, goals, current_tech_stack, existing_crm, calendar_link, business_hours, timezone, faq_document_url, additional_notes, wants_voice_agent, voice_agent_phone"
}
```

#### Node: Update Customer Phase

```json
{
  "name": "Update to Phase 3",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "operation": "update",
  "table": "onboarding_customers",
  "updateKey": "email",
  "additionalFields": {
    "values": {
      "status": "questionnaire_completed",
      "phase": 3,
      "questionnaire_completed_at": "={{ new Date().toISOString() }}"
    }
  }
}
```

---

## <a id="phase-3"></a>PHASE 3: AI Agent Setup (Automated — Day 2-3)

**Trigger:** Webhook from questionnaire completion OR scheduled check for `status = 'questionnaire_completed'`

### Node 1: Generate Agent Configuration

```json
{
  "name": "Generate Agent Config",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [250, 300],
  "code": "// Generates AI agent configuration from questionnaire data\n\nconst customer = $('Get Customer Data').item.json;\nconst questionnaire = $('Get Questionnaire').item.json;\n\n// Industry-specific prompts\nconst industryPrompts = {\n  real_estate: `You are an AI assistant for a real estate business. You help with:\\n- Scheduling property viewings\\n- Answering questions about listings\\n- Qualifying leads\\n- Following up with prospects\\n\nAlways be professional, knowledgeable about the local market, and prompt in responses.`,\n  \n  medspa: `You are an AI assistant for a medical spa. You help with:\\n- Booking treatment appointments\\n- Answering questions about procedures\\n- Providing pre/post-care instructions\\n- Handling pricing inquiries\\n\nAlways be warm, reassuring, and knowledgeable about aesthetic treatments.`,\n  \n  hvac: `You are an AI assistant for an HVAC company. You help with:\\n- Scheduling service appointments\\n- Triage urgent vs. non-urgent issues\\n- Providing basic troubleshooting\\n- Quoting service calls\\n\nAlways be helpful, safety-conscious, and efficient.`,\n  \n  other: `You are an AI assistant for ${questionnaire.business_name}. You help with customer inquiries, scheduling, and general support based on the business's knowledge base. Always be professional and helpful.`\n};\n\n// Build goals context\nconst goalsContext = questionnaire.goals.map(g => `- ${g.replace(/_/g, ' ')}`).join('\\n');\n\nconst systemPrompt = `${industryPrompts[questionnaire.industry] || industryPrompts.other}\n\nBusiness: ${questionnaire.business_name}\nBusiness Hours: ${JSON.stringify(questionnaire.business_hours)}\nTimezone: ${questionnaire.timezone}\n\nYour main goals for this business:\\n${goalsContext}\n\nWhen you don't know an answer, check the knowledge base first. If the answer isn't there, politely say you'll have a team member follow up.`;\n\nconst agentConfig = {\n  agent_name: `${questionnaire.business_name} AI Assistant`,\n  agent_type: questionnaire.wants_voice_agent ? 'hybrid' : 'chat',\n  llm_model: 'gpt-4o',\n  system_prompt: systemPrompt,\n  business_hours: questionnaire.business_hours,\n  timezone: questionnaire.timezone,\n  greeting_message: `Hi! Welcome to ${questionnaire.business_name}. I'm your AI assistant — how can I help you today?`,\n  fallback_behavior: 'Create a support ticket and notify the team',\n  test_scenarios: [\n    {\n      name: 'Basic greeting',\n      input: 'Hi, I need help',\n      expected_contains: ['welcome', 'help', questionnaire.business_name]\n    },\n    {\n      name: 'Appointment request',\n      input: 'I want to book an appointment',\n      expected_contains: ['schedule', 'availability', 'book']\n    },\n    {\n      name: 'After hours inquiry',\n      input: 'Are you open right now?',\n      expected_contains: ['hours', questionnaire.business_name]\n    }\n  ]\n};\n\nreturn [{ json: agentConfig }];"
}
```

### Node 2: Create Knowledge Base

```json
{
  "name": "Setup Knowledge Base",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 3,
  "position": [470, 300],
  "method": "POST",
  "url": "={{ $env.DREAM_AI_API_URL }}/api/knowledge-base",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "body": {
    "name": "={{ $('Generate Agent Config').item.json.agent_name }} — Knowledge Base",
    "sources": [
      {
        "type": "url",
        "value": "={{ $('Get Questionnaire').item.json.faq_document_url }}"
      }
    ],
    "metadata": {
      "customer_id": "={{ $('Get Customer Data').item.json.id }}",
      "business_name": "={{ $('Get Questionnaire').item.json.business_name }}"
    }
  }
}
```

### Node 3: Configure Voice Agent (Conditional)

```json
{
  "name": "Configure Voice Agent",
  "type": "n8n-nodes-base.if",
  "typeVersion": 1,
  "position": [690, 200],
  "conditions": {
    "boolean": [
      {
        "value1": "={{ $('Get Questionnaire').item.json.wants_voice_agent }}",
        "value2": true
      }
    ]
  }
}
```

**Voice Agent Config Node:**
```json
{
  "name": "Setup Voice Agent",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 3,
  "position": [910, 140],
  "method": "POST",
  "url": "={{ $env.DREAM_AI_API_URL }}/api/voice-agent/setup",
  "body": {
    "phone_number": "={{ $('Get Questionnaire').item.json.voice_agent_phone }}",
    "agent_config_id": "={{ $('Store Agent Config').item.json.id }}",
    "voice_provider": "elevenlabs",
    "voice_id": "default_business_voice",
    "greeting": "={{ $('Generate Agent Config').item.json.greeting_message }}",
    "language": "en",
    "speech_speed": 1.0
  }
}
```

### Node 4: Store Agent Config in Database

```json
{
  "name": "Store Agent Config",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [1130, 300],
  "operation": "insert",
  "table": "agent_configs",
  "columns": "customer_id, agent_name, agent_type, llm_model, system_prompt, knowledge_base_urls, business_hours, greeting_message, fallback_behavior, test_scenarios"
}
```

### Node 5: Create Test Scenarios & Run

```json
{
  "name": "Run Test Scenarios",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 3,
  "position": [1350, 300],
  "method": "POST",
  "url": "={{ $env.DREAM_AI_API_URL }}/api/agent/test",
  "body": {
    "agent_id": "={{ $('Store Agent Config').item.json.id }}",
    "scenarios": "={{ $('Generate Agent Config').item.json.test_scenarios }}"
  }
}
```

### Node 6: Send to Internal Testing

```json
{
  "name": "Alert Testing Complete",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1,
  "position": [1570, 300],
  "resource": "message",
  "operation": "send",
  "chatId": "686076918",
  "text": "=🧪 *AI AGENT READY FOR REVIEW*\n\n👤 *Customer:* {{ $('Get Customer Data').item.json.first_name }} {{ $('Get Customer Data').item.json.last_name }}\n🏢 *Business:* {{ $('Get Questionnaire').item.json.business_name }}\n📦 *Product:* {{ $('Get Customer Data').item.json.product_name }}\n\n🤖 *Agent:* {{ $('Generate Agent Config').item.json.agent_name }}\n📱 *Type:* {{ $('Generate Agent Config').item.json.agent_type }}\n\nTest results: {{ $('Run Test Scenarios').item.json.summary || 'Pending review' }}\n\nReview & approve for Phase 4 handoff.",
  "parseMode": "Markdown"
}
```

### Phase 3 Flow Diagram

```
Questionnaire Complete ──▶ Generate Agent Config ──▶ Setup Knowledge Base
                               │                         │
                               ▼                         ▼
                        Voice Agent? (if Yes)    Store Agent Config
                               │                         │
                               └──────▶ Run Tests ──────┘
                                              │
                                              ▼
                                    Telegram Alert (Review)
```

---

## <a id="phase-4"></a>PHASE 4: Client Handoff (Day 4)

### Trigger: Manual approval OR auto-trigger after testing

### Node 1: Schedule Demo Call

```json
{
  "name": "Create Calendar Event",
  "type": "n8n-nodes-base.googleCalendar",
  "typeVersion": 1,
  "position": [250, 300],
  "resource": "event",
  "operation": "create",
  "calendarId": "primary",
  "startDateTime": "={{ /* Find next available slot + 3 days */ }}",
  "endDateTime": "={{ /* startDateTime + 30 min */ }}",
  "summary": "=Dream AI — {{ $('Get Customer Data').item.json.product_name }} Demo: {{ $('Get Customer Data').item.json.first_name }} {{ $('Get Customer Data').item.json.last_name }}",
  "description": "=AI Agent Demo & Training Session\n\nCustomer: {{ $('Get Customer Data').item.json.email }}\nProduct: {{ $('Get Customer Data').item.json.product_name }}\nBusiness: {{ $('Get Questionnaire').item.json.business_name }}\n\nMeeting link: (auto-generated)",
  "attendees": [
    {
      "email": "={{ $('Get Customer Data').item.json.email }}"
    },
    {
      "email": "stephen@dream-ai.co"
    }
  ],
  "conferenceData": {
    "createRequest": {
      "requestId": "={{ $now }}",
      "conferenceSolutionKey": {
        "type": "hangoutsMeet"
      }
    }
  }
}
```

### Node 2: Send Training Materials Email

```json
{
  "name": "Send Training Materials",
  "type": "n8n-nodes-base.emailSend",
  "typeVersion": 2,
  "position": [470, 300],
  "toEmail": "={{ $('Get Customer Data').item.json.email }}",
  "subject": "={{ $('Get Customer Data').item.json.first_name }}, your AI agent is live! Training materials inside 🎉",
  "html": "={{ /* See training email template below */ }}"
}
```

### Node 3: Create Support Channel (Telegram)

```json
{
  "name": "Create Support Channel",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1,
  "position": [690, 200],
  "resource": "chat",
  "operation": "create",
  "name": "=AI Support — {{ $('Get Questionnaire').item.json.business_name }}"
}
```

### Node 4: Update Customer to Active

```json
{
  "name": "Mark Customer Active",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2,
  "position": [910, 300],
  "operation": "update",
  "table": "onboarding_customers",
  "updateKey": "id",
  "additionalFields": {
    "values": {
      "status": "active",
      "phase": 4,
      "demo_scheduled_at": "={{ $('Create Calendar Event').item.json.start.dateTime }}",
      "demo_call_link": "={{ $('Create Calendar Event').item.json.hangoutLink }}",
      "training_sent_at": "={{ new Date().toISOString() }}",
      "launched_at": "={{ new Date().toISOString() }}",
      "support_channel_id": "={{ $('Create Support Channel').item.json.id }}"
    }
  }
}
```

### Node 5: Final Telegram Alert

```json
{
  "name": "Telegram — Launch Complete",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1,
  "position": [1130, 300],
  "resource": "message",
  "operation": "send",
  "chatId": "686076918",
  "text": "=🚀 *ONBOARDING COMPLETE*\n\n👤 *Customer:* {{ $('Get Customer Data').item.json.first_name }} {{ $('Get Customer Data').item.json.last_name }}\n🏢 *Business:* {{ $('Get Questionnaire').item.json.business_name }}\n📦 *Product:* {{ $('Get Customer Data').item.json.product_name }}\n💰 *Revenue:* ${{ $('Get Customer Data').item.json.product_price / 100 }}\n\n✅ AI Agent deployed\n✅ Training materials sent\n✅ Demo call scheduled\n✅ Support channel created\n\nDay 4 — Customer is LIVE!",
  "parseMode": "Markdown"
}
```

---

## <a id="email-templates"></a>Email Templates

### 1. Welcome Email (Phase 1)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body style="margin: 0; padding: 0; background-color: #0a0a0a; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;">
  <table width="100%" cellpadding="0" cellspacing="0" style="max-width: 600px; margin: 0 auto; background-color: #111111; border-radius: 12px; overflow: hidden;">
    
    <!-- Header -->
    <tr>
      <td style="padding: 40px 40px 20px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%); text-align: center;">
        <h1 style="color: #ffffff; font-size: 28px; margin: 0; font-weight: 700;">Welcome to Dream AI</h1>
        <p style="color: #8892b0; font-size: 14px; margin-top: 8px;">Your AI agent journey starts now</p>
      </td>
    </tr>
    
    <!-- Body -->
    <tr>
      <td style="padding: 40px;">
        <p style="color: #e0e0e0; font-size: 16px; line-height: 1.6; margin: 0 0 20px;">
          Hi {{firstName}},
        </p>
        <p style="color: #e0e0e0; font-size: 16px; line-height: 1.6; margin: 0 0 20px;">
          Thank you for choosing Dream AI! Your <strong style="color: #64ffda;">{{productName}}</strong> purchase is confirmed and we're excited to build your custom AI agent.
        </p>
        
        <!-- What Happens Next -->
        <div style="background-color: #1a1a2e; border-left: 4px solid #64ffda; padding: 20px; margin: 24px 0; border-radius: 0 8px 8px 0;">
          <h3 style="color: #64ffda; margin: 0 0 12px; font-size: 16px;">📋 What Happens Next</h3>
          <table width="100%" cellpadding="0" cellspacing="0">
            <tr>
              <td style="padding: 6px 0; color: #ccd6f6; font-size: 14px;">
                <strong style="color: #ffffff;">Day 1</strong> — We send you a quick questionnaire about your business
              </td>
            </tr>
            <tr>
              <td style="padding: 6px 0; color: #ccd6f6; font-size: 14px;">
                <strong style="color: #ffffff;">Day 2-3</strong> — We build and test your custom AI agent
              </td>
            </tr>
            <tr>
              <td style="padding: 6px 0; color: #ccd6f6; font-size: 14px;">
                <strong style="color: #ffffff;">Day 4</strong> — Demo call + training + you're LIVE 🚀
              </td>
            </tr>
          </table>
        </div>
        
        <!-- Starter Materials -->
        <div style="background-color: #1a1a2e; border-radius: 8px; padding: 20px; margin: 24px 0;">
          <h3 style="color: #64ffda; margin: 0 0 12px; font-size: 16px;">📚 Your Starter Materials</h3>
          <p style="color: #ccd6f6; font-size: 14px; margin: 0 0 12px;">While you wait, explore these resources:</p>
          <ul style="color: #ccd6f6; font-size: 14px; padding-left: 20px; margin: 0;">
            <li style="padding: 4px 0;"><a href="https://dream-ai.co/resources/ai-readiness" style="color: #64ffda;">AI Readiness Checklist</a></li>
            <li style="padding: 4px 0;"><a href="https://dream-ai.co/resources/knowledge-base-guide" style="color: #64ffda;">How to Build a Great Knowledge Base</a></li>
            <li style="padding: 4px 0;"><a href="https://dream-ai.co/resources/faq-templates" style="color: #64ffda;">Industry FAQ Templates</a></li>
          </ul>
        </div>
        
        <!-- CTA Button -->
        <table width="100%" cellpadding="0" cellspacing="0" style="margin: 32px 0;">
          <tr>
            <td align="center">
              <a href="https://form.dream-ai.co/onboarding?email={{email}}&ref={{stripeSessionId}}" 
                 style="display: inline-block; background: linear-gradient(135deg, #64ffda, #48c9b0); color: #0a0a0a; font-size: 16px; font-weight: 700; padding: 16px 40px; border-radius: 8px; text-decoration: none;">
                Complete Your Onboarding Questionnaire →
              </a>
            </td>
          </tr>
        </table>
        
        <p style="color: #8892b0; font-size: 13px; line-height: 1.6; margin: 0;">
          The questionnaire takes about 5 minutes and helps us customize your AI agent perfectly for your business.
        </p>
        
        <!-- Footer -->
        <div style="border-top: 1px solid #1a1a2e; padding-top: 24px; margin-top: 32px; text-align: center;">
          <p style="color: #8892b0; font-size: 13px; margin: 0 0 8px;">
            Questions? Reply to this email or reach us at <a href="mailto:support@dream-ai.co" style="color: #64ffda;">support@dream-ai.co</a>
          </p>
          <p style="color: #4a4a5a; font-size: 12px; margin: 0;">
            Dream AI — Intelligent agents for modern businesses
          </p>
        </div>
      </td>
    </tr>
  </table>
</body>
</html>
```

### 2. Questionnaire Reminder Email (Day 1)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
</head>
<body style="margin: 0; padding: 0; background-color: #0a0a0a; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;">
  <table width="100%" cellpadding="0" cellspacing="0" style="max-width: 600px; margin: 0 auto; background-color: #111111; border-radius: 12px; overflow: hidden;">
    <tr>
      <td style="padding: 40px;">
        <h1 style="color: #ffffff; font-size: 24px; margin: 0 0 16px;">Hey {{firstName}}! 👋</h1>
        <p style="color: #e0e0e0; font-size: 16px; line-height: 1.6; margin: 0 0 20px;">
          We're ready to start building your AI agent, but we need a few details about your business first.
        </p>
        <p style="color: #e0e0e0; font-size: 16px; line-height: 1.6; margin: 0 0 20px;">
          The questionnaire takes <strong style="color: #64ffda;">about 5 minutes</strong> — totally painless, we promise.
        </p>
        <table width="100%" cellpadding="0" cellspacing="0" style="margin: 24px 0;">
          <tr>
            <td align="center">
              <a href="https://form.dream-ai.co/onboarding?email={{email}}" 
                 style="display: inline-block; background: linear-gradient(135deg, #64ffda, #48c9b0); color: #0a0a0a; font-size: 16px; font-weight: 700; padding: 16px 40px; border-radius: 8px; text-decoration: none;">
                Start Questionnaire →
              </a>
            </td>
          </tr>
        </table>
        <p style="color: #8892b0; font-size: 13px; margin: 0;">
          Each answer helps us build a smarter, more personalized AI agent for your business.
        </p>
      </td>
    </tr>
  </table>
</body>
</html>
```

### 3. Training Materials Email (Phase 4)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
</head>
<body style="margin: 0; padding: 0; background-color: #0a0a0a; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;">
  <table width="100%" cellpadding="0" cellspacing="0" style="max-width: 600px; margin: 0 auto; background-color: #111111; border-radius: 12px; overflow: hidden;">
    
    <!-- Header -->
    <tr>
      <td style="padding: 40px 40px 20px; background: linear-gradient(135deg, #0d4f3c 0%, #1a6b4a 100%); text-align: center;">
        <h1 style="color: #ffffff; font-size: 28px; margin: 0;">🎉 Your AI Agent Is Live!</h1>
        <p style="color: #a0ffcf; font-size: 14px; margin-top: 8px;">Time to see your new team member in action</p>
      </td>
    </tr>
    
    <!-- Body -->
    <tr>
      <td style="padding: 40px;">
        <p style="color: #e0e0e0; font-size: 16px; line-height: 1.6; margin: 0 0 20px;">
          Hi {{firstName}},
        </p>
        <p style="color: #e0e0e0; font-size: 16px; line-height: 1.6; margin: 0 0 20px;">
          Your custom AI agent for <strong style="color: #64ffda;">{{businessName}}</strong> is now deployed and ready! Here's everything you need to get started.
        </p>
        
        <!-- Demo Call Info -->
        <div style="background-color: #1a1a2e; border-radius: 8px; padding: 20px; margin: 24px 0;">
          <h3 style="color: #64ffda; margin: 0 0 12px; font-size: 16px;">📅 Demo Call Scheduled</h3>
          <p style="color: #ccd6f6; font-size: 14px; margin: 0 0 16px;">
            <strong style="color: #ffffff;">When:</strong> {{demoDate}} at {{demoTime}} ({{timezone}})<br>
            <strong style="color: #ffffff;">Duration:</strong> 30 minutes<br>
            <strong style="color: #ffffff;">What we'll cover:</strong> Live demo, Q&A, optimization tips
          </p>
          <a href="{{demoLink}}" style="display: inline-block; background-color: #64ffda; color: #0a0a0a; font-size: 14px; font-weight: 700; padding: 12px 24px; border-radius: 6px; text-decoration: none;">
            Join Demo Call
          </a>
        </div>
        
        <!-- Training Materials -->
        <div style="background-color: #1a1a2e; border-radius: 8px; padding: 20px; margin: 24px 0;">
          <h3 style="color: #64ffda; margin: 0 0 16px; font-size: 16px;">📚 Training Materials</h3>
          <table width="100%" cellpadding="0" cellspacing="0">
            <tr>
              <td style="padding: 8px 0; border-bottom: 1px solid #2a2a3e;">
                <a href="https://training.dream-ai.co/dashboard-guide" style="color: #64ffda; text-decoration: none; font-size: 14px;">📊 Dashboard Guide — How to monitor your AI agent</a>
              </td>
            </tr>
            <tr>
              <td style="padding: 8px 0; border-bottom: 1px solid #2a2a3e;">
                <a href="https://training.dream-ai.co/knowledge-base-updates" style="color: #64ffda; text-decoration: none; font-size: 14px;">📝 Updating Your Knowledge Base</a>
              </td>
            </tr>
            <tr>
              <td style="padding: 8px 0; border-bottom: 1px solid #2a2a3e;">
                <a href="https://training.dream-ai.co/conversation-review" style="color: #64ffda; text-decoration: none; font-size: 14px;">💬 Reviewing AI Conversations</a>
              </td>
            </tr>
            <tr>
              <td style="padding: 8px 0;">
                <a href="https://training.dream-ai.co/optimization-tips" style="color: #64ffda; text-decoration: none; font-size: 14px;">⚡ Optimization Tips & Best Practices</a>
              </td>
            </tr>
          </table>
        </div>
        
        <!-- Support Channel -->
        <div style="background-color: #1a1a2e; border-left: 4px solid #64ffda; padding: 20px; margin: 24px 0; border-radius: 0 8px 8px 0;">
          <h3 style="color: #64ffda; margin: 0 0 8px; font-size: 16px;">💬 Your Support Channel</h3>
          <p style="color: #ccd6f6; font-size: 14px; margin: 0;">
            We've set up a dedicated support channel for you. Reach us anytime for quick questions, tweaks, or help.
          </p>
        </div>
        
        <!-- Footer -->
        <div style="border-top: 1px solid #1a1a2e; padding-top: 24px; margin-top: 32px; text-align: center;">
          <p style="color: #8892b0; font-size: 13px; margin: 0;">
            Welcome aboard! 🚀<br>
            — The Dream AI Team
          </p>
        </div>
      </td>
    </tr>
  </table>
</body>
</html>
```

---

## <a id="telegram-alerts"></a>Telegram Alert Formats

### Alert 1: New Customer (Phase 1)

```
🟢 NEW CUSTOMER — Dream AI

👤 Name: John Smith
📧 Email: john@example.com
📦 Product: Starter Kit
💰 Revenue: $297.00
⏰ Time: 3/18/2026, 10:00:00 PM UTC

✅ Welcome email sent
✅ HubSpot contact & deal created

Next: Questionnaire link goes out Day 1.
```

### Alert 2: Agent Ready for Review (Phase 3)

```
🧪 AI AGENT READY FOR REVIEW

👤 Customer: John Smith
🏢 Business: Smith Real Estate Group
📦 Product: Starter Kit

🤖 Agent: Smith Real Estate Group AI Assistant
📱 Type: chat

Test Results:
✅ Basic greeting — PASS
✅ Appointment request — PASS  
✅ After hours inquiry — PASS

Review & approve for Phase 4 handoff.
```

### Alert 3: Onboarding Complete (Phase 4)

```
🚀 ONBOARDING COMPLETE

👤 Customer: John Smith
🏢 Business: Smith Real Estate Group
📦 Product: Starter Kit
💰 Revenue: $297.00

✅ AI Agent deployed
✅ Training materials sent
✅ Demo call scheduled: Mar 22, 2:00 PM UTC
✅ Support channel created

Day 4 — Customer is LIVE! 🎉
```

### Alert 4: Questionnaire Completed (Phase 2)

```
📝 QUESTIONNAIRE COMPLETED

👤 Customer: John Smith
🏢 Business: Smith Real Estate Group
🏷️ Industry: Real Estate
🎯 Goals: Lead capture, Appointment booking
📱 Voice Agent: No

Starting AI agent configuration...
```

---

## <a id="n8n-workflow-json"></a>Complete N8N Workflow JSON

```json
{
  "name": "Dream AI — Customer Onboarding",
  "nodes": [
    {
      "parameters": {},
      "id": "webhook-stripe",
      "name": "Stripe Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [250, 300],
      "webhook": "stripe-payment",
      "httpMethod": "POST",
      "responseMode": "responseNode"
    },
    {
      "parameters": {
        "jsCode": "// Extract and validate Stripe payload\nconst event = $input.first().json;\n\nif (event.type !== 'checkout.session.completed') {\n  return [{ json: { skip: true, reason: 'Not a checkout completion' } }];\n}\n\nconst session = event.data.object;\nconst nameParts = (session.customer_details.name || '').split(' ');\n\nreturn [{\n  json: {\n    stripeCustomerId: session.customer,\n    stripeSessionId: session.id,\n    email: session.customer_details.email,\n    firstName: nameParts[0] || '',\n    lastName: nameParts.slice(1).join(' ') || '',\n    productName: session.metadata.product_name || 'Unknown Product',\n    productSlug: session.metadata.product_slug || 'unknown',\n    amountTotal: session.amount_total,\n    amountFormatted: `$${(session.amount_total / 100).toFixed(2)}`,\n    paymentStatus: session.payment_status\n  }\n}];"
      },
      "id": "code-parse-payment",
      "name": "Parse Payment",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [470, 300]
    },
    {
      "parameters": {
        "operation": "insert",
        "table": "onboarding_customers",
        "columns": "stripe_customer_id, stripe_session_id, email, first_name, last_name, product_name, product_price, payment_status, phase, status",
        "options": {}
      },
      "id": "postgres-insert",
      "name": "Create Onboarding Record",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [690, 300]
    },
    {
      "parameters": {
        "resource": "contact",
        "operation": "create",
        "additionalFields": {}
      },
      "id": "hubspot-contact",
      "name": "Create HubSpot Contact",
      "type": "n8n-nodes-base.hubspot",
      "typeVersion": 2,
      "position": [690, 140]
    },
    {
      "parameters": {
        "resource": "deal",
        "operation": "create",
        "additionalFields": {}
      },
      "id": "hubspot-deal",
      "name": "Create HubSpot Deal",
      "type": "n8n-nodes-base.hubspot",
      "typeVersion": 2,
      "position": [690, 460]
    },
    {
      "parameters": {
        "fromEmail": "hello@dream-ai.co",
        "fromName": "Dream AI",
        "toEmail": "={{ $('Parse Payment').item.json.email }}",
        "subject": "=Welcome to Dream AI, {{ $('Parse Payment').item.json.firstName }}! Here's your next step 🚀",
        "html": "={{ /* Full HTML template from above */ }}"
      },
      "id": "email-welcome",
      "name": "Send Welcome Email",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 2,
      "position": [910, 220]
    },
    {
      "parameters": {
        "chatId": "686076918",
        "text": "=🟢 *NEW CUSTOMER — Dream AI*\n\n👤 *Name:* {{ $('Parse Payment').item.json.firstName }} {{ $('Parse Payment').item.json.lastName }}\n📧 *Email:* {{ $('Parse Payment').item.json.email }}\n📦 *Product:* {{ $('Parse Payment').item.json.productName }}\n💰 *Revenue:* {{ $('Parse Payment').item.json.amountFormatted }}\n\n✅ Welcome email sent\n✅ HubSpot contact & deal created\n\nNext: Questionnaire link goes out Day 1.",
        "additionalFields": {
          "parseMode": "Markdown"
        }
      },
      "id": "telegram-alert-new",
      "name": "Telegram Alert — New Customer",
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1,
      "position": [910, 400]
    },
    {
      "parameters": {},
      "id": "respond-stripe",
      "name": "Respond to Stripe",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1,
      "position": [1130, 300]
    }
  ],
  "connections": {
    "Stripe Webhook": {
      "main": [
        [{ "node": "Parse Payment", "type": "main", "index": 0 }]
      ]
    },
    "Parse Payment": {
      "main": [
        [
          { "node": "Create Onboarding Record", "type": "main", "index": 0 },
          { "node": "Create HubSpot Contact", "type": "main", "index": 0 },
          { "node": "Create HubSpot Deal", "type": "main", "index": 0 },
          { "node": "Send Welcome Email", "type": "main", "index": 0 },
          { "node": "Telegram Alert — New Customer", "type": "main", "index": 0 }
        ]
      ]
    }
  },
  "settings": {
    "executionOrder": "v1"
  }
}
```

---

## <a id="timeline"></a>Timeline: Day 0 → Day 4

| Day | Phase | Action | Automated? | Owner |
|-----|-------|--------|-----------|-------|
| **Day 0** | 1 | Stripe payment received | ✅ | System |
| **Day 0** | 1 | PostgreSQL record created | ✅ | System |
| **Day 0** | 1 | HubSpot contact + deal created | ✅ | System |
| **Day 0** | 1 | Welcome email sent | ✅ | System |
| **Day 0** | 1 | Telegram alert to Stephen | ✅ | System |
| | | | | |
| **Day 1** | 2 | Questionnaire reminder email sent | ✅ | System (cron 9 AM UTC) |
| **Day 1** | 2 | Customer fills out questionnaire | ⏳ | Customer |
| **Day 1** | 2 | Questionnaire data stored | ✅ | System |
| **Day 1** | 2 | Telegram alert: questionnaire complete | ✅ | System |
| | | | | |
| **Day 2-3** | 3 | AI agent config generated | ✅ | System |
| **Day 2-3** | 3 | Knowledge base populated | ✅ | System |
| **Day 2-3** | 3 | Voice agent configured (if applicable) | ✅ | System |
| **Day 2-3** | 3 | Test scenarios executed | ✅ | System |
| **Day 2-3** | 3 | Agent ready for review | ⏳ | Stephen (manual review) |
| **Day 2-3** | 3 | Agent approved | ⏳ | Stephen (approval) |
| | | | | |
| **Day 4** | 4 | Demo call scheduled | ✅ | System |
| **Day 4** | 4 | Training materials sent | ✅ | System |
| **Day 4** | 4 | Support channel created | ✅ | System |
| **Day 4** | 4 | Customer marked Active | ✅ | System |
| **Day 4** | 4 | Final launch alert to Stephen | ✅ | System |
| **Day 4** | 4 | Demo call conducted | ⏳ | Stephen |

### Automation Rate: **85% automated, 15% manual touchpoints**

**Manual touchpoints:**
1. Customer fills out questionnaire (Day 1)
2. Stephen reviews agent test results (Day 2-3)
3. Stephen approves agent deployment (Day 2-3)
4. Stephen conducts demo call (Day 4)

---

## Environment Variables Required

```env
# N8N Credentials
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_API_KEY=sk_live_...

# Database
DATABASE_URL=postgresql://root:...@sjc1.clusters.zeabur.com:22417/zeabur

# Email (SendGrid)
SENDGRID_API_KEY=SG...

# HubSpot
HUBSPOT_API_KEY=pat-...

# Telegram
TELEGRAM_BOT_TOKEN=...
TELEGRAM_CHAT_ID=686076918

# Google Calendar
GOOGLE_CALENDAR_CREDENTIALS=...

# Dream AI API (Agent Builder)
DREAM_AI_API_URL=https://api.dream-ai.co
DREAM_AI_API_KEY=...
```

---

## Quick Setup Checklist

- [ ] Import workflow JSON into N8N (dreamain8n.zeabur.app)
- [ ] Create PostgreSQL tables (run schema above)
- [ ] Set up Stripe webhook endpoint → `https://dreamain8n.zeabur.app/webhook/stripe-payment`
- [ ] Configure SendGrid credentials in N8N
- [ ] Configure HubSpot credentials in N8N
- [ ] Configure Telegram bot credentials in N8N
- [ ] Set up Google Calendar OAuth in N8N
- [ ] Create Typeform for questionnaire (or use N8N native form)
- [ ] Set questionnaire webhook URL: `https://dreamain8n.zeabur.app/webhook/questionnaire-complete`
- [ ] Test with Stripe test mode (`sk_test_...`)
- [ ] Enable workflow
- [ ] Test end-to-end with a real purchase

---

*Document generated by SOVEREIGN — 2026-03-18*  
*Review before production deployment. All Stripe Price IDs and API URLs are placeholders — replace with actual values.*
