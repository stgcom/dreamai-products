# Dream AI — Lead Scoring System Implementation Guide

> **Goal:** Score every inbound lead from 0–100 based on engagement signals, automatically update HubSpot, alert Stephen on hot leads via Telegram, and decay scores for inactive contacts.
>
> **Prerequisites:** Basic computer skills. No CRM or coding experience required.

---

## Table of Contents

1. [HubSpot Free CRM Setup](#1-hubspot-free-crm-setup)
2. [Create Custom Contact Properties](#2-create-custom-contact-properties)
3. [Lead Scoring Rules & Point Values](#3-lead-scoring-rules--point-values)
4. [Temperature Thresholds](#4-temperature-thresholds)
5. [Lead Decay (Inactivity Penalty)](#5-lead-decay-inactivity-penalty)
6. [N8N Workflow: Automated Scoring Engine](#6-n8n-workflow-automated-scoring-engine)
7. [Daily Scoring Sync Cron Job](#7-daily-scoring-sync-cron-job)
8. [Telegram Alerts for Hot Leads](#8-telegram-alerts-for-hot-leads)
9. [Testing & Validation](#9-testing--validation)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. HubSpot Free CRM Setup

### 1a. Create Your HubSpot Account

1. Go to **https://www.hubspot.com/products/crm**
2. Click **"Get started free"**
3. Enter your business email (use **hello@dream-ai.com** or Stephen's email)
4. Fill in:
   - **Company name:** Dream AI
   - **Company size:** 1–10
   - **Your role:** Founder / CEO
5. Click **"Get my free account"**
6. Check your email for verification — click the link
7. Skip the onboarding wizard (we'll configure manually)

### 1b. Enable Website Tracking (Required for Page Visit Scoring)

1. In HubSpot, go to **Settings** (gear icon, top right)
2. Navigate to **Tracking & Analytics** → **Tracking Code**
3. Click **"Copy code"**
4. Add this code to every page on your website (before `</head>`)
5. If using a CMS (WordPress, Webflow, etc.), use their "custom code" or "head scripts" feature
6. Verify it works: visit your site, then check **Contacts** in HubSpot — you should see a "Website Visitors" section

### 1c. Enable Email Tracking

1. Go to **Settings** → **Sales** → **Email**
2. Enable **"Track email opens"** and **"Track link clicks"**
3. Connect your email account (Gmail/Outlook) if using HubSpot's email tools

---

## 2. Create Custom Contact Properties

These are the custom fields we add to HubSpot contacts. Follow these steps **exactly** — each property must be created in order.

### How to Create a Custom Property

For each property below:
1. In HubSpot, click the **Settings** (gear icon) → **Data Management** → **Properties**
2. Click **"Create property"** (top right)
3. Fill in the fields as specified below
4. Click **"Create"**

---

### Property 1: Lead Score

| Field | Value |
|---|---|
| **Object type** | Contact |
| **Group** | Contact information |
| **Label** | Dream AI Lead Score |
| **Internal name** | `dream_ai_lead_score` (auto-filled) |
| **Description** | Automated lead score 0-100. Higher = more engaged. |
| **Field type** | Number |
| **Format** | Whole number |
| **Minimum value** | 0 |
| **Maximum value** | 100 |

---

### Property 2: Lead Source

| Field | Value |
|---|---|
| **Object type** | Contact |
| **Group** | Contact information |
| **Label** | Dream AI Lead Source |
| **Internal name** | `dream_ai_lead_source` |
| **Description** | How this lead first found Dream AI |
| **Field type** | Enumeration (dropdown) |
| **Options** | (add each one below) |

**Dropdown options to add:**
1. `calculator` — Missed Call Calculator
2. `scorecard` — AI Readiness Scorecard
3. `landing-page` — Landing Page
4. `referral` — Customer Referral
5. `direct` — Direct / Organic

---

### Property 3: Industry Vertical

| Field | Value |
|---|---|
| **Object type** | Contact |
| **Group** | Contact information |
| **Label** | Dream AI Industry Vertical |
| **Internal name** | `dream_ai_industry_vertical` |
| **Description** | Lead's business vertical |
| **Field type** | Enumeration (dropdown) |
| **Options** | (add each one below) |

**Dropdown options to add:**
1. `real-estate` — Real Estate
2. `medspa` — Med Spa
3. `hvac` — HVAC
4. `other` — Other

---

### Property 4: Last Engagement Date

| Field | Value |
|---|---|
| **Object type** | Contact |
| **Group** | Contact activity |
| **Label** | Dream AI Last Engagement |
| **Internal name** | `dream_ai_last_engagement_date` |
| **Description** | Date of last known engagement (page visit, email open, etc.) |
| **Field type** | Date |
| **Format** | Show date only |

---

### Property 5: Hot Lead

| Field | Value |
|---|---|
| **Object type** | Contact |
| **Group** | Contact information |
| **Label** | Dream AI Hot Lead |
| **Internal name** | `dream_ai_hot_lead` |
| **Description** | TRUE if lead score > 70. Auto-set by scoring system. |
| **Field type** | Checkbox (true/false) |
| **Default value** | false |

---

### Verify All Properties

Go to **Settings** → **Data Management** → **Properties** and search for `dream_ai_`. You should see:

- ☐ `dream_ai_lead_score` (Number)
- ☐ `dream_ai_lead_source` (Dropdown)
- ☐ `dream_ai_industry_vertical` (Dropdown)
- ☐ `dream_ai_last_engagement_date` (Date)
- ☐ `dream_ai_hot_lead` (Checkbox)

---

## 3. Lead Scoring Rules & Point Values

### Scoring Table

| Signal | Points | Triggered By |
|---|---|---|
| **Missed Call Calculator completed** | **+20** | Form submission on calculator page |
| **AI Readiness Scorecard completed** | **+25** | Form submission on scorecard page |
| **Demo booked** | **+30** | Calendar integration webhook |
| **Reply to email** | **+20** | Email reply detected |
| **Pricing page visited** | **+15** | Website tracking |
| **Link clicked (in email)** | **+10** | Email link click tracking |
| **Landing page visited** | **+5** | Website tracking on key pages |
| **Email opened** | **+5** | Email open tracking |
| **Website visit (any page)** | **+3** | Website tracking pixel |

### Scoring Rules

1. **Score floors and ceilings:** Minimum score is 0, maximum is 100.
2. **One-time vs repeatable:**
   - **One-time events** (calculator completed, scorecard completed, demo booked): points awarded once per contact. If the same event fires again, no additional points.
   - **Repeatable events** (email opened, link clicked, website visit): points awarded each time, but capped at 3 occurrences per day to prevent gaming.
3. **Score is additive:** All signals stack. A lead who completes the calculator (+20), visits the pricing page (+15), and opens 3 emails (+15) reaches 50.
4. **Order doesn't matter:** Signals are processed as they arrive, but the final score is the sum of all qualifying signals minus decay.

### How Scores Flow

```
User Action (website/email)
        ↓
N8N receives webhook or detects event
        ↓
N8N calculates new score
        ↓
N8N updates HubSpot contact via API
        ↓
HubSpot stores score in dream_ai_lead_score
        ↓
If score > 70: N8N sends Telegram alert to Stephen
```

---

## 4. Temperature Thresholds

Once scored, leads fall into one of four temperature buckets:

| Score Range | Temperature | What It Means | Action |
|---|---|---|---|
| **0–30** | ❄️ **Cold** | Minimal engagement. Maybe visited once, no meaningful interaction. | Add to nurture email sequence. Do not contact directly. Score will decay further if inactive. |
| **31–50** | 🌡️ **Warm** | Some interest shown. Visited multiple pages or completed one tool. | SDR should review weekly. Continue nurture sequence. Monitor for score increase. |
| **51–70** | 🔥 **Hot** | Strong interest. Completed a tool OR visited multiple high-intent pages. | SDR should reach out within 24 hours. Send demo invitation. |
| **71+** | 🚨 **Sales Ready** | Highly engaged. Multiple high-intent actions. Ready to buy. | **Alert Stephen immediately via Telegram.** Same-day outreach required. Move to "Qualified" pipeline stage. |

### Threshold Actions (Automated)

These happen automatically via the N8N scoring workflow:

- **Score crosses 31 (Cold → Warm):** Update contact note: "Warmed up — review for outreach"
- **Score crosses 51 (Warm → Hot):** Send notification to SDR queue. Tag contact for priority review.
- **Score crosses 71 (Hot → Sales Ready):** Send **Telegram alert to Stephen** (see Section 8). Auto-set `dream_ai_hot_lead = true`. Move to "Qualified" pipeline stage if not already.
- **Score drops below 31 (Warm → Cold):** Return to nurture sequence. Clear any priority tags.

---

## 5. Lead Decay (Inactivity Penalty)

**Rule:** Leads lose **−5 points per 7 days of inactivity.**

### How Inactivity Is Calculated

1. `dream_ai_last_engagement_date` is updated every time a contact interacts (page visit, email open, link click, form submission, reply).
2. The daily scoring sync (Section 7) checks each contact's `dream_ai_last_engagement_date`.
3. If today minus last engagement ≥ 7 days, subtract 5 points.
4. If 14 days of inactivity: −10 points (−5 × 2 seven-day periods).
5. If 21 days: −15 points. And so on.
6. **Floor:** Score never goes below 0.

### Decay Examples

| Lead | Last Engagement | Days Inactive | Current Score | After Decay |
|---|---|---|---|---|
| Alice | Today | 0 | 45 | 45 (no decay) |
| Bob | 8 days ago | 8 | 45 | 40 (−5) |
| Carol | 15 days ago | 15 | 45 | 35 (−10, two periods) |
| Dave | 22 days ago | 22 | 45 | 30 (−15, three periods) |

### Decay Guardrails

- **Minimum score floor:** 0 (scores don't go negative)
- **Protected statuses:** Contacts marked as `customer` are never decayed
- **Re-engagement resets decay:** Any new engagement resets the clock and prevents further decay
- **Decay doesn't trigger threshold alerts:** A lead dropping from 75 to 65 due to decay does NOT trigger a "went cold" alert — only genuine downward movement does

---

## 6. N8N Workflow: Automated Scoring Engine

This is the core automation. It lives on **https://dreamain8n.zeabur.app**.

### How to Build It in N8N

Each step below is a "node" in the N8N visual workflow builder. You drag and connect them left to right.

---

### Workflow: `Dream AI — Lead Scoring Engine`

#### Step 1: Webhook Trigger Node

| Setting | Value |
|---|---|
| **Node type** | Webhook |
| **Name** | `Lead Event Webhook` |
| **HTTP Method** | POST |
| **Path** | `dreamai-score-event` |
| **Authentication** | Header Auth |
| **Header Name** | `X-DreamAI-Signature` |
| **Response Code** | 200 |

**What it does:** Receives scoring events from your website, email system, or other tools. This is the entry point for all scoring signals.

**Expected payload format:**
```json
{
  "email": "lead@example.com",
  "event": "calculator_completed",
  "timestamp": "2026-03-18T21:00:00Z",
  "metadata": {
    "source_url": "https://dreamai.com/calculator",
    "industry": "real-estate"
  }
}
```

#### Step 2: HMAC Signature Verification Node

| Setting | Value |
|---|---|
| **Node type** | Code (JavaScript) |
| **Name** | `Verify Signature` |

**Code:**
```javascript
const crypto = require('crypto');

const secret = $env.DREAM_AI_WEBHOOK_SECRET;
const signature = $input.header('X-DreamAI-Signature');
const body = JSON.stringify($input.body);

const expected = crypto.createHmac('sha256', secret).update(body).digest('hex');

if (signature !== expected) {
  throw new Error('Invalid signature');
}

return $input.body;
```

**What it does:** Validates that the webhook came from a trusted source. Rejects tampered or unauthorized requests.

#### Step 3: Lookup Contact in HubSpot

| Setting | Value |
|---|---|
| **Node type** | HubSpot |
| **Name** | `Find Contact` |
| **Operation** | Search |
| **Resource** | Contact |
| **Search Field** | Email |
| **Value** | `{{ $json.email }}` |

**What it does:** Finds the existing HubSpot contact by email. If no contact is found, the workflow creates one (see Step 3b).

#### Step 3b: Create Contact If Not Found (Conditional Branch)

| Setting | Value |
|---|---|
| **Node type** | IF |
| **Name** | `Contact Exists?` |
| **Condition** | HubSpot Contact ID exists |

**If TRUE (contact exists):** Continue to Step 4.

**If FALSE (new lead):** Route to HubSpot → Create Contact node:

| Setting | Value |
|---|---|
| **Node type** | HubSpot |
| **Name** | `Create Contact` |
| **Operation** | Create |
| **Resource** | Contact |
| **Email** | `{{ $json.email }}` |
| **First Name** | `{{ $json.firstName }}` (if provided) |
| **Last Name** | `{{ $json.lastName }}` (if provided) |
| **dream_ai_lead_source** | `{{ $json.leadSource }}` (default: `direct`) |
| **dream_ai_industry_vertical** | `{{ $json.industry }}` (default: `other`) |
| **dream_ai_lead_score** | `0` (starting score) |
| **dream_ai_hot_lead** | `false` |
| **dream_ai_last_engagement_date** | `{{ $json.timestamp }}` |

#### Step 4: Score Calculator Node

| Setting | Value |
|---|---|
| **Node type** | Code (JavaScript) |
| **Name** | `Calculate New Score` |

**Code:**
```javascript
// ─── SCORING RULES ───
const SCORE_MAP = {
  'calculator_completed':  20,
  'scorecard_completed':   25,
  'demo_booked':           30,
  'email_reply':           20,
  'pricing_page_visit':    15,
  'link_clicked':          10,
  'landing_page_visit':     5,
  'email_opened':           5,
  'website_visit':          3
};

// Get event data
const event = $json.event;
const currentScore = $('Find Contact').item.json.properties.dream_ai_lead_score || 0;

// Look up points for this event
const eventPoints = SCORE_MAP[event] || 0;

// Calculate new score (cap at 100, floor at 0)
const newScore = Math.min(100, Math.max(0, currentScore + eventPoints));

return {
  email: $json.email,
  event: event,
  pointsEarned: eventPoints,
  previousScore: currentScore,
  newScore: newScore,
  timestamp: $json.timestamp || new Date().toISOString(),
  industry: $json.metadata?.industry || null
};
```

**What it does:** Looks up the event type in the scoring table, adds points to the current score, and caps at 100.

#### Step 5: Update HubSpot Contact

| Setting | Value |
|---|---|
| **Node type** | HubSpot |
| **Name** | `Update Lead Score` |
| **Operation** | Update |
| **Resource** | Contact |
| **Contact ID** | `{{ $('Find Contact').item.json.id }}` |
| **Properties to update:** | |
| `dream_ai_lead_score` | `{{ $json.newScore }}` |
| `dream_ai_last_engagement_date` | `{{ $json.timestamp }}` |

**What it does:** Pushes the new score and engagement date to HubSpot.

#### Step 6: Check Threshold (IF Node)

| Setting | Value |
|---|---|
| **Node type** | IF |
| **Name** | `Sales Ready?` |
| **Condition** | `{{ $json.newScore }} >= 71` |

**If TRUE (score ≥ 71):** Route to Telegram Alert (Step 7).
**If FALSE:** End workflow (or route to Hot Lead check).

#### Step 6b: Hot Lead Check (for score 51–70)

| Setting | Value |
|---|---|
| **Node type** | IF |
| **Name** | `Hot Lead?` |
| **Condition** | `{{ $json.newScore }} >= 51 AND {{ $json.previousScore }} < 51` |

**If TRUE (just crossed into Hot):** Update HubSpot `dream_ai_hot_lead = true`, send SDR notification.
**If FALSE:** End workflow.

#### Step 7: Send Telegram Alert

| Setting | Value |
|---|---|
| **Node type** | HTTP Request |
| **Name** | `Telegram Alert` |
| **Method** | POST |
| **URL** | `https://api.telegram.org/bot{{ $env.TELEGRAM_BOT_TOKEN }}/sendMessage` |
| **Body (JSON):** | See format below |

**Alert Message Format:**
```json
{
  "chat_id": "686076918",
  "text": "🚨 *Sales Ready Lead*\n\n👤 {{ $json.email }}\n📊 Score: {{ $json.newScore }} (was {{ $json.previousScore }})\n📈 +{{ $json.pointsEarned }} from: {{ $json.event }}\n🏢 Industry: {{ $json.industry || 'Unknown' }}\n⏰ {{ $json.timestamp }}\n\n→ Immediate outreach required.",
  "parse_mode": "Markdown"
}
```

#### Step 8: Set HubSpot Hot Lead Flag

| Setting | Value |
|---|---|
| **Node type** | HubSpot |
| **Name** | `Mark Hot Lead` |
| **Operation** | Update |
| **Resource** | Contact |
| **dream_ai_hot_lead** | `true` |

---

### Complete Workflow Diagram

```
[Webhook] → [Verify Signature] → [Find Contact]
                                         ↓
                                   [Contact Exists?]
                                    ↓          ↓
                                 TRUE        FALSE
                                  ↓          ↓
                           [Calculate]   [Create Contact]
                              Score           ↓
                                  ↓       [Calculate]
                                  ↓          Score
                                  ↓           ↓
                                  └───── [Update HubSpot]
                                              ↓
                                     [Sales Ready? (≥71)]
                                       ↓           ↓
                                     TRUE        FALSE
                                      ↓           ↓
                                [Telegram     [Hot? (≥51)]
                                 Alert]        ↓       ↓
                                  ↓          TRUE    FALSE
                               [Mark Hot       ↓      ↓
                                Lead]     [Mark Hot  [END]
                                          Lead]
```

---

## 7. Daily Scoring Sync Cron Job

### Purpose

Every day, recalculate ALL lead scores to apply **lead decay** (−5 per 7 inactive days) and generate a **daily lead report**.

### How to Set Up in N8N

#### Workflow: `Dream AI — Daily Scoring Sync`

**Trigger:** Cron node

| Setting | Value |
|---|---|
| **Node type** | Cron |
| **Name** | `Daily 6AM UTC` |
| **Trigger** | Every day at 06:00 UTC |

---

#### Step 1: Fetch All Leads from HubSpot

| Setting | Value |
|---|---|
| **Node type** | HubSpot |
| **Name** | `Get All Leads` |
| **Operation** | Search / List |
| **Resource** | Contact |
| **Filter** | `dream_ai_customer_status` is NOT `customer` |
| **Properties to retrieve** | `email, dream_ai_lead_score, dream_ai_last_engagement_date, dream_ai_industry_vertical, dream_ai_hot_lead` |
| **Limit** | 500 (page through if more) |

---

#### Step 2: Calculate Decay (Code Node)

| Setting | Value |
|---|---|
| **Node type** | Code (JavaScript) |
| **Name** | `Apply Decay` |

**Code:**
```javascript
const results = [];
const now = new Date();

for (const contact of $input.all()) {
  const lastEngagement = new Date(contact.json.properties.dream_ai_last_engagement_date);
  const currentScore = contact.json.properties.dream_ai_lead_score || 0;

  // Calculate days since last engagement
  const daysInactive = Math.floor((now - lastEngagement) / (1000 * 60 * 60 * 24));

  // Calculate decay: -5 points per full 7-day period
  const decayPeriods = Math.floor(daysInactive / 7);
  const decayPoints = decayPeriods * 5;
  const newScore = Math.max(0, currentScore - decayPoints);

  // Only update if score changed
  const scoreChanged = newScore !== currentScore;

  results.push({
    contactId: contact.json.id,
    email: contact.json.properties.email,
    previousScore: currentScore,
    newScore: newScore,
    daysInactive: daysInactive,
    decayApplied: decayPoints,
    scoreChanged: scoreChanged,
    wasHotLead: contact.json.properties.dream_ai_hot_lead || false,
    nowCold: newScore < 71 && (contact.json.properties.dream_ai_hot_lead || false)
  });
}

return results;
```

---

#### Step 3: Batch Update HubSpot (Only Changed Scores)

| Setting | Value |
|---|---|
| **Node type** | IF |
| **Name** | `Score Changed?` |
| **Condition** | `{{ $json.scoreChanged }}` is TRUE |

**If TRUE:** Route to HubSpot Update node:

| Setting | Value |
|---|---|
| **Node type** | HubSpot |
| **Name** | `Update Decayed Score` |
| **Operation** | Update |
| **Contact ID** | `{{ $json.contactId }}` |
| **dream_ai_lead_score** | `{{ $json.newScore }}` |
| **dream_ai_hot_lead** | `{{ $json.nowCold ? false : $json.wasHotLead }}` |

**If FALSE:** Skip (no HubSpot API call needed).

---

#### Step 4: Generate Daily Report

| Setting | Value |
|---|---|
| **Node type** | Code (JavaScript) |
| **Name** | `Build Report` |

**Code:**
```javascript
const allContacts = $input.all().map(c => c.json);

const report = {
  date: new Date().toISOString().split('T')[0],
  totalContacts: allContacts.length,
  scoresUpdated: allContacts.filter(c => c.scoreChanged).length,
  salesReady: allContacts.filter(c => c.newScore >= 71).length,
  hot: allContacts.filter(c => c.newScore >= 51 && c.newScore < 71).length,
  warm: allContacts.filter(c => c.newScore >= 31 && c.newScore < 51).length,
  cold: allContacts.filter(c => c.newScore < 31).length,
  newSalesReady: allContacts.filter(c => c.newScore >= 71 && c.previousScore < 71).length,
  wentCold: allContacts.filter(c => c.nowCold).length,
  topLeads: allContacts
    .filter(c => c.newScore >= 51)
    .sort((a, b) => b.newScore - a.newScore)
    .slice(0, 10)
    .map(c => `  ${c.email}: ${c.newScore} (was ${c.previousScore})`)
};

return report;
```

---

#### Step 5: Send Daily Report to Telegram

| Setting | Value |
|---|---|
| **Node type** | HTTP Request |
| **Name** | `Daily Report` |
| **Method** | POST |
| **URL** | `https://api.telegram.org/bot{{ $env.TELEGRAM_BOT_TOKEN }}/sendMessage` |

**Message format:**
```json
{
  "chat_id": "686076918",
  "text": "📊 *Daily Lead Scoring Report — {{ $json.date }}*\n\n🔢 Contacts scored: {{ $json.totalContacts }}\n🎯 Scores updated: {{ $json.scoresUpdated }}\n\n🚨 Sales Ready (71+): {{ $json.salesReady }}\n🔥 Hot (51-70): {{ $json.hot }}\n🌡️ Warm (31-50): {{ $json.warm }}\n❄️ Cold (0-30): {{ $json.cold }}\n\n📈 New Sales Ready today: {{ $json.newSalesReady }}\n📉 Dropped from Hot: {{ $json.wentCold }}\n\n🏆 *Top Leads:*\n{{ $json.topLeads.join('\\n') }}\n\n→ Review and act.",
  "parse_mode": "Markdown"
}
```

---

## 8. Telegram Alerts for Hot Leads

### Real-Time Alert (Sales Ready — Score > 70)

When any lead's score crosses 71, Stephen gets an **immediate** Telegram alert:

```
🚨 Sales Ready Lead

👤 jane@acmerealty.com
📊 Score: 75 (was 55)
📈 +20 from: calculator_completed
🏢 Industry: Real Estate
⏰ 2026-03-18T21:00:00Z

→ Immediate outreach required.
```

### Alert Triggers

| Event | Condition | Alert Type |
|---|---|---|
| Lead becomes Sales Ready | Score crosses 71 (newScore ≥ 71, previousScore < 71) | 🚨 Immediate Telegram |
| Lead drops from Hot | Score drops below 71 AND was previously flagged Hot | 📉 Daily report only |
| Daily summary | Every day at 06:00 UTC | 📊 Daily report |
| New Sales Ready in daily sync | Score ≥ 71 found in daily sync (wasn't caught by real-time) | 🚨 Included in daily report |

### Bot Setup (One-Time)

1. Open Telegram, search for **@BotFather**
2. Send `/newbot`
3. Name: `Dream AI Alerts`
4. Username: `dream_ai_alerts_bot`
5. Copy the bot token (looks like `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`)
6. Add the token to N8N environment variable: `TELEGRAM_BOT_TOKEN`
7. Start a chat with your bot (search for it and send `/start`)
8. Get your chat ID:
   - Visit `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
   - Find the `chat.id` in the response (for Stephen: `686076918`)

---

## 9. Testing & Validation

### Test Checklist

Run these tests after setup:

#### Test 1: Custom Properties Exist
- [ ] Go to HubSpot → Contacts → click any contact
- [ ] Scroll to find: Dream AI Lead Score, Lead Source, Industry Vertical, Last Engagement, Hot Lead
- [ ] All 5 properties should appear

#### Test 2: Webhook Receives Events
- [ ] In N8N, open the "Lead Scoring Engine" workflow
- [ ] Click "Test workflow" to activate listening
- [ ] Use curl or Postman to send:
```bash
curl -X POST https://dreamain8n.zeabur.app/webhook/dreamai-score-event \
  -H "Content-Type: application/json" \
  -H "X-DreamAI-Signature: <your-hmac-signature>" \
  -d '{"email":"test@example.com","event":"calculator_completed","timestamp":"2026-03-18T21:00:00Z"}'
```
- [ ] Check HubSpot — contact should have `dream_ai_lead_score = 20`

#### Test 3: Score Adds Up
- [ ] Send multiple events for the same test contact:
  - `website_visit` → score should be 3
  - `pricing_page_visit` → score should be 18 (3+15)
  - `email_opened` → score should be 23 (18+5)
- [ ] Verify cumulative score in HubSpot

#### Test 4: Sales Ready Alert Fires
- [ ] Send events that push a test lead past 70:
  - Start at 50, add `demo_booked` (+30) = 80
- [ ] Check Telegram — you should receive the 🚨 alert

#### Test 5: Daily Decay Works
- [ ] Manually set a contact's `dream_ai_last_engagement_date` to 8 days ago in HubSpot
- [ ] Run the Daily Scoring Sync workflow manually in N8N
- [ ] Score should drop by 5 points

#### Test 6: Lead Source & Industry Tracked
- [ ] Create a new contact via webhook with `leadSource: "calculator"` and `industry: "real-estate"`
- [ ] Verify the dropdown properties are set correctly in HubSpot

---

## 10. Troubleshooting

### Problem: Scores not updating in HubSpot

**Check:**
1. N8N workflow is active (toggle is ON, not in test mode)
2. HubSpot API key/credentials are valid in N8N (Settings → Credentials)
3. Contact exists in HubSpot (search by email)
4. Webhook is being called (check N8N execution log)

### Problem: Telegram alert not sending

**Check:**
1. `TELEGRAM_BOT_TOKEN` is set in N8N environment
2. Bot has been started by Stephen (send `/start` to the bot)
3. Chat ID `686076918` is correct
4. Check N8N execution log for the HTTP Request node

### Problem: Decay over-penalizing leads

**Check:**
1. `dream_ai_last_engagement_date` is being updated on every event (not just some)
2. Daily sync runs at 06:00 UTC — verify cron is active
3. Check the Contact's date format in HubSpot (should be ISO 8601)

### Problem: Multiple duplicate contacts

**Check:**
1. Webhook handler should check for existing contact by email before creating
2. HubSpot has built-in duplicate detection — enable it in Settings → Data Management → Data Quality
3. Standardize email format (lowercase, trimmed) before sending to webhook

---

## Quick Reference Card

| What | Where | When |
|---|---|---|
| Lead Score (0-100) | HubSpot contact property | Updated on every event + daily sync |
| Sales Ready alert | Telegram (Stephen) | Instant when score > 70 |
| Daily report | Telegram (Stephen) | 06:00 UTC every day |
| Decay | Daily sync cron | −5 pts per 7 inactive days |
| N8N workflows | dreamain8n.zeabur.app | Scoring engine + daily sync |
| Scoring rules | See Section 3 | Never change without Stephen's approval |

---

## Maintenance Notes

- **Monthly:** Review scoring rules. Are certain signals over/under-weighted? Adjust +5/-5 max with Stephen's approval.
- **Quarterly:** Check if new engagement signals need scoring (webinar attendance, case study downloads, etc.).
- **Never:** Change the Sales Ready threshold (71) without Stephen's explicit approval — this controls who gets immediate alerts.

---

*Last updated: 2026-03-18*
*Owner: SOVEREIGN (Dream AI Operating System)*
*Related: [crm-spec.md](./crm-spec.md)*
