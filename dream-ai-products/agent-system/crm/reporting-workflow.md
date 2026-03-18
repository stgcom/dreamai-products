# Dream AI Automated Reporting — Weekly Report Workflow

> N8N workflow for Monday 9 AM UTC weekly business report
> Last updated: 2026-03-18

---

## 1. Report Overview

| Field | Value |
|---|---|
| **Report** | Weekly Business Intelligence Report |
| **Frequency** | Every Monday |
| **Time** | 09:00 UTC |
| **Deliveries** | Email (HTML) + Telegram (Markdown) + Notion (archive) |
| **Owner** | METRIC (analytics agent) via SOVEREIGN routing |

---

## 2. Report Sections

### Section 1: Lead Generation Metrics

```
📊 LEAD GENERATION — Week of Mar 11–17, 2026
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Total Leads:        47    (↑ 12% vs last week)
Unique Visitors:    1,240 (↑ 5% vs last week)
Lead Rate:          3.8%  (↑ 0.3pp)

By Source:
  Calculator:     18  ██████████████████  38%
  Scorecard:      12  ████████████        26%
  Landing Page:    9  █████████           19%
  Referral:        5  █████               11%
  Organic:         3  ███                  6%

By Vertical:
  Real Estate:    16  ████████████████  34%
  MedSpa:         12  ████████████      26%
  HVAC:            9  █████████         19%
  Other:          10  ██████████        21%

Quality Distribution:
  Hot (70+):       8  (17%)
  Warm (50-69):   14  (30%)
  Cold (0-49):    25  (53%)
```

**N8N Query:**
```sql
-- HubSpot API equivalent via HTTP Request node
GET /crm/v3/objects/contacts
  properties: dream_ai_lead_source,dream_ai_lead_score,
              dream_ai_industry_vertical,createdate
  filters: createdate >= [week_start] AND createdate < [week_end]
Group by: dream_ai_lead_source, dream_ai_industry_vertical
Calculate: count, avg(dream_ai_lead_score), distribution buckets
```

### Section 2: Conversion Funnel Analysis

```
🔄 CONVERSION FUNNEL — Week of Mar 11–17
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Stage              Count    Stage Conv.    Cumulative
─────────────────────────────────────────────────────
Visitor           1,240        —             —
Lead                47       3.8%           3.8%
Qualified           18      38.3%           1.5%
Demo Booked         12      66.7%           1.0%
Proposal Sent        8      66.7%           0.6%
Closed Won           5      62.5%           0.4%

Funnel visualization:
  Visitors  ████████████████████████████████████████████████  1,240
  Leads     ██                                                47
  Qualified █                                                   18
  Demos     █                                                   12
  Proposals ▌                                                    8
  Won       ▌                                                    5

Week-over-Week:
  Visitor→Lead:     3.8% (↑ 0.3pp)
  Lead→Qualified:   38.3% (↓ 2.1pp) ⚠️ investigate
  Qualified→Demo:   66.7% (↑ 5.2pp) ✅
  Demo→Proposal:    66.7% (→)
  Proposal→Won:     62.5% (↑ 8.3pp) ✅

Biggest Drop-off: Lead → Qualified (81% don't qualify)
```

**Implementation:**
- Query contacts/deals created in the week
- Count how many progressed through each stage
- Calculate stage-over-stage conversion rate
- Compare to prior week for trend
- Flag any stage with >5pp drop

### Section 3: Revenue Breakdown by Product

```
💰 REVENUE — March 2026 (Week 2)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Week Revenue:     $9,994
Month to Date:    $18,492
Month Target:     $25,000
Run Rate:         $25,400 (on track ✅)

By Product:
  AI Employee    $9,994  ████████████████████████████  2 deals
  Starter Kit    $0      █                             0 deals
  Products       $4,997  ████████████                  1 deal
  Consulting     $5,000  █████████████                 1 deal
                        ─────────────────────────────────────
                        Total:      $18,492 (MTD)     4 deals

By Vertical:
  Real Estate    $9,994  ████████████████████████████  34%
  MedSpa         $4,997  ████████████                  17%
  HVAC           $4,997  ████████████                  17%
  Other          $0      —                              0%

MRR Impact:
  New MRR added:     $1,491/mo (3 new AI Employee customers)
  Total MRR:         $4,473/mo (9 active AI Employee subscriptions)
  Churn this month:  $497/mo (1 customer cancelled)
  Net MRR change:    +$994/mo
```

### Section 4: Customer Acquisition Cost (CAC)

```
💎 CAC ANALYSIS — March 2026
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Total Sales & Marketing Spend:  $8,500
  Paid Ads:          $4,000
  Content/SEO:       $1,500
  Tools (HubSpot, N8N, etc.): $1,000
  SDR (contract):    $2,000

New Customers Acquired:  7
CAC = $8,500 / 7 = $1,214

LTV (AI Employee):       $9,994 (avg 20-month retention × $497/mo)
LTV:CAC Ratio:            8.2x  ✅ (target >3x)

CAC by Channel:
  Calculator funnel:     $890   (best ROI) ✅
  Scorecard funnel:      $1,120
  Referral:              $450   (lowest cost) ✅
  Paid Ads:              $2,200

Payback Period: 2.4 months (avg)
```

**CAC Calculation Logic:**
```
For each channel in the week/month:
  1. Get total spend on that channel
  2. Count customers acquired from that channel
  3. CAC = spend / customers
  4. Compare across channels to identify best ROI
```

### Section 5: Churn Risk Indicators

```
⚠️ CHURN RISK — March 2026
━━━━━━━━━━━━━━━━━━━━━━━━━━━

Active Customers: 9
At-Risk Customers: 2 (22%)

HIGH RISK (Score >70):
  1. Bob Martinez (Cool Air HVAC)    Risk: 82
     • Payment failed 2x last month
     • No login in 14 days
     • Support tickets: 3 unresolved
     Action: PULSE outreach within 24h

  2. Lisa Wang (Glow MedSpa)         Risk: 75
     • Usage dropped 60% vs last month
     • Demo engagement declining
     Action: Schedule check-in call

MEDIUM RISK (Score 40-70):
  • 2 customers — monitor

HEALTHY (Score 0-39):
  • 5 customers — no action needed

Churn Rate: 1/10 customers this month (10%)
Churn Revenue Lost: $497/mo
```

**Churn Risk Scoring:**
| Signal | Points |
|---|---|
| Payment failure | +25 per failure |
| No login in 7+ days | +20 |
| Usage drop > 50% | +15 |
| Support ticket unresolved 48h+ | +10 |
| Cancel request / complaint | +30 |
| Reduced engagement (emails unopened) | +5 |

**Auto-escalation:** Score > 70 → alert PULSE agent immediately

### Section 6: Next Week Priorities

```
🎯 NEXT WEEK PRIORITIES (Mar 18–24)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Revenue Goals:
  • Close $6,508 to hit March target
  • 3 proposals in pipeline — prioritize follow-up
  • Launch AI Employee promo (MedSpa vertical)

Pipeline Priorities:
  • 8 hot leads need demo booking this week
  • 3 demos scheduled — prep and follow up same-day
  • Lead→Qualified conversion dropped 2.1pp — review qualification criteria

Retention:
  • Follow up with 2 at-risk customers
  • NPS survey to 5 healthy customers (get testimonials)

Marketing:
  • Publish calculator results case study (Real Estate)
  • Run MedSpa vertical ad test ($500 budget)
  • Update scorecard with spring seasonal data

System:
  • Review lead scoring weights (qualified conversion drop)
  • Test new webhook integration
  • Update email templates for proposal stage
```

**Priority generation logic:**
- Auto-flag overdue pipeline stages
- Identify gap to monthly target
- Pull at-risk customers from churn scoring
- Flag any conversion rate drops > 5pp
- Human review before sending (optional)

---

## 3. N8N Workflow Design

### Workflow: "Weekly Report — Monday 9AM UTC"

```
┌──────────────┐
│  Cron Trigger│  0 9 * * 1 (Every Monday 09:00 UTC)
└──────┬───────┘
       ▼
┌──────────────┐
│  Fetch Date  │  Calculate week_start, week_end, month_start
│  Parameters  │
└──────┬───────┘
       ▼
┌──────────────────────────────────────────────────────────────────┐
│                     Parallel Data Fetching                        │
│                                                                  │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐    │
│  │ Contacts   │ │ Deals      │ │ Revenue    │ │ Marketing  │    │
│  │ This Week  │ │ By Stage   │ │ This Month │ │ Spend      │    │
│  │ + History  │ │ + Products │ │ vs Target  │ │ By Channel │    │
│  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘    │
│        │              │              │              │             │
│        └──────────────┴──────────────┴──────────────┘             │
│                              │                                    │
└──────────────────────────────┼────────────────────────────────────┘
                               ▼
                    ┌─────────────────────┐
                    │  Data Transformation │
                    │  (Code Node)         │
                    │                      │
                    │  • Lead metrics      │
                    │  • Funnel analysis   │
                    │  • Revenue breakdown │
                    │  • CAC calculation   │
                    │  • Churn scoring     │
                    │  • Priority flags    │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │  Format Report      │
                    │                      │
                    │  • HTML (email)      │
                    │  • Markdown (Tg)     │
                    │  • JSON (Notion)     │
                    └──────────┬──────────┘
                               ▼
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
     │   Email      │ │  Telegram    │ │  Notion      │
     │   SendGrid   │ │  via SOVEREIGN│ │  Create Page │
     └──────────────┘ └──────────────┘ └──────────────┘
```

### Node Details

#### Node 1: Cron Trigger
```json
{
  "rule": {
    "interval": [{ "field": "cronExpression", "expression": "0 9 * * 1" }]
  }
}
```

#### Node 2: Date Parameters (Function Node)
```javascript
const now = new Date();
const weekEnd = new Date(now);
const weekStart = new Date(now);
weekStart.setDate(weekStart.getDate() - 7);

const monthStart = new Date(now.getFullYear(), now.getMonth(), 1);
const monthEnd = new Date(now.getFullYear(), now.getMonth() + 1, 0);

return {
  week_start: weekStart.toISOString(),
  week_end: weekEnd.toISOString(),
  month_start: monthStart.toISOString(),
  month_end: monthEnd.toISOString(),
  week_label: `${weekStart.toISOString().split('T')[0]} to ${weekEnd.toISOString().split('T')[0]}`,
  month_label: now.toLocaleDateString('en-US', { month: 'long', year: 'numeric' })
};
```

#### Node 3–6: Parallel Data Fetching (HubSpot API)

**Contacts query:**
```http
GET https://api.hubapi.com/crm/v3/objects/contacts
Authorization: Bearer {{HUBSPOT_TOKEN}}
Properties: email,firstname,lastname,company,dream_ai_lead_source,
            dream_ai_lead_score,dream_ai_industry_vertical,dream_ai_customer_status,
            dream_ai_purchase_date,createdate
Filter: createdate gte {{week_start}} AND createdate lt {{week_end}}
Limit: 200
```

**Deals query:**
```http
GET https://api.hubapi.com/crm/v3/objects/deals
Authorization: Bearer {{HUBSPOT_TOKEN}}
Properties: dealname,amount,dealstage,closedate,createdate,
            dream_ai_product_interest,dream_ai_mrr
Filter: createdate gte {{month_start}} OR closedate gte {{month_start}}
Limit: 500
```

**Historical contacts (for funnel):**
```http
GET https://api.hubapi.com/crm/v3/objects/contacts
Filter: createdate gte {{week_start}} AND createdate lt {{week_end}}
        AND dream_ai_customer_status IN (lead, prospect, customer)
```

**Marketing spend (static config or Google Ads API):**
```javascript
// Or pull from Google Ads / Meta Ads API
return {
  paid_ads: 4000,
  content_seo: 1500,
  tools: 1000,
  sdr: 2000
};
```

#### Node 7: Data Transformation (Code Node)

```javascript
const contacts = $node["Fetch Contacts"].json.results;
const deals = $node["Fetch Deals"].json.results;

// 1. Lead Generation Metrics
const bySource = {};
const byVertical = {};
contacts.forEach(c => {
  const src = c.properties.dream_ai_lead_source || 'unknown';
  const vert = c.properties.dream_ai_industry_vertical || 'other';
  bySource[src] = (bySource[src] || 0) + 1;
  byVertical[vert] = (byVertical[vert] || 0) + 1;
});

// 2. Funnel Analysis
// Count contacts at each stage
const stages = ['lead', 'prospect', 'customer'];
const funnel = {};
stages.forEach(s => {
  funnel[s] = contacts.filter(c =>
    c.properties.dream_ai_customer_status === s
  ).length;
};

// 3. Revenue Breakdown
const byProduct = {};
const closedDeals = deals.filter(d => d.properties.dealstage === 'closed_won');
closedDeals.forEach(d => {
  const prod = d.properties.dream_ai_product_interest || 'other';
  if (!byProduct[prod]) byProduct[prod] = { count: 0, revenue: 0 };
  byProduct[prod].count++;
  byProduct[prod].revenue += parseFloat(d.properties.amount) || 0;
});

// 4. CAC
const totalSpend = 8500;
const customersAcquired = closedDeals.length;
const cac = totalSpend / Math.max(customersAcquired, 1);

// 5. Churn Risk
const activeCustomers = contacts.filter(c =>
  c.properties.dream_ai_customer_status === 'customer'
);
const atRisk = activeCustomers.filter(c =>
  parseFloat(c.properties.dream_ai_churn_risk_score || 0) > 40
);

return [{
  leads: { total: contacts.length, by_source: bySource, by_vertical: byVertical },
  funnel,
  revenue: { by_product: byProduct, mtd_total: closedDeals.reduce((s, d) => s + (parseFloat(d.properties.amount) || 0), 0) },
  cac: { spend: totalSpend, customers: customersAcquired, ratio: cac },
  churn: { total: activeCustomers.length, at_risk: atRisk.length, details: atRisk }
}];
```

#### Node 8: Format HTML Email
- Use HubSpot HTML email template or SendGrid dynamic template
- Responsive design (mobile-first)
- Each section as a card with header + data
- Links to HubSpot CRM for drill-down
- "Reply to add notes" footer

#### Node 9: Send Email (SendGrid)
```json
{
  "to": "sales@dreamai.com",
  "subject": "📊 Dream AI Weekly Report — Mar 11–17, 2026",
  "html": "{{HTML_CONTENT}}",
  "attachments": ["{{CSV_EXPORT}}"]
}
```

#### Node 10: Send Telegram (HTTP Request)
```json
{
  "method": "POST",
  "url": "https://api.telegram.org/bot{{BOT_TOKEN}}/sendMessage",
  "body": {
    "chat_id": "686076918",
    "text": "{{MARKDOWN_SUMMARY}}",
    "parse_mode": "Markdown"
  }
}
```

#### Node 11: Save to Notion (HTTP Request)
```json
{
  "method": "POST",
  "url": "https://api.notion.com/v1/pages",
  "headers": {
    "Authorization": "Bearer {{NOTION_TOKEN}}",
    "Content-Type": "application/json"
  },
  "body": {
    "parent": { "database_id": "{{REPORTS_DB_ID}}" },
    "properties": {
      "Title": { "title": [{ "text": { "content": "Weekly Report — Mar 11–17" } }] },
      "Date": { "date": { "start": "2026-03-17" } },
      "Leads": { "number": 47 },
      "Revenue": { "number": 9994 },
      "Customers": { "number": 5 }
    },
    "children": [{ "object": "block", "type": "paragraph", "paragraph": { "rich_text": [{ "text": { "content": "..." } }] } }]
  }
}
```

---

## 4. Scheduling & Configuration

### Cron Schedule
```
Daily Brief:     0 8 * * *    (08:00 UTC every day)
Weekly Report:   0 9 * * 1    (09:00 UTC every Monday)
Monthly Review:  0 9 1 * *    (09:00 UTC, 1st of month)
```

### Environment Variables (N8N)
```
HUBSPOT_TOKEN=pat-na1-xxxxx
NOTION_TOKEN=ntn_xxxxx
NOTION_REPORTS_DB_ID=xxxxx
TELEGRAM_BOT_TOKEN=xxxxx
TELEGRAM_CHAT_ID=686076918
SENDGRID_API_KEY=SG.xxxxx
REPORT_EMAIL_TO=sales@dreamai.com
```

### Config: `reporting-config.json`
```json
{
  "targets": {
    "monthly_revenue": 25000,
    "weekly_leads": 50,
    "weekly_demos": 10,
    "cac_ceiling": 2000,
    "churn_ceiling": 5
  },
  "alerts": {
    "weekly_revenue_below_target_pct": 80,
    "cac_above_ceiling": true,
    "churn_risk_high": true,
    "funnel_drop_pp": 5
  },
  "products": [
    {"name": "AI Employee", "price": 4997, "mrr": 497},
    {"name": "Starter Kit", "price": 2497, "mrr": 0},
    {"name": "Products Bundle", "price": 4997, "mrr": 0},
    {"name": "Consulting", "price": 3500, "mrr": 0}
  ]
}
```

---

## 5. Implementation Checklist

- [ ] Create N8N workflow "Weekly Report"
- [ ] Configure HubSpot API connection
- [ ] Configure SendGrid API connection
- [ ] Configure Notion API connection
- [ ] Set up Telegram bot token
- [ ] Create cron triggers (daily + weekly + monthly)
- [ ] Build data transformation code nodes
- [ ] Create HTML email template (SendGrid dynamic template)
- [ ] Create Telegram markdown formatter
- [ ] Create Notion reports database
- [ ] Set up environment variables
- [ ] Test with mock data
- [ ] Run first real report
- [ ] Set up alert webhooks for threshold violations
- [ ] Document any customization needed for specific reports

---

## 6. Report Delivery Matrix

| Report | Email | Telegram | Notion | Frequency |
|---|---|---|---|---|
| Daily Sales Brief | ✅ | ✅ | ❌ | Daily 08:00 |
| Hot Leads Alert | ❌ | ✅ | ❌ | Real-time |
| Weekly Report | ✅ | ✅ | ✅ | Mon 09:00 |
| Monthly Review | ✅ | ✅ | ✅ | 1st of month |
| Churn Alert | ❌ | ✅ | ❌ | Real-time |
| Revenue Milestone | ❌ | ✅ | ❌ | Real-time |

---

*See also: [crm-spec.md](./crm-spec.md) | [sales-dashboard.md](./sales-dashboard.md)*
