# Dream AI CRM Specification

> Foundation: HubSpot Free CRM + N8N Automation Layer
> Last updated: 2026-03-18

---

## 1. Contact Properties

Custom properties added to HubSpot contacts. All prefixed with `dream_ai_` to avoid conflicts.

### Core Fields

| Property | Type | Options / Range | Notes |
|---|---|---|---|
| `dream_ai_lead_source` | Enumeration | `calculator`, `scorecard`, `landing_page`, `referral`, `organic`, `paid_ad`, `demo_request` | Set on first touch |
| `dream_ai_lead_score` | Number | 0–100 | Auto-calculated, updated daily via scoring sync |
| `dream_ai_industry_vertical` | Enumeration | `real_estate`, `med_spa`, `hvac`, `other` | Captured from lead form or enrichment |
| `dream_ai_product_interest` | Enumeration | `starter_kit`, `ai_employee`, `products`, `consulting`, `multiple` | Multi-select where applicable |
| `dream_ai_demo_booked` | Boolean | `true` / `false` | Set by calendar integration webhook |
| `dream_ai_purchase_date` | Date | ISO 8601 | Set on Closed Won |
| `dream_ai_customer_status` | Enumeration | `lead`, `prospect`, `customer`, `churned` | Master lifecycle field |
| `dream_ai_mrr` | Number | USD | Monthly recurring value for customers |
| `dream_ai_deal_value` | Number | USD | One-time product revenue |
| `dream_ai_last_score_update` | DateTime | ISO 8601 | Audit trail for scoring freshness |
| `dream_ai_nurture_sequence` | Enumeration | `none`, `industry_re`, `industry_medspa`, `industry_hvac`, `general`, `re-engagement` | Tracks which email sequence contact is in |
| `dream_ai_churn_risk_score` | Number | 0–100 | Calculated from engagement + payment history |

### Scoring Model (Lead Score 0–100)

Points accumulate from multiple signals:

| Signal | Points | Source |
|---|---|---|
| Visited calculator page | +5 | Website tracking |
| Completed calculator | +15 | Form submission |
| Visited scorecard page | +5 | Website tracking |
| Completed scorecard | +20 | Form submission |
| Downloaded lead magnet | +10 | N8N workflow |
| Attended demo | +25 | Calendar webhook |
| Replied to email | +10 | Email tracking |
| Opened 3+ emails | +5 | Email tracking |
| Visited pricing page | +10 | Website tracking |
| Referred by existing customer | +15 | Referral form |
| Industry vertical match (top 3) | +10 | Manual/enrichment |
| Company size > 10 employees | +5 | Enrichment API |
| Engaged on social | +3 | Social tracking |

**Scoring tiers:**
- 0–30: Cold — nurture sequence
- 31–50: Warm — qualify via SDR
- 51–70: Hot — demo invitation
- 71–85: Very Hot — priority follow-up, same day
- 86–100: VIP — founder/exec outreach

---

## 2. Pipeline Stages

### Pipeline: "Dream AI Sales Pipeline"

```
New Lead → Qualified → Demo Booked → Proposal Sent → Closed Won → Onboarding
```

### Stage Details

#### Stage 1: New Lead
- **Entry:** Contact created via webhook or form submission
- **Exit criteria:** Lead score ≥ 31 AND industry vertical identified AND contacted (email/call sent)
- **Max dwell time:** 3 days (auto-escalate if exceeded)
- **Automation triggers:**
  - Send welcome email sequence (industry-specific if vertical known)
  - Assign lead source tag
  - Add to N8N daily scoring queue
  - Notify SDR if score ≥ 50

#### Stage 2: Qualified
- **Entry:** SDR confirms BANT (Budget, Authority, Need, Timeline)
- **Exit criteria:** Demo scheduled (calendar booking confirmed)
- **Max dwell time:** 7 days
- **Automation triggers:**
  - Send qualification summary to Slack/Telegram
  - Create demo prep task in Notion
  - Enrich contact with company data (Clearbit/Apollo)
  - Trigger industry-specific nurture if demo not booked in 48h

#### Stage 3: Demo Booked
- **Entry:** Calendar booking confirmed
- **Exit criteria:** Demo completed AND follow-up sent
- **Max dwell time:** 5 days (demo must happen)
- **Automation triggers:**
  - 24h and 1h demo reminder emails
  - Send pre-demo questionnaire
  - Create demo recording task
  - After demo: send summary email with CTA

#### Stage 4: Proposal Sent
- **Entry:** Proposal delivered (email or DocuSign)
- **Exit criteria:** Contract signed OR deal lost
- **Max dwell time:** 14 days
- **Automation triggers:**
  - Proposal viewed tracking (email open/click)
  - 3-day follow-up reminder if no response
  - 7-day escalation to founder
  - Send case studies / social proof if proposal stalled

#### Stage 5: Closed Won
- **Entry:** Contract signed / payment received
- **Exit criteria:** Handoff to Onboarding complete
- **Automation triggers:**
  - Set `dream_ai_purchase_date`
  - Update `dream_ai_customer_status` → `customer`
  - Create onboarding project in Notion
  - Send welcome package email
  - Remove from nurture sequences
  - Add to customer Slack/Telegram channel
  - Generate invoice if applicable
  - Update MRR / revenue tracking

#### Stage 6: Onboarding
- **Entry:** Deal closed, onboarding started
- **Exit criteria:** Onboarding checklist complete, customer activated
- **Max dwell time:** 14 days
- **Automation triggers:**
  - Daily onboarding status check
  - NPS survey at day 7
  - Auto-transition to customer success (PULSE agent)
  - Set `dream_ai_churn_risk_score` = 0

### Lost Deal Handling
- At any stage, if deal is marked Lost:
  - Log reason (too expensive, wrong timing, competitor, no response, other)
  - Set `dream_ai_customer_status` = `lead` (stay in nurture)
  - Add to re-engagement sequence (90-day drip)
  - Re-evaluate in 90 days

---

## 3. N8N Integration

### Webhook Endpoints

All webhooks use HMAC-SHA256 signature in `X-DreamAI-Signature` header for verification.

#### Webhook 1: New Lead → Create Contact

```
POST /webhook/dreamai-new-lead
```

**Payload:**
```json
{
  "email": "prospect@example.com",
  "firstName": "Jane",
  "lastName": "Doe",
  "company": "Acme Realty",
  "phone": "+15551234567",
  "leadSource": "calculator",
  "industryVertical": "real_estate",
  "productInterest": "ai_employee",
  "sourceUrl": "https://dreamai.com/calculator",
  "metadata": {
    "calculatorResult": "recommended_ai_employee",
    "monthlyRevenue": "150000",
    "employeeCount": "12"
  }
}
```

**Workflow:**
1. Verify HMAC signature
2. Check for duplicate (email match)
3. Create/update HubSpot contact with all properties
4. Set initial lead score based on source (+15 for calculator, etc.)
5. Send to SDR queue if score ≥ 50
6. Trigger welcome email sequence
7. Log to Notion lead database
8. Return 200 with contact ID

#### Webhook 2: Score Update → Update Contact

```
POST /webhook/dreamai-score-update
```

**Payload:**
```json
{
  "email": "prospect@example.com",
  "newScore": 72,
  "scoreDelta": +15,
  "reason": "completed_scorecard",
  "signals": [
    {"signal": "completed_scorecard", "points": 20, "timestamp": "2026-03-18T21:00:00Z"},
    {"signal": "visited_pricing", "points": 10, "timestamp": "2026-03-18T20:45:00Z"}
  ]
}
```

**Workflow:**
1. Verify signature
2. Update `dream_ai_lead_score` on HubSpot contact
3. Update `dream_ai_last_score_update` timestamp
4. If score crosses tier threshold (30, 50, 70, 85):
   - Send alert to SDR Telegram
   - Auto-move to next pipeline stage if appropriate
   - Trigger priority follow-up workflow
5. Log score change to Notion

#### Webhook 3: Purchase → Move to Customer

```
POST /webhook/dreamai-purchase
```

**Payload:**
```json
{
  "email": "prospect@example.com",
  "product": "ai_employee",
  "dealValue": 4997,
  "mrr": 497,
  "purchaseDate": "2026-03-18",
  "paymentMethod": "stripe",
  "invoiceId": "inv_abc123"
}
```

**Workflow:**
1. Verify signature
2. Update HubSpot deal → Closed Won
3. Set `dream_ai_purchase_date`, `dream_ai_mrr`, `dream_ai_deal_value`
4. Set `dream_ai_customer_status` → `customer`
5. Create onboarding project in Notion
6. Send welcome package email
7. Trigger revenue reporting update
8. Remove from nurture sequences
9. Notify PULSE agent (client success)

#### Webhook 4: Demo Booked

```
POST /webhook/dreamai-demo-booked
```

**Payload:**
```json
{
  "email": "prospect@example.com",
  "demoDate": "2026-03-20T15:00:00Z",
  "demoType": "product_walkthrough",
  "calendarEventId": "evt_xyz789"
}
```

**Workflow:**
1. Update contact `dream_ai_demo_booked` → true
2. Move deal to "Demo Booked" stage
3. Send confirmation email with calendar link
4. Send pre-demo questionnaire
5. Schedule reminder (24h + 1h before)
6. Create demo prep task for presenter

### Daily: Lead Scoring Sync

**Cron:** Every day at 06:00 UTC

**Workflow:**
1. Query all HubSpot contacts where `dream_ai_customer_status` = `lead` or `prospect`
2. For each contact, recalculate score from:
   - Website activity (last 7 days)
   - Email engagement (opens, clicks, replies)
   - Form submissions
   - Social engagement
   - Decay: -2 points per day of inactivity (floor at current base score from form data)
3. Update HubSpot if score changed
4. Generate daily report:
   - New hot leads (score ≥ 70)
   - Leads going cold (score dropped ≥ 10)
   - Leads exceeding dwell time limits
5. Send report to SDR team (Telegram + email)

---

## 4. Data Flow Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Dream AI Website                      │
│  (Calculator / Scorecard / Landing Pages / Blog)        │
└───────────────────┬─────────────────────────────────────┘
                    │ Form submission / page visit
                    ▼
┌─────────────────────────────────────────────────────────┐
│                   N8N Webhook Layer                      │
│  /webhook/dreamai-new-lead                              │
│  /webhook/dreamai-score-update                          │
│  /webhook/dreamai-purchase                              │
│  /webhook/dreamai-demo-booked                           │
└───────┬─────────────┬───────────────┬───────────────────┘
        │             │               │
        ▼             ▼               ▼
┌──────────┐  ┌──────────────┐  ┌──────────────┐
│ HubSpot  │  │    Notion    │  │   Telegram   │
│   CRM    │  │  (NotionDB)  │  │   Alerts     │
└────┬─────┘  └──────────────┘  └──────────────┘
     │
     │  Daily scoring sync
     ▼
┌─────────────────────────────────────────────────────────┐
│              Daily/Weekly Reporting Cron                 │
│  (sales-dashboard.md / reporting-workflow.md)           │
└─────────────────────────────────────────────────────────┘
```

---

## 5. Implementation Checklist

- [ ] Create HubSpot custom properties (12 fields)
- [ ] Create HubSpot deal pipeline with 6 stages
- [ ] Set up N8N instance (dreamain8n.zeabur.app)
- [ ] Create 4 webhook workflows in N8N
- [ ] Configure HMAC signature verification
- [ ] Build daily scoring sync cron job
- [ ] Set up HubSpot contact/deal creation workflows
- [ ] Configure Telegram alerts via SOVEREIGN
- [ ] Build Notion lead database
- [ ] Test end-to-end: calculator → lead → qualified → demo → won
- [ ] Set up email templates (welcome, demo prep, follow-up, proposal, won)
- [ ] Configure reporting workflows (see sales-dashboard.md)

---

*Next: See [sales-dashboard.md](./sales-dashboard.md) and [reporting-workflow.md](./reporting-workflow.md)*
