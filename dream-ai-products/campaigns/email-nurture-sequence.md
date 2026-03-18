# Dream AI — Nurture Email Sequence

## Overview

| Field | Value |
|-------|-------|
| **Sequence** | Nurture Sequence |
| **Trigger** | Product page view without purchase (abandoned browse) |
| **Emails** | 4 emails over 14 days |
| **Goal** | Convert warm leads → Starter Kit purchase |
| **Platform** | N8N → HubSpot/PostgreSQL |
| **Segment** | Leads who viewed product/pricing but didn't purchase |

---

## Email 1: Day 0 (Abandoned Browse — Immediate)

### Header

| Field | Value |
|-------|-------|
| **Subject** | Still thinking about it? |
| **Preview text** | No pressure — but here's what you might have missed |
| **Trigger** | Product/pricing page viewed, no purchase within 30 minutes |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
Hey {{firstName}},

I noticed you were looking at the AI Starter Kit earlier today.

No pressure at all — but I wanted to make sure you saw the full picture before deciding.

Here's what {{productName}} actually does for businesses like {{businessName}}:

🎯 IT SOLVES {{painPoint1}}
→ {{painPoint1Detail}}

🎯 IT SOLVES {{painPoint2}}
→ {{painPoint2Detail}}

🎯 IT SOLVES {{painPoint3}}
→ {{painPoint3Detail}}

But I get it — you might have some questions or concerns.

Here are the three I hear most often:

"I'm not technical" → You don't need to be. We set everything up for you.

"What if it doesn't work for my business?" → 30-day money-back guarantee. Zero risk.

"Is this actually worth $297?" → Our average customer sees 8.2x ROI in 90 days. Do the math.

Not ready to jump in yet? That's totally fine.

[CTA BUTTON: Try the free calculator first →]
{{calculatorLink}}

It takes 3 minutes and shows you exactly how much revenue you're leaving on the table. No commitment, no credit card.

Talk soon,
Stephen

P.S. — If you're comparing options, here's the shortcut: nothing else on the market gives you a custom-trained AI employee + 5 automation templates + 30-day guarantee at this price. Not even close.
```

### Personalization Variables

| Variable | Source |
|----------|--------|
| `{{firstName}}` | CRM data |
| `{{businessName}}` | CRM data |
| `{{productName}}` | Product they viewed (Starter Kit, Pro, etc.) |
| `{{painPoint1-3}}` | Dynamic based on their quiz data or industry |
| `{{calculatorLink}}` | Scorecard/calculator URL |

### Tracking

- Page they viewed before email (URL + time on page)
- Whether they returned to site after email
- Calculator completion after click

---

## Email 2: Day 3 (Objection Handling via FAQ)

### Header

| Field | Value |
|-------|-------|
| **Subject** | What's holding you back? |
| **Preview text** | The honest answers to the questions everyone asks |
| **Trigger** | 3 days after Email 1, IF no purchase |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
{{firstName}},

When people don't buy, it's usually one of four things.

I figured I'd just address them head-on. No sales tricks — just straight answers.

---

💰 Q: "Is $297 really worth it?"

A: Our average customer adds $2,400/month in new revenue within 90 days. That's 8x their investment. And if it doesn't work? Full refund. The math is: you either gain $2,400/month or get your $297 back. There's no version of this where you lose.

---

⚙️ Q: "I'm not technical. Is this going to be complicated?"

A: Nope. If you can send an email, you can use Dream AI. We handle all the setup — your AI employee is trained on YOUR business, YOUR processes, YOUR voice. You just tell it what to do, like you would a human employee.

---

🤔 Q: "How do I know this actually works?"

A: Fair question. Here's the proof:
→ 500+ businesses deployed
→ 4.9/5 average rating
→ Case studies with real numbers (not projections)
→ 30-day money-back guarantee (we put our money where our mouth is)

---

⏰ Q: "I don't have time to set this up."

A: That's literally the point. 😄 Setup takes 15 minutes. Then your AI employee handles the work you're spending hours on right now. The time investment is the opposite of what you think — you're investing 15 minutes to save 15+ hours/month.

---

Still on the fence? Start risk-free:

[CTA BUTTON: See our 30-day guarantee →]
{{guaranteePageLink}}

If it doesn't deliver, you pay nothing. That's the deal.

Stephen
```

### FAQ Response Mapping

| If Clicked Section | CRM Tag | Follow-up |
|-------------------|---------|-----------|
| Price section | `objection-price` | Send ROI calculator email |
| Technical section | `objection-complexity` | Send demo video link |
| Trust section | `objection-trust` | Send case study collection |
| Time section | `objection-time` | Send "setup in 15 min" walkthrough |

### CTA

- **Primary:** View guarantee page (30-day guarantee)
- **Secondary:** None

---

## Email 3: Day 7 (Free Value / Lead Magnet)

### Header

| Field | Value |
|-------|-------|
| **Subject** | I made you something |
| **Preview text** | Free guide: 5 AI quick wins you can set up this weekend |
| **Trigger** | 7 days after Email 1, IF no purchase |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
Hey {{firstName}},

I know you've been checking out Dream AI, and I appreciate that you're doing your homework.

Instead of another sales email, I wanted to give you something useful.

I put together a free guide: "5 AI Quick Wins You Can Set Up This Weekend"

No fluff. No "theory." Just five concrete automations you can implement in under an hour each, even if you've never touched AI before.

Here's what's inside:

📘 Quick Win #1: The 24/7 Lead Responder
→ Never miss a lead again, even at 2am. Set this up in 20 minutes.

📘 Quick Win #2: The Content Machine
→ Turn one piece of content into 10. Reuse everywhere.

📘 Quick Win #3: The Inbox Zero System
→ Sort, prioritize, and draft responses to every email automatically.

📘 Quick Win #4: The Meeting Prep Bot
→ Walk into every meeting fully prepared. AI does the homework.

📘 Quick Win #5: The Follow-Up Engine
→ Never let a deal go cold. Automated follow-up sequences that feel personal.

The best part? Four of these five can be set up with free tools.

[CTA BUTTON: Download the free guide →]
{{guideDownloadLink}}

No email gate, no credit card, no strings attached.

If you like these and want to go deeper — the Starter Kit is the next step. But this guide is valuable on its own. Grab it, use it, see for yourself.

Enjoy,
Stephen

P.S. — Quick Win #1 alone (the 24/7 lead responder) has generated an average of $3,200/month in recovered revenue for our customers. And it takes 20 minutes to set up. That's a pretty good ROI on a Saturday afternoon. 🙂
```

### Lead Magnet Details

| Field | Value |
|-------|-------|
| **Title** | 5 AI Quick Wins You Can Set Up This Weekend |
| **Format** | PDF guide (8 pages) |
| **Download link** | Direct download or gated landing page |
| **File size** | ~2MB PDF |
| **Lead magnet tag** | `downloaded-quick-wins-guide` |

### Value Tracking

| Metric | Target |
|--------|--------|
| Download rate | 40%+ of email opens |
| Guide completion (if tracked) | 60%+ |
| Post-download purchase | 15%+ within 7 days |

---

## Email 4: Day 14 (Urgency / Discount)

### Header

| Field | Value |
|-------|-------|
| **Subject** | Last chance: 20% off ends Friday |
| **Preview text** | I rarely do discounts, but this one's too good to sit on |
| **Trigger** | 14 days after Email 1, IF no purchase |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
{{firstName}},

I'm going to keep this short.

I don't do discounts often. But you've been looking at Dream AI for two weeks now, and I want to make it a no-brainer.

For the next 5 days, the AI Starter Kit is 20% off:

BEFORE: $297
NOW: $237

That's $60 off. Use it on whatever — lunch, a coffee habit, or reinvest it back into your business.

Here's what you get at $237:

✅ Your first custom AI employee (trained on your business)
✅ 5 pre-built automation templates
✅ 30-day money-back guarantee
✅ Lifetime access + all future updates
✅ Priority onboarding support

This offer expires Friday at midnight. After that, it goes back to $297.

[CTA BUTTON: Claim your 20% discount — $237 →]
{{discountCheckoutLink}}

Why am I doing this?

Honestly? Because I know the product works. Every customer who implements sees results. The only barrier is getting started. A small discount removes that friction.

But I can't do this all the time — it would devalue what we've built.

So this is it. $237 for the next 5 days.

Stephen

P.S. — Still covered by the 30-day guarantee. If it doesn't save you 10+ hours in the first month, full refund. You literally cannot lose.
```

### Offer Details

| Field | Value |
|-------|-------|
| **Original price** | $297 |
| **Discounted price** | $237 |
| **Discount %** | 20% |
| **Coupon code** | `STARTER20` (auto-applied) |
| **Expires** | 5 days from email send |
| **Guarantee** | 30-day money-back (unchanged) |

### Urgency Sequence (Day 14-19)

| Day | Email | Subject |
|-----|-------|---------|
| 14 | Email 4 | "Last chance: 20% off ends Friday" |
| 17 | Reminder | "3 days left — your $237 discount is waiting" |
| 19 | Final | "Expires tomorrow at midnight" |
| 20 | Expired | "Discount expired — but here's what I can do" |

### Exit Conditions

| Condition | Action |
|-----------|--------|
| Purchases at discount | Tag `purchased-discount-20pct`, enter onboarding |
| Clicks but doesn't purchase | Send reminder emails (days 17, 19) |
| No engagement after Email 4 | Move to re-engagement sequence after 30 days |
| Purchases at full price (pre-discount) | Remove from discount sequence |

---

## Sequence Flow Diagram

```
[Product Page View — No Purchase]
        │
        ▼
   Email 1 (Day 0)
   "Still thinking about it?"
        │
        ├──→ Returned to site & purchased? → EXIT (onboarding)
        │
        ▼
   [Wait 3 days]
        │
        ▼
   Email 2 (Day 3)
   "What's holding you back?" (FAQ/objections)
        │
        ├──→ Clicked objection? → Tag in CRM, route follow-up
        │
        ▼
   [Wait 4 days]
        │
        ▼
   Email 3 (Day 7)
   "I made you something" (free guide)
        │
        ├──→ Downloaded guide? → Tag, watch for post-download purchase
        │
        ▼
   [Wait 7 days]
        │
        ▼
   Email 4 (Day 14)
   "20% off ends Friday" (discount offer)
        │
        ├──→ Purchased? → EXIT (onboarding)
        ├──→ Clicked, no purchase? → Reminder sequence (days 17, 19, 20)
        └──→ No engagement? → EXIT to re-engagement (day 44+)
```

## Segmentation Rules

| Segment | Criteria | Action |
|---------|----------|--------|
| Hot lead | Viewed product page 3+ times | Skip to Email 4 (discount) immediately |
| Warm lead | Viewed product page 1-2 times | Standard sequence |
| Calculator user | Completed calculator before viewing product | Add ROI data to all emails |
| Return visitor | Visited site 5+ times total | Add "social proof" email variant |

## A/B Test Plan

| Email | Test Element | Variant A | Variant B |
|-------|-------------|-----------|-----------|
| Email 1 | Subject | "Still thinking about it?" | "I noticed you were checking us out" |
| Email 2 | Format | FAQ style | "3 reasons people don't buy" |
| Email 3 | CTA | "Download the free guide" | "Get your free quick wins" |
| Email 4 | Discount amount | 20% off ($237) | 15% off + bonus template ($252) |
| Email 4 | Urgency framing | Time-based ("ends Friday") | Quantity-based ("10 spots left") |
