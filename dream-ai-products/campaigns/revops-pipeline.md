# Dream AI — Revenue Operations Pipeline Design

> **Product:** AI Employee Platform (AI receptionists, AI SDRs, AI schedulers)
> **ICP:** SMEs in Real Estate, MedSpa, HVAC — 2-50 employees, $500K-$10M revenue
> **GTM Motion:** Sales-led with PLG triggers (free trial / demo-first)
> **Target ACV:** $3,600-$36,000/yr ($300-$3,000/mo)
> **Target Sales Cycle:** 7-21 days (SMB velocity)

---

## 1. Lead Lifecycle Stages

### Stage Definitions

| # | Stage | Entry Criteria | Exit Criteria | Owner | Max Duration |
|---|-------|---------------|---------------|-------|-------------|
| 0 | **Unknown** | Cookie tracked, no identity | Provides email or form fill | Marketing (auto) | ∞ |
| 1 | **Aware** | Email captured (blog sub, lead magnet, free tool) | Shows engagement OR firmographic fit confirmed | Marketing | 30 days |
| 2 | **Engaged** | 2+ engagement actions (page visits, email opens, content downloads) OR firmographic fit + 1 engagement | Score reaches MQL threshold OR goes dormant | Marketing | 14 days |
| 3 | **MQL** | Lead score ≥ 60 AND firmographic fit score ≥ 40 AND engagement score ≥ 20 | Sales accepts (converted to SQL) or rejects within SLA | Marketing → Sales | 48 hours |
| 4 | **SQL** | Sales rep accepts, confirms fit via conversation, qualifies BANT-lite | Discovery call booked OR recycled | Sales (SDR/AE) | 7 days |
| 5 | **Demo** | Discovery completed, pain points confirmed, demo scheduled | Demo delivered, evaluation initiated | Sales (AE) | 14 days |
| 6 | **Proposal** | Pricing discussed, decision-makers identified, proposal requested | Proposal delivered and reviewed | Sales (AE) | 10 days |
| 7 | **Won** | Contract signed, payment method on file, onboarding triggered | — | Sales → CS | — |
| 8 | **Lost** | Deal closed-lost with reason code | — | Sales | — |

### Lifecycle Stage Table (PostgreSQL)

```sql
CREATE TYPE lead_stage AS ENUM (
  'unknown', 'aware', 'engaged', 'mql', 'sql',
  'demo', 'proposal', 'won', 'lost'
);

CREATE TYPE loss_reason AS ENUM (
  'price', 'no_budget', 'no_decision_maker', 'timing',
  'competitor', 'no_need', 'no_response', 'other'
);

CREATE TYPE deal_source AS ENUM (
  'organic_search', 'paid_google', 'paid_meta', 'paid_linkedin',
  'referral', 'partner', 'outbound_email', 'outbound_linked',
  'webinar', 'event', 'content_download', 'free_tool',
  'direct', 'other'
);

CREATE TYPE niche AS ENUM ('real_estate', 'medspa', 'hvac', 'other');
```

---

## 2. Lead Scoring Model (0-100)

Scoring is split into two independent dimensions:

| Dimension | Max Score | Weight | Purpose |
|-----------|-----------|--------|---------|
| **Firmographic Fit** | 40 | ICP match | "Is this the right customer?" |
| **Engagement Score** | 60 | Buying intent | "Are they ready to buy?" |
| **Total** | 100 | Combined | MQL threshold = **60** |

### 2A. Firmographic Fit Score (0-40)

| Attribute | Points | Rationale |
|-----------|--------|-----------|
| **Industry match** | | |
| → Real Estate / MedSpa / HVAC | +10 | Core verticals |
| → Adjacent (dental, plumbing, insurance) | +5 | Adjacent verticals |
| → Other | +0 | Not ICP |
| **Company size** | | |
| → 2-20 employees | +8 | Sweet spot |
| → 21-50 employees | +6 | Good fit |
| → 51-200 employees | +4 | Possible, longer cycle |
| → 1-employee or 200+ | +0 | Usually not right |
| **Revenue range** | | |
| → $500K-$10M | +8 | Target revenue band |
| → $250K-$500K or $10M-$25M | +4 | Borderline |
| → Other | +0 | Outside range |
| **Decision-maker title** | | |
| → Owner / CEO / Founder | +8 | Direct decision maker |
| → Ops Manager / GM / Office Manager | +6 | Influencer / champion |
| → Individual contributor / admin | +2 | Not enough authority |
| → Student / intern / personal email | -5 | Disqualifying signal |
| **Location (US-based)** | | |
| → US-based business | +6 | Primary market |
| → Canada | +3 | Secondary market |
| → Other | +0 | Out of scope |

### 2B. Engagement Score (0-60)

**High-intent actions (buying signals):**

| Action | Points | Decay |
|--------|--------|-------|
| Requested a demo / booked meeting | +25 | 30-day decay |
| Visited pricing page | +15 | 14-day decay |
| Used free AI employee tool / trial | +20 | 21-day decay |
| Requested a quote / ROI calc | +15 | 14-day decay |
| Attended a webinar / live demo | +10 | 14-day decay |
| Watched 75%+ of a product video | +8 | 14-day decay |

**Medium-intent actions (research phase):**

| Action | Points | Decay |
|--------|--------|-------|
| Downloaded case study | +6 | 14-day decay |
| Visited pricing page (repeat, 2nd+) | +5 | 7-day decay |
| Opened sales outreach email | +4 | 7-day decay |
| Clicked link in outreach email | +6 | 7-day decay |
| Visited "how it works" / product page | +4 | 14-day decay |
| LinkedIn profile engaged (reaction/comment) | +3 | 7-day decay |

**Low-intent actions (awareness):**

| Action | Points | Decay |
|--------|--------|-------|
| Blog post visit | +1 | 3-day decay |
| Content download (ebook, guide) | +2 | 7-day decay |
| Newsletter signup | +1 | 14-day decay |
| Homepage visit | +1 | 3-day decay |
| Social media follow | +1 | 30-day decay |

**Negative scoring (deductions):**

| Signal | Points | Notes |
|--------|--------|-------|
| Competitor email domain | -20 | Hard disqualify |
| Student / personal email (gmail.com etc.) | -5 | Unless firmographic fit is strong |
| Unsubscribed from emails | -10 | Active rejection |
| No engagement in 30+ days | -15 | Stale lead penalty |
| Rejected by sales (bad fit) | -25 | Recycle to nurture |

### Scoring Decay Rules

- Scores decay **daily** based on the decay rate listed above
- A lead's score is the **sum of all active (non-decayed) signals**
- Minimum score is **0** (never negative from decay alone)
- Recalculated every **6 hours** via N8N workflow

### MQL Threshold

```
MQL = (Firmographic Fit ≥ 25) AND (Engagement Score ≥ 35) AND (Total Score ≥ 60)
```

Both dimensions must meet their minimum — prevents perfectly-targeted but completely passive leads from becoming MQLs.

---

## 3. Lead Routing Rules

### 3A. Routing Priority Matrix

Routing is evaluated in order. First match wins.

| Priority | Rule | Route To | Rationale |
|----------|------|----------|-----------|
| 1 | **Free trial signup + scored > 40** | Product specialist (round-robin) | PLG-assisted selling |
| 2 | **Niche: Real Estate** | Real Estate AE team (round-robin) | Vertical expertise |
| 3 | **Niche: MedSpa** | MedSpa AE team (round-robin) | Vertical expertise |
| 4 | **Niche: HVAC** | HVAC AE team (round-robin) | Vertical expertise |
| 5 | **Deal size > $2,000/mo (estimated)** | Senior AE (round-robin) | Higher-touch for bigger deals |
| 6 | **Outbound-sourced leads** | Original prospecting SDR | Attribution to outreach owner |
| 7 | **Referral leads** | Referring partner's assigned AE | Maintain partner relationship |
| 8 | **All other MQLs** | General AE pool (round-robin) | Default catch-all |

### 3B. Routing Rules (N8N Logic)

```
IF lead.source = 'free_trial' AND lead.score > 40
  → route('product_specialists', round_robin)

ELSE IF lead.niche = 'real_estate'
  → route('ae_real_estate', round_robin)

ELSE IF lead.niche = 'medspa'
  → route('ae_medspa', round_robin)

ELSE IF lead.niche = 'hvac'
  → route('ae_hvac', round_robin)

ELSE IF lead.estimated_acv > 2000  -- monthly
  → route('ae_senior', round_robin)

ELSE IF lead.source = 'referral'
  → route(ae_for_partner[lead.partner_id])

ELSE IF lead.outbound_owner IS NOT NULL
  → route(lead.outbound_owner)

ELSE
  → route('ae_general', round_robin)
```

### 3C. Round-Robin Configuration

```sql
CREATE TABLE sales_reps (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('sdr', 'ae', 'senior_ae', 'ae_manager')),
  teams TEXT[] NOT NULL,  -- e.g., {'ae_real_estate', 'ae_general'}
  is_active BOOLEAN DEFAULT true,
  max_open_deals INT DEFAULT 20,
  current_open_deals INT DEFAULT 0,
  out_of_office_until TIMESTAMPTZ,
  last_assigned_at TIMESTAMPTZ
);
```

Round-robin respects:
- **Active status** (skip inactive/out-of-office)
- **Capacity** (skip reps at max open deals)
- **Last assigned** (assign to rep with oldest `last_assigned_at`)

### 3D. Unassigned Lead Fallback

If no rep matches routing rules or all reps are at capacity:
1. Alert sales manager via Slack/email
2. Assign to `ae_manager` as fallback
3. Auto-create task: "Assign lead within 2 hours"

---

## 4. MQL → SQL Handoff Criteria

### Acceptance Criteria (Sales must confirm ALL of the following)

| # | Criteria | Verification Method |
|---|----------|-------------------|
| 1 | Company is in a target vertical (Real Estate, MedSpa, HVAC) | Confirmed via call, website, or enrichment |
| 2 | Company has 2+ employees | Website, LinkedIn, or conversation |
| 3 | Contact has decision-making authority (or strong influence) | Title + conversation confirmation |
| 4 | Has a clear pain point AI can solve (reception, scheduling, lead follow-up) | Discovery conversation |
| 5 | Timeline to purchase is within 90 days | Verbal confirmation from prospect |
| 6 | Budget authority or access to budget holder | Role or conversation confirmation |

### MQL→SQL Conversion Process

```
1. MQL created (score ≥ 60, fit ≥ 25, engagement ≥ 35)
2. Lead assigned to rep via routing rules
3. SLA clock starts: rep must contact within 4 business hours
4. Rep conducts qualification (call, email, or LinkedIn message)
5. Within 48 hours, rep marks as:
   → "Accepted" (SQL) — all 6 criteria confirmed
   → "Rejected" (not a fit) — reason code required
   → "Recycled" (good fit, bad timing) — re-enter nurture
6. If rep doesn't act within SLA:
   → Alert sent to rep (hour 4)
   → Escalate to sales manager (hour 8)
   → Re-route to backup rep (hour 12)
```

### Rejection Reason Codes

| Code | Meaning | Next Action |
|------|---------|-------------|
| `wrong_industry` | Not in target vertical | Recycle to broad nurture |
| `too_small` | Company too small | Recycle; may revisit at scale |
| `no_authority` | Contact is not a decision-maker | Request to connect with owner |
| `no_budget` | Confirmed no budget | Recycle to 90-day nurture |
| `bad_timing` | Interested but not now | Recycle with 30/60/90-day follow-up |
| `competitor` | Using a competitor solution | Add to competitive nurture track |
| `wrong_geography` | Outside serviceable market | Archive |

---

## 5. SLA Definitions

### Response Time SLAs

| Scenario | SLA | Escalation |
|----------|-----|------------|
| **New MQL assigned** | Contact within **4 business hours** | Manager alert at hour 6, re-route at hour 12 |
| **Demo request** | Contact within **1 business hour** | Manager alert at hour 2, re-route at hour 4 |
| **Free trial signup** | Welcome + check-in within **2 hours** | Product specialist alert at hour 4 |
| **Hot lead (score > 75)** | Contact within **30 minutes** | Immediate Slack + SMS alert to manager |
| **Referral lead** | Contact within **4 business hours** | Partner manager notified if SLA breached |

### Follow-up Cadence

| Lead Type | Touchpoint 1 | Touchpoint 2 | Touchpoint 3 | Touchpoint 4 |
|-----------|-------------|-------------|-------------|-------------|
| **MQL (inbound)** | Call + email (Day 0) | Email + call (Day 1) | Email (Day 3) | LinkedIn (Day 5) |
| **Free trial** | Welcome call (Day 0) | Check-in call (Day 2) | Value email (Day 5) | Demo invite (Day 7) |
| **Referral** | Personal email (Day 0) | Call (Day 1) | Thank-you to referrer (Day 1) | Follow-up (Day 3) |
| **Recycled MQL** | Email sequence (Day 0) | Content share (Day 14) | Re-engagement email (Day 30) | Break-up email (Day 60) |

### Escalation Triggers

| Trigger | Action |
|---------|--------|
| MQL uncontacted after 4 hours | Slack alert to rep + manager |
| MQL uncontacted after 8 hours | Email to sales manager |
| MQL uncontacted after 12 hours | Auto-re-route to backup rep |
| Demo no-show | Auto-reschedule email within 1 hour |
| Proposal not sent within 5 days of request | Alert to AE + manager |
| Deal stuck in stage 2x average | Alert to AE + pipeline review flag |
| Hot lead (score > 75) detected | Instant Slack + SMS to assigned rep and manager |

---

## 6. Data Schema (PostgreSQL)

### Contacts Table

```sql
CREATE TABLE contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  first_name TEXT,
  last_name TEXT,
  phone TEXT,
  linkedin_url TEXT,
  job_title TEXT,
  seniority TEXT CHECK (seniority IN ('owner_ceo', 'manager', 'individual', 'unknown')),
  lifecycle_stage lead_stage DEFAULT 'unknown',
  firmographic_score INT DEFAULT 0 CHECK (firmographic_score BETWEEN 0 AND 40),
  engagement_score INT DEFAULT 0 CHECK (engagement_score BETWEEN 0 AND 60),
  total_score INT GENERATED ALWAYS AS (firmographic_score + engagement_score) STORED,
  source deal_source,
  utm_source TEXT,
  utm_medium TEXT,
  utm_campaign TEXT,
  utm_content TEXT,
  assigned_to UUID REFERENCES sales_reps(id),
  assigned_at TIMESTAMPTZ,
  mql_at TIMESTAMPTZ,
  sql_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
```

### Accounts (Companies) Table

```sql
CREATE TABLE accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  domain TEXT UNIQUE,
  name TEXT NOT NULL,
  niche niche DEFAULT 'other',
  employee_count INT,
  annual_revenue BIGINT,
  industry TEXT,
  city TEXT,
  state TEXT,
  country TEXT DEFAULT 'US',
  timezone TEXT,
  tech_stack TEXT[],  -- detected tools
  enrichment_data JSONB,  -- Clearbit/Apollo data
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Link contacts to accounts
ALTER TABLE contacts ADD COLUMN account_id UUID REFERENCES accounts(id);
```

### Lead Scoring Events Table

```sql
CREATE TABLE lead_scoring_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID REFERENCES contacts(id),
  event_type TEXT NOT NULL,  -- 'page_visit', 'email_open', 'demo_request', etc.
  event_data JSONB,
  points_change INT NOT NULL,
  decay_at TIMESTAMPTZ,  -- when this event's points expire
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### Deals / Opportunities Table

```sql
CREATE TABLE deals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID REFERENCES contacts(id),
  account_id UUID REFERENCES accounts(id),
  owner_id UUID REFERENCES sales_reps(id),
  stage lead_stage NOT NULL CHECK (stage IN ('sql', 'demo', 'proposal', 'won', 'lost')),
  estimated_acv NUMERIC(12,2),
  actual_acv NUMERIC(12,2),
  product_interest TEXT,  -- which AI employee: receptionist, sdr, scheduler, etc.
  niche niche,
  loss_reason loss_reason,
  loss_detail TEXT,
  demo_date TIMESTAMPTZ,
  proposal_sent_at TIMESTAMPTZ,
  closed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
```

### Audit / Activity Log

```sql
CREATE TABLE activity_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID REFERENCES contacts(id),
  deal_id UUID REFERENCES deals(id),
  actor_id UUID REFERENCES sales_reps(id),
  activity_type TEXT NOT NULL,  -- 'call', 'email', 'demo', 'proposal_sent', 'stage_change', etc.
  details JSONB,
  previous_stage lead_stage,
  new_stage lead_stage,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

---

## 7. Field Mappings for N8N

### Webhook Payload (Form Submission → N8N)

```
contact.email          ← form.email
contact.first_name     ← form.first_name
contact.last_name      ← form.last_name
contact.phone          ← form.phone
contact.job_title      ← form.job_title
contact.utm_source     ← url.utm_source
contact.utm_medium     ← url.utm_medium
contact.utm_campaign   ← url.utm_campaign
contact.source         ← map_source(form.referrer, form.utm_source)
account.name           ← form.company_name
account.domain         ← extract_domain(form.email)
account.niche          ← form.industry OR infer_from_domain
```

### N8N → PostgreSQL Mapping

```
contacts.email         ← {{ $json.email }}
contacts.first_name    ← {{ $json.first_name }}
contacts.last_name     ← {{ $json.last_name }}
contacts.firmographic_score ← {{ $json.firmographic_fit_score }}
contacts.engagement_score   ← {{ $json.engagement_score }}
contacts.lifecycle_stage    ← {{ $json.lifecycle_stage }}
contacts.source             ← {{ $json.source }}
contacts.utm_source         ← {{ $json.utm_source }}
accounts.domain             ← {{ $json.domain }}
accounts.name               ← {{ $json.company_name }}
accounts.niche              ← {{ $json.niche }}
```
