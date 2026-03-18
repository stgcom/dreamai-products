# Dream AI Sales Dashboard & Daily Report

> Daily automated email + Telegram report for sales visibility
> Last updated: 2026-03-18

---

## 1. Report Overview

| Report | Frequency | Delivery | Time (UTC) |
|---|---|---|---|
| Daily Sales Brief | Daily | Email + Telegram | 08:00 |
| Hot Leads Alert | Real-time | Telegram | Immediate |
| Weekly Report | Weekly (Mon) | Email + Telegram | 09:00 |

---

## 2. Daily Sales Brief — 08:00 UTC

### Section A: New Leads Today

```
📊 NEW LEADS — Wed, Mar 18
━━━━━━━━━━━━━━━━━━━━━━━━━
Total: 12 new leads

  Calculator:      ████████████  5 (42%)
  Scorecard:       ██████        3 (25%)
  Landing Page:    ████          2 (17%)
  Referral:        ██            1 (8%)
  Organic:         ██            1 (8%)

  Avg Lead Score: 34
  Hottest: jane@realty.com (Score: 68)
```

**Data source:** HubSpot contacts created in last 24h, grouped by `dream_ai_lead_source`

**Implementation (N8N):**
```
Trigger: Schedule — 08:00 UTC daily
Action: HubSpot API — Get contacts created_date = today
Group by: dream_ai_lead_source
Calculate: count, avg(dream_ai_lead_score), max()
Format: markdown → send via email + Telegram
```

### Section B: Hot Leads Needing Follow-Up

```
🔥 HOT LEADS (Score >70) — 3 need follow-up
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  1. Mike Chen (Acme Realty)     Score: 82  Source: Calculator
     Status: New Lead — 2 days in stage ⚠️
     Action: Schedule demo call

  2. Sarah Kim (Glow MedSpa)     Score: 76  Source: Scorecard
     Status: Qualified — awaiting demo booking
     Action: Send demo calendar link

  3. Tom Rivera (Cool Air HVAC)  Score: 71  Source: Landing Page
     Status: New Lead — just entered
     Action: Send intro email
```

**Data source:** HubSpot contacts where `dream_ai_lead_score` > 70 AND `dream_ai_customer_status` ≠ `customer`
Sorted by score desc, flagging contacts exceeding stage dwell time

**Hot Leads Alert (Real-time):**
Triggered immediately when any contact's score crosses 70. Sends to Telegram with full context.

### Section C: Demos Booked This Week

```
📅 DEMOS THIS WEEK
━━━━━━━━━━━━━━━━━━
  This week:  8 demos booked
  Completed:  5  ✅
  Upcoming:   3  📆

  Tomorrow:
    • 10:00 UTC — Jane Doe (Acme Realty) — AI Employee demo
    • 14:00 UTC — Sarah Kim (Glow MedSpa) — Products overview

  Week-over-week: +25% (6 → 8)
```

**Data source:** HubSpot deals in "Demo Booked" stage + calendar events

### Section D: Pipeline Value by Stage

```
💰 PIPELINE VALUE
━━━━━━━━━━━━━━━━━
  New Lead        $14,900  ████               (5 leads)
  Qualified       $29,940  ████████           (8 leads)
  Demo Booked     $24,985  ██████             (5 demos)
  Proposal Sent   $19,980  █████              (3 proposals)
  ─────────────────────────────────────────
  Total Pipeline: $89,805  21 opportunities

  Weighted Pipeline: $38,400 (probability-adjusted)
```

**Data source:** HubSpot deals by stage × deal value. Weighted = stage probability × deal value.

**Stage probabilities:**
| Stage | Close Probability |
|---|---|
| New Lead | 10% |
| Qualified | 25% |
| Demo Booked | 40% |
| Proposal Sent | 65% |
| Closed Won | 100% |

### Section E: Revenue This Month vs Target

```
📈 REVENUE — March 2026
━━━━━━━━━━━━━━━━━━━━━━━
  Target:   $25,000
  Closed:   $18,492  ████████████████░░░░  74%
  Pipeline: $29,940  (potential adds)

  Deals Closed: 4
    • AI Employee ×2  @ $4,997 each
    • Starter Kit ×1  @ $2,497
    • Products Bundle ×1 @ $4,997

  To target: $6,508 needed (1-2 more deals)
```

**Data source:** HubSpot deals closed this month (date >= 1st of month). Target set in config.

### Section F: Top Converting Channel

```
🏆 TOP CHANNELS — 30-Day Conversion Rate
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  1. Calculator      12.3%  ██████████████████████  (8 closed / 65 leads)
  2. Scorecard       8.7%   ██████████████          (5 closed / 58 leads)
  3. Referral        7.2%   ███████████             (3 closed / 42 leads)
  4. Landing Page    3.1%   █████                   (2 closed / 64 leads)
  5. Paid Ads        1.8%   ███                     (1 closed / 55 leads)
```

**Data source:** HubSpot contacts grouped by `dream_ai_lead_source`, filtered to closed won in last 30 days, divided by total leads from that source in last 30 days.

### Section G: Products Sold

```
🛒 PRODUCTS SOLD — This Month
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  AI Employee:       3 units   $14,991
  Starter Kit:       2 units   $4,994
  Products Bundle:   1 unit    $4,997
  Consulting:        1 unit    $3,500
  ─────────────────────────────────
  Total Revenue:     $28,482
```

**Data source:** HubSpot deals with product line items, grouped by `dream_ai_product_interest`

---

## 3. N8N Daily Report Workflow

### Workflow: "Daily Sales Brief"

```
┌──────────────┐    ┌─────────────────┐    ┌──────────────┐
│   Cron       │───▶│  HubSpot API    │───▶│  Format      │
│  08:00 UTC   │    │  Query Contact  │    │  Report      │
└──────────────┘    │  & Deal Data    │    └──────┬───────┘
                    └─────────────────┘           │
                         ┌────────────────────────┼─────────────────┐
                         ▼                        ▼                 ▼
                  ┌──────────────┐    ┌────────────────┐   ┌──────────────┐
                  │  Email       │    │   Telegram     │   │  Notion      │
                  │  (HTML)      │    │   (Markdown)   │   │  Log Entry   │
                  └──────────────┘    └────────────────┘   └──────────────┘
```

### N8N Nodes Detail

**Node 1: Cron Trigger**
- Schedule: `0 8 * * *` (08:00 UTC daily)

**Node 2: HubSpot — Fetch Contacts**
```
GET /crm/v3/objects/contacts
Query params:
  properties: dream_ai_lead_source,dream_ai_lead_score,
              dream_ai_industry_vertical,dream_ai_customer_status,
              createdate,email,firstname,lastname,company
  filters: dream_ai_customer_status IN (lead, prospect)
           AND createdate >= today 00:00 UTC
Limit: 200
```

**Node 3: HubSpot — Fetch Deals**
```
GET /crm/v3/objects/deals
Query params:
  properties: dealname,amount,dealstage,closedate,pipeline,
              dream_ai_product_interest
  filters: dealstage IN (all stages)
           AND createdate >= start_of_month OR closedate >= start_of_month
Limit: 500
```

**Node 4: HubSpot — Fetch Hot Leads**
```
GET /crm/v3/objects/contacts
Query params:
  properties: dream_ai_lead_score,dream_ai_industry_vertical,
              dream_ai_lead_source,dream_ai_customer_status,email,
              firstname,lastname,company
  filters: dream_ai_lead_score > 70
           AND dream_ai_customer_status IN (lead, prospect)
Sort: dream_ai_lead_score DESC
Limit: 50
```

**Node 5: Data Transformation (Code node)**
- Group contacts by source
- Calculate conversion rates (30-day window)
- Build pipeline value by stage
- Calculate revenue vs target
- Format product sales breakdown

**Node 6: Format HTML Email**
- Responsive HTML template
- Sections A through G
- Include "View in HubSpot" links
- Attach CSV export option

**Node 7: Send Email**
- To: sales@dreamai.com (configurable)
- Subject: `📊 Dream AI Daily Sales Brief — [DATE]`
- Format: HTML

**Node 8: Send Telegram**
- Channel: SOVEREIGN → Telegram (target: 686076918)
- Format: Markdown (optimized for mobile)
- Include key metrics only (A, B, E)
- Full report available via link

---

## 4. Configuration

### Config File: `dashboard-config.json`

```json
{
  "targets": {
    "monthly_revenue": 25000,
    "quarterly_revenue": 75000,
    "monthly_leads": 150,
    "monthly_demos": 30
  },
  "thresholds": {
    "hot_lead_score": 70,
    "warm_lead_score": 50,
    "max_new_lead_dwell_days": 3,
    "max_qualified_dwell_days": 7,
    "max_demo_dwell_days": 5,
    "max_proposal_dwell_days": 14
  },
  "reporting": {
    "daily_email_to": ["sales@dreamai.com"],
    "daily_telegram_target": "686076918",
    "weekly_report_day": "monday",
    "weekly_report_hour_utc": 9
  },
  "stage_probabilities": {
    "new_lead": 0.10,
    "qualified": 0.25,
    "demo_booked": 0.40,
    "proposal_sent": 0.65,
    "closed_won": 1.00,
    "onboarding": 1.00
  }
}
```

---

## 5. Implementation Checklist

- [ ] Create N8N workflow "Daily Sales Brief"
- [ ] Configure HubSpot API credentials in N8N
- [ ] Set up cron trigger (08:00 UTC daily)
- [ ] Build data aggregation nodes (contacts + deals)
- [ ] Create HTML email template
- [ ] Set up Telegram delivery via SOVEREIGN
- [ ] Configure `dashboard-config.json`
- [ ] Test with 30 days of sample data
- [ ] Set up real-time hot lead alert trigger
- [ ] Create Notion logging node for report history

---

*See also: [crm-spec.md](./crm-spec.md) | [reporting-workflow.md](./reporting-workflow.md)*
