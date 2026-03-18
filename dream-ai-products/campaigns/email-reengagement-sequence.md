# Dream AI — Re-engagement Email Sequence

## Overview

| Field | Value |
|-------|-------|
| **Sequence** | Re-engagement Sequence |
| **Trigger** | 30+ days of inactivity (no opens, no clicks, no site visits) |
| **Emails** | 3 emails over 30 days (Day 30, 45, 60) |
| **Goal** | Reactivate dormant subscribers OR clean list |
| **Platform** | N8N → HubSpot/PostgreSQL |
| **Segment** | Subscribers with 30+ days zero engagement |

---

## Definition of "Inactive"

A subscriber is considered inactive when ALL of the following are true for 30+ consecutive days:

| Signal | Criteria |
|--------|----------|
| Email opens | 0 opens in last 30 days |
| Email clicks | 0 clicks in last 30 days |
| Website visits | 0 site visits (with tracking cookie) |
| Product views | 0 product page views |
| Replies | 0 email replies |

### Exclusions

Do NOT send re-engagement to:
- Customers who purchased in last 90 days
- Contacts tagged `vip` or `enterprise`
- Contacts currently in another active sequence
- Contacts who already completed a re-engagement sequence in last 180 days

---

## Email 1: Day 30 (The Soft Reconnect)

### Header

| Field | Value |
|-------|-------|
| **Subject** | We miss you, {{firstName}} |
| **Preview text** | Things have changed at Dream AI — and you'll want to see this |
| **Trigger** | 30 days of zero engagement |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
Hey {{firstName}},

It's been a while since we've heard from you, and I wanted to reach out personally.

A lot has changed at Dream AI since you first signed up. Here's what's new:

🚀 NEW: {{feature1Name}}
{{feature1Description}}
Impact: {{feature1Impact}}

🚀 NEW: {{feature2Name}}
{{feature2Description}}
Impact: {{feature2Impact}}

📊 NEW: {{caseStudyHighlight}}
{{caseStudySummary}}

⭐ NEW: {{testimonialHighlight}}
{{testimonialQuote}}
— {{testimonialPerson}}, {{testimonialBusiness}}

---

Since you joined, we've crossed some milestones:

→ {{totalCustomers}}+ businesses now running Dream AI employees
→ {{avgROI}}x average ROI in the first 90 days
→ {{hourSaved}}+ hours saved across all customers (and counting)
→ {{newRating}}/5 average rating

We've also added:
→ Instant setup (was 24hrs, now 15 minutes)
→ 5 new automation templates
→ Live weekly training calls
→ A private community of {{communitySize}}+ business owners

I don't want you to miss out on what's working right now.

[CTA BUTTON: Come back and see what's new →]
{{reengagementLandingLink}}

Or if you'd rather just chat — hit reply. I'd love to hear what you've been working on.

Stephen

P.S. — If you've been busy and just haven't had time to check in, that's okay. That's exactly what Dream AI is built to help with — giving you your time back.
```

### Personalization Variables

| Variable | Source |
|----------|--------|
| `{{firstName}}` | CRM |
| `{{feature1Name}}` | Latest feature release |
| `{{feature1Description}}` | 1-2 sentence description |
| `{{feature1Impact}}` | Business outcome metric |
| `{{caseStudyHighlight}}` | Recent top-performing case study |
| `{{testimonialHighlight}}` | Recent 5-star testimonial |
| `{{totalCustomers}}` | Live metric from DB |
| `{{avgROI}}` | Aggregated customer data |
| `{{hourSaved}}` | Aggregate metric |
| `{{communitySize}}` | Community member count |

### Landing Page

The CTA link goes to a dedicated "What's New" landing page:
- Highlights of all new features
- Recent case studies
- Updated pricing if changed
- Re-opt-in button ("Yes, I'm back!")
- Secondary: "Update my preferences" link

### Engagement Check

After sending, wait 48 hours then check:
| Result | Action |
|--------|--------|
| Opened email | Tag `reengaged-day30`, remove from re-engagement |
| Clicked CTA | Tag `reengaged-active`, move to nurture |
| Opened + replied | Personal response, move to priority |
| No open | Continue to Email 2 |

---

## Email 2: Day 45 (The Direct Ask)

### Header

| Field | Value |
|-------|-------|
| **Subject** | Is this goodbye? |
| **Preview text** | One click to stay — or we'll part as friends |
| **Trigger** | 45 days of zero engagement (15 days after Email 1, no engagement) |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
{{firstName}},

I'll be straightforward with you.

You signed up for Dream AI updates, but I've noticed you haven't opened our emails in a while.

That could mean a few things:

1. You're busy (totally get it)
2. The content isn't relevant to you anymore
3. You've moved on to other solutions

Whatever the case — no hard feelings. I'd rather have a small list of engaged people than a big list of people who don't want to hear from us.

So here's what I need from you:

[CTA BUTTON: 👍 Yes, keep me subscribed →]
{{keepSubscribedLink}}

One click. That's all it takes to stay on the list and keep getting updates, case studies, and the occasional free resource.

Or if you'd rather not — I completely understand. You don't need to do anything. If you don't click the button above, we'll quietly remove you from our list in 15 days. No drama, no guilt trips.

Either way, I appreciate the time you've spent with us.

Stephen

P.S. — If the timing was just bad and you want to come back, hit the button above AND reply to this email. I'd love to know what's going on and how we can help.
```

### CTA Design

| CTA | Link | Action |
|-----|------|--------|
| "Yes, keep me subscribed" | `{{keepSubscribedLink}}` | Tags `reengaged-day45`, removes from re-engagement |
| No action (default) | — | Auto-remove after 15 days |

### The Keep-Subscribed Link

When clicked:
1. Redirects to a simple confirmation page: "You're still subscribed! Here's what's coming..."
2. Updates PostgreSQL: `status = 'active'`, `reengagement_status = 're-engaged'`
3. Triggers: Add to monthly newsletter segment
4. Logs: `reengaged_via = 'day45_direct_ask'`

### Auto-Removal Logic

If no engagement 15 days after Email 2 (Day 60 total):
- Update PostgreSQL: `status = 'inactive'`
- Move contact to `inactive_contacts` table
- Send internal notification: "Contact {{email}} auto-archived after 60 days"
- Do NOT delete — retain for re-import if they opt back in

---

## Email 3: Day 60 (Final Attempt + Gift)

### Header

| Field | Value |
|-------|-------|
| **Subject** | Final email + a gift before we go |
| **Preview text** | A free resource to say thanks — no strings attached |
| **Trigger** | 60 days of zero engagement (15 days after Email 2, no engagement) |
| **From** | Stephen @ Dream AI <stephen@dreamai.co> |

### Content

```
Hey {{firstName}},

This is my last email to you unless you tell me to keep going.

Before I go, I wanted to leave you something useful — whether you stay subscribed or not.

Here's a free resource I think you'll get value from:

🎁 {{giftTitle}}
{{giftDescription}}

[CTA BUTTON: Get your free gift →]
{{giftDownloadLink}}

No email gate. No credit card. Just a useful thing I made that I want you to have.

---

If you do want to stay connected after this:

→ Hit reply and say "stay" — I'll keep you on the list
→ Follow me on Twitter: @dreamaiHQ
→ Check the blog: blog.dreamai.co

And if you ever change your mind, you can always sign up again at dreamai.co. No penalty, no "why did you leave" emails. Clean slate.

Thank you for being part of Dream AI, even for a little while.

All the best,
Stephen

P.S. — That free gift really is good. I use a version of it myself every week. Grab it even if you unsubscribe — it's worth it.
```

### Gift Options (Rotated)

| Gift | Format | URL |
|------|--------|-----|
| "5 AI Quick Wins" mini-guide | PDF (8 pages) | `/resources/quick-wins-guide` |
| "AI Automation Checklist" | PDF (3 pages) | `/resources/automation-checklist` |
| "Weekly Time Audit Template" | Google Sheets | `/resources/time-audit-template` |
| "AI Readiness Scorecard" (lite) | Interactive | `/scorecard/lite` |

### Post-Email 3 Logic

```
IF clicked gift link:
    → Tag: `downloaded-exit-gift`
    → Keep in database but mark `newsletter_only = true`
    → Add to monthly newsletter (lower frequency)

IF opened but no click:
    → Tag: `saw-exit-email-no-action`
    → 15-day grace period → move to `inactive_contacts`

IF not opened:
    → Move to `inactive_contacts` immediately
    → Send internal alert

IF replied:
    → Personal response from Stephen
    → Tag: `manually_reengaged`
    → Restore to active list
```

---

## Sequence Flow Diagram

```
[30 Days Zero Engagement]
        │
        ▼
   Email 1 (Day 30)
   "We miss you" — what's new at Dream AI
        │
        ├──→ Opened/Clicked? → RE-ENGAGED ✅
        │                       Tag: reengaged-day30
        │                       Move to: nurture sequence
        │
        ▼
   [Wait 15 days — no engagement]
        │
        ▼
   Email 2 (Day 45)
   "Is this goodbye?" — one-click to stay
        │
        ├──→ Clicked "keep me"? → RE-ENGAGED ✅
        │                         Tag: reengaged-day45
        │                         Move to: monthly newsletter
        │
        ▼
   [Wait 15 days — no engagement]
        │
        ▼
   Email 3 (Day 60)
   "Final email + free gift"
        │
        ├──→ Downloaded gift? → Tag: exit-gift-downloaded
        │                       Move to: newsletter_only
        │
        ├──→ Replied? → RE-ENGAGED ✅
        │               Personal response
        │
        ▼
   [15 day grace period]
        │
        ▼
   [MOVE TO INACTIVE LIST]
   Status: inactive
   Table: inactive_contacts
   Retain for: 365 days (re-import eligible)
```

## Inactive List Handling

### What Happens at Inactive Status

| Action | Detail |
|--------|--------|
| PostgreSQL update | `contacts.status = 'inactive'` |
| Move row | Copy to `inactive_contacts` table |
| Remove from all sequences | Unenroll from welcome, nurture, re-engagement |
| Keep data | Retain email, name, quiz results for 365 days |
| Newsletter exclusion | Remove from all newsletter segments |

### Reactivation Path

If an inactive contact:
- Opens any future marketing email (from a one-off campaign)
- Visits the website and opts back in
- Subscribes via a new lead magnet

Then:
1. Check `inactive_contacts` for existing data
2. Restore to `contacts` table with `status = 'active'`
3. Tag: `reactivated_from_inactive`
4. Send a "Welcome back" one-off email
5. Add to nurture sequence (not welcome — they've been through it)

### Cleanup Schedule

| Timeframe | Action |
|-----------|--------|
| 30 days inactive | Start re-engagement sequence |
| 60 days inactive | Move to inactive list |
| 180 days inactive | Anonymize personal data (keep email hash) |
| 365 days inactive | Delete from database |

---

## A/B Test Plan

| Email | Test Element | Variant A | Variant B |
|-------|-------------|-----------|-----------|
| Email 1 | Subject | "We miss you, {{firstName}}" | "It's been a while" |
| Email 2 | Tone | Direct ("Is this goodbye?") | Soft ("We want to make sure...") |
| Email 2 | CTA style | Button only | Button + inline text link |
| Email 3 | Gift type | PDF guide | Interactive tool/calculator |
| Email 3 | Subject | "Final email + a gift" | "A parting gift from Dream AI" |

## Metrics Dashboard

| Metric | Email 1 Target | Email 2 Target | Email 3 Target |
|--------|---------------|---------------|---------------|
| Open rate | 15% | 20% | 25% |
| Click rate | 3% | 5% | 8% |
| Re-engagement rate | 5% | 8% | 4% |
| Reply rate | 1% | 3% | 2% |
| **Total re-engagement** | **15-20% of inactive segment** |

## Internal Notifications

| Event | Notification | Channel |
|-------|-------------|---------|
| Email 1 sent (batch) | "Re-engagement batch: {{count}} contacts" | Internal Slack/Discord |
| Re-engagement (any) | "{{firstName}} re-engaged (Day {{day}})" | Internal channel |
| Mass inactive move | "{{count}} contacts moved to inactive" | Internal + Stephen DM |
| Reply received | "Reply from {{email}}: '{{subjectPreview}}'" | Stephen's inbox + alert |
