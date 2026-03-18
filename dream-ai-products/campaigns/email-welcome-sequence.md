# Dream AI — Welcome Email Sequence

## Overview

| Field | Value |
|-------|-------|
| **Sequence** | Welcome Sequence |
| **Trigger** | Email opt-in OR quiz/scorecard completion |
| **Emails** | 4 emails over 7 days |
| **Goal** | Convert quiz leads → Starter Kit purchase |
| **Platform** | N8N → HubSpot/PostgreSQL |
| **Segment** | New leads with completed quiz/calculator data |

---

## Email 1: Immediate (Quiz Results Delivery)

### Header

| Field | Value |
|-------|-------|
| **Subject** | Your AI Revenue Report is ready, {{firstName}} |
| **Preview text** | Your personalized AI Readiness Score is in — here's what it means for {{businessName}} |
| **Trigger** | Immediately on quiz submission |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
Hey {{firstName}},

Your AI Readiness Report is ready — and your score is in.

Here's the headline from your results:

📊 YOUR AI READINESS SCORE: {{aiReadinessScore}}/100

Based on your answers about {{businessName}}, here are the three key insights:

1. {{insight1}} — {{insight1Detail}}
2. {{insight2}} — {{insight2Detail}}
3. {{insight3}} — {{insight3Detail}}

The biggest opportunity: {{biggestOpportunity}}

Businesses in your situation typically see {{projectedROI}} in annual revenue left on the table by not automating their {{topProcessArea}}.

Your full report has the complete breakdown, including:
→ Step-by-step automation roadmap for {{businessName}}
→ Estimated ROI if you implement in the next 90 days
→ The 3 processes you should automate first

[CTA BUTTON: See your full AI Readiness Report →]
{{reportLink}}

Talk soon,
Stephen

P.S. — This report was generated specifically for {{businessName}} based on {{industry}} business benchmarks. It's not a generic template — it's your roadmap.
```

### Personalization Variables

| Variable | Source |
|----------|--------|
| `{{firstName}}` | Quiz form submission |
| `{{businessName}}` | Quiz form submission |
| `{{aiReadinessScore}}` | Calculated from quiz answers |
| `{{insight1-3}}` | Generated based on quiz responses |
| `{{biggestOpportunity}}` | Highest-scoring automation gap |
| `{{projectedROI}}` | Calculator output |
| `{{topProcessArea}}` | Highest-priority process from quiz |
| `{{reportLink}}` | Personalized report URL with tracking |

### Metrics to Track

- Open rate (target: 60%+)
- Click rate to report (target: 35%+)
- Report view duration
- Forward/share rate

---

## Email 2: Day 2 (The Cost of Inaction)

### Header

| Field | Value |
|-------|-------|
| **Subject** | The $6,200 mistake {{businessName}} is making |
| **Preview text** | Most {{industry}} businesses don't realize they're bleeding money here |
| **Trigger** | 2 days after Email 1, IF email 1 was opened |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
{{firstName}},

I looked at your AI Readiness Report and one number jumped out.

Right now, {{businessName}} is spending roughly {{monthlyHoursLost}} hours/month on {{specificProcess}} — work that an AI employee could handle in a fraction of the time.

At your hourly rate, that's approximately ${{estimatedMonthlyLoss}}/month in owner time lost to tasks that don't need a human.

That's ${{estimatedAnnualLoss}}/year. Gone.

Here's the thing — this isn't unusual. A {{industry}} business just like yours, {{competitorAnalogy}} in {{analogousCity}}, was in the same spot 6 months ago.

They were:
- Spending 25+ hours/week on {{similarProcess}}
- Missing leads because follow-up was manual
- Losing deals to faster competitors

They implemented a Dream AI employee — and within 90 days:

✅ {{benefit1}}
✅ {{benefit2}}
✅ {{benefit3}}

The result? They freed up {{hoursSaved}} hours/month and added ${{revenueIncrease}} in new revenue.

You have the same opportunity in front of you right now.

[CTA BUTTON: See how we helped a business like yours →]
{{caseStudyLink}}

Stephen

P.S. — That ${{estimatedMonthlyLoss}}/month? That's not a hypothetical. That's what your calculator results showed. The question is whether you keep losing it or start recapturing it this quarter.
```

### Personalization Variables

| Variable | Source |
|----------|--------|
| `{{monthlyHoursLost}}` | Calculated from quiz (time spent on automatable tasks) |
| `{{specificProcess}}` | Top automation opportunity from quiz |
| `{{estimatedMonthlyLoss}}` | hours × hourly rate |
| `{{estimatedAnnualLoss}}` | monthly × 12 |
| `{{competitorAnalogy}}` | Rotated from similar business database |
| `{{analogousCity}}` | Different city for anonymity |
| `{{benefit1-3}}` | Case study outcomes |
| `{{hoursSaved}}` | Case study metric |
| `{{revenueIncrease}}` | Case study metric |
| `{{caseStudyLink}}` | Landing page URL |

### Engagement Check

- IF Email 1 NOT opened → Send Email 2 anyway (may have missed Email 1)
- IF Email 1 opened but report NOT clicked → Stronger urgency in copy
- IF report clicked → Standard version

---

## Email 3: Day 4 (Conversation Starter)

### Header

| Field | Value |
|-------|-------|
| **Subject** | Quick question, {{firstName}} |
| **Preview text** | I'd love to know what's on your mind about AI automation |
| **Trigger** | 4 days after opt-in |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
Hey {{firstName}},

Quick one — no pitch, just a genuine question.

What's the #1 thing holding you back from automating {{businessName}}'s {{topProcessArea}}?

I ask because I talk to {{industry}} business owners every day, and the answer is almost always one of these:

→ "I don't know where to start"
→ "I'm worried it's too expensive"
→ "I'm not sure AI can actually handle my work"
→ "I don't have time to set it up"
→ "I've been burned by 'automation' tools before"

Whatever yours is — I'd love to hear it.

Just hit reply and type one sentence. I read every response personally.

Why do I care? Because if I know what's stopping you, I can help. And if Dream AI isn't the right fit, I'll tell you that too.

Talk soon,
Stephen

P.S. — If you're already thinking "I don't have time for this" — that's exactly why automation exists. 🙂
```

### CTA

- **Primary:** Reply to email (direct response)
- **Secondary:** None (single action)

### Reply Handling

| Reply Contains | Action |
|----------------|--------|
| Objection about price | Route to "pricing" nurture tag, send ROI breakdown email |
| Objection about complexity | Route to "ease" tag, send demo video link |
| Objection about trust/results | Route to "proof" tag, send case study collection |
| General interest/question | Personal reply from Stephen within 4 hours |
| Unsubscribe request | Immediate unsubscribe + remove from all sequences |

### Metrics to Track

- Reply rate (target: 5-10%)
- Reply sentiment (positive/neutral/negative)
- Objection type distribution

---

## Email 4: Day 7 (Social Proof + Offer)

### Header

| Field | Value |
|-------|-------|
| **Subject** | Case study: +$45,000/mo from one AI employee |
| **Preview text** | How Sarah Mitchell added a $540K/year revenue stream with AI |
| **Trigger** | 7 days after opt-in |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
{{firstName}},

I want to tell you about Sarah Mitchell.

Sarah runs a real estate agency in Austin. When she came to Dream AI 8 months ago, she was:
- Spending 30+ hours/week on lead follow-up and appointment scheduling
- Losing 40% of inbound leads because she couldn't respond fast enough
- Working 60-hour weeks just to keep up

Sound familiar?

Here's what happened after she implemented her Dream AI employee:

📈 Month 1: All lead follow-up automated. Sarah freed up 20 hours/week.

📈 Month 3: Response time dropped from hours to 47 seconds. Lead conversion up 34%.

📈 Month 6: Revenue up $45,000/month. Sarah took her first vacation in 2 years.

📈 Month 8: She now has 3 AI employees handling different parts of her business.

Sarah's exact words: "I wish I'd done this 3 years ago. It's like I hired a team of 5 for the cost of one."

---

This isn't a one-off.

500+ businesses have deployed Dream AI employees.
⭐ 4.9/5 average rating
📊 Average ROI: 8.2x in the first 90 days

The AI Starter Kit gets you everything Sarah started with:

→ Your first custom AI employee (trained on your business)
→ 5 pre-built automation templates
→ 30-day money-back guarantee
→ Lifetime access + updates

[CTA BUTTON: Get the Starter Kit — $297 →]
{{starterKitLink}}

Stephen

P.S. — The Starter Kit includes a 30-day money-back guarantee. If it doesn't save you at least 10 hours in the first month, you pay nothing. No risk, no catch.
```

### Social Proof Elements

| Element | Value |
|---------|-------|
| Case study | Sarah Mitchell, real estate, Austin |
| Revenue increase | +$45,000/month |
| Time saved | 20+ hours/week |
| Total businesses | 500+ |
| Rating | 4.9/5 |
| Average ROI | 8.2x in 90 days |

### Offer Details

| Field | Value |
|-------|-------|
| Product | AI Starter Kit |
| Price | $297 |
| Guarantee | 30-day money-back |
| Includes | Custom AI employee, 5 templates, support |

---

## Sequence Flow Diagram

```
[Quiz/Opt-in Complete]
        │
        ▼
   Email 1 (Immediate)
   "Your AI Revenue Report is ready"
        │
        ▼
   [Wait 2 days]
        │
        ▼
   Email 2 (Day 2)
   "The $6,200 mistake"
        │
        ▼
   [Wait 2 days]
        │
        ▼
   Email 3 (Day 4)
   "Quick question" (reply-request)
        │
        ▼
   [Wait 3 days]
        │
        ▼
   Email 4 (Day 7)
   "Case study: +$45K/mo" → Starter Kit offer
        │
        ├──→ Purchased? → Move to onboarding sequence
        ├──→ Replied? → Handle in CRM
        └──→ No purchase? → Enter nurture sequence
```

## Exit Conditions

| Condition | Action |
|-----------|--------|
| Purchases Starter Kit | Exit welcome → Enter onboarding/delivery |
| Replies to Email 3 | Tag in CRM, personal follow-up |
| Unsubscribes | Immediate removal from all sequences |
| No opens after Email 4 | Move to re-engagement sequence after 30 days |
| Clicks but no purchase | Enter nurture sequence (day 0) |

## A/B Test Plan

| Email | Test Element | Variant A | Variant B |
|-------|-------------|-----------|-----------|
| Email 1 | Subject line | "Your AI Revenue Report is ready" | "Your score is in, {{firstName}}" |
| Email 2 | Dollar amount | $6,200 (calculated) | $10,000 (rounded up) |
| Email 4 | CTA button color | Blue | Green |
| Email 4 | Price framing | "$297" | "Less than $10/day" |
