# Dream AI — N8N RevOps Automation Specifications

> **Platform:** N8N (self-hosted) + PostgreSQL
> **Webhook endpoint pattern:** `https://your-n8n.com/webhook/[trigger-name]`
> **DB connection:** PostgreSQL node or HTTP Postgres API

---

## 1. Master Workflow: Lead Capture → Enrichment → Scoring → Routing

### Workflow: `lead-lifecycle-master`

```
[Webhook Trigger] → [Dedup Check] → [Enrich] → [Score] → [Stage Router] → [Route & Alert]
```

### Node Definitions

#### Node 1: Webhook Trigger
```
Node Type: Webhook
Method: POST
Path: /webhook/lead-capture
Response: 200 OK (immediate, async processing)
```

**Expected Payload:**
```json
{
  "email": "jane@propertygroup.com",
  "first_name": "Jane",
  "last_name": "Smith",
  "phone": "+15551234567",
  "job_title": "Managing Broker",
  "company_name": "Property Group Realty",
  "industry": "real_estate",
  "form_type": "demo_request",
  "referrer": "https://google.com/...",
  "utm_source": "google",
  "utm_medium": "cpc",
  "utm_campaign": "ai-receptionist-real-estate",
  "page_url": "https://dreamai.com/demo-request"
}
```

#### Node 2: Dedup Check
```
Node Type: PostgreSQL
Query:
  SELECT * FROM contacts WHERE email = $1

IF contact exists:
  → Merge existing contact data
  → Add new scoring event
  → Update lifecycle stage if needed (never downgrade)
  → Route to Node 7 (Update Scoring)

IF contact does NOT exist:
  → Route to Node 3 (Enrichment)
```

#### Node 3: Enrichment (Firmographic)
```
Node Type: HTTP Request → Clearbit / Apollo API

Input: {{ $json.email }}
API Call: GET https://person.clearbit.com/v2/combined/find?email={{ $json.email }}

Extract & Map:
  account.domain         ← response.company.domain
  account.name           ← response.company.name
  account.employee_count ← response.company.metrics.employees
  account.annual_revenue ← response.company.metrics.annualRevenue
  account.industry       ← response.company.category.industry
  account.city           ← response.company.geo.city
  account.state          ← response.company.geo.state
  contact.job_title      ← response.person.occupation.title (override if missing)
  contact.seniority      ← map_seniority(response.person.occupation.title)

Fallback: If Clearbit fails, try Apollo:
  GET https://api.apollo.io/v1/people/match?email={{ $json.email }}
```

#### Node 4: Niche Inference
```
Node Type: Code (JavaScript)

Logic:
  const nicheMap = {
    'real_estate': ['real estate', 'realty', 'realtor', 'property', 'brokerage', 'mortgage'],
    'medspa': ['medspa', 'medical spa', 'aesthetic', 'botox', 'laser', 'dermatolog', 'plastic surg'],
    'hvac': ['hvac', 'heating', 'cooling', 'air conditioning', 'furnace', 'ac ', 'ventilation']
  };

  let niche = $json.industry || 'other';

  // Check company name + industry against niche keywords
  const searchText = ($json.company_name + ' ' + ($json.industry || '')).toLowerCase();

  for (const [key, keywords] of Object.entries(nicheMap)) {
    if (keywords.some(kw => searchText.includes(kw))) {
      niche = key;
      break;
    }
  }

  return { niche };
```

#### Node 5: Calculate Firmographic Score
```
Node Type: Code (JavaScript)

Logic:
  let score = 0;

  // Industry match
  const niche = $json.niche;
  if (['real_estate', 'medspa', 'hvac'].includes(niche)) score += 10;
  else score += 0;

  // Company size
  const emp = $json.employee_count;
  if (emp >= 2 && emp <= 20) score += 8;
  else if (emp >= 21 && emp <= 50) score += 6;
  else if (emp >= 51 && emp <= 200) score += 4;

  // Revenue
  const rev = $json.annual_revenue;
  if (rev >= 500000 && rev <= 10000000) score += 8;
  else if ((rev >= 250000 && rev < 500000) || (rev > 10000000 && rev <= 25000000)) score += 4;

  // Decision maker title
  const title = ($json.job_title || '').toLowerCase();
  if (title.match(/owner|ceo|founder|president|principal/)) score += 8;
  else if (title.match(/manager|director|gm|office|operations/)) score += 6;
  else if (title.match(/assistant|admin|intern|student/)) score += 2;

  // Personal email penalty
  const email = $json.email;
  if (email.match(/@(gmail|yahoo|hotmail|outlook|aol)\./)) score -= 5;

  // Geography
  const country = $json.country || 'US';
  if (country === 'US') score += 6;
  else if (country === 'CA' || country === 'Canada') score += 3;

  // Clamp to 0-40
  score = Math.max(0, Math.min(40, score));

  return { firmographic_score: score };
```

#### Node 6: Calculate Engagement Score
```
Node Type: PostgreSQL + Code

Query: Sum active scoring events for contact
  SELECT COALESCE(SUM(points_change), 0) as engagement_score
  FROM lead_scoring_events
  WHERE contact_id = $1
    AND (decay_at IS NULL OR decay_at > NOW())

IF new contact: engagement_score = form_type base points
  - 'demo_request': +25
  - 'free_trial': +20
  - 'quote_request': +15
  - 'content_download': +2
  - 'newsletter_signup': +1
  - 'contact_form': +5
```

#### Node 7: Update Scoring & Stage
```
Node Type: Code (JavaScript) + PostgreSQL

Logic:
  const total = $json.firmographic_score + $json.engagement_score;

  let newStage = 'unknown';
  if (total >= 60 && $json.firmographic_score >= 25 && $json.engagement_score >= 35) {
    newStage = 'mql';
  } else if ($json.engagement_score >= 5) {
    newStage = 'engaged';
  } else if (total > 0) {
    newStage = 'aware';
  }

  // Never downgrade
  IF stage_priority(newStage) > stage_priority(existingStage):
    UPDATE contact with new scores + stage
    IF newStage = 'mql': SET mql_at = NOW()
    IF newStage = 'sql': SET sql_at = NOW()

  Route to Node 8 (Routing)
```

#### Node 8: Lead Routing
```
Node Type: Switch + PostgreSQL

Switch conditions (priority order):

1. IF source = 'free_trial' AND total_score > 40:
   → Query sales_reps WHERE 'product_specialists' = ANY(teams) AND is_active = true
   → Assign to rep with oldest last_assigned_at

2. IF niche = 'real_estate':
   → Query sales_reps WHERE 'ae_real_estate' = ANY(teams) AND is_active = true
   → Round-robin

3. IF niche = 'medspa':
   → Query sales_reps WHERE 'ae_medspa' = ANY(teams) AND is_active = true
   → Round-robin

4. IF niche = 'hvac':
   → Query sales_reps WHERE 'ae_hvac' = ANY(teams) AND is_active = true
   → Round-robin

5. IF estimated_acv > 24000 (>$2k/mo):
   → Query sales_reps WHERE role = 'senior_ae' AND is_active = true
   → Round-robin

6. DEFAULT:
   → Query sales_reps WHERE 'ae_general' = ANY(teams) AND is_active = true
   → Round-robin

Fallback: Assign to sales manager, create escalation task.
```

#### Node 9: Alert & Task Creation
```
Node Type: IF + Slack/Email + N8N

IF stage = 'mql':
  → Slack: #sales-leads channel
     "🎯 New MQL: {{ $json.first_name }} {{ $json.last_name }} | {{ $json.company_name }} | Score: {{ $json.total }} | Niche: {{ $json.niche }} | Assigned to: {{ $json.rep_name }}"
  → Email: To assigned rep with full lead context
  → N8N: Create task in task system (Notion/Linear/etc.)
  → Set SLA timer (4 hours)

IF stage = 'mql' AND total_score > 75:
  → ALL of the above PLUS:
  → Slack: #sales-hot-leads channel
     "🔥 HOT LEAD: {{ $json.first_name }} {{ $json.company_name }} | Score: {{ $json.total }} | Contact immediately!"
  → SMS via Twilio to assigned rep
  → Alert sales manager

IF source = 'demo_request':
  → Priority override: immediate call notification
```

---

## 2. Auto-Qualification Rules

### Rule Engine (N8N Function Node)

```javascript
// Auto-qualification decision matrix
const lead = $input.first().json;

const rules = {
  auto_mql: {
    // Demo/trial requests from ICP companies auto-qualify
    condition: () =>
      ['demo_request', 'free_trial', 'quote_request'].includes(lead.form_type) &&
      ['real_estate', 'medspa', 'hvac'].includes(lead.niche) &&
      lead.employee_count >= 2,
    action: 'auto_mql',
    score_override: Math.max(lead.total_score, 65)
  },

  fast_track: {
    // High-value signals: pricing page + ICP match
    condition: () =>
      lead.firmographic_score >= 25 &&
      lead.engagement_score >= 20 &&
      lead.page_url?.includes('pricing'),
    action: 'fast_track_mql'
  },

  nurture: {
    // Good fit but low engagement — keep nurturing
    condition: () =>
      lead.firmographic_score >= 25 &&
      lead.engagement_score < 20 &&
      lead.engagement_score >= 5,
    action: 'nurture'
  },

  recycle: {
    // Was MQL but went cold
    condition: () =>
      lead.previous_stage === 'mql' &&
      lead.engagement_score < 15 &&
      daysSince(lead.mql_at) > 7,
    action: 'recycle_to_nurture'
  },

  disqualify: {
    // Clear disqualifiers
    condition: () =>
      lead.firmographic_score < 10 ||
      lead.email.match(/@(competitor1|competitor2)\./) ||
      lead.employee_count === 1 && !['real_estate', 'medspa'].includes(lead.niche),
    action: 'disqualify'
  }
};

// Evaluate rules in priority order
for (const [name, rule] of Object.entries(rules)) {
  if (rule.condition()) {
    return {
      qualification_result: rule.action,
      score_override: rule.score_override || null,
      rule_triggered: name
    };
  }
}

return { qualification_result: 'standard_flow' };
```

---

## 3. Follow-Up Automation by Lead Source

### Workflow: `follow-up-by-source`

Triggered when: Lead enters `engaged` or `mql` stage.

```
[Stage Change Webhook] → [Source Router] → [Email Sequence] → [Task Scheduler] → [SLA Timer]
```

### Source-Specific Sequences

#### Source: Inbound (organic_search, paid_google, paid_meta)

| Day | Channel | Action | N8N Node |
|-----|---------|--------|----------|
| 0 | Email | Welcome + value proposition for their niche | SendGrid / Gmail |
| 0 | Task | Rep call within 4 hours | Create task |
| 1 | Email | Case study from their vertical | SendGrid |
| 1 | Task | Follow-up call | Create task |
| 3 | Email | ROI calculator or free assessment offer | SendGrid |
| 5 | LinkedIn | Connect with assigned rep | LinkedIn automation |
| 7 | Email | "Still interested?" + demo CTA | SendGrid |
| 14 | Email | New case study or webinar invite | SendGrid |
| 30 | Email | Re-engagement: "What changed?" | SendGrid |
| 60 | Email | Break-up email: "Should I close your file?" | SendGrid |

#### Source: Free Trial

| Day | Channel | Action | N8N Node |
|-----|---------|--------|----------|
| 0 | In-app | Welcome modal + onboarding checklist | App webhook |
| 0 | Email | Welcome + setup guide | SendGrid |
| 0 | Task | Product specialist check-in call (2 hours) | Create task |
| 1 | Email | "How's your trial going?" + tips | SendGrid |
| 2 | Task | Check-in call #2 | Create task |
| 3 | Email | Advanced feature guide for their niche | SendGrid |
| 5 | Email | Social proof: "Companies like yours use this for X" | SendGrid |
| 7 | Email | Trial ending in 7 days + upgrade CTA | SendGrid |
| 10 | Email | Trial ending in 3 days + special offer | SendGrid |
| 14 | Email | Trial expired: final offer + demo invite | SendGrid |

#### Source: Referral

| Day | Channel | Action |
|-----|---------|--------|
| 0 | Email | "Referred by [partner name]" — personal intro |
| 0 | Task | Call within 4 hours |
| 1 | Email | Thank-you to referrer + update on engagement |
| 3 | Email | Tailored case study + demo offer |
| 7 | Email | Follow-up + value-add content |

#### Source: Outbound (outbound_email, outbound_linked)

| Day | Channel | Action |
|-----|---------|--------|
| 0 | Email | Personalized cold email (from sequences) |
| 3 | LinkedIn | Connect request with note |
| 5 | Email | Follow-up #2 — different angle |
| 7 | Email | Follow-up #3 — value offer (free assessment) |
| 14 | LinkedIn | Engage with their content |
| 21 | Email | Break-up / last attempt |

---

## 4. Task Creation Rules for Sales Team

### Workflow: `task-creation-engine`

```
[Trigger: Stage Change] → [Task Router] → [Create Task] → [Set SLA Timer]
```

### Task Templates

| Trigger Event | Task Type | Assigned To | Priority | Due |
|--------------|-----------|-------------|----------|-----|
| MQL created | Call + qualify | Assigned rep | High | +4 business hours |
| Demo requested | Schedule demo | Assigned rep | Urgent | +1 business hour |
| Free trial signup | Check-in call | Product specialist | High | +2 hours |
| SQL accepted | Discovery call | AE | High | +2 business days |
| Demo completed | Send proposal | AE | Medium | +3 business days |
| Proposal sent | Follow-up | AE | Medium | +2 business days |
| Deal stage > 2x avg days | Pipeline check-in | AE + manager | Medium | +1 day |
| Hot lead (score > 75) | Immediate call | Assigned rep + backup | Urgent | +30 minutes |

### N8N Task Creation Node

```javascript
const lead = $input.first().json;

const taskTemplates = {
  mql: {
    title: `New MQL: ${lead.first_name} from ${lead.company_name}`,
    description: `Score: ${lead.total_score} | Niche: ${lead.niche} | Source: ${lead.source}\n\nFirmographic: ${lead.firmographic_score}/40 | Engagement: ${lead.engagement_score}/60\n\nAction: Call and qualify within 4 hours.`,
    priority: 'high',
    due_hours: 4,
    type: 'call_and_qualify'
  },
  demo_request: {
    title: `Demo Request: ${lead.first_name} from ${lead.company_name}`,
    description: `Requested demo for ${lead.niche}.\n\nCall within 1 hour to schedule.`,
    priority: 'urgent',
    due_hours: 1,
    type: 'schedule_demo'
  },
  hot_lead: {
    title: `🔥 HOT LEAD: ${lead.first_name} from ${lead.company_name}`,
    description: `Score: ${lead.total_score}/100 — Contact IMMEDIATELY!\n\nAll context:\nFirmographic: ${lead.firmographic_score}/40\nEngagement: ${lead.engagement_score}/60\nNiche: ${lead.niche}\nSource: ${lead.source}`,
    priority: 'urgent',
    due_hours: 0.5,
    type: 'immediate_call'
  }
};

return taskTemplates[lead.task_type] || taskTemplates.mql;
```

---

## 5. Hot Lead Alerts (Score > 75)

### Workflow: `hot-lead-alerts`

```
[Score Update] → [IF score > 75 AND stage = 'mql'] → [Multi-channel Alert] → [Escalation Timer]
```

### Alert Channels

| Channel | Recipient | Message Template | Timing |
|---------|-----------|-----------------|--------|
| **Slack** | #sales-hot-leads | 🔥 `{{lead.first_name}} {{lead.last_name}}` at `{{lead.company_name}}` — Score: `{{lead.total_score}}` — Niche: `{{lead.niche}}` — Assigned: `{{lead.rep_name}}` | Immediate |
| **Slack DM** | Assigned rep | 🔥 Hot lead assigned: `{{lead.first_name}}` (`{{lead.company_name}}`). Score {{lead.total_score}}. Call within 30 min! | Immediate |
| **Email** | Assigned rep + manager | Full lead context HTML email with one-click call link | Immediate |
| **SMS** | Assigned rep (if score > 85) | "HOT LEAD: {{lead.first_name}} {{lead.company_name}} — Score {{lead.total_score}}. Call NOW: {{lead.phone}}" | Immediate |
| **SMS** | Sales manager (if score > 85) | "Rep {{lead.rep_name}} assigned hot lead: {{lead.first_name}} {{lead.company_name}}. Score {{lead.total_score}}. Monitor response." | Immediate |

### Hot Lead SLA Escalation

```
[Hot Lead Created] → [Start Timer: 30 min]

IF rep contacts within 30 min:
  → Log activity, close escalation

IF no contact after 30 min:
  → Alert manager via Slack
  → Create escalation task
  → Send SMS to rep again

IF no contact after 60 min:
  → Re-route to backup rep
  → Manager calls directly
  → Log SLA breach in activity_log
```

---

## 6. Re-Engagement Workflow for Cold Leads

### Workflow: `re-engagement-nurture`

**Trigger conditions:**
- Engagement score decayed below 10 AND was previously MQL/SQL
- No activity in 30+ days
- Lifecycle stage = 'engaged' or 'mql' but no contact in 7+ days

### Re-Engagement Sequence

| Day | Action | Channel | N8N Node |
|-----|--------|---------|----------|
| 0 | **We miss you** email: "Not the right time? No worries." + offer free assessment | Email | SendGrid |
| 0 | Remove from all marketing emails (except this sequence) | Action | PostgreSQL |
| 3 | **New content** email: Share relevant case study or new feature | Email | SendGrid |
| 7 | **Social touch**: Rep likes/comments on prospect's LinkedIn | LinkedIn | Automation |
| 14 | **Different angle** email: Lead with a new pain point or use case | Email | SendGrid |
| 21 | **Personal outreach**: Rep sends personal email (not templated) | Email | Manual trigger task |
| 30 | **Break-up** email: "Should I close your file? Click here if you want to stay on our radar." | Email | SendGrid |

### Re-Engagement Outcomes

```
IF prospect re-engages (opens, clicks, replies, visits site):
  → Score rebuilt with fresh engagement points
  → Re-enter main pipeline at 'engaged' stage
  → Alert assigned rep: "Cold lead re-engaged!"
  → Cancel re-engagement sequence

IF prospect clicks "close my file" or unsubscribes:
  → Set lifecycle_stage = 'lost'
  → Set loss_reason = 'no_response'
  → Suppress from all marketing
  → Log in activity_log

IF no response after 30-day sequence:
  → Set lifecycle_stage = 'dormant'
  → Suppress from active sequences
  → Add to quarterly re-check list
  → Re-evaluate in 90 days
```

### N8N Cron Trigger for Identifying Cold Leads

```
Schedule: Daily at 8:00 AM UTC

Query:
  SELECT c.*, a.niche,
         NOW() - c.updated_at as days_inactive,
         c.engagement_score
  FROM contacts c
  LEFT JOIN accounts a ON c.account_id = a.id
  WHERE c.lifecycle_stage IN ('engaged', 'mql', 'sql')
    AND c.updated_at < NOW() - INTERVAL '14 days'
    AND c.lifecycle_stage != 'dormant'
    AND NOT EXISTS (
      SELECT 1 FROM lead_scoring_events lse
      WHERE lse.contact_id = c.id
        AND lse.created_at > NOW() - INTERVAL '14 days'
    )

→ Route each to re-engagement sequence
→ Update contact.in_re_engagement = true
```

---

## 7. Complete Workflow Architecture

```
                        ┌─────────────────────────────────────┐
                        │         INCOMING LEADS              │
                        │  (forms, trials, outbound, etc.)    │
                        └──────────────┬──────────────────────┘
                                       │
                              [lead-capture webhook]
                                       │
                        ┌──────────────▼──────────────────────┐
                        │     lead-lifecycle-master            │
                        │  ┌─────────┐  ┌─────────┐           │
                        │  │ Dedup   │→ │ Enrich  │           │
                        │  └─────────┘  └────┬────┘           │
                        │                ┌───▼───┐             │
                        │                │ Score │             │
                        │                └───┬───┘             │
                        │           ┌────────▼────────┐        │
                        │           │  Auto-Qualify   │        │
                        │           └───┬─────┬───────┘        │
                        │         MQL   │     │ Nurture        │
                        │          ┌────┘     └────┐          │
                        └──────────┼────────────────┼─────────┘
                                   │                │
                    ┌──────────────▼──┐    ┌────────▼──────────┐
                    │  Routing Engine │    │ Nurture Sequences  │
                    │  (by niche,     │    │ (email drip)       │
                    │   size, source) │    └────────┬───────────┘
                    └────────┬───────┘              │
                             │              [if re-engages]
                    ┌────────▼───────┐         ┌────▼──────┐
                    │ Hot Lead Check │         │ Re-enter  │
                    │ (score > 75?)  │         │ pipeline  │
                    └──┬─────────┬───┘         └───────────┘
                   YES │         │ NO
              ┌────────▼───┐  ┌──▼──────────┐
              │ Multi-     │  │ Alert rep   │
              │ channel    │  │ (Slack +    │
              │ alerts     │  │  email)     │
              │ + SMS      │  └─────────────┘
              └────────────┘

         ┌────────────────────────────────────┐
         │     follow-up-by-source            │
         │  (auto-triggered per lead source)  │
         └────────────────────────────────────┘

         ┌────────────────────────────────────┐
         │     re-engagement-nurture          │
         │  (cron: daily cold lead scan)      │
         └────────────────────────────────────┘

         ┌────────────────────────────────────┐
         │     task-creation-engine           │
         │  (creates sales tasks per stage)   │
         └────────────────────────────────────┘

         ┌────────────────────────────────────┐
         │     SLA-monitoring                 │
         │  (cron: hourly — check SLA breaches)│
         └────────────────────────────────────┘
```

---

## 8. N8N Credential & Connection Setup

### PostgreSQL Connection

```json
{
  "name": "Dream AI PostgreSQL",
  "type": "postgres",
  "host": "your-db-host",
  "port": 5432,
  "database": "dream_ai_crm",
  "user": "n8n_user",
  "password": "secure_password"
}
```

### Required N8N Credentials

| Credential | Type | Used By |
|-----------|------|---------|
| `Dream AI PostgreSQL` | PostgreSQL | All DB queries |
| `Clearbit API` | HTTP Header | Enrichment node |
| `SendGrid` | API Key | Email sequences |
| `Slack Sales` | OAuth2 | Lead alerts |
| `Twilio` | API Key | SMS alerts (hot leads) |
| `LinkedIn` | OAuth2 | Social automation |

### Webhook URLs to Register

| Webhook | URL | Source |
|---------|-----|--------|
| Lead capture (forms) | `https://n8n.dreamai.com/webhook/lead-capture` | Website, landing pages |
| Score update | `https://n8n.dreamai.com/webhook/score-update` | Analytics events |
| Demo booking | `https://n8n.dreamai.com/webhook/demo-booked` | Calendly/scheduling tool |
| Free trial signup | `https://n8n.dreamai.com/webhook/trial-signup` | Product |
| Re-engagement webhook | `https://n8n.dreamai.com/webhook/re-engage` | Email platform |
