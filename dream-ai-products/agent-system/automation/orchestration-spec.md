# Dream AI — Multi-Agent Orchestration Specification

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    INCOMING REQUEST                       │
│         (Webchat, Telegram, Email, Webhook)              │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│              SOVEREIGN — Orchestrator Agent               │
│                                                          │
│  ┌────────────┐  ┌────────────┐  ┌───────────────────┐ │
│  │  Classify   │  │   Route    │  │  Track Completion  │ │
│  │  Request    │→ │  to Agent  │  │  & Escalate        │ │
│  └────────────┘  └────────────┘  └───────────────────┘ │
└──────────┬──────────┬──────────┬──────────┬─────────────┘
           │          │          │          │
           ▼          ▼          ▼          ▼
     ┌──────────┐┌──────────┐┌──────────┐┌──────────┐
     │  SALES   ││ SUPPORT  ││ CONTENT  ││ ANALYTICS│
     │  Agent   ││  Agent   ││  Agent   ││  Agent   │
     │    📈    ││    🛟    ││    ✍️    ││    📊    │
     └──────────┘└──────────┘└──────────┘└──────────┘
           │          │          │          │
           ▼          ▼          ▼          ▼
┌─────────────────────────────────────────────────────────┐
│            N8N WORKFLOWS + POSTGRESQL STATE              │
│                                                          │
│    ┌──────────┐  ┌──────────┐  ┌──────────────────┐    │
│    │ Webhooks │  │  State   │  │  Retry / Alert   │    │
│    │ (Comms)  │  │  Store   │  │  (Failure Handl.) │    │
│    └──────────┘  └──────────┘  └──────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

---

## 1. Orchestrator Agent — SOVEREIGN

### Role
Central command router. Every incoming request hits SOVEREIGN first. It classifies, routes, tracks, and escalates.

### Core Responsibilities

| Responsibility | Description | Success Metric |
|---------------|-------------|----------------|
| **Classification** | Determine request type, urgency, value, risk | >95% correct routing |
| **Routing** | Send to correct specialist agent | <2s routing decision |
| **Tracking** | Monitor task completion across all agents | 100% task state visibility |
| **Escalation** | Escalate failures to human (Stephen) | <5min escalation time |

### Classification Schema

Every request is tagged with:

```json
{
  "task_id": "task_<uuid>",
  "classification": {
    "type": "quick_answer|research|sales|proposal|delivery|content|engineering|analytics|success|escalation",
    "urgency": "low|medium|high|critical",
    "business_value": "low|medium|high|revenue_critical",
    "risk": "low|medium|high",
    "reversibility": "reversible|semi|hard|irreversible"
  },
  "created_at": "2026-03-18T21:43:00Z",
  "source": "webchat|telegram|email|webhook"
}
```

### Routing Decision Matrix

| Classification | Urgency | Route To | Notification |
|---------------|---------|----------|-------------|
| sales/prospecting | any | **SALES Agent** | Telegram on qualified lead |
| client delivery | any | **SALES Agent** (or assigned specialist) | Telegram on completion |
| support/question | low | **SUPPORT Agent** | No notification unless blocked |
| support/question | high | **SUPPORT Agent** + Telegram Stephen | Immediate escalation path |
| content/brand | any | **CONTENT Agent** | Telegram on publish-ready |
| analytics/report | any | **ANALYTICS Agent** | Telegram on report ready |
| engineering/auto | any | Route to **BOLT** (specialist) | Telegram on deploy |
| critical/escalation | critical | **Direct to Stephen** | Telegram immediately |
| quick_answer | any | **SOVEREIGN answers directly** | None |

### Tracking System

```json
{
  "task_id": "task_a1b2c3",
  "status": "in_progress",
  "assigned_to": "sales_agent",
  "state_history": [
    {"state": "received", "at": "21:43:00", "by": "sovereign"},
    {"state": "classified", "at": "21:43:01", "by": "sovereign"},
    {"state": "routed", "at": "21:43:02", "by": "sovereign", "to": "sales_agent"},
    {"state": "in_progress", "at": "21:43:05", "by": "sales_agent"}
  ],
  "timeout_seconds": 300,
  "escalation_path": "stephen_telegram",
  "retry_count": 0
}
```

---

## 2. Specialist Agents

### 2.1 Sales Agent — Lead Qualification & Demo Booking

**Scope:** Inbound lead processing, qualification, demo scheduling, follow-up

**Capabilities:**
- Qualify leads using BANT/MEDDPICC frameworks
- Book demos via Calendly/Cal.com integration
- Send proposals and pricing
- Nurture sequences for warm leads
- Update CRM (GoHighLevel / Notion)

**Input Format:**
```json
{
  "task_type": "lead_qualification",
  "lead_data": {
    "name": "Jane Smith",
    "email": "jane@techcorp.com",
    "company": "TechCorp",
    "source": "website_form",
    "message": "Interested in AI automation for our support team"
  },
  "priority": "high"
}
```

**Output Format:**
```json
{
  "task_id": "task_a1b2c3",
  "agent": "sales_agent",
  "result": {
    "qualification_score": 85,
    "lead_grade": "A",
    "bant": {
      "budget": "confirmed $10k-50k",
      "authority": "VP Operations",
      "need": "support automation, 500+ tickets/day",
      "timeline": "Q2 2026"
    },
    "next_action": "demo_booked",
    "demo_datetime": "2026-03-20T14:00:00Z",
    "crm_updated": true
  }
}
```

**Workflow Integration:**
- Triggers `demo-booking-reminder-confirm` workflow on demo booking
- Triggers `lead-capture-score-alert` workflow on high-scoring lead
- Logs all interactions to PostgreSQL

---

### 2.2 Support Agent — Questions & Troubleshooting

**Scope:** Customer support, product questions, technical troubleshooting, FAQ

**Capabilities:**
- Answer product/service questions from knowledge base
- Troubleshoot technical issues (N8N, integrations, API)
- Escalate complex issues to Stephen
- Create help documentation from repeated questions
- Track satisfaction scores

**Input Format:**
```json
{
  "task_type": "support_query",
  "customer": {
    "id": "cust_001",
    "name": "John Doe",
    "email": "john@startup.io",
    "tier": "pro"
  },
  "query": "My N8N workflow isn't triggering when leads come in from Typeform",
  "channel": "telegram"
}
```

**Output Format:**
```json
{
  "task_id": "task_d4e5f6",
  "agent": "support_agent",
  "result": {
    "category": "integration_issue",
    "resolution": "step_by_step",
    "steps": [
      "Check Typeform webhook URL matches N8N endpoint",
      "Verify Typeform submission payload includes required fields",
      "Test with manual Typeform submission"
    ],
    "resolved": false,
    "escalated_to": "stephen",
    "escalation_reason": "Requires access to customer's N8N instance"
  }
}
```

**Escalation Rules:**
- Auto-escalate if: requires customer account access, error persists after 3 attempts, customer tier = enterprise
- Always escalate if customer directly requests Stephen

---

### 2.3 Content Agent — Posts, Emails, Copy

**Scope:** Social media posts, email sequences, landing page copy, blog content

**Capabilities:**
- Generate platform-specific content (Twitter, LinkedIn, email)
- Create email sequences (welcome, nurture, re-engagement)
- Write landing page copy and headlines
- Repurpose long-form content into social snippets
- A/B test variations

**Input Format:**
```json
{
  "task_type": "content_generation",
  "brief": {
    "type": "linkedin_post",
    "topic": "AI automation saving time for small businesses",
    "tone": "professional_conversational",
    "length": "medium",
    "cta": "link to AI Toolkit product",
    "brand_guidelines": "use 'we' not 'I', include data points"
  }
}
```

**Output Format:**
```json
{
  "task_id": "task_g7h8i9",
  "agent": "content_agent",
  "result": {
    "variations": [
      {
        "version": "A",
        "headline": "We just saved 47 hours/month with one AI workflow",
        "body": "Most small businesses are drowning in repetitive tasks...",
        "cta": "Steal our AI automation playbook → [link]",
        "char_count": 847
      },
      {
        "version": "B",
        "headline": "The $2M mistake small businesses keep making",
        "body": "Here's what we found when we audited 50 small business workflows...",
        "cta": "See the full breakdown → [link]",
        "char_count": 902
      }
    ],
    "recommended": "A",
    "ready_to_publish": false,
    "awaiting_approval": true
  }
}
```

**Approval Rules:**
- Always require approval for public-facing content
- Auto-publish only for approved content types (configurable)
- Never publish without explicit "yes, publish" from Stephen

---

### 2.4 Analytics Agent — Metrics & Reporting

**Scope:** Performance tracking, dashboard updates, automated reports, anomaly detection

**Capabilities:**
- Query PostgreSQL for business metrics
- Generate daily/weekly/monthly reports
- Track pipeline, revenue, conversion rates
- Anomaly detection (traffic drops, conversion changes)
- Competitor monitoring summaries

**Input Format:**
```json
{
  "task_type": "analytics_report",
  "report_config": {
    "type": "weekly_summary",
    "metrics": ["revenue", "leads", "conversion_rate", "avg_deal_size"],
    "period": "last_7_days",
    "compare_to": "previous_period"
  }
}
```

**Output Format:**
```json
{
  "task_id": "task_j1k2l3",
  "agent": "analytics_agent",
  "result": {
    "period": "2026-03-11 to 2026-03-18",
    "metrics": {
      "revenue": {"current": 12400, "previous": 8900, "change": "+39.3%"},
      "leads": {"current": 47, "previous": 32, "change": "+46.9%"},
      "conversion_rate": {"current": 6.4, "previous": 5.1, "change": "+1.3pp"},
      "avg_deal_size": {"current": 2640, "previous": 2780, "change": "-5.0%"}
    },
    "insights": [
      "Lead volume up significantly — likely from LinkedIn content push",
      "Deal size slightly down — investigate if discounting started",
      "Conversion rate improvement suggests qualification tightening"
    ],
    "anomalies": [],
    "report_sent_to": "stephen_telegram"
  }
}
```

---

## 3. Communication Protocol

### Inter-Agent Message Format

All agent-to-agent communication uses a standardized format over N8N webhooks:

```json
{
  "handoff": {
    "task_id": "task_a1b2c3",
    "from_agent": "sales_agent",
    "to_agent": "content_agent",
    "context": {
      "customer_id": "cust_001",
      "summary": "New customer onboarding — needs welcome email sequence",
      "required_data": {
        "customer_name": "Jane Smith",
        "tier": "pro",
        "onboarding_date": "2026-03-20"
      },
      "linked_entities": {
        "deal_id": "deal_xyz",
        "workflow_id": "wf_onboarding_001"
      }
    },
    "priority": "medium",
    "deadline": "2026-03-20T12:00:00Z",
    "expected_output": "3-email welcome sequence, ready for approval"
  }
}
```

### N8N Webhook Endpoints

| Endpoint | Purpose | Agents |
|----------|---------|--------|
| `/hooks/agent/handoff` | Agent-to-agent task transfer | All → All |
| `/hooks/agent/status` | Task status update | All → SOVEREIGN |
| `/hooks/agent/escalate` | Escalate to human | All → Stephen |
| `/hooks/agent/notify` | Cross-channel notification | All → Telegram/Email |
| `/hooks/agent/dlq` | Dead letter queue (failed tasks) | System → SOVEREIGN |

### State Machine

Every task follows this state machine:

```
                    ┌──────────┐
                    │ RECEIVED │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │CLASSIFIED│
                    └────┬─────┘
                         │
                    ┌────▼─────┐
              ┌─────│  ROUTED  │─────┐
              │     └──────────┘     │
              │                      │
         ┌────▼─────┐         ┌─────▼────┐
         │ IN_PROGRESS│        │ BLOCKED  │
         └────┬─────┘         └────┬─────┘
              │                     │
         ┌────▼─────┐        ┌─────▼────┐
    ┌────│COMPLETED │        │ RETRYING │───┐
    │    └──────────┘        └──────────┘   │
    │                                       │
┌───▼──────┐                         ┌─────▼────┐
│ DELIVERED │                        │ FAILED   │
└───────────┘                        └────┬─────┘
                                          │
                                    ┌─────▼─────┐
                                    │ ESCALATED │
                                    └───────────┘
```

### State Transitions (PostgreSQL)

```sql
CREATE TABLE task_states (
    task_id VARCHAR(64) PRIMARY KEY,
    current_state VARCHAR(32) NOT NULL,
    assigned_agent VARCHAR(32),
    priority VARCHAR(16) DEFAULT 'medium',
    context JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deadline TIMESTAMP,
    retry_count INT DEFAULT 0,
    max_retries INT DEFAULT 3,
    escalated BOOLEAN DEFAULT FALSE
);

CREATE TABLE state_history (
    id SERIAL PRIMARY KEY,
    task_id VARCHAR(64),
    from_state VARCHAR(32),
    to_state VARCHAR(32),
    changed_by VARCHAR(32),
    metadata JSONB,
    changed_at TIMESTAMP DEFAULT NOW()
);
```

---

## 4. Failure Handling

### Retry Logic

```
Attempt 1 → Failure
    ↓ (wait 30s)
Attempt 2 → Failure
    ↓ (wait 60s)
Attempt 3 → Failure
    ↓
ESCALATE to human (Stephen)
```

**Retry Configuration:**

```json
{
  "retry_policy": {
    "max_attempts": 3,
    "backoff": "exponential",
    "base_delay_seconds": 30,
    "max_delay_seconds": 300,
    "retryable_errors": ["timeout", "rate_limit", "connection_error", "5xx"],
    "non_retryable_errors": ["invalid_input", "unauthorized", "404"]
  }
}
```

### Escalation Protocol

When all retries fail, SOVEREIGN escalates to Stephen:

```json
{
  "escalation": {
    "task_id": "task_a1b2c3",
    "failed_agent": "sales_agent",
    "failure_reason": "CRM API timeout after 3 attempts",
    "original_request": {
      "type": "lead_qualification",
      "summary": "Qualify lead Jane Smith from TechCorp"
    },
    "context": {
      "last_error": "ETIMEDOUT: Connection to GoHighLevel API timed out",
      "attempts": 3,
      "total_time_seconds": 180,
      "data_loss_risk": "low"
    },
    "suggested_action": "Check GoHighLevel API status, manually qualify if urgent"
  }
}
```

### Critical Failure Alerts

Alerts that bypass normal routing and go directly to Stephen via Telegram:

| Severity | Criteria | Response Time Target |
|----------|---------|---------------------|
| 🔴 **Critical** | Revenue impact, client-facing outage, data loss risk | < 15 minutes |
| 🟠 **High** | Agent down > 30min, integration broken, stuck pipeline | < 1 hour |
| 🟡 **Medium** | Non-blocking errors, degraded performance | < 4 hours |
| 🟢 **Low** | Logging failures, cosmetic issues | Next business day |

**Critical Alert Template:**
```
🚨 CRITICAL — [Agent Name] Failure

Task: [task_id]
Agent: [agent_name]
Error: [error_summary]
Impact: [what's affected]
Time: [first_failure → escalation_time]
Suggested: [recommended action]

Reply:
/approve [retry_command] — to retry
/done [task_id] — to mark resolved manually
/ignore [task_id] — to dismiss
```

### Dead Letter Queue (DLQ)

Failed tasks that can't be retried and weren't manually resolved within 24h:

```json
{
  "dlq_entry": {
    "task_id": "task_a1b2c3",
    "original_agent": "sales_agent",
    "failure_count": 3,
    "first_failure": "2026-03-18T21:45:00Z",
    "last_failure": "2026-03-18T21:50:00Z",
    "escalated_to": "stephen",
    "manual_resolved": false,
    "context_snapshot": { "..." }
  }
}
```

**Daily DLQ Review:** Cron job at 09:00 UTC sends Telegram summary of unresolved DLQ entries.

---

## 5. Database Schema

### PostgreSQL Tables

```sql
-- Agent definitions
CREATE TABLE agents (
    agent_id VARCHAR(32) PRIMARY KEY,
    name VARCHAR(64),
    emoji VARCHAR(8),
    capabilities JSONB,
    status VARCHAR(16) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insert default agents
INSERT INTO agents VALUES
('sales_agent', 'Sales Agent', '📈', '["lead_qualification","demo_booking","proposal"]', 'active', NOW()),
('support_agent', 'Support Agent', '🛟', '["questions","troubleshooting","faq"]', 'active', NOW()),
('content_agent', 'Content Agent', '✍️', '["social_posts","emails","copywriting"]', 'active', NOW()),
('analytics_agent', 'Analytics Agent', '📊', '["metrics","reports","anomaly_detection"]', 'active', NOW());

-- Task tracking
CREATE TABLE tasks (
    task_id VARCHAR(64) PRIMARY KEY,
    classification_type VARCHAR(32),
    urgency VARCHAR(16),
    business_value VARCHAR(16),
    risk VARCHAR(16),
    source VARCHAR(32),
    current_state VARCHAR(32),
    assigned_agent VARCHAR(32),
    priority VARCHAR(16),
    context JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deadline TIMESTAMP,
    completed_at TIMESTAMP,
    retry_count INT DEFAULT 0,
    escalated BOOLEAN DEFAULT FALSE
);

-- State transitions
CREATE TABLE state_history (
    id SERIAL PRIMARY KEY,
    task_id VARCHAR(64) REFERENCES tasks(task_id),
    from_state VARCHAR(32),
    to_state VARCHAR(32),
    changed_by VARCHAR(32),
    metadata JSONB,
    changed_at TIMESTAMP DEFAULT NOW()
);

-- Agent communications
CREATE TABLE agent_messages (
    id SERIAL PRIMARY KEY,
    task_id VARCHAR(64),
    from_agent VARCHAR(32),
    to_agent VARCHAR(32),
    message_type VARCHAR(32),
    payload JSONB,
    sent_at TIMESTAMP DEFAULT NOW(),
    acknowledged_at TIMESTAMP,
    status VARCHAR(16) DEFAULT 'sent'
);

-- Dead letter queue
CREATE TABLE dead_letter_queue (
    id SERIAL PRIMARY KEY,
    task_id VARCHAR(64),
    agent_id VARCHAR(32),
    failure_reason TEXT,
    retry_count INT,
    context JSONB,
    failed_at TIMESTAMP DEFAULT NOW(),
    escalated_at TIMESTAMP,
    resolved BOOLEAN DEFAULT FALSE,
    resolved_at TIMESTAMP,
    resolution_notes TEXT
);

-- Workflow execution log
CREATE TABLE workflow_runs (
    id SERIAL PRIMARY KEY,
    workflow_name VARCHAR(64),
    trigger_source VARCHAR(32),
    status VARCHAR(16),
    input JSONB,
    output JSONB,
    duration_ms INT,
    started_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP,
    error_message TEXT
);
```

---

## 6. Implementation Checklist

### Phase 1: Foundation (Week 1)
- [ ] PostgreSQL schema deployed (Zeabur)
- [ ] SOVEREIGN classification logic implemented
- [ ] Basic routing to 4 specialist agents
- [ ] N8N webhook endpoints created
- [ ] Task state machine in PostgreSQL

### Phase 2: Specialist Agents (Week 2)
- [ ] Sales Agent: Lead qualification + demo booking
- [ ] Support Agent: FAQ knowledge base + escalation
- [ ] Content Agent: Post generation + approval flow
- [ ] Analytics Agent: Daily report generation

### Phase 3: Failure Handling (Week 3)
- [ ] Retry logic (3 attempts, exponential backoff)
- [ ] Telegram escalation alerts
- [ ] Dead letter queue + daily digest
- [ ] Agent health monitoring

### Phase 4: Workflow Integration (Week 4)
- [ ] N8N workflow templates deployed
- [ ] Agent-triggered workflow execution
- [ ] Cross-agent handoff via N8N
- [ ] End-to-end testing with sample tasks

---

## 7. Monitoring & Observability

### Dashboard Metrics (SOVEREIGN should report daily)

| Metric | Target | Alert Threshold |
|--------|--------|----------------|
| Tasks classified/day | Track volume | Anomaly: ±50% |
| Routing accuracy | >95% | <90% |
| Avg task completion time | <5 min | >15 min |
| Agent failure rate | <5% | >10% |
| Escalation rate | <5% | >15% |
| DLQ entries/day | 0 | >3 |

### Health Check Endpoint

```
GET /hooks/agent/health
→ 200: All agents healthy
→ 503: One or more agents down (details in response)
```

---

## 8. Security & Access Control

- **Agent API keys:** Each agent has unique webhook authentication token
- **Data isolation:** Agents only access context relevant to their task
- **Audit trail:** All state transitions logged in `state_history`
- **Human override:** Stephen can manually reassign any task via command
- **Rate limiting:** Max 100 agent messages/minute to prevent loops

---

*Specification v1.0 — 2026-03-18*  
*Dream AI Internal Operating System*
