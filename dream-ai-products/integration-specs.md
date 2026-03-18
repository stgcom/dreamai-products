# Integration Specifications — Dream AI

> API reference and data mapping for all external service integrations. Designed for N8N HTTP Request nodes + PostgreSQL MCP persistence.

---

## 1. SMS — Twilio

**Purpose:** SMS notifications, two-factor auth, lead follow-up sequences, appointment reminders.

### Authentication

```
Account SID: TWILIO_ACCOUNT_SID (env)
Auth Token: TWILIO_AUTH_TOKEN (env)
From Number: TWILIO_PHONE_NUMBER (env)
```

N8N credential type: **HTTP Basic Auth** (user = Account SID, password = Auth Token).

### Key Endpoints

#### Send SMS

```
POST https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Messages.json
```

**N8N HTTP Request node config:**
```json
{
  "method": "POST",
  "url": "https://api.twilio.com/2010-04-01/Accounts/{{$env.TWILIO_ACCOUNT_SID}}/Messages.json",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpBasicAuth",
  "sendBody": true,
  "bodyParameters": {
    "To": "={{$json.lead_phone}}",
    "From": "={{$env.TWILIO_PHONE_NUMBER}}",
    "Body": "={{$json.message}}"
  }
}
```

**Response:**
```json
{
  "sid": "SM1234567890abcdef",
  "status": "queued",
  "to": "+15551234567",
  "from": "+15559876543",
  "body": "Your demo is confirmed for March 20 at 2pm EST.",
  "num_segments": 1,
  "price": "-0.00790",
  "direction": "outbound-api",
  "date_created": "2026-03-18T03:00:00Z"
}
```

#### List Messages (for audit)

```
GET https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Messages.json?To={to}&Limit=50
```

#### Message Status Webhook

Configure in Twilio console:
```
POST https://your-n8n.example.com/webhooks/twilio-status
```

Webhook receives:
```json
{
  "MessageSid": "SM1234567890abcdef",
  "MessageStatus": "delivered",
  "To": "+15551234567",
  "From": "+15559876543"
}
```

### Data Mapping

| Dream AI Field | Twilio Field | Notes |
|----------------|-------------|-------|
| `lead.phone` | `To` | E.164 format required (`+15551234567`) |
| `message_body` | `Body` | Max 1600 chars (auto-segmented) |
| `from_number` | `From` | Must be verified Twilio number |
| `lead.id` | `ApplicationSid` (optional) | For analytics linking |
| status callback | `StatusCallback` | URL to receive delivery events |

### Error Handling

| HTTP Code | Meaning | Action |
|-----------|---------|--------|
| 201 | Queued | Success — monitor via StatusCallback |
| 21211 | Invalid phone number | Mark lead `phone_valid = false`, skip future SMS |
| 21610 | Unsubscribed / stopped | Set `sms_opt_out = true` on lead |
| 21408 | Permission denied (geo) | Log, skip SMS, try email instead |
| 401 | Auth failed | **CRITICAL** — alert ops, check env vars |

### Database Integration

```sql
CREATE TABLE sms_events (
  id SERIAL PRIMARY KEY,
  lead_id UUID REFERENCES leads(id),
  twilio_sid TEXT UNIQUE,
  to_number TEXT,
  message_body TEXT,
  status TEXT DEFAULT 'queued', -- queued, sent, delivered, failed, undelivered
  price NUMERIC(5,4),
  error_code TEXT,
  error_message TEXT,
  sent_at TIMESTAMPTZ DEFAULT now(),
  delivered_at TIMESTAMPTZ
);

-- Webhook handler updates status:
-- UPDATE sms_events SET status = $1, delivered_at = now() WHERE twilio_sid = $2;
```

### N8N Workflow Integration

- **Lead Capture → SMS:** Hot leads (score ≥ 70) trigger immediate SMS: "Hi {name}, thanks for your interest! We'll be in touch shortly."
- **Demo Booking → SMS:** Confirmation SMS with time + Zoom link.
- **Email Sequence → SMS:** Step 5 (breakup) sends SMS fallback if email bounced.

---

## 2. Voice AI — ElevenLabs

**Purpose:** AI voice agents for inbound calls, outbound follow-up calls, demo scheduling confirmation, voicemail drops.

### Authentication

```
API Key: ELEVENLABS_API_KEY (env)
Base URL: https://api.elevenlabs.io
```

N8N credential type: **Header Auth** (`xi-api-key`).

### Key Endpoints

#### List Voices

```
GET https://api.elevenlabs.io/v1/voices
```

#### Generate Speech (TTS)

```
POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}
```

**N8N HTTP Request node:**
```json
{
  "method": "POST",
  "url": "https://api.elevenlabs.io/v1/text-to-speech/{{$env.ELEVENLABS_VOICE_ID}}",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendBody": true,
  "bodyEncoding": "json",
  "body": {
    "text": "={{$json.tts_text}}",
    "model_id": "eleven_multilingual_v2",
    "voice_settings": {
      "stability": 0.5,
      "similarity_boost": 0.75,
      "style": 0.3,
      "use_speaker_boost": true
    }
  },
  "response": {
    "responseFormat": "file",
    "filePath": "/tmp/voice_{{$json.lead_id}}.mp3"
  }
}
```

**Response:** Binary audio (MP3). Store or stream directly.

#### Conversational AI (Voice Agent)

```
POST https://api.elevenlabs.io/v1/convai/conversation/create
```

Used for AI agents that can handle inbound calls:
```json
{
  "agent_id": "YOUR_AGENT_ID",
  "conversation_config": {
    "agent": {
      "prompt": "You are a Dream AI assistant. Help schedule demos and answer questions about our AI employee services.",
      "first_message": "Thanks for calling Dream AI! How can I help you today?"
    },
    "tts": {
      "voice_id": "YOUR_VOICE_ID"
    }
  }
}
```

#### Voice Cloning (for branded voices)

```
POST https://api.elevenlabs.io/v1/voices/add
```

Multipart form: upload audio samples + voice name/description.

### Data Mapping

| Dream AI Field | ElevenLabs Field | Notes |
|----------------|-----------------|-------|
| `tts_text` | `text` | Content to synthesize |
| `voice_profile` | `voice_id` | Brand voice, default, or custom |
| `tone` | `voice_settings.style` | 0.0 neutral → 1.0 expressive |
| `audio_output` | Response binary | MP3 format, store in S3 or serve directly |

### Error Handling

| HTTP Code | Meaning | Action |
|-----------|---------|--------|
| 200 | Success | Audio generated |
| 401 | Invalid API key | Alert ops |
| 422 | Invalid text/voice | Log, fix params, retry |
| 429 | Rate limited | Queue and retry after `Retry-After` header |
| 500 | ElevenLabs error | Fallback: use text-based communication |

### Use Cases in Workflows

1. **Outbound follow-up calls:** When lead goes cold after email sequence, generate AI voice call to re-engage.
2. **Appointment reminders:** Auto-generated voice call 24h before demo.
3. **Voicemail drops:** Pre-generated personalized voicemail for high-value leads.
4. **Website voice widget:** Conversational AI agent embedded on site for real-time Q&A.

---

## 3. CRM — Follow Up Boss

**Purpose:** Master contact & deal database. All lead/client data syncs here.

### Authentication

```
API Key: FUB_API_KEY (env)
Base URL: https://api.followupboss.com/v1
```

N8N credential type: **Header Auth** (`X-Boomtown-Api-Key` or `X-Api-Key` depending on account).

### Key Endpoints

#### Create/Update Person

```
POST /v1/people
```

**Payload:**
```json
{
  "firstName": "={{$json.first_name}}",
  "lastName": "={{$json.last_name}}",
  "emails": [{"value": "={{$json.email}}", "type": "work"}],
  "phones": [{"value": "={{$json.phone}}", "type": "mobile"}],
  "source": "={{$json.source}}",
  "tags": ["dream-ai", "{{$json.plan_type}}"],
  "customFields": {
    "leadScore": "={{$json.score}}",
    "utmSource": "={{$json.utm_source}}",
    "utmCampaign": "={{$json.utm_campaign}}"
  },
  "assignedTo": "={{$json.assigned_agent}}"
}
```

**Response (201):**
```json
{
  "id": 12345,
  "firstName": "John",
  "lastName": "Doe",
  "emails": [{"value": "john@example.com", "type": "work"}],
  "createdTime": "2026-03-18T03:00:00Z"
}
```

#### Create Action/Event

```
POST /v1/actions
```

```json
{
  "type": "Email",
  "personId": "={{$json.fub_person_id}}",
  "name": "Welcome Email Sent",
  "notes": "Step 1 of email sequence"
}
```

#### Create Deal

```
POST /v1/deals
```

```json
{
  "personId": "={{$json.fub_person_id}}",
  "name": "={{$json.company_name}} - AI Employee Package",
  "price": "={{$json.deal_value}}",
  "stage": "New Lead",
  "pipeline": "Dream AI Services",
  "source": "={{$json.lead_source}}"
}
```

#### List People (search)

```
GET /v1/people?email={{email}}&limit=5
```

### Data Mapping

| Dream AI Field | FUB Field | Notes |
|----------------|-----------|-------|
| `lead.id` | Internal UUID | Cross-reference only |
| `lead.email` | `emails[0].value` | Primary email |
| `lead.phone` | `phones[0].value` | E.164 recommended |
| `lead.score` | `customFields.leadScore` | 0-100 integer |
| `lead.source` | `source` | Free text or enum |
| `utm_*` | `customFields.utm*` | Attribution tracking |
| `client.plan` | `deal.name` + `deal.price` | Plan selection |
| `demo.slot` | FUB Calendar Event | Via actions API |

### Webhooks (FUB → N8N)

Configure in FUB Settings → API → Webhooks:
- `person.created` → Lead capture sync
- `deal.stage_changed` → Onboarding trigger (when stage = "Won")
- `person.assigned` → Team notification

### Error Handling

| HTTP Code | Action |
|-----------|--------|
| 200/201 | Success |
| 409 | Duplicate — GET by email, use existing `id` |
| 422 | Validation error — log, review payload mapping |
| 429 | Rate limit — N8N retry with backoff (max 5/min) |

### Database Mirror

```sql
-- Mirror FUB people locally for fast queries + offline access
CREATE TABLE fub_sync_log (
  id SERIAL PRIMARY KEY,
  fub_id TEXT,
  entity_type TEXT, -- 'person', 'deal', 'action'
  operation TEXT,    -- 'create', 'update'
  payload JSONB,
  synced_at TIMESTAMPTZ DEFAULT now(),
  status TEXT -- 'success', 'failed'
);
```

---

## 4. CRM — kvCORE (Optional/Secondary)

**Purpose:** Real estate–specific CRM. Use if clients are in real estate vertical or for secondary lead routing.

### Authentication

```
API Key: KVCORE_API_KEY (env)
Base URL: https://api.kvcore.com/api/v1
```

### Key Endpoints

#### Create Contact

```
POST /contacts
```

```json
{
  "first_name": "={{$json.first_name}}",
  "last_name": "={{$json.last_name}}",
  "email": "={{$json.email}}",
  "phone": "={{$json.phone}}",
  "source": "={{$json.source}}",
  "tags": ["dream-ai-lead"],
  "notes": "Lead score: {{$json.score}}. Source UTM: {{$json.utm_source}}"
}
```

#### Create Task

```
POST /tasks
```

```json
{
  "contact_id": "={{$json.kvcore_contact_id}}",
  "type": "call",
  "due_date": "={{$json.follow_up_date}}",
  "note": "Follow up on demo booking request"
}
```

### Data Mapping (FUB ↔ kvCORE)

| Field | FUB | kvCORE |
|-------|-----|--------|
| Person ID | `id` (integer) | `_id` (string) |
| Name | `firstName` + `lastName` | `first_name` + `last_name` |
| Email | `emails[].value` | `email` |
| Tags | `tags[]` | `tags[]` |
| Stage | `deal.stage` | `lead_status` |

### When to Use

- **FUB:** Primary CRM for most clients and internal operations.
- **kvCORE:** When client is a real estate team and needs industry-specific features (IDX, property alerts, etc.).
- **Sync both:** Hot leads push to both CRMs via parallel N8N branches.

---

## 5. Calendar — Google Calendar

**Purpose:** Demo bookings, team scheduling, client kickoff meetings.

### Authentication

Google OAuth2 in N8N. Scopes required:
```
https://www.googleapis.com/auth/calendar
https://www.googleapis.com/auth/calendar.events
```

### Key Endpoints

#### Check Availability

```
POST https://www.googleapis.com/calendar/v3/calendars/primary/freeBusy
```

```json
{
  "timeMin": "2026-03-18T09:00:00Z",
  "timeMax": "2026-03-22T17:00:00Z",
  "items": [{"id": "primary"}]
}
```

**Response:**
```json
{
  "calendars": {
    "primary": {
      "busy": [
        {"start": "2026-03-19T14:00:00Z", "end": "2026-03-19T15:00:00Z"}
      ]
    }
  }
}
```

#### Create Event

```
POST https://www.googleapis.com/calendar/v3/calendars/primary/events
```

```json
{
  "summary": "Dream AI Demo — {{$json.company_name}}",
  "description": "Demo call with {{$json.lead_name}}\n\nLead Score: {{$json.score}}\nSource: {{$json.source}}\n\nPrep notes:\n- Review their website\n- Identify 2 AI employee use cases\n- Prepare custom ROI estimate",
  "start": {
    "dateTime": "={{$json.slot}}",
    "timeZone": "America/New_York"
  },
  "end": {
    "dateTime": "={{$json.slot_end}}",
    "timeZone": "America/New_York"
  },
  "attendees": [
    {"email": "={{$json.lead_email}}", "displayName": "={{$json.lead_name}}"},
    {"email": "={{$json.rep_email}}", "displayName": "={{$json.rep_name}}"}
  ],
  "conferenceData": {
    "createRequest": {
      "requestId": "demo-{{$json.lead_id}}",
      "conferenceSolutionKey": {"type": "hangoutsMeet"}
    }
  },
  "reminders": {
    "overrides": [
      {"method": "popup", "minutes": 60},
      {"method": "popup", "minutes": 15}
    ]
  }
}
```

**Response:** Returns event with `hangoutsMeet` conference link.

#### List Upcoming Events

```
GET /calendars/primary/events?timeMin=now&maxResults=10&singleEvents=true&orderBy=startTime
```

### Data Mapping

| Dream AI Field | Calendar Field | Notes |
|----------------|---------------|-------|
| `demo.slot` | `start.dateTime` | ISO 8601 with timezone |
| `demo.duration_min` | `end.dateTime` | Calculate `start + duration` |
| `lead.name` | `attendees[0].displayName` | |
| `lead.email` | `attendees[0].email` | Receives invite |
| zoom_link | `conferenceData` (auto-gen) | Or inject custom Zoom link |
| prep_notes | `description` | Pre-populated for sales team |

### Error Handling

| Code | Action |
|------|--------|
| 409 | Conflict — find next available slot, return alternatives |
| 403 | Permissions — re-run Google OAuth flow |
| 404 | Calendar not found — verify calendar ID |
| 429 | Rate limit — N8N retry with backoff |

### N8N Cron: Cleanup

Daily cron checks for cancelled demos and cleans up:
```
DELETE FROM booked_demos WHERE status = 'cancelled' AND slot < now()
UPDATE booked_demos SET status = 'no-show' WHERE status = 'confirmed' AND slot < now() - interval '1 hour' AND completed_at IS NULL
```

---

## 6. Payments — Stripe

**Purpose:** Client billing, subscription management, payment collection during onboarding.

### Authentication

```
Secret Key: STRIPE_SECRET_KEY (env)
Publishable Key: STRIPE_PUBLISHABLE_KEY (env — frontend only)
Webhook Secret: STRIPE_WEBHOOK_SECRET (env)
```

Base URL: `https://api.stripe.com/v1`

### Key Endpoints

#### Create Customer

```
POST https://api.stripe.com/v1/customers
```

```json
{
  "email": "={{$json.client_email}}",
  "name": "={{$json.client_name}}",
  "metadata": {
    "fub_id": "={{$json.fub_person_id}}",
    "client_id": "={{$json.client_id}}",
    "company": "={{$json.company_name}}"
  }
}
```

**Response:**
```json
{
  "id": "cus_ABC123",
  "email": "client@company.com",
  "name": "John Doe",
  "created": 1742275200
}
```

#### Create Subscription

```
POST https://api.stripe.com/v1/subscriptions
```

```json
{
  "customer": "={{$json.stripe_customer_id}}",
  "items": [{"price": "={{$json.stripe_price_id}}"}],
  "payment_behavior": "default_incomplete",
  "metadata": {
    "plan_name": "={{$json.plan}}",
    "client_id": "={{$json.client_id}}"
  },
  "expand": ["latest_invoice.payment_intent"]
}
```

#### Create Payment Intent (one-time)

```
POST https://api.stripe.com/v1/payment_intents
```

```json
{
  "amount": "={{$json.amount_cents}}",
  "currency": "usd",
  "customer": "={{$json.stripe_customer_id}}",
  "metadata": {
    "description": "={{$json.description}}"
  }
}
```

#### Generate Customer Portal Link

```
POST https://api.stripe.com/v1/billing_portal/sessions
```

```json
{
  "customer": "={{$json.stripe_customer_id}}",
  "return_url": "https://dream-ai.com/client-portal"
}
```

### Webhooks (Stripe → N8N)

```
POST /webhooks/stripe
```

Events to handle:

| Event | Action |
|-------|--------|
| `customer.subscription.created` | Confirm onboarding, unlock services |
| `customer.subscription.updated` | Update `clients.plan` and `clients.mrr` |
| `customer.subscription.deleted` | Set `clients.status = 'churned'`, trigger offboarding |
| `invoice.payment_succeeded` | Log payment, update `clients.last_payment` |
| `invoice.payment_failed` | Alert ops, notify client, pause services after 3 failures |
| `payment_intent.succeeded` | One-time payment confirmed |

### Data Mapping

| Dream AI Field | Stripe Field | Notes |
|----------------|-------------|-------|
| `client.email` | `customer.email` | |
| `client.id` | `customer.metadata.client_id` | Cross-reference |
| `plan.price_monthly` | `items[0].price` (Price ID) | Must be created in Stripe Dashboard first |
| `client.mrr` | `subscription.items.data[0].price.unit_amount / 100` | Convert cents → dollars |
| `fub_deal_id` | `customer.metadata.fub_id` | CRM link |

### Pricing Plans (Stripe Price IDs)

| Plan | Monthly | Stripe Price ID (placeholder) |
|------|---------|-------------------------------|
| AI Employee Basic | $497 | `price_basic_497` |
| AI Employee Pro | $997 | `price_pro_997` |
| AI Employee Enterprise | $2,497 | `price_ent_2497` |
| Consulting (hourly) | Variable | `price_consulting_h` |
| Voice AI Add-on | $197 | `price_voice_197` |

*(Replace placeholder IDs with actual Stripe Price IDs from Dashboard.)*

### Error Handling

| Code | Action |
|------|--------|
| 400 | Bad request — log payload, review mapping |
| 402 | Card declined — notify client, provide portal link |
| 404 | Customer/subscription not found — check `stripe_customer_id` |
| 409 | Already exists — retrieve and update instead |
| Webhook verification | Always verify signature with `STRIPE_WEBHOOK_SECRET` |

### Database Integration

```sql
CREATE TABLE payments (
  id SERIAL PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  stripe_invoice_id TEXT,
  stripe_payment_intent_id TEXT,
  amount NUMERIC(10,2),
  currency TEXT DEFAULT 'usd',
  status TEXT, -- succeeded, failed, pending
  paid_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

---

## Data Flow Architecture

```
                        ┌──────────────┐
                        │   Website /  │
                        │   Landing    │
                        │   Page       │
                        └──────┬───────┘
                               │ webhook
                               ▼
┌─────────────────────────────────────────────────┐
│              N8N Automation Layer                │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ Lead     │→ │ Email    │→ │ Demo         │  │
│  │ Capture  │  │ Sequence │  │ Booking      │  │
│  └────┬─────┘  └────┬─────┘  └──────┬───────┘  │
│       │              │               │           │
│       ▼              ▼               ▼           │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ Health   │  │ Onboard  │  │ Integration  │  │
│  │ Check    │  │          │  │ Router       │  │
│  └──────────┘  └──────────┘  └──────────────┘  │
└──────────────────┬──────────────────────────────┘
                   │
       ┌───────────┼───────────┬───────────┐
       ▼           ▼           ▼           ▼
┌──────────┐ ┌──────────┐ ┌────────┐ ┌──────────┐
│PostgreSQL│ │ Follow   │ │ Stripe │ │ Twilio / │
│ (MCP)    │ │ Up Boss  │ │        │ │ ElevenL  │
│          │ │          │ │        │ │ Google   │
└──────────┘ └──────────┘ └────────┘ │ Cal      │
                                     └──────────┘
```

---

## Implementation Checklist

- [ ] N8N instance deployed with PostgreSQL MCP connected
- [ ] All API credentials stored in N8N credentials (encrypted)
- [ ] Environment variables set for all services
- [ ] Webhook URLs registered with all external services
- [ ] Database schemas created via PostgreSQL MCP
- [ ] All 5 core workflows imported from `/n8n-exports/`
- [ ] Health check workflow active and posting to `#system-health`
- [ ] Stripe webhooks verified and endpoint configured
- [ ] Test each workflow end-to-end with sandbox data
- [ ] Set up N8N error workflows for each production workflow
- [ ] Document credential rotation schedule (quarterly)

---

*Integration specs version 1.0 — Created 2026-03-18 by BOLT*
