# N8N Workflow Improvements for Dream AI Digital Products

## How N8N enhances each product — from delivery to customer experience to automation they receive.

---

## Product #1: Missed Call Calculator ($3)

### Workflow: Calculator → Email Sequence → Upsell Pipeline

```
[Webhook: Calculator Submitted]
  → [PostgreSQL: Store result + email + industry]
  → [IF score > $5,000/month lost]: Tag as "hot lead"
  → [Gmail: Send immediate results email]
  → [Wait 2 days]
  → [Gmail: "Did you see your number?" email with tip]
  → [Wait 3 days]
  → [Gmail: Case study email + Starter Kit offer]
  → [IF clicked Starter Kit link]: Notify in Telegram
  → [CRM: Update lead score + stage]
```

**Why this matters:**
- Automatic lead scoring (hot leads get instant Telegram alert)
- Zero manual email follow-up
- Every calculator user enters the funnel automatically
- Tracks conversion from free tool → paid product

**N8N nodes needed:** Webhook, PostgreSQL, IF, Gmail, Wait, Telegram, HTTP Request

---

## Product #2: AI Readiness Scorecard ($5)

### Workflow: Quiz → Segmented Nurture → Personalized Upsell

```
[Webhook: Quiz Completed]
  → [PostgreSQL: Store score, category breakdown, email]
  → [IF score 0-30]: Route to "Critical" nurture sequence
  → [IF score 31-60]: Route to "Developing" nurture sequence
  → [IF score 61-85]: Route to "Ready" nurture sequence
  → [IF score 86-100]: Route to "Advanced" nurture sequence
  → [Gmail: Send personalized results based on score band]
  → [Wait 2 days]
  → [Gmail: Weakest category deep-dive email]
  → [Wait 3 days]
  → [Gmail: Quick win implementation email]
  → [Wait 3 days]
  → [Gmail: Case study from their industry]
  → [Wait 4 days]
  → [IF score < 60]: Offer Starter Kit ($297)
  → [IF score > 60]: Offer Playbook ($27) → upsell Starter Kit
  → [Telegram: Alert for score > 70 — high-value lead]
```

**Why this matters:**
- Personalized nurture based on actual quiz results
- Different offers for different readiness levels
- Automatic identification of high-value leads
- Each email references their specific weak areas

**N8N nodes needed:** Webhook, PostgreSQL, IF (multi-branch), Gmail, Wait, Telegram, Switch

---

## Product #3: SMS Booking Template ($7)

### Workflow: Purchase → Setup Automation → Customer Onboarding

```
[Webhook: Stripe Purchase]
  → [HTTP: Create Twilio phone number via API]
  → [PostgreSQL: Store customer + purchase + phone number]
  → [Gmail: Send setup instructions + N8N workflow JSON]
  → [Wait 1 day]
  → [SMS: "Your booking system is live! Reply with 'test' to try it"]
  → [IF replied within 24h]: Log as "activated"
  → [IF no reply 48h]: Send reminder with video tutorial link
  → [Wait 7 days]
  → [Gmail: "How's your booking system going? Here's how to optimize"]
  → [IF activated]: Ask for feedback / review
  → [IF not activated]: Offer 15-min setup call
  → [Stripe: Issue refund if not activated within 14 days]
```

**Why this matters:**
- Reduces refund rate by ensuring activation
- Automated onboarding without human involvement
- Identifies struggling customers early
- Collects reviews from successful users

**N8N nodes needed:** Webhook (Stripe), PostgreSQL, Gmail, SMS, Wait, IF, HTTP Request

---

## Product #4: Script Pack ($12)

### Workflow: Delivery → Usage Tracking → Upsell

```
[Webhook: Stripe Purchase]
  → [PostgreSQL: Store customer + purchase]
  → [Gmail: Send script pack + "Copy to Clipboard" guide]
  → [Wait 3 days]
  → [Gmail: "Which scripts are you using? Here's how to customize them"]
  → [Wait 7 days]
  → [Gmail: "Want these scripts to run automatically? → AI Employee Starter Kit"]
  → [IF clicked link]: Create hot lead alert → Telegram
  → [Wait 14 days]
  → [Gmail: "The done-for-you version" — testimonial + Starter Kit offer]
  → [IF purchased Starter Kit]: Tag as upsell conversion
  → [PostgreSQL: Track: delivered, opened, upsell_click, upsell_convert]
```

**Why this matters:**
- Tracks which customers are actively using scripts
- Natural progression from DIY → done-for-you
- Upsell conversions tracked in PostgreSQL
- Low-friction → high-ticket path

**N8N nodes needed:** Webhook, PostgreSQL, Gmail, Wait, IF, Telegram, HTTP Request

---

## Product #5: Google Review Rocket Kit ($15)

### Workflow: Automated Review Collection (THE PRODUCT IS THE AUTOMATION)

```
[Webhook: Service Completed (from their CRM/scheduling)]
  → [PostgreSQL: Log customer + service + date]
  → [Wait 24 hours]
  → [SMS: Review request #1 — warm & personal]
  → [Wait 48 hours]
  → [IF review left]: Log success → Thank customer
  → [IF no review]: [Email: Follow-up #2 with direct link]
  → [Wait 72 hours]
  → [IF review left]: Log success
  → [IF no review]: [SMS: Last nudge with incentive — 10% off next visit]
  → [IF negative review]: [Telegram alert to business owner + draft response]
  → [PostgreSQL: Track conversion rate by customer]
  → [Weekly: Generate review collection report]
```

**Why this matters:**
- This IS the product — N8N automates what the kit teaches
- The product becomes a managed service ($147/mo)
- Negative review interception adds massive value
- Weekly reports show ROI

**N8N nodes needed:** Webhook, PostgreSQL, Wait, SMS, Email, IF, Telegram, Code (for report generation)

---

## Product #6: Real Estate AI Playbook ($27)

### Workflow: Delivery → Implementation Tracker → Managed Service Upsell

```
[Webhook: Stripe Purchase]
  → [PostgreSQL: Store agent + brokerage + CRM type]
  → [Gmail: Send playbook PDF + welcome video]
  → [Day 3]: [Gmail: "Week 1 checklist: Set up AI Receptionist"]
  → [Day 7]: [Gmail: "How's the setup going? Here's the CRM integration guide"]
  → [Day 14]: [Gmail: "Speed-to-Lead setup — here's the SMS template"]
  → [Day 21]: [Gmail: "You're halfway through! Client nurture setup"]
  → [Day 28]: [Gmail: "Final week: Social media automation"]
  → [Day 30]: [Gmail: "Implementation complete? Book a free optimization call"]
  → [IF calendar booking]: Create demo task → Notify Stephen
  → [PostgreSQL: Track: delivered, checklist opens, CTA clicks]
  → [Day 45]: [Gmail: "One month in — results check-in + managed service offer"]
```

**Why this matters:**
- Guided implementation increases completion rate
- Drip sequence keeps them engaged for 30 days
- Each email drives toward the managed service ($297/mo)
- Calendar booking = hot sales lead

**N8N nodes needed:** Webhook, PostgreSQL, Gmail, Wait, IF, Calendly API, Telegram

---

## Product #7: All Products — Post-Purchase Review Automation

### Workflow: Universal Review Collection (applies to ALL products)

```
[Webhook: Any Stripe Purchase]
  → [Wait 5 days]
  → [Email: "How's your [Product Name]? Quick review request"]
  → [IF positive reply]: [Email: Direct Google review link]
  → [IF negative reply]: [Telegram alert → Support ticket created]
  → [IF no reply 3 days]: [Email: Follow-up with value add (bonus tip)]
  → [PostgreSQL: Log review status]
  → [Monthly: Aggregate review data → Report to Stephen]
```

**Why this matters:**
- Every product sale becomes a review opportunity
- Negative feedback caught before public review
- Monthly review report shows brand health
- Social proof compounds over time

---

## Product #8: N8N Customer Dashboard (NEW PRODUCT — $47)

### Workflow: Give customers a live dashboard of their automations

```
[Webhook: Customer visits dashboard]
  → [PostgreSQL: Pull their metrics — reviews collected, leads captured, calls answered]
  → [IF metrics > target]: Show green "On Track" status
  → [IF metrics < target]: Show yellow "Needs Attention" + suggestions
  → [Render: Custom dashboard with their real data]
  → [Weekly: Auto-email dashboard summary]
  → [Monthly: Auto-generate PDF report]
```

**Why this matters:**
- Customers see ROI clearly → lower churn
- Could be sold as a standalone ($47/mo) or included in managed service
- Builds on the PostgreSQL data already being collected

---

## MASTER AUTOMATION: Dream AI Lead Pipeline

### The N8N workflow that runs Dream AI's entire business

```
[Trigger: Any lead source — Calculator, Scorecard, Website, LinkedIn, Referral]

1. CAPTURE
   → [Webhook receives lead data]
   → [PostgreSQL: Create lead record]
   → [Enrichment: Look up company, LinkedIn, review count]
   → [Score: Calculate lead quality (0-100)]

2. ROUTE
   → [IF score 80-100]: Immediate Telegram alert → Human follows up same hour
   → [IF score 50-79]: Add to "nurture" email sequence
   → [IF score 0-49]: Add to "awareness" email sequence

3. NURTURE
   → [Automated email sequence based on lead source + score]
   → [Track: opens, clicks, replies]
   → [IF engagement spike]: Upgrade lead score → re-route

4. CONVERT
   → [IF purchases product]: Activate onboarding workflow
   → [IF books demo]: Create sales task + prep brief
   → [IF goes cold 30 days]: Re-engagement sequence

5. RETAIN
   → [Post-purchase: Activate relevant product workflow]
   → [Day 7, 30, 90]: Check-in + review request
   → [IF health score drops]: Alert + intervention
   → [IF health score high]: Upsell opportunity flag

6. OPTIMIZE
   → [Weekly: Pipeline report — leads, conversions, revenue]
   → [Monthly: Funnel analysis — where are we leaking?]
   → [Quarterly: Channel ROI — which sources produce best customers?]
```

---

## N8N IMPLEMENTATION PRIORITY

| Priority | Workflow | Build Time | Revenue Impact |
|----------|----------|------------|----------------|
| P0 | Calculator → Email Sequence | 2 hours | High (lead gen) |
| P0 | Scorecard → Segmented Nurture | 3 hours | High (lead gen) |
| P0 | Master Lead Pipeline | 4 hours | Critical (core ops) |
| P1 | Review Collection (product delivery) | 3 hours | High (delivers value) |
| P1 | Post-Purchase Review Automation | 2 hours | Medium (social proof) |
| P2 | SMS Onboarding | 2 hours | Medium (reduce refunds) |
| P2 | Playbook Drip Sequence | 2 hours | Medium (upsell) |
| P3 | Customer Dashboard | 4 hours | Medium (retention) |

**Total build time: ~20 hours for complete automation stack**

---

## N8N + PostgreSQL Tables Needed

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `leads` | All captured leads | email, source, score, status, created_at |
| `customers` | Purchased products | email, product_id, purchase_date, activated |
| `calculator_results` | Calculator submissions | email, industry, missed_calls, revenue_lost |
| `scorecard_results` | Quiz completions | email, score, category_breakdown, band |
| `review_campaigns` | Review request tracking | customer_id, sms_sent, email_sent, review_left, rating |
| `nurture_sequences` | Email sequence tracking | customer_id, sequence_id, email_number, sent, opened, clicked |
| `lead_scores` | Scoring history | lead_id, score, source, calculated_at |
| `product_health` | Customer health tracking | customer_id, usage_score, engagement_score, health_band |

All these tables are compatible with our existing PostgreSQL MCP (`postgres-mcp`).
All workflows are importable as N8N JSON via our N8N MCP (`n8n-mcp`).
