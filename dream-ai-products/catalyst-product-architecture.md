# Dream AI — Product Architecture & Go-to-Market

> **Owner:** CATALYST (Digital Products Strategist)
> **Created:** 2026-03-18
> **Based on:** `niches/00-niche-strategy.md`

---

## 1. Product Ladder

A 5-tier ascension model designed to capture leads at zero cost, convert through progressive commitment, and retain via recurring managed services.

```
TIER 5: Custom Deployment (scope template)    — $2,997–$7,997 one-time
TIER 4: Managed AI Employee (SLA-backed)      — $297–$997/mo recurring
TIER 3: AI Employee Starter Kit               — $297 one-time
TIER 2: Niche Playbook (PDF + templates)      — $27–$47 one-time
TIER 1: AI Readiness Quiz                     — Free (lead magnet)
```

---

### 1.1 Tier 1 — AI Readiness Quiz (Free Lead Magnet)

**Purpose:** Capture email + phone, segment by niche, score readiness, deliver instant value + CTA to Tier 2.

#### Quiz Flow (8–10 questions)

| # | Question | Type | Scoring Weight |
|---|----------|------|---------------|
| 1 | What industry are you in? | Single select (Real Estate / MedSpa / HVAC / Barbershop / Dental / Other) | Segments niche (no score) |
| 2 | How many calls/leads do you miss per week? | Range (0 / 1–5 / 6–15 / 16+) | 0 / 5 / 10 / 15 |
| 3 | How do you currently handle appointment booking? | Single select (Phone / Text / Online tool / Don't have a system) | 2 / 5 / 10 / 15 |
| 4 | How much revenue do you estimate you lose from missed leads annually? | Range (<$10K / $10–50K / $50–100K / $100K+) | 3 / 8 / 13 / 20 |
| 5 | Do you use a CRM or booking software? | Yes / No / What's a CRM? | 10 / 5 / 0 |
| 6 | How many hours per week does your team spend on phone calls, scheduling, and follow-ups? | Range (0–2 / 3–5 / 6–10 / 10+) | 0 / 5 / 10 / 15 |
| 7 | How do you currently follow up with new leads? | Single select (Immediately / Within a day / When I get to it / I don't) | 10 / 6 / 3 / 0 |
| 8 | What would you do with 10 extra hours per week? | Open text (qualitative, not scored) | 0 |

#### Scoring Bands

| Score | Band | Result Page CTA |
|-------|------|----------------|
| 0–25 | 🟢 "AI-Curious" | "You're wasting less than most — but imagine zero friction. Start with our free guide →" → Tier 2 upsell |
| 26–50 | 🟡 "AI-Ready" | "You're leaving $20K–$50K+/yr on the table. Your AI Readiness Score: X. Here's exactly what to fix →" → Tier 2 + Tier 3 offer |
| 51–75 | 🟠 "AI-Hungry" | "You're losing serious money every week. You need an AI employee NOW. Book a free setup call →" → Tier 3 + Tier 4 |
| 76–100 | 🔴 "AI-Critical" | "You're bleeding revenue. This is urgent. Let's fix this in 48 hours →" → Tier 4 + Tier 5 |

#### Technical Implementation

- **Form tool:** Typeform or Tally (embeddable, free tier)
- **Scoring:** Typeform logic / conditional branches or webhook → simple Node.js scoring function
- **Email capture:** Question 0 (name + email required to see results)
- **CRM sync:** Webhook → N8N → Google Sheets / HubSpot / Notion CRM
- **Result page:** Custom landing page per niche + score band (dynamic content)
- **Follow-up sequence:** Automated email sequence via ConvertKit / Mailchimp / N8N

#### Email Follow-Up Sequence (Post-Quiz)

| Day | Subject | Content |
|-----|---------|---------|
| 0 | "Your AI Readiness Score: [X]" | Results recap + quick win they can implement today |
| 1 | "The #1 thing [niche] businesses automate first" | Niche-specific case study / pain point |
| 3 | "How [competitor/peer] added $X/mo with AI" | Social proof + playbook CTA |
| 5 | "Your free playbook is ready" | Direct Tier 2 offer with urgency |
| 7 | "Last chance: playbook bonus expires" | Scarcity + Tier 2 + Tier 3 bundle |

---

### 1.2 Tier 2 — Niche Playbook ($27–$47)

**Purpose:** Low-commitment purchase that proves expertise, builds trust, and qualifies buyers for Tier 3+.

**Price:** $27 (single niche) / $37 (bundle of 2) / $47 (all niches + bonus templates)

#### Table of Contents Template (Per Niche)

*Example: Real Estate Agent AI Playbook*

```
Introduction: Why AI Is the #1 Competitive Edge in [Niche] Right Now        (1 page)

SECTION 1: The Revenue Leak Audit
  1.1 — How to calculate what missed calls cost your business (worksheet)   (2 pages)
  1.2 — The "speed-to-lead" math: 78% buy from first responder             (1 page)
  1.3 — Your personal revenue leak scorecard                                (1 page)

SECTION 2: The 5 Automations That Pay for Themselves in Week 1
  2.1 — 24/7 AI Voice Receptionist (what it does, what it says)             (3 pages)
  2.2 — Instant SMS Lead Follow-Up (template scripts included)              (2 pages)
  2.3 — Appointment Booking Agent (flow diagram + script)                   (2 pages)
  2.4 — CRM Auto-Fill (no more manual data entry)                          (2 pages)
  2.5 — Post-Visit Review Solicitor (copy-paste scripts)                    (1 page)

SECTION 3: Build It Yourself vs. Done-For-You
  3.1 — DIY option: tools, time investment, limitations                     (2 pages)
  3.2 — DFY option: what Dream AI handles for you                          (1 page)
  3.3 — Decision matrix: which path fits your situation                    (1 page)

SECTION 4: Implementation Roadmap
  4.1 — Week 1: Set up your first AI agent                                 (checklist)
  4.2 — Week 2: Add booking + follow-up                                    (checklist)
  4.3 — Week 3: Optimize and expand                                        (checklist)
  4.4 — Month 2+: Scale to full automation stack                           (checklist)

BONUS MATERIALS
  - Pre-written AI voice scripts for your niche (copy-paste ready)
  - SMS follow-up templates (5 variations)
  - ROI calculator spreadsheet (Google Sheets)
  - "Explain AI to my team" one-pager
```

**Delivery:** PDF download + Notion template (interactive version) + ROI calculator spreadsheet

**Upsell Bridge:** Last page = "Ready to skip the DIY? Get a pre-built AI employee installed in 48 hours → $297 Starter Kit"

---

### 1.3 Tier 3 — AI Employee Starter Kit ($297)

**Purpose:** Deliver a working AI agent with minimal friction. First "real" revenue event. Gateway to managed services.

**Price:** $297 one-time (payment plan: 2×$167)

#### What's Included

| Component | Details |
|-----------|---------|
| **1 Pre-Built AI Agent** | Configured for their niche (receptionist, booking, or follow-up — they choose 1) |
| **Phone Number** | Dedicated Twilio number with AI voice pickup |
| **Setup & Configuration** | Dream AI team installs + configures (48-hour turnaround) |
| **Onboarding Video** | 15-min walkthrough: how to use, monitor, and tweak |
| **30-Day Email Support** | Direct access to support (response within 24h) |
| **Templates Library** | Scripts, prompts, and flow diagrams for their niche |
| **Basic Dashboard** | Call logs, lead count, booking stats (simple reporting) |

#### SLA (30-day support window)

| Metric | Commitment |
|--------|-----------|
| Setup completion | 48 business hours from order + info |
| Support response time | <24 hours (business days) |
| Uptime target | 99% (excluding scheduled maintenance) |
| Rework guarantee | 1 free revision within 14 days |

**Upsell Bridge:** "Want us to manage, optimize, and scale this for you every month? → Managed Service ($297+/mo)"

---

### 1.4 Tier 4 — Managed AI Employee ($297–$997/mo Recurring)

**Purpose:** Core recurring revenue. Ongoing optimization, support, and expansion.

#### Tier Comparison

| Feature | Starter ($297/mo) | Growth ($497/mo) | Pro ($997/mo) |
|---------|-------------------|-------------------|----------------|
| AI Agent(s) | 1 | 2 | 4 |
| Monthly optimization calls | — | 1× (30 min) | 2× (30 min) |
| Script & prompt updates | As needed | Weekly review | Continuous optimization |
| Reporting | Basic dashboard | Weekly email report | Custom dashboard + weekly |
| Support priority | Email (24h) | Email + chat (4h) | Email + chat + phone (1h) |
| Integrations | 1 (CRM or booking) | 3 | Unlimited |
| A/B testing | — | Monthly | Continuous |
| New agent additions | Add-on ($197 each) | 1 free/quarter | 2 free/quarter |
| Contract | Month-to-month | Month-to-month or annual (15% off) | Month-to-month or annual (20% off) |

#### Managed Service SLA

| Commitment | Details |
|-----------|---------|
| Uptime | 99.5% (monitored, with credits for downtime) |
| Response time | Per tier (see above) |
| Monthly reporting | Call volume, conversion rate, revenue impact estimate |
| Quarterly business review | 60-min strategic call (Growth + Pro only) |
| Optimization guarantee | Measurable improvement in key metric within 90 days or 1 month free |
| Data security | Encrypted storage, access controls, data retention policy |

---

### 1.5 Tier 5 — Custom Deployment ($2,997–$7,997 + optional managed)

**Purpose:** High-ticket, high-touch deployment for multi-location businesses, teams, or complex use cases.

#### Scope Template

```markdown
## Custom AI Deployment — Scope of Work

### Client: [Business Name]
### Date: [Date]
### Deploy Lead: [Team Member]

---

### 1. Discovery & Requirements
- [ ] Stakeholder interviews (up to 3 sessions × 60 min)
- [ ] Current workflow audit (tools, call flow, CRM, scheduling)
- [ ] Pain point prioritization (ranked by revenue impact)
- [ ] Integration mapping (what systems need to connect)

### 2. Solution Design
- [ ] AI agent architecture (voice + text + automation flows)
- [ ] Conversation scripts & decision trees
- [ ] Integration plan (CRM, booking, phone, messaging)
- [ ] Data flow diagram
- [ ] Approval checkpoint with client

### 3. Build & Configuration
- [ ] AI agent(s) built and configured
- [ ] Phone number provisioning + routing
- [ ] CRM / booking software integration
- [ ] Custom dashboard / reporting setup
- [ ] Testing (internal QA + client UAT)

### 4. Launch & Training
- [ ] Go-live with monitoring
- [ ] Team training session (up to 90 min, recorded)
- [ ] Written SOPs / runbook delivered
- [ ] Handoff to support

### 5. Support & Optimization (optional managed add-on)
- [ ] [Per managed tier SLA above]

---

### Deliverables
| Item | Format | Timeline |
|------|--------|----------|
| Discovery summary | PDF/Notion | Day 3 |
| Solution design | Notion + diagrams | Day 7 |
| Agent build | Live system | Day 14 |
| Training + SOPs | Video + PDF | Day 16 |
| Go-live | Production | Day 14–21 |

### Investment
| Component | Price |
|-----------|-------|
| Discovery & Design | $997 |
| Build & Launch | $2,000–$7,000 (based on complexity) |
| Managed Service (optional) | $297–$997/mo |
| **Total One-Time** | **$2,997–$7,997** |

### Change Order Policy
- Scope changes after sign-off: billed at $150/hr
- Additional agents post-launch: from $497 each
- Rush delivery (48h): +30%
```

---

## 2. Self-Serve Build

For buyers who want to DIY (typically playbooks + starter kit purchasers who prefer autonomy).

### 2.1 Tech Stack

| Layer | Tool | Why | Cost |
|-------|------|-----|------|
| **Voice AI** | OpenClaw + ElevenLabs + Twilio | Realistic voice, phone integration, agent orchestration | $20–100/mo per agent |
| **Chat/SMS AI** | OpenClaw + Twilio SMS / WhatsApp | Omnichannel text interface | Usage-based |
| **Workflow Automation** | N8N (self-hosted on OpenClaw) | Visual builder, webhook-native, no vendor lock-in | Free (self-hosted) |
| **CRM** | HubSpot Free / Notion / Google Sheets | Contact management, pipeline tracking | Free–$20/mo |
| **Scheduling** | Cal.com / Calendly | Appointment booking integration | Free–$12/mo |
| **Payment** | Stripe | Checkout, subscriptions, invoicing | 2.9% + $0.30/txn |
| **Email Marketing** | ConvertKit / Mailchimp | Sequences, segmentation | Free–$29/mo |
| **Landing Pages** | Carrd / Framer / Notion | Fast, clean pages | Free–$19/mo |
| **Hosting / Infrastructure** | OpenClaw (managed) | Agent runtime, monitoring, scaling | Included in OpenClaw |
| **Analytics** | Google Analytics + custom dashboard | Funnel tracking, agent performance | Free |
| **Community** | Discord / Circle | Peer support, Q&A, updates | Free–$30/mo |

### 2.2 Onboarding Funnel

```
┌─────────────────────────────────────────────────────────────────┐
│  AWARENESS                                                      │
│  Ad / Content / Referral → Quiz (free)                         │
│  Email captured, niche segmented, score delivered               │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  EDUCATION                                                      │
│  Email sequence (7 days) → Playbook purchase ($27–47)           │
│  Self-paced: watch tutorials, read playbook, join community     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  ACTIVATION                                                     │
│  Playbook buyer → Starter Kit upsell ($297)                     │
│  OR: Free activation call (15 min) to set up first agent        │
│  "Aha moment" = first call answered / lead booked by AI         │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  EXPANSION                                                      │
│  Active user → Managed service upsell ($297+/mo)                │
│  Triggers: >20 calls/mo, >1 agent needed, complex integrations  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  ADVOCACY                                                        │
│  Case study / referral program / affiliate commission            │
│  10% referral fee on first 3 months                             │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Video Training Outline

**"Build Your AI Employee" — 10-Module Video Course (Self-Paced)**

| Module | Title | Length | Content |
|--------|-------|--------|---------|
| 1 | "Why Your Business Needs an AI Employee" | 8 min | Pain points, ROI framing, what AI can do today |
| 2 | "The Tech Stack Explained (No Jargon)" | 12 min | Walkthrough of each tool, why it's chosen, setup overview |
| 3 | "Setting Up Your OpenClaw Agent" | 15 min | Account setup, first agent creation, voice selection |
| 4 | "Writing Scripts That Convert" | 20 min | Conversation design, tone, objection handling, niches |
| 5 | "Connecting Your Phone Number" | 10 min | Twilio setup, number provisioning, call routing |
| 6 | "Booking & CRM Integration" | 18 min | Cal.com/Calendly + CRM setup, data flow |
| 7 | "SMS & WhatsApp Automation" | 15 min | Text-based agents, follow-up sequences |
| 8 | "Your First Automation Workflow (N8N)" | 20 min | Visual builder walkthrough, trigger → action → webhook |
| 9 | "Monitoring & Optimization" | 12 min | Dashboards, call review, script iteration |
| 10 | "Scaling: From 1 Agent to a Full Team" | 10 min | Multi-agent architectures, delegation, advanced flows |

**Bonus Videos:**
- "Niche-Specific Script Workshop" (1 per niche, 15 min each)
- "Troubleshooting Common Issues" (10 min)
- "Monthly AI Trends Update" (recurring, 10 min)

**Delivery:** Vimeo/YouTube (unlisted) + Notion course hub with downloads

### 2.4 Community Structure

**Platform:** Discord (primary) or Circle (if premium feel needed)

```
📢 ANNOUNCEMENTS
  # updates — New features, course updates, wins
  # deals — Special offers, early access

🎓 LEARNING
  # course-discussion — Module Q&A, progress sharing
  # tutorials — Written guides, tips, how-tos
  # templates — Shared scripts, prompts, workflows

💬 NICHES (one channel per priority niche)
  # real-estate — Agents sharing wins, scripts, strategies
  # medspa — Owners discussing AI for practices
  # hvac — Contractors, dispatchers, owners
  # barbershops — Stylists and shop owners
  # other-industries — Catch-all for other niches

🔧 TECHNICAL
  # openclaw-help — Agent setup, troubleshooting
  # integrations — CRM, booking, telephony setup
  # n8n-workflows — Automation building, sharing

🏆 WINS & ACCOUNTABILITY
  # wins — Share revenue recovered, calls answered, leads booked
  # accountability — Weekly goal setting, progress tracking

🤝 NETWORKING
  # introductions — New member intros
  # partnerships — JV, referrals, cross-promotion
  # off-topic — General chat, community bonding
```

**Community Engagement Cadence:**
- **Daily:** Automated "AI Tip of the Day" post (N8N → Discord webhook)
- **Weekly:** "Win Wednesday" prompt — members share results
- **Bi-weekly:** Live Q&A / office hours (30 min, hosted by Dream AI team)
- **Monthly:** Guest expert AMA (niche-specific)
- **Quarterly:** Community challenge ("Book 50 appointments with AI in 30 days")

---

## 3. Upsell / Cross-Sell Logic

### 3.1 Trigger-Based Upsells

| Trigger Event | Upsell Offer | Channel | Timing |
|---------------|-------------|---------|--------|
| Quiz score ≥ 51 | Starter Kit ($297) | Email + retargeting ad | Immediately after quiz |
| Playbook purchased | Starter Kit ($297) — bundle at $247 | Email sequence day 3 | Within 72h of purchase |
| Starter Kit active, >20 calls/mo | Managed Growth ($497/mo) | Email + in-app banner | Week 3–4 post-activation |
| Starter Kit active, >50 calls/mo | Managed Pro ($997/mo) | Personal outreach (email/call) | When threshold hit |
| Single agent active 30+ days | Add 2nd agent (cross-sell) | In-app prompt + email | Day 30 |
| Multi-location inquiry detected | Custom Deployment scope | Consultation offer | Qualification call |
| Annual plan consideration | 2-month discount (save $594 on Pro) | Email campaign | Month 2 of monthly plan |
| NPS ≥ 9 (promoter) | Referral program invite | Post-support survey | After positive interaction |
| NPS ≤ 6 (detractor) | Free optimization call | Personal outreach within 24h | After low score |

### 3.2 Expansion Paths

```
                        ┌─────────────────┐
                        │  QUIZ TAKER     │
                        │  (Free)         │
                        └────────┬────────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
              ┌──────────┐ ┌──────────┐ ┌──────────┐
              │ PLAYBOOK │ │ PLAYBOOK │ │ PLAYBOOK │
              │ $27      │ │ $37 (2×) │ │ $47 (all)│
              └────┬─────┘ └────┬─────┘ └────┬─────┘
                   │            │            │
                   └────────────┼────────────┘
                                ▼
                     ┌───────────────────┐
                     │  STARTER KIT      │
                     │  $297             │
                     │  (1 agent)        │
                     └───────┬───────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌──────────────┐ ┌───────────┐ ┌──────────────┐
     │ SELF-SERVE   │ │ MANAGED   │ │ ADD-ON       │
     │ (Community   │ │ $297/mo   │ │ AGENTS       │
     │  support)    │ │ (1 agent) │ │ $197/ea      │
     └──────────────┘ └─────┬─────┘ └──────────────┘
                            │
                   ┌────────┼────────┐
                   ▼        ▼        ▼
            ┌──────────┐ ┌──────┐ ┌──────────┐
            │ GROWTH   │ │ PRO  │ │ CUSTOM   │
            │ $497/mo  │ │$997  │ │ DEPLOY   │
            │ (2 agts) │ │/mo   │ │ $3K–8K   │
            └──────────┘ └──────┘ └──────────┘
```

### 3.3 Cross-Sell Matrix

| If They Have... | Sell Them Next... | Reason |
|-----------------|-------------------|--------|
| Receptionist agent | Booking agent | Complementary: captures → converts |
| Booking agent | Follow-up / review agent | Complementary: books → retains |
| 1 agent (any type) | SMS/WhatsApp channel | Same agent, more touchpoints |
| SMS agent | Voice agent | Multi-channel expansion |
| Any agent | CRM integration | Deeper value, higher switching cost |
| Starter Kit | Playbook (if didn't buy) | Education increases stickiness |
| Managed service | Custom deployment | Complexity grows with success |

---

## 4. Launch Sequencing

### 4.1 MVP Scope (Weeks 1–2)

**Goal:** Ship the minimum viable product stack to start generating revenue and collecting data.

| Priority | Deliverable | Status | Owner |
|----------|------------|--------|-------|
| **P0** | AI Readiness Quiz (1 niche: Real Estate) | Build | CATALYST + BOLT |
| **P0** | Email capture + scoring + results page | Build | CATALYST + BOLT |
| **P0** | Email follow-up sequence (7 emails) | Write | QUILL |
| **P0** | Real Estate Playbook (PDF) | Write | CATALYST + QUILL |
| **P0** | Stripe checkout (quiz → playbook → starter kit) | Build | FORGE |
| **P1** | Starter Kit: 1 AI receptionist agent template | Build | BOLT |
| **P1** | Onboarding email sequence (post-purchase) | Write | QUILL |
| **P1** | Basic landing page (dream-ai.co or similar) | Build | PIXEL |
| **P2** | Discord community setup | Setup | CATALYST |
| **P2** | Twilio phone number provisioning flow | Build | BOLT |

**MVP Budget Target:** < $500 (tools + domains + Twilio credits)

### 4.2 30/60/90 Day Roadmap

#### Days 1–30: Foundation & First Revenue

| Week | Focus | Key Actions | Success Metric |
|------|-------|-------------|---------------|
| **Week 1** | Build MVP | Quiz live, playbook published, checkout working | Quiz accepting submissions |
| **Week 2** | Launch & validate | Promote to warm network, first 50 quiz takers, 5+ playbook sales | First $200 revenue |
| **Week 3** | Optimize funnel | A/B test quiz results page, improve email seq, add 2nd niche (MedSpa) | 10% quiz→playbook conversion |
| **Week 4** | First Starter Kits | Sell 3+ Starter Kits, deliver, collect testimonials | First $891 revenue, 3 testimonials |

**End of Month 1 Target:**
- 200+ quiz takers
- 20+ playbook sales ($540–$940)
- 3–5 Starter Kits ($891–$1,485)
- **Total: $1,431–$2,425**

#### Days 31–60: Scale & Systematize

| Week | Focus | Key Actions | Success Metric |
|------|-------|-------------|---------------|
| **Week 5** | Add niches 3–4 | Build playbooks for HVAC + Barbershops, launch niche-specific ads | 4 niches live |
| **Week 6** | Paid acquisition tests | Test $20/day on Meta/Google (quiz funnel), track CPA | CPA < $8/lead |
| **Week 7** | Managed service launch | Launch $297/mo tier, convert first Starter Kit buyers | 2+ managed subscribers |
| **Week 8** | Community + content | Launch Discord, first 20 members, weekly content cadence | 20 community members |

**End of Month 2 Target:**
- 600+ total quiz takers
- 50+ playbook sales
- 10–15 Starter Kits
- 3–5 managed subscribers
- **Total MRR: $891–$4,985**

#### Days 61–90: Compound & Validate

| Week | Focus | Key Actions | Success Metric |
|------|-------|-------------|---------------|
| **Week 9** | Video course launch | Ship "Build Your AI Employee" course (10 modules), bundle with playbook | Course completion rate >40% |
| **Week 10** | Referral program | Launch 10% referral commission, activate 5 referrers | 3+ referral sales |
| **Week 11** | Custom deployment pilot | Sign 1st Custom Deployment client ($2,997+), use as case study | $2,997+ project closed |
| **Week 12** | Retention & expansion | NPS survey, churn analysis, upsell campaign to existing base | Churn < 5%, NPS > 50 |

**End of Month 3 Target:**
- 1,500+ total quiz takers
- 100+ playbook sales
- 25–35 Starter Kits
- 8–12 managed subscribers
- 1–2 custom deployments
- **Monthly Revenue: $5,000–$15,000**
- **ARR Run Rate: $60,000–$180,000**

---

### 4.3 Pricing Experiments

A/B test matrix — run each experiment for minimum 2 weeks or 100 conversions, whichever comes first.

| # | Experiment | Control | Variant | Hypothesis | Metric |
|---|-----------|---------|---------|------------|--------|
| **E1** | Playbook price point | $37 | $27 | Lower price = higher volume, net positive | Revenue per 100 visitors |
| **E2** | Playbook price point | $37 | $47 | Higher price signals quality, same conversion | Revenue per 100 visitors |
| **E3** | Bundle vs. single | Single ($27) | All-access ($47) | Bundle increases AOV more than it decreases conversion | AOV, conversion rate |
| **E4** | Starter Kit price | $297 | $197 | Lower entry → more volume → more managed upsells | Starter Kit sales + managed conversion % |
| **E5** | Starter Kit price | $297 | $497 | Higher price attracts more committed buyers | Churn rate, managed upsell % |
| **E6** | Payment plan | Pay in full | 2×$167 | Payment plan increases conversion on $297 item | Conversion rate |
| **E7** | Quiz length | 8 questions | 5 questions | Shorter quiz = more completions, but less qualified leads | Completion rate + downstream conversion |
| **E8** | Quiz gate | Email required | Email optional (results visible) | Ungated = more completions but fewer leads | Total leads + lead quality score |
| **E9** | Managed service tier | $297/mo only | $297 / $497 / $997 | 3-tier pricing creates anchor effect, lifts mid-tier | Avg managed MRR per customer |
| **E10** | Annual discount | Monthly only | Annual (20% off) | Annual prepay improves cash flow + retention | % choosing annual, retention at 12mo |
| **E11** | Free trial (managed) | No trial | 7-day free trial | Trial reduces perceived risk for $297+/mo | Trial→paid conversion rate |
| **E12** | Urgency/scarcity | No deadline | "5 spots this month" | Artificial scarcity drives faster decisions | Days-to-purchase, conversion rate |

#### Experiment Run Protocol

1. **Setup:** Split traffic 50/50 using Typeform/Tally conditional logic or landing page A/B tool
2. **Duration:** Minimum 2 weeks OR 100 conversions per variant
3. **Significance:** Target 95% confidence (use a simple calculator like Optimizely)
4. **Decision:** If winner → implement as new default. If inconclusive → extend 1 week or kill
5. **Documentation:** Log all results in experiment tracker (Notion database)

---

## Appendix: Revenue Projections (90-Day Conservative)

| Product | Unit Sales | Avg Price | Revenue |
|---------|-----------|-----------|---------|
| Quiz (free) | 1,500 | $0 | $0 |
| Playbooks | 100 | $37 | $3,700 |
| Starter Kits | 30 | $297 | $8,910 |
| Managed (avg 8 subs × 1.5 months) | — | $397 | $4,764 |
| Custom Deployments | 2 | $4,997 | $9,994 |
| **Total (90 days)** | | | **$27,368** |
| **Monthly Average** | | | **$9,123** |
| **ARR Run Rate** | | | **$109,472** |

*Aggressive scenario (2× conversions): ~$55K/90 days → $220K ARR*

---

## File Manifest

| File | Purpose |
|------|---------|
| `niches/00-niche-strategy.md` | Niche research, pain points, positioning |
| `catalyst-product-architecture.md` | This file — full product ladder, self-serve build, upsell logic, launch plan |

---

*Document maintained by CATALYST. Review and update bi-weekly as data comes in.*
