# Dream AI — Client Onboarding Checklist

> **Version:** 1.0.0 | **Last Updated:** 2026-03-18
> **Timeline:** 4 days (Day 1: Setup → Day 2–3: Testing → Day 4: Go-Live)
> **Purpose:** Ensure consistent, thorough onboarding for every Dream AI client.

---

## Overview

```
Day 1: Setup & Configuration
Day 2: Internal Testing & Script Refinement
Day 3: Client Testing & Approval
Day 4: Go-Live & Handoff
```

**Total Steps:** 15
**Owner:** Customer Success Manager (CSM)
**Supporting:** Solutions Engineer, AI Builder

---

## Day 1: Setup & Configuration (Steps 1–5)

### ✅ Step 1: Client Intake & Access Collection
**Owner:** CSM | **Time:** 30 min

- [ ] Send welcome email with onboarding packet
- [ ] Collect completed client intake form:
  - Business name, address, phone, website
  - Business hours and timezone
  - Services/products offered
  - Pricing information
  - Current promotions
  - Brand voice description (formal, casual, etc.)
  - Competitor mentions (who clients compare them to)
- [ ] Collect required access:
  - [ ] Calendar system credentials (OAuth or API key)
  - [ ] CRM credentials (API key or OAuth)
  - [ ] Phone number for call routing
  - [ ] Website access (for chat embed)
  - [ ] Brand assets (logo, colors)
- [ ] Create client account in Dream AI Dashboard
- [ ] Assign Customer Success Manager

**Deliverable:** Complete client profile in Dream AI system

---

### ✅ Step 2: Integration Setup
**Owner:** Solutions Engineer | **Time:** 45 min

- [ ] Connect calendar integration:
  - [ ] Authenticate with calendar platform
  - [ ] Configure available hours and days
  - [ ] Set appointment durations
  - [ ] Add buffer time between appointments
  - [ ] Set blackout dates
  - [ ] Test: Book test appointment → verify on calendar
- [ ] Connect CRM integration:
  - [ ] Authenticate with CRM platform
  - [ ] Map data fields (name, email, phone, source, status)
  - [ ] Configure lead stages/pipeline
  - [ ] Set up tags/labels for AI-generated leads
  - [ ] Test: Create test lead → verify in CRM
- [ ] Configure phone routing (voice agents):
  - [ ] Set up Twilio number or configure forwarding
  - [ ] Test inbound call reaches AI agent
  - [ ] Configure voicemail fallback (if needed)
- [ ] Configure chat embed (chat agents):
  - [ ] Generate embed code
  - [ ] Customize widget appearance (colors, logo, position)
  - [ ] Provide embed code to client (or install on their site)

**Deliverable:** All integrations connected and tested

---

### ✅ Step 3: Knowledge Base & Script Creation
**Owner:** AI Builder | **Time:** 1–2 hours

- [ ] Load niche-specific template from knowledge-bases/niches-kb.md
- [ ] Customize system prompt using system-prompt-template.md:
  - [ ] Fill in all `[PLACEHOLDER]` values
  - [ ] Add client-specific business information
  - [ ] Configure personality/tone per client preference
  - [ ] Set escalation rules
- [ ] Build knowledge base content:
  - [ ] Services/products with descriptions
  - [ ] Pricing information
  - [ ] FAQ (minimum 15 questions)
  - [ ] Common objections and responses
  - [ ] Location/hours/scheduling info
  - [ ] Promotions and special offers
- [ ] Create conversation scripts:
  - [ ] Greeting script
  - [ ] Appointment booking flow
  - [ ] Service inquiry flow
  - [ ] Pricing discussion flow
  - [ ] Transfer to human flow
  - [ ] After-hours handling
  - [ ] Emergency handling (if applicable)
- [ ] Configure lead qualification questions (niche-specific)

**Deliverable:** Complete AI agent configuration with custom scripts and knowledge base

---

### ✅ Step 4: Voice Configuration (Voice Agents Only)
**Owner:** AI Builder | **Time:** 15 min

- [ ] Present 3–5 voice options to client
- [ ] Client selects preferred voice
- [ ] Configure voice settings:
  - [ ] Speed (0.8x–1.2x recommended)
  - [ ] Pitch (adjust to preference)
  - [ ] Emotion level (professional: 0.3, friendly: 0.6)
- [ ] Generate sample recordings:
  - [ ] Greeting: "Hi, this is [Business Name]..."
  - [ ] Booking: "I'd be happy to schedule that for you..."
  - [ ] Transfer: "Let me connect you with someone..."
- [ ] Client approves voice selection
- [ ] Document voice choice for future reference

**Deliverable:** Voice model selected, configured, and approved

---

### ✅ Step 5: End-to-End Smoke Test
**Owner:** Solutions Engineer | **Time:** 30 min

- [ ] Test all call/chat scenarios:
  - [ ] **Happy path:** Customer books appointment successfully
  - [ ] **FAQ:** Customer asks common question → correct answer
  - [ ] **Pricing:** Customer asks about price → answer within range
  - [ ] **After-hours:** Call/chat outside business hours → handled correctly
  - [ ] **Transfer request:** Customer asks for human → transferred
  - [ ] **Emergency:** Urgent request → prioritized appropriately
  - [ ] **Calendar conflict:** Requested time unavailable → alternative offered
  - [ ] **Out-of-scope:** Question outside KB → gracefully handled
- [ ] Verify all integrations:
  - [ ] Calendar: Test appointment appears
  - [ ] CRM: Test lead created with correct data
  - [ ] Phone: Test call quality and routing
  - [ ] Chat: Test widget loads and responds
- [ ] Check for:
  - [ ] No AI hallucination (making up info)
  - [ ] Correct business information
  - [ ] Appropriate tone and personality
  - [ ] Proper data collection before booking
- [ ] Document any issues found
- [ ] Fix critical issues; log minor ones for Day 2–3

**Deliverable:** Smoke test results with pass/fail for all scenarios

---

## Day 2–3: Testing & Refinement (Steps 6–10)

### ✅ Step 6: Internal Testing Sprint
**Owner:** QA Team | **Time:** 2–4 hours

- [ ] Run 50+ test conversations covering:
  - [ ] All service types / products
  - [ ] All FAQ questions
  - [ ] Edge cases (angry customer, confused customer, etc.)
  - [ ] Niche-specific scenarios
  - [ ] Multilingual tests (if applicable)
- [ ] Score each conversation:
  - Accuracy (did it get the facts right?)
  - Tone (was it on-brand?)
  - Completion (did it accomplish the goal?)
  - Handoff (was the transfer smooth?)
- [ ] Identify and prioritize issues:
  - **Critical:** Wrong info, broken flow, safety issue → Fix immediately
  - **Major:** Awkward phrasing, missed opportunity → Fix before client test
  - **Minor:** Polishing, optimization → Log for post-launch
- [ ] Update scripts, knowledge base, and prompts based on findings

**Deliverable:** Test results report + updated agent configuration

---

### ✅ Step 7: Client Testing Session
**Owner:** CSM + Client | **Time:** 30–45 min

- [ ] Schedule testing call with client contact
- [ ] Walk client through the agent:
  - [ ] Show dashboard and analytics
  - [ ] Explain what the agent does and doesn't do
  - [ ] Highlight key capabilities
- [ ] Live test calls/chats:
  - [ ] Client calls the agent (hears their chosen voice)
  - [ ] Client asks common questions
  - [ ] Client tests appointment booking
  - [ ] Client tests edge cases
- [ ] Review test conversations together:
  - [ ] Did the AI say the right things?
  - [ ] Was the tone appropriate?
  - [ ] Were there any "ooh, change that" moments?
- [ ] Document client feedback
- [ ] Prioritize changes:
  - [ ] Must-fix before launch
  - [ ] Can-fix post-launch
  - [ ] Nice-to-have

**Deliverable:** Client feedback documented, must-fix items identified

---

### ✅ Step 8: Script Refinement
**Owner:** AI Builder | **Time:** 1–2 hours

- [ ] Implement all must-fix items from client feedback
- [ ] Refine scripts based on real test conversations
- [ ] Update knowledge base with additional Q&A
- [ ] Adjust AI personality/tone if client requested
- [ ] Re-test all changes
- [ ] Get final client approval on revised scripts
- [ ] **No launch without written client approval on scripts**

**Deliverable:** Final, approved scripts and agent configuration

---

### ✅ Step 9: Analytics & Reporting Setup
**Owner:** Solutions Engineer | **Time:** 15 min

- [ ] Configure reporting:
  - [ ] Daily summary email (calls handled, appointments booked, leads captured)
  - [ ] Weekly analytics report (full metrics dashboard)
  - [ ] Real-time alerts (missed calls, escalations, errors)
- [ ] Set up dashboard access:
  - [ ] Client can view: conversations, appointments, leads, analytics
  - [ ] Client cannot access: backend config, other client data
- [ ] Share dashboard login credentials with client
- [ ] Walk client through dashboard on Day 3 call

**Deliverable:** Reporting configured and dashboard access granted

---

### ✅ Step 10: Go/No-Go Decision
**Owner:** CSM | **Time:** 10 min

- [ ] Review all checklist items
- [ ] Confirm:
  - [ ] All integrations working
  - [ ] Scripts approved by client
  - [ ] Testing passed (90%+ of scenarios)
  - [ ] Client contact available for launch day
  - [ ] Payment method confirmed
- [ ] Make go/no-go decision:
  - **GO:** Proceed to Day 4 launch
  - **NO-GO:** Identify blockers and set new timeline
- [ ] Communicate decision to all stakeholders

**Deliverable:** Go-live confirmed or blockers documented

---

## Day 4: Go-Live & Handoff (Steps 11–15)

### ✅ Step 11: Soft Launch
**Owner:** Solutions Engineer | **Time:** 30 min

- [ ] Route 10–20% of traffic to AI agent:
  - Voice: Set call forwarding to AI for subset of calls
  - Chat: Enable widget on specific pages (not all)
- [ ] Monitor first 2 hours of live traffic:
  - [ ] Real-time conversation review
  - [ ] Check for any errors or failures
  - [ ] Verify integrations are writing data
- [ ] If issues arise:
  - **Minor:** Fix and continue
  - **Major:** Roll back to human, fix, re-deploy
- [ ] Collect first real conversation data

**Deliverable:** Soft launch running, initial data collected

---

### ✅ Step 12: Full Launch
**Owner:** Solutions Engineer | **Time:** 15 min

- [ ] After successful soft launch (2+ hours, no critical issues):
  - [ ] Route 100% of traffic to AI agent
  - [ ] Enable widget on all website pages
  - [ ] Activate full phone routing
- [ ] Confirm with client: "We're live!"
- [ ] Monitor for first 4 hours of full traffic

**Deliverable:** AI agent fully live and handling all traffic

---

### ✅ Step 13: Client Handoff Call
**Owner:** CSM | **Time:** 30 min

- [ ] Schedule handoff call with client (same day as launch or next morning)
- [ ] Cover:
  - [ ] How to read the dashboard
  - [ ] How to review conversations
  - [ ] How to edit scripts (if self-service)
  - [ ] How to request changes
  - [ ] How to escalate issues
  - [ ] Key metrics to watch (first week)
- [ ] Share resources:
  - [ ] Support contact info
  - [ ] Knowledge base documentation
  - [ ] FAQ on "What to expect in the first week"
- [ ] Set expectations for first 30 days:
  - Week 1: Monitor closely, expect script tweaks
  - Week 2: Stabilization, optimization begins
  - Week 3: Review metrics, identify wins
  - Week 4: 30-day review call with full report

**Deliverable:** Client empowered and informed

---

### ✅ Step 14: Post-Launch Monitoring (First 48 Hours)
**Owner:** CSM + QA | **Time:** 2 hours (spread across 48h)

- [ ] Review every conversation for the first 48 hours
- [ ] Flag and fix issues:
  - Wrong information provided
  - Missed booking opportunities
  - Tone/brand inconsistencies
  - Integration failures
- [ ] Make real-time script adjustments
- [ ] Document patterns and optimization opportunities
- [ ] Send client a 48-hour status update

**Deliverable:** First 48 hours reviewed, issues resolved

---

### ✅ Step 15: Success Confirmation
**Owner:** CSM | **Time:** 15 min

- [ ] After 7 days, confirm success criteria:
  - [ ] AI handling 90%+ of incoming calls/chats without transfer
  - [ ] Appointments being booked correctly
  - [ ] Leads flowing to CRM
  - [ ] No critical errors or outages
  - [ ] Client satisfaction confirmed (check in with contact)
- [ ] Schedule 30-day review call
- [ ] Mark onboarding as **COMPLETE** in project tracker
- [ ] Move client to standard support cycle

**Deliverable:** Onboarding complete, client in steady state

---

## Success Criteria Checklist

The onboarding is successful when ALL of the following are true:

| Criteria | Target | Status |
|----------|--------|--------|
| AI handles inbound calls/chats | 90%+ without human transfer | ⬜ |
| Appointments booked correctly | 100% sync to calendar | ⬜ |
| Leads captured in CRM | 100% sync to CRM | ⬜ |
| FAQ accuracy | 95%+ correct answers | ⬜ |
| Client satisfaction | Client confirms satisfaction | ⬜ |
| No critical errors | 0 critical bugs in first 48h | ⬜ |
| Scripts approved | Written client approval obtained | ⬜ |
| Payment confirmed | Active billing on file | ⬜ |
| Client trained | Client can use dashboard independently | ⬜ |
| Support contact established | Client knows how to reach support | ⬜ |

---

## Escalation During Onboarding

| Issue | Severity | Action |
|-------|----------|--------|
| Integration won't connect | Critical | Solutions Engineer resolves within 4 hours |
| Client unreachable for approval | High | CSM attempts contact 3x over 48h; escalate to management if no response |
| AI gives wrong info during testing | High | AI Builder fixes immediately; retest before proceeding |
| Client unhappy with AI tone | Medium | Adjust scripts; retest; get approval |
| Minor bugs found during testing | Low | Log for post-launch; note in handoff |
| Payment not set up | Critical | Block go-live until resolved |

---

*End of Onboarding Checklist*
