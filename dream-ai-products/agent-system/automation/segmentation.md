# Dream AI — Customer Segmentation System

> Automated segmentation for lifecycle marketing, retention, and upsell
> Last updated: 2026-03-18

---

## 1. Segmentation by Value Tier

### Tier Definitions

| Tier | Revenue Range | Criteria | Count (est.) |
|---|---|---|---|
| **Entry** | $3 – $27 | Low-ticket digital products, lead magnets | — |
| **Starter** | $297 one-time | Starter kit purchase | — |
| **Growth** | $497/mo | Recurring service, monthly retainer | — |
| **Enterprise** | $1,497+ | Full deployment, multi-seat, custom | — |

### Segment Logic

```
IF total_lifetime_spend <= 27 → Entry
IF total_lifetime_spend >= 297 AND max_single_purchase = 297 → Starter
IF monthly_recurring_revenue >= 497 → Growth
IF monthly_recurring_revenue >= 1497 → Enterprise
```

### Value Tier Characteristics

| Attribute | Entry | Starter | Growth | Enterprise |
|---|---|---|---|---|
| Support level | Self-serve | Email | Priority | Dedicated |
| Onboarding | Automated | Light-touch | Guided | Full white-glove |
| Contact frequency | Weekly newsletter | Bi-weekly | Weekly check-in | Daily availability |
| Upsell target | Starter kit | Growth plan | Enterprise | Add-ons |

---

## 2. Segmentation by Behavior (Activity)

### Definitions

| Segment | Activity Window | Label Color |
|---|---|---|
| **Active** | Logged in / used product ≤ 7 days | 🟢 Green |
| **Engaged** | Last activity 8–30 days | 🟡 Yellow |
| **At Risk** | Last activity 31–60 days | 🟠 Orange |
| **Churned** | Last activity 61–90+ days | 🔴 Red |

### Segment Logic

```
days_since_last_activity = NOW() - last_login_at

IF days_since_last_activity <= 7   → Active
IF days_since_last_activity <= 30  → Engaged
IF days_since_last_activity <= 60  → At Risk
IF days_since_last_activity > 60   → Churned
```

### Behavior Segment Actions

| Segment | Action | Channel | Timing |
|---|---|---|---|
| Active | Feature education, ask for referral | In-app, email | Ongoing |
| Engaged | Re-engagement content, tips | Email | Day 8, Day 14 |
| At Risk | Win-back offer, check-in | Email + SMS | Day 31, Day 45 |
| Churned | Re-engagement survey, final offer | Email | Day 61, Day 90 |

---

## 3. Segmentation by Industry

### Industry Definitions

| Industry | Tag | Common Use Case | Key Metrics |
|---|---|---|---|
| **Real Estate** | `industry:real-estate` | Lead gen, CRM integration, follow-up automation | Leads, showings, closings |
| **MedSpa** | `industry:medspa` | Booking, reviews, patient follow-up, compliance | Appointments, reviews, retention |
| **HVAC** | `industry:hvac` | Dispatch, seasonal campaigns, maintenance plans | Bookings, maintenance renewal |
| **Other** | `industry:other` | General business automation | Custom per client |

### Industry Routing

| Industry | Assigned Agent | Compliance | Special Notes |
|---|---|---|---|
| Real Estate | Standard | — | IDX integration support |
| MedSpa | Standard + HIPAA | BAA required | PHI handling, consent flows |
| HVAC | Standard | — | Seasonal campaign templates |
| Other | Standard | Case-by-case | Custom scoping required |

---

## 4. Segmentation by Product Stage

### Stage Definitions

| Stage | Description | Revenue Impact |
|---|---|---|
| **Never Purchased** | Lead/prospect, no purchase history | $0 potential |
| **Entry Products Only** | Purchased low-ticket items only ($3-27) | Low — needs starter upsell |
| **Starter Kit Purchased** | Bought starter kit ($297) | Medium — upsell to Growth |
| **Recurring Service Customer** | Active monthly retainer ($497+/mo) | High — retain + expand |

### Stage Logic

```
IF purchase_count = 0 → Never Purchased
IF purchase_count > 0 AND max_purchase <= 27 → Entry Products Only
IF EXISTS purchase = 297 → Starter Kit Purchased
IF MRR >= 497 → Recurring Service Customer
```

### Stage Progression Funnel

```
[Lead] → [Entry Buyer] → [Starter] → [Growth] → [Enterprise]
  ↓           ↓              ↓           ↓            ↓
 Score     Upsell        Upsell     Retain+Expand  White-glove
          to Starter    to Growth     Account Mgmt   Custom Work
```

### Stage-Specific Campaigns

| From Stage | To Stage | Campaign | Trigger |
|---|---|---|---|
| Never Purchased | Entry | Lead magnet offer | Score > 70 |
| Entry Products Only | Starter | Starter kit pitch | Post-purchase day 3 |
| Starter Kit | Growth | Case study + ROI calc | 30 days post-purchase |
| Growth | Enterprise | Custom proposal | Usage > 80% capacity |
| Any → Churned | — | Win-back | 60 days inactive |

---

## 5. Automation Rules

### Rule 1: High Score + No Purchase → Sales Alert

```
TRIGGER: Lead score > 70 AND purchase_count = 0
ACTION:
  1. Tag lead as "hot prospect"
  2. Create task in sales queue
  3. Send alert to sales@dreamai (Telegram + email)
  4. Assign to next available sales rep
  5. Auto-send personalized intro within 1 hour

PRIORITY: High
SLA: Respond within 4 business hours
```

### Rule 2: Starter Kit + 30 Days → Upsell to Growth

```
TRIGGER: Product stage = "Starter Kit Purchased" AND days_since_purchase >= 30
ACTION:
  1. Generate ROI report from their usage data
  2. Send upsell email with Growth plan benefits
  3. Offer 10% first-month discount
  4. Schedule follow-up call if no response (day 37)
  5. Track: opened, clicked, converted

PRIORITY: Medium
CADENCE: Day 30 (email), Day 37 (call offer), Day 44 (final nudge)
```

### Rule 3: At Risk Segment → Win-Back Campaign

```
TRIGGER: Behavior segment = "At Risk" (31-60 days inactive)
ACTION:
  1. Send "We miss you" email with usage recap
  2. Day 45: SMS check-in ("Need help?")
  3. Day 52: Offer 15% discount or free consultation
  4. Day 60: Final email — survey ("Why did you go quiet?")
  5. Day 61: Move to Churned segment

PRIORITY: High
GOAL: Re-activate 15% of At Risk contacts
```

### Rule 4: Enterprise Segment → Priority Support

```
TRIGGER: Value tier = "Enterprise" OR MRR >= 1,497
ACTION:
  1. Tag account as "enterprise-priority"
  2. Route all support to priority queue
  3. Assign dedicated success contact
  4. Monthly check-in call scheduled
  5. Quarterly business review (QBR) auto-scheduled
  6. SLA: < 1 hour response during business hours

PRIORITY: Critical
REVENUE PROTECT: Enterprise accounts = highest LTV
```

---

## 6. Segmentation Data Schema

### Customer Record (minimum fields)

```json
{
  "customer_id": "uuid",
  "email": "string",
  "industry": "real-estate | medspa | hvac | other",
  "value_tier": "entry | starter | growth | enterprise",
  "behavior_segment": "active | engaged | at_risk | churned",
  "product_stage": "never_purchased | entry_only | starter | recurring",
  "total_lifetime_spend": 0.00,
  "monthly_recurring_revenue": 0.00,
  "last_login_at": "ISO8601",
  "first_purchase_at": "ISO8601",
  "lead_score": 0,
  "tags": ["string"],
  "automation_state": {
    "active_campaigns": ["string"],
    "last_campaign_at": "ISO8601",
    "next_scheduled_action": "ISO8601"
  },
  "created_at": "ISO8601",
  "updated_at": "ISO8601"
}
```

### Segment Calculation (daily batch)

```
Every day at 02:00 UTC:
1. Recalculate all segment assignments
2. Identify segment transitions
3. Trigger automation rules for transitions
4. Update dashboard counts
5. Flag any data quality issues
```

---

## 7. N8N Segmentation Workflows

### Workflow: Daily Segment Recalculation

| Step | Node | Action |
|---|---|---|
| 1 | Cron | Trigger at 02:00 UTC |
| 2 | Postgres | Query all customers |
| 3 | Code | Calculate segments per rules |
| 4 | Postgres | Update customer records |
| 5 | Code | Identify transitions |
| 6 | Switch | Route by transition type |
| 7 | HTTP/Telegram | Fire automation rules |

### Workflow: Real-Time Scoring

| Step | Node | Action |
|---|---|---|
| 1 | Webhook | Receive event (purchase, login, page view) |
| 2 | Postgres | Fetch customer record |
| 3 | Code | Recalculate score + segment |
| 4 | Postgres | Update record |
| 5 | IF | Check automation trigger conditions |
| 6 | Email/Telegram | Send alert if triggered |

---

## 8. KPI Tracking

| Metric | Target | Current | Segment Owner |
|---|---|---|---|
| Entry → Starter conversion | 15% | — | Marketing |
| Starter → Growth conversion | 20% | — | Sales |
| Growth → Enterprise conversion | 10% | — | Sales |
| At Risk → Active recovery | 15% | — | Success |
| Enterprise retention (annual) | 95% | — | Success |
| Churn recovery rate | 5% | — | Marketing |
| Sales alert response time | < 4h | — | Sales |

---

## Appendix: Segment Export Queries

```sql
-- Active Enterprise Customers
SELECT * FROM customers
WHERE value_tier = 'enterprise'
  AND behavior_segment = 'active'
ORDER BY monthly_recurring_revenue DESC;

-- At Risk Starter Customers (upsell candidates)
SELECT * FROM customers
WHERE value_tier = 'starter'
  AND behavior_segment = 'at_risk'
ORDER BY last_login_at ASC;

-- High Score + No Purchase (sales leads)
SELECT * FROM customers
WHERE lead_score > 70
  AND product_stage = 'never_purchased'
ORDER BY lead_score DESC;

-- Monthly Segment Distribution
SELECT
  value_tier,
  behavior_segment,
  product_stage,
  COUNT(*) as count,
  SUM(monthly_recurring_revenue) as total_mrr
FROM customers
GROUP BY value_tier, behavior_segment, product_stage
ORDER BY total_mrr DESC;
```
