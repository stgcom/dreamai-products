# N8N Automation Blueprints — Dream AI

> Engineering specs for production N8N workflows. All workflows use PostgreSQL MCP for persistence and are triggerable/auditable via N8N MCP.

---

## 1. Lead Capture Workflow

**Trigger:** Webhook (`POST /webhooks/lead-capture`)

### Node Flow

```
[Webhook Trigger] → [Validate Payload] → [Dedupe Check] → [Upsert to DB] → [Enrich + Score] → [Route to CRM] → [Notify]
```

### Nodes

| # | Node Type | Config | Purpose |
|---|-----------|--------|---------|
| 1 | Webhook | `POST /webhooks/lead-capture`, response mode "last node" | Entry point. Accepts JSON body. |
| 2 | Function (Code) | Validate required fields: `email`, `firstName`, `source` | Reject 400 if missing. Log bad payloads to `lead_errors` table. |
| 3 | PostgreSQL (Execute Query) | `SELECT id FROM leads WHERE email = $1` | Dedupe check — skip enrichment if lead exists < 24h old. |
| 4 | PostgreSQL (Insert) | Insert into `leads` table (see schema below) | Persist raw lead. |
| 5 | HTTP Request | enrichment API (Clearbit/PeopleDataLabs) | Append firmographic + social data. |
| 6 | Function (Code) | Score formula: `source_weight + enrichment_match + recency` | Assign 0–100 score. Store in `leads.score`. |
| 7 | IF / Switch | `score >= 70 → Hot; 40-69 → Warm; <40 → Cold` | Branch routing. |
| 8 | HTTP Request | Push to Follow Up Boss API (`POST /v1/people`) | CRM sync. Map fields per integration-specs. |
| 9 | HTTP Request | Slack webhook or N8N notification | Alert sales channel for hot leads. |

### Data Schema (PostgreSQL)

```sql
CREATE TABLE leads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  first_name TEXT,
  last_name TEXT,
  phone TEXT,
  source TEXT,              -- 'website', 'ad', 'referral', 'event'
  utm_source TEXT,
  utm_medium TEXT,
  utm_campaign TEXT,
  enrichment JSONB,         -- raw enrichment payload
  score INTEGER DEFAULT 0,
  status TEXT DEFAULT 'new', -- new, contacted, qualified, lost
  fub_id TEXT,              -- Follow Up Boss person ID
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE lead_errors (
  id SERIAL PRIMARY KEY,
  payload JSONB,
  error TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### Error Handling

- **Webhook timeout:** N8N retry policy = 3 attempts, exponential backoff.
- **Enrichment API down:** Catch error, proceed without enrichment, set `score = 25` (cold default), log to `lead_errors`.
- **CRM push fail:** Queue retry via N8N error workflow → re-attempt at 1h, 4h, 24h. Persist `fub_status = 'pending'` on lead record.
- **Dedupe collision:** Update existing record with new source/UTM data, increment `touch_count`.

---

## 2. Email Sequence Workflow

**Trigger:** Schedule (daily cron) + Event (`lead_scored` from Workflow 1)

### Node Flow

```
[Cron / Webhook] → [Query Due Contacts] → [Fetch Template] → [Personalize] → [Send via Provider] → [Log Event] → [Schedule Next]
```

### Nodes

| # | Node Type | Config | Purpose |
|---|-----------|--------|---------|
| 1 | Cron / Webhook | `0 9 * * *` (9 AM daily) or event trigger `lead_scored` | Kick off sequence check. |
| 2 | PostgreSQL | `SELECT * FROM email_sequence_queue WHERE send_at <= now() AND status = 'pending'` | Get contacts due for next email. |
| 3 | Switch | Route by `sequence_step`: 1=welcome, 2=value, 3=case-study, 4=offer, 5=breakup | Pick template. |
| 4 | PostgreSQL | `SELECT subject, body_html FROM email_templates WHERE step = $1` | Fetch template. |
| 5 | Function (Code) | Replace `{{first_name}}`, `{{company}}`, `{{pain_point}}` in template | Personalization. |
| 6 | HTTP Request | SendGrid / Mailchimp API: `POST /v3/mail/send` or N8N built-in Email node | Send email. |
| 7 | PostgreSQL (Insert) | Insert into `email_events` (lead_id, step, event_type, provider_msg_id) | Log send. |
| 8 | PostgreSQL (Update) | Update `email_sequence_queue`: set `status = 'sent'`, `next_send_at = now() + interval '3 days'` | Schedule next step. |

### Sequence Schedule

| Step | Delay | Purpose |
|------|-------|---------|
| 1 (Welcome) | Immediate | Confirm opt-in, deliver lead magnet |
| 2 (Value) | +3 days | Educational content, build authority |
| 3 (Case Study) | +3 days | Social proof, relevant client story |
| 4 (Offer) | +3 days | Direct CTA — book demo / start trial |
| 5 (Breakup) | +5 days | "Last chance" urgency, remove if no action |

### Error Handling

- **Send provider down:** N8N error trigger → log to `email_events` with `status = 'failed'`. Retry next cron cycle.
- **Bounce:** Webhook from provider → set lead `status = 'bounced'`, remove from sequence.
- **Unsubscribe:** Webhook → set `opted_out = true` on lead, halt all sequences.
- **Template render error:** Catch in personalize node, skip send, alert via Slack.

---

## 3. Demo Booking Workflow

**Trigger:** Webhook (`POST /webhooks/book-demo`)

### Node Flow

```
[Webhook] → [Validate Slots] → [Check Double-Book] → [Create Calendar Event] → [Update CRM] → [Confirm to Lead] → [Prep Internal]
```

### Nodes

| # | Node Type | Config | Purpose |
|---|-----------|--------|---------|
| 1 | Webhook | `POST /webhooks/book-demo` | Accept: `{lead_id, preferred_times[], name, email, company}` |
| 2 | Function (Code) | Validate: at least 1 time slot, email format, future dates | Reject 400 on bad input. |
| 3 | HTTP Request | Google Calendar API: `GET /calendar/v3/calendars/primary/freebusy` | Check availability for requested slots. |
| 4 | PostgreSQL | `SELECT COUNT(*) FROM booked_demos WHERE slot = $1 AND status = 'confirmed'` | Double-book check against internal DB. |
| 5 | HTTP Request | Google Calendar API: `POST /calendar/v3/calendars/primary/events` | Create event. Include Zoom link + prep notes. |
| 6 | HTTP Request | Follow Up Boss: `POST /v1/events` → type: "Appointment", linked to person | CRM update. |
| 7 | HTTP Request | Email/SMS to lead: confirmation with `.ics` file + Zoom link | Lead-facing confirmation. |
| 8 | HTTP Request | Slack: Internal prep channel with lead summary + score | Sales team heads-up. |
| 9 | PostgreSQL (Insert) | Insert into `booked_demos` | Audit record. |

### Data Schema

```sql
CREATE TABLE booked_demos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id UUID REFERENCES leads(id),
  calendar_event_id TEXT,
  slot TIMESTAMPTZ NOT NULL,
  duration_min INTEGER DEFAULT 30,
  status TEXT DEFAULT 'confirmed', -- confirmed, completed, no-show, cancelled
  zoom_link TEXT,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### Error Handling

- **No availability:** Return 422 with alternative slots (next 3 open slots from FreeBusy API).
- **Calendar API fail:** Retry 2x → if still fail, return 503 and queue for manual booking in Slack.
- **CRM fail:** Demo still confirmed; CRM sync retried hourly.
- **Cancellation:** Webhook `POST /webhooks/cancel-demo` → delete calendar event, update CRM, notify lead.

---

## 4. Client Onboarding Workflow

**Trigger:** CRM event (Follow Up Boss `deal_moved_to_won`) or manual trigger

### Node Flow

```
[Trigger] → [Pull Client Data] → [Create Workspace] → [Provision Services] → [Send Welcome] → [Assign Team] → [Schedule Kickoff]
```

### Nodes

| # | Node Type | Config | Purpose |
|---|-----------|--------|---------|
| 1 | Webhook / FUB Event | `POST /webhooks/fub-deal-won` | Triggered when deal status = "won". |
| 2 | HTTP Request | Follow Up Boss: `GET /v1/people/{id}` | Pull full client record. |
| 3 | Function (Code) | Map FUB fields → internal `clients` table schema | Data transformation layer. |
| 4 | PostgreSQL (Insert) | Insert into `clients` + `client_services` | Create internal records. |
| 5 | HTTP Request | Stripe: `POST /v1/customers` + `POST /v1/subscriptions` | Create customer + start subscription. |
| 6 | HTTP Request | Create Notion workspace/page for client (via Notion API or webhook) | Project tracking space. |
| 7 | HTTP Request | SendGrid: Send welcome email sequence (separate from lead sequence) | Client welcome. |
| 8 | Function (Code) | Round-robin assign from team pool (or rule-based by service type) | Team assignment. |
| 9 | HTTP Request | Calendar API: Create kickoff meeting 5 business days out | Schedule first call. |
| 10 | Slack | Post to `#new-clients` with full summary | Internal notification. |

### Data Schema

```sql
CREATE TABLE clients (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_name TEXT,
  primary_contact TEXT,
  email TEXT,
  phone TEXT,
  fub_deal_id TEXT,
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT,
  notion_page_id TEXT,
  assigned_to TEXT,
  status TEXT DEFAULT 'active', -- active, paused, churned
  plan TEXT,
  mrr NUMERIC(10,2),
  onboarded_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE client_services (
  id SERIAL PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  service_type TEXT,     -- 'ai-employees', 'voice-ai', 'automation', 'consulting'
  config JSONB,
  active BOOLEAN DEFAULT true
);
```

### Error Handling

- **Stripe creation fail:** Pause onboarding, alert `#ops-alerts`, retry after manual review.
- **Notion workspace fail:** Non-blocking — log warning, proceed with rest of onboarding.
- **Kickoff scheduling conflict:** Auto-retry next business day until successful (max 3 attempts).
- **Partial failure:** Store `onboarding_status` JSONB on client record with per-step success/fail.

---

## 5. Health Check Workflow

**Trigger:** Schedule (`*/15 * * * *` — every 15 minutes) + Manual webhook

### Node Flow

```
[Cron] → [Check External APIs] → [Check DB] → [Check Active Workflows] → [Evaluate Thresholds] → [Alert if Degraded] → [Log Metrics]
```

### Nodes

| # | Node Type | Config | Purpose |
|---|-----------|--------|---------|
| 1 | Cron | `*/15 * * * *` | Scheduled execution. |
| 2 | HTTP Request | Ping: Follow Up Boss, Stripe, SendGrid, Twilio, Google Calendar, ElevenLabs | Check API health (status code + latency). |
| 3 | PostgreSQL | `SELECT 1` + `SELECT count(*) FROM leads WHERE created_at > now() - interval '1 hour'` | DB connectivity + recent lead volume. |
| 4 | N8N MCP | List active workflows, check last execution time | Ensure no workflow is "stuck" (no run in > 2x expected interval). |
| 5 | Function (Code) | Evaluate: response times, error rates, workflow staleness | Score overall health 0-100. |
| 6 | IF | `health_score < 80 OR any_critical_down` | Alert branch. |
| 7 | HTTP Request | Slack `#system-health` alert + optional PagerDuty/webhook | Escalation. |
| 8 | PostgreSQL (Insert) | Insert into `system_health_log` | Metrics persistence. |

### Data Schema

```sql
CREATE TABLE system_health_log (
  id SERIAL PRIMARY KEY,
  checked_at TIMESTAMPTZ DEFAULT now(),
  health_score INTEGER,
  services JSONB,    -- {service: {status, latency_ms}}
  db_connected BOOLEAN,
  active_workflows INTEGER,
  stuck_workflows INTEGER,
  alerts_sent JSONB  -- [{channel, message, severity}]
);
```

### Alert Thresholds

| Condition | Severity | Action |
|-----------|----------|--------|
| Any critical API down | CRITICAL | Slack + PagerDuty + retry in 5 min |
| Response time > 5s | WARNING | Slack to `#system-health` |
| DB query fail | CRITICAL | Slack + halt dependent workflows |
| Workflow stuck > 30 min | WARNING | Slack + auto-restart attempt |
| Health score < 60 | CRITICAL | Escalate to on-call |

### Error Handling

- **Health check itself fails:** Log to local file, send bare-minimum alert via secondary channel.
- **Alert fatigue:** Deduplicate: don't re-alert for same service within 30 minutes (store last alert time in memory/state).
- **Metrics retention:** Auto-purge `system_health_log` entries older than 90 days (monthly cleanup workflow).

---

## Cross-Workflow Patterns

### Database Access

All workflows use **PostgreSQL MCP** for:
- Lead/client data persistence
- Sequence state management
- Audit logging
- Health metrics

### N8N MCP Integration

Use N8N MCP for:
- **Workflow management:** List, activate, deactivate workflows
- **Execution monitoring:** Check run status, debug failures
- **Credential rotation:** Update API keys without redeploying

### Naming Conventions

- Webhook URLs: `/webhooks/{workflow-name}`
- DB tables: snake_case, singular nouns
- N8N workflow names: `DreamAI — {Workflow Name}`
- Error workflows: `DreamAI — Error: {Workflow Name}`

### Version Control

Export all N8N workflows as JSON and store in repo:
```
/n8n-exports/
  ├── lead-capture.json
  ├── email-sequence.json
  ├── demo-booking.json
  ├── client-onboarding.json
  └── health-check.json
```

---

*Blueprint version 1.0 — Created 2026-03-18 by BOLT*
