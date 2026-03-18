# Dream AI — Retargeting Strategy

> **Purpose:** Capture warm audiences who've already engaged but haven't purchased the highest-value product.
> **Budget rule:** 20-30% of total ad spend allocated to retargeting.
> **Key principle:** Retargeting is where ROAS compounds. These audiences already know you — the job is conversion, not awareness.

---

## PIXEL EVENTS TO TRACK

### Meta Pixel (Facebook/Instagram)

| Event | Trigger | Priority | Use |
|-------|---------|----------|-----|
| `PageView` | Any page load | Standard | Site visitor retargeting |
| `ViewContent` | Calculator/Scorecard landing page | Standard | Intent retargeting |
| `InitiateCheckout` | Click "Calculate Now" / "Get Scored" (before payment) | Standard | Cart abandonment |
| `Purchase` | Successful payment | Standard | Exclude buyers + lookalike creation |
| `Lead` | Email capture (calculator/scorecard results page) | Standard | Nurture sequence triggers |
| `AddToCart` | Add order bump / upsell item | Standard | Upsell retargeting |
| `Search` | Use of calculator tool / scorecard questions | Custom | High-intent signal |

### Custom Conversions to Set Up

| Name | Rule | Value |
|------|------|-------|
| Calculator Completed | URL contains `/calculator/results` | $3 |
| Scorecard Completed | URL contains `/scorecard/results` | $5 |
| Email Captured | URL contains `/thank-you` or fires `Lead` event | $5 |
| Starter Kit Purchase | URL contains `/thank-you` + product = starter-kit | $297 |
| Playbook Purchase | URL contains `/thank-you` + product = playbook | $22 |
| Script Pack Purchase | URL contains `/thank-you` + product = script-pack | $12 |

### Google Analytics 4 Events

| Event | Parameters | Purpose |
|-------|-----------|---------|
| `purchase` | value, currency, items | Revenue tracking, ROAS |
| `generate_lead` | lead_type (calculator/scorecard) | Funnel attribution |
| `scroll_depth` | percent_scrolled (75, 90, 100) | Engagement quality |
| `video_progress` | percent (25, 50, 75, 100) | Video ad optimization |

### Google Ads Conversion Tracking

- Import GA4 `purchase` events
- Import GA4 `generate_lead` events
- Set up enhanced conversions (hashed first-party data)
- Value-based bidding once 50+ conversions in 30 days

---

## AUDIENCE SEGMENTS

### Segment 1: Site Visitors (Cold-ish)

| Attribute | Detail |
|-----------|--------|
| **Definition** | Any visitor to dreamai.co in the last 30 days |
| **Exclusion** | Purchasers (7-day window), bounces (<10 sec) |
| **Size estimate** | 2,000-10,000/month (grows with ad spend) |
| **Temperature** | Warm — they know the brand but haven't engaged deeply |
| **Primary goal** | Get them to use a tool (calculator or scorecard) |
| **Revenue potential** | Low direct, high funnel entry |

### Segment 2: Calculator Users (Warm)

| Attribute | Detail |
|-----------|--------|
| **Definition** | Completed the Missed Call Calculator (30-day window) |
| **Exclusion** | Starter Kit purchasers |
| **Size estimate** | 500-2,000/month |
| **Temperature** | Hot — they saw their number, they feel the pain |
| **Primary goal** | Convert to Starter Kit ($297) or Script Pack ($12) |
| **Revenue potential** | HIGH — highest converting retargeting segment |

### Segment 3: Scorecard Users (Warm)

| Attribute | Detail |
|-----------|--------|
| **Definition** | Completed the AI Readiness Scorecard (30-day window) |
| **Exclusion** | Playbook or Starter Kit purchasers |
| **Size estimate** | 300-1,500/month |
| **Temperature** | Warm-to-hot — they want to improve but need a plan |
| **Primary goal** | Convert to Playbook ($17-27) or Starter Kit ($297) |
| **Revenue potential** | MEDIUM-HIGH — requires nurture, then upsell |

### Segment 4: Cart Abandoners (Hottest)

| Attribute | Detail |
|-----------|--------|
| **Definition** | Initiated checkout but didn't complete (14-day window) |
| **Exclusion** | None (they haven't bought yet) |
| **Size estimate** | 100-500/month |
| **Temperature** | HOT — they were ready to buy |
| **Primary goal** | Complete purchase + order bump |
| **Revenue potential** | HIGHEST — closest to conversion |

### Segment 5: Email Engagers (Warm)

| Attribute | Detail |
|-----------|--------|
| **Definition** | Opened 2+ emails in nurture sequence (30-day window) |
| **Exclusion** | Purchasers of the product being promoted |
| **Size estimate** | 200-1,000/month |
| **Temperature** | Warm — engaged with email but haven't clicked through to buy |
| **Primary goal** | Drive to landing page / purchase |
| **Revenue potential** | MEDIUM — responsive but need another touchpoint |

---

## AD SEQUENCES BY SEGMENT

### Segment 1: Site Visitors

**Goal:** Tool entry (calculator or scorecard)
**Duration:** 14 days
**Frequency cap:** 3-5x per week
**Budget:** 30% of retargeting spend

#### Week 1 (Days 1-7)

| Day | Ad | Creative | Copy Angle |
|-----|-----|----------|------------|
| 1-2 | Calculator Retarget | Single image | "You visited. You didn't calculate. Every day costs you money." |
| 3-4 | Scorecard Retarget | Carousel | "94% of businesses aren't AI-ready. Are you one of them?" |
| 5-6 | Social Proof | Video testimonial | "This plumber saved $41K/year after using our calculator" |
| 7 | Urgency | Single image | "Week 1 without fixing this cost you $[X]." |

#### Week 2 (Days 8-14)

| Day | Ad | Creative | Copy Angle |
|-----|-----|----------|------------|
| 8-9 | Calculator Retarget v2 | Different image | New hook: "Your phone is a $50K/year liability" |
| 10-11 | Scorecard Retarget v2 | Different carousel | "The AI gap is widening. Score yourself in 2 min." |
| 12-13 | Founder Story | Video (30 sec) | Personal, authentic — why these tools exist |
| 14 | Last Chance | Single image | "Last ad on this. Take the calculator or we'll stop asking." |

**After Day 14:** Move to "stale" segment at reduced frequency (1-2x/week) for 30 more days.

---

### Segment 2: Calculator Users → Paid Products

**Goal:** Convert to Starter Kit ($297) or Script Pack ($12)
**Duration:** 21 days
**Frequency cap:** 5x per week (high intent, higher frequency OK)
**Budget:** 35% of retargeting spend

#### Phase 1: Direct Conversion (Days 1-7)

| Day | Ad | Creative | Copy Angle |
|-----|-----|----------|------------|
| 1 | Starter Kit Lead | Single image | "You saw the number. Here's the fix." |
| 1 | Script Pack | Single image (alt) | A/B test: lower price point ($12 vs $297) |
| 2-3 | Results Proof | Carousel | Before/after: before Starter Kit vs. after |
| 4 | Founder Video | Video (30 sec) | "I built this because I was losing $3K/month to missed calls" |
| 5 | Objection handler | Single image | "$297 sounds like a lot. Until you realize you lost $[X] last month." |
| 6 | Testimonial | Single image | Customer quote + result |
| 7 | Soft close | Single image | "Still thinking? The Starter Kit comes with a 30-day guarantee." |

#### Phase 2: Objection Handling (Days 8-14)

| Day | Ad | Creative | Copy Angle |
|-----|-----|----------|------------|
| 8-9 | Price anchor | Single image | "I charge $2,000/day for consulting. This is everything I'd teach you — for $297." |
| 10 | Guarantee | Single image | "30-day money-back guarantee. What's the risk?" |
| 11-12 | Specificity | Single image | "47 scripts. 12 templates. 1 playbook. Every objection covered." |
| 13 | Scarcity | Single image | "Limited support slots this quarter. Price goes up April 1." |
| 14 | Final CTA | Single image | "Last call. The Starter Kit — or we move on." |

#### Phase 3: Reduced frequency (Days 15-21)
- 1-2 ads/week, softer messaging
- Focus on Playbook ($17) or Script Pack ($12) as lower-commitment entry
- After Day 21: move to "stale" retargeting at 1-2x/week

---

### Segment 3: Scorecard Users → Playbook/Starter Kit

**Goal:** Convert to Playbook ($17-27) first, then upsell Starter Kit
**Duration:** 21 days
**Frequency cap:** 4x per week
**Budget:** 20% of retargeting spend

#### Phase 1: Playbook Push (Days 1-7)

| Day | Ad | Creative | Copy Angle |
|-----|-----|----------|------------|
| 1 | Score result reminder | Single image | "Your AI Score: [X]. Here's how to raise it." |
| 2-3 | Playbook features | Carousel | "What's inside: 12 chapters, 90-day roadmap, 15 tool templates" |
| 4 | Case study | Single image | "Went from 34 to 81 in 90 days. Here's the playbook." |
| 5 | Price anchor | Single image | "$17 for a 90-day plan that could add $50K to your revenue." |
| 6 | Testimonial | Single image | Customer quote: "This is the AI strategy I couldn't write myself" |
| 7 | Direct CTA | Single image | "Get the Playbook → raise your score" |

#### Phase 2: Cross-sell to Starter Kit (Days 8-21)

| Day | Ad | Creative | Copy Angle |
|-----|-----|----------|------------|
| 8-10 | "Plan + Tools" | Single image | "You have the plan (Playbook). Now get the tools (Starter Kit)." |
| 11-12 | Bundle offer | Single image | "Playbook + Starter Kit bundle: $297 (save $30)" |
| 13-14 | Results proof | Carousel | "Playbook gives you the roadmap. Starter Kit gives you the engine." |
| 15-21 | Reduced freq | Mix | 2x/week, alternating Playbook and Starter Kit messages |

---

### Segment 4: Cart Abandoners (Highest Priority)

**Goal:** Complete purchase immediately
**Duration:** 7 days (hot window)
**Frequency cap:** Daily (Day 1-3), then every other day
**Budget:** 15% of retargeting spend

#### The Sequence

| Day | Ad | Creative | Copy Angle |
|-----|-----|----------|------------|
| 0 (same day) | Urgency nudge | Single image | "You were almost there. Complete your [product] purchase." |
| 1 | Objection handler | Single image | "Was it the price? Here's what $[X] buys you." |
| 2 | Social proof | Single image | "[X,XXX] people bought this. Average rating: 4.8/5" |
| 3 | Guarantee | Single image | "30-day money-back guarantee. Zero risk." |
| 4 | Scarcity | Single image | "Limited time: add [order bump] for just $[X] more" |
| 5-6 | Soft exit | Single image | "Still interested? Our SMS templates are just $7 to start." |
| 7 | Final | Single image | "We'll stop showing this after today. Last chance." |

**Critical:** Cart abandoners get email + SMS abandonment sequence simultaneously (not just ads). Layer channels for maximum recovery.

**Expected recovery rates:**
- Day 0-1: 30-40% recovery
- Day 2-4: 10-15% recovery
- Day 5-7: 5-8% recovery
- **Total cart recovery: 45-63%**

---

### Segment 5: Email Engagers → Landing Page

**Goal:** Drive from email to landing page / purchase
**Duration:** 14 days
**Frequency cap:** 3x per week
**Budget:** 0% of retargeting spend (handled via email sequences)

This segment is primarily handled through email sequences, not paid retargeting. However, if email engagement drops:
- Trigger a 3-day retargeting burst to re-engage
- Creative: "Check your inbox — we sent you something important"
- Budget: pulled from Segment 1 allocation temporarily

---

## BUDGET ALLOCATION

### Total Retargeting Budget = 20-30% of Total Ad Spend

Assuming $10,000/mo total ad spend:

| | Conservative (20%) | Aggressive (30%) |
|---|---|---|
| **Retargeting total** | $2,000/mo | $3,000/mo |

### Allocation by Segment

| Segment | % of Retargeting | Conservative ($2K) | Aggressive ($3K) |
|---------|-----------------|-------------------|-----------------|
| Site Visitors → Tool | 30% | $600 | $900 |
| Calculator Users → Paid | 35% | $700 | $1,050 |
| Scorecard Users → Paid | 20% | $400 | $600 |
| Cart Abandoners | 15% | $300 | $450 |
| **Total** | **100%** | **$2,000** | **$3,000** |

### Expected Retargeting Performance

| Segment | Expected CPA | Expected ROAS | Monthly Revenue (Conservative) |
|---------|-------------|-------------|-------------------------------|
| Site Visitors | $5-10 | 3-5x | $1,800-$3,000 |
| Calculator Users | $15-30 | 8-15x | $5,600-$10,500 |
| Scorecard Users | $12-25 | 5-10x | $2,000-$6,000 |
| Cart Abandoners | $3-8 | 15-30x | $4,500-$13,500 |
| **Total retargeting** | | | **$13,900-$33,000** |

**Retargeting ROAS should be 3-5x higher than prospecting ROAS** — if it's not, the landing page or offer is the problem, not the ads.

---

## CREATIVE ROTATION SCHEDULE

### Rotation Rules

| Rule | Detail |
|------|--------|
| **Primary creative** | Runs first 5-7 days |
| **Secondary creative** | Runs days 3-10 (overlap for A/B testing) |
| **Tertiary creative** | Launch day 8 if primary + secondary both fatigue |
| **Refresh trigger** | CTR drops 30%+ from baseline OR frequency >4.0 |
| **Retire rule** | Creative that hasn't converted after 7 days → pause |
| **Winning creative** | Scale budget 20%, keep running until CPA 2x target |

### Monthly Creative Calendar

| Week | Calculator Retarget | Scorecard Retarget | Cart Abandon | Site Visitor |
|------|--------------------|--------------------|-------------|--------------|
| 1 | Direct conversion (new) | Playbook push (new) | Urgency sequence (new) | Tool intro (new) |
| 2 | Results proof (new) | Cross-sell (new) | Objection handler (new) | Social proof (new) |
| 3 | Founder story (new) | Bundle offer (new) | Scarcity (new) | Different tool angle |
| 4 | Testimonial (new) | Soft close (new) | Final push (new) | Founder story (new) |

---

## EXCLUSIONS & NEGATIVE AUDIENCES

### Always Exclude These from Retargeting:

| Exclusion | Duration | Why |
|-----------|----------|-----|
| Purchasers of promoted product | 30 days | Don't sell what they bought |
| All purchasers (site-wide) | 14 days | Prevent accidental upsell fatigue before they experience value |
| Bounced visitors (<10 sec) | Permanent | Low quality, waste of spend |
| Employees / internal IP | Permanent | Wasted impressions |
| Existing email subscribers (if segment 5 active) | Active | They're being hit via email already |

### Platform-Specific Exclusions

**Meta:**
- Upload customer list as Custom Audience → Exclude
- Create exclusion from `Purchase` event (auto-exclude)

**Google Ads:**
- Exclude converters via GA4 audience
- Exclude bounced sessions (<10 sec, 0 pages viewed)

**LinkedIn:**
- Upload customer email list → Exclude
- Exclude by company name (internal)

---

## MEASUREMENT & OPTIMIZATION

### Weekly Retargeting Review Checklist

- [ ] Check frequency per segment (flag if >4.0)
- [ ] Compare CPA vs. target for each segment
- [ ] Identify top and bottom performing creatives
- [ ] Check audience overlap between segments (reduce cannibalization)
- [ ] Review cart abandonment recovery rate
- [ ] Verify pixel is firing correctly (no data gaps)
- [ ] Check landing page conversion rate (is the bottleneck post-click?)

### Key Metrics by Segment

| Segment | Primary Metric | Secondary Metric | Health Threshold |
|---------|---------------|-----------------|-----------------|
| Site Visitors | Tool completion rate | CPC | CPA < $15 |
| Calculator Users | Starter Kit conversion | CPA | CPA < $50, ROAS > 5x |
| Scorecard Users | Playbook conversion | CPA | CPA < $30, ROAS > 4x |
| Cart Abandoners | Recovery rate | CPA | CPA < $10, Recovery > 40% |

### When to Scale vs. Cut

**Scale (increase budget 20-25%):**
- CPA below target for 7+ days
- ROAS above 5x for 14+ days
- Frequency below 2.0 with strong conversion
- Creative still gaining impressions without fatigue

**Cut (pause and re-evaluate):**
- CPA above 2x target for 7+ days
- Frequency above 4.0 with declining CTR
- Zero conversions after 100+ clicks
- CTR declined 50%+ from launch (creative fatigue)

**Do nothing (let it run):**
- CPA at target ±20%
- ROAS at 2-4x with stable trend
- In learning phase (<50 conversions in 14 days)

---

## CROSS-CHANNEL RETARGETING INTEGRATION

### The Full Picture

Retargeting doesn't happen in isolation. Here's how channels work together:

```
User sees Meta ad → Visits site → Leaves without buying
        │
        ├── [Hour 0-1] Cart abandonment email fires (if cart was started)
        ├── [Hour 1-6] Retargeting ad appears on Facebook/Instagram
        ├── [Day 1] Second retargeting creative + email #2
        ├── [Day 2-3] Google Display retargeting kicks in
        ├── [Day 3-5] LinkedIn retargeting (if B2B audience)
        ├── [Day 5-7] Email #3: objection handler / testimonial
        ├── [Day 7-14] Reduced ad frequency + email #4: urgency
        └── [Day 14+] Move to long-term nurture list (weekly email, low-frequency retargeting)
```

### Channel Priority (by CPA efficiency)

1. **Email** (cheapest, ~$0.01-0.05 per send)
2. **Meta retargeting** (~$3-8 CPA for tool purchases)
3. **Google Display retargeting** (~$5-12 CPA)
4. **LinkedIn retargeting** (~$10-25 CPA, but higher AOV)
5. **Google Search retargeting** (RLSA, ~$2-5 CPA for high-intent keywords)

**Rule of thumb:** Hit them with email first. Layer paid retargeting 12-24 hours later. Different channels, same message.

---

*Track everything. The difference between a 3x ROAS retargeting campaign and a 15x one is usually just 2-3 creative changes and better audience segmentation. Review weekly, optimize monthly.*
