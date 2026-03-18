# Dream AI — Quick Start Deployment Guide

> **Version:** 1.0.0 | **Last Updated:** 2026-03-18
> **Time to Complete:** 30 minutes (setup) + 5–7 business days (build & deploy)
> **Purpose:** Fastest path from "yes" to a live AI agent for any client.

---

## Prerequisites — What You Need Before Starting

| Requirement | Details | Example |
|-------------|---------|---------|
| **Phone Number** | A dedicated number for the voice agent, OR call forwarding from existing number | (555) 123-4567 |
| **Calendar System** | Google Calendar, Calendly, Cal.com, or Acuity — with API access | Google Calendar account |
| **CRM Access** | GoHighLevel, HubSpot, Salesforce, or Pipedrive — API key or OAuth | GHL sub-account |
| **Business Info** | Name, address, hours, services, pricing | See onboarding form |
| **Contact Person** | Someone who can approve scripts and test within 48 hours | Office manager or owner |
| **Payment Method** | Credit card or ACH on file | Visa ending in 4242 |

---

## Step-by-Step: 30-Minute Setup Session

### Step 1: Welcome & Access (5 min)
1. Open the onboarding call/meeting
2. Confirm client's business details
3. Collect calendar + CRM credentials
4. Set up access to their accounts:
   - [ ] Calendar: Connect via API/OAuth
   - [ ] CRM: Connect via API key
   - [ ] Phone: Configure call routing (Twilio/SIP)
5. **Deliverable:** All integrations connected

### Step 2: Script & Knowledge Base (10 min)
1. Walk through the niche-specific script template
2. Customize for their business:
   - [ ] Business name, hours, location
   - [ ] Services / products offered
   - [ ] Pricing ranges
   - [ ] Current promotions
   - [ ] Brand voice (formal, casual, friendly, professional)
3. Gather FAQs (ask: "What do customers ask most often?")
4. **Deliverable:** Approved script + FAQ list

### Step 3: Voice Selection (Voice Agents Only — 3 min)
1. Present 3–5 voice options (from ElevenLabs / OpenAI voice library)
2. Client selects preferred voice
3. Adjust speed and tone if needed
4. Play sample: "Hi, this is [Business Name]. How can I help you today?"
5. **Deliverable:** Voice model selected and confirmed

### Step 4: Calendar Configuration (5 min)
1. Connect calendar integration
2. Set available hours:
   - [ ] Available days
   - [ ] Time windows (e.g., 9am–5pm)
   - [ ] Appointment durations (30 min / 60 min / custom)
   - [ ] Buffer time between appointments
   - [ ] Blackout dates (holidays, closures)
3. Test: Book a fake appointment, verify it appears on calendar
4. **Deliverable:** Calendar tested and working

### Step 5: Go/No-Go Checklist (2 min)
1. Review the deployment checklist:
   - [ ] Phone routing working
   - [ ] Calendar connected and tested
   - [ ] CRM connected and tested
   - [ ] Scripts approved
   - [ ] Voice selected (if applicable)
   - [ ] FAQ uploaded
   - [ ] Payment on file
   - [ ] Client contact confirmed
2. Confirm go-live date
3. **Deliverable:** Deployment approved

### Step 6: Timeline Confirmation (5 min)
1. Set expectations:
   - **Day 1–2:** AI agent build + script finalization
   - **Day 3–4:** Internal testing (our team)
   - **Day 5:** Client testing + approval
   - **Day 6:** Soft launch (limited routing)
   - **Day 7:** Full launch
2. Schedule the Day 5 testing call
3. Share the support contact info
4. **Deliverable:** Clear timeline + next steps

---

## Niche-Specific Quick Start Guides

### Real Estate (30 min setup)

**Special Requirements:**
- Lead source tracking (Zillow, Realtor.com, signs, website)
- Lead qualification script (timeline, budget, buyer/seller, pre-approval)
- CRM pipeline stages for leads

**Setup Steps:**
1. Connect IDX/website lead form (if applicable)
2. Configure CRM lead stages: New → Qualified → Appointment → Showing → Offer → Closed
3. Set up lead scoring rules:
   - +10: Pre-approved
   - +10: Active timeline (within 90 days)
   - +5: Currently working with agent
   - -5: Just browsing / 6+ month timeline
4. Configure AI script for buyer vs. seller qualification
5. Test with a simulated inbound lead

**Calendar:** Block showing windows (e.g., Tue–Sat 10am–4pm)
**CRM Tags:** `ai-qualified`, `buyer`/`seller`, `lead-source-[x]`

---

### MedSpa (30 min setup)

**Special Requirements:**
- Treatment menu with descriptions + price ranges
- Contraindication screening questions
- Consultation booking (not treatment booking — treatments require in-person assessment)

**Setup Steps:**
1. Upload treatment menu to knowledge base
2. Configure AI to discuss treatments (what they do, what to expect)
3. Set contraindication screening:
   - Pregnancy/nursing
   - Active skin conditions
   - Allergies
   - Recent procedures
4. Set up consultation booking (not treatment booking)
5. Configure pre-appointment intake form link

**Calendar:** Block consultation slots (15–30 min), no same-day treatment booking
**CRM Tags:** `ai-consult`, `treatment-interest-[x]`, `new-client`/`returning`
**Compliance:** AI must NOT diagnose or promise outcomes. Transfer medical questions.

---

### HVAC / Home Services (30 min setup)

**Special Requirements:**
- Emergency vs. routine call triage
- Service area mapping (zip codes covered)
- Dispatch rules (which tech, which area)
- Seasonal service menu

**Setup Steps:**
1. Configure emergency triage:
   - No heat/AC → Priority dispatch
   - Gas smell → Emergency (advise safety, dispatch immediately)
   - Water leak → Urgent (same-day or next-day)
   - Maintenance → Standard (schedule within week)
2. Set up service area: List zip codes / cities covered
3. Upload service menu with pricing
4. Configure dispatch rules (if multi-tech):
   - Route by area
   - Route by specialty (commercial vs. residential)
5. Set up estimate follow-up sequence (24hr, 48hr, 7-day)

**Calendar:** Block tech schedule slots (2-hour windows), include travel time
**CRM Tags:** `ai-dispatch`, `emergency`/`routine`, `service-area-[x]`, `estimate-pending`
**Safety:** AI must transfer ANY safety concern to human immediately.

---

## Post-Deployment Checklist

After go-live, monitor for 48 hours:
- [ ] Check first 10 conversations for quality
- [ ] Verify all appointments booked correctly
- [ ] Confirm CRM data is syncing
- [ ] Review any failed calls/chats
- [ ] Adjust scripts based on real interactions
- [ ] Schedule 2-week optimization call with client

---

## Troubleshooting Quick Reference

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| AI not answering | Phone routing misconfigured | Check Twilio forwarding; test with a call |
| Wrong info given | Knowledge base not updated | Update KB; clear cache; retest |
| Calendar not booking | Integration disconnected | Re-authenticate calendar; test |
| CRM not syncing | API key expired | Regenerate API key; reconnect |
| Customer complaints about AI | Script too robotic | Adjust tone; update scripts; A/B test |

---

## Tools & Links

| Resource | URL |
|----------|-----|
| Twilio Console | console.twilio.com |
| Google Calendar | calendar.google.com |
| GoHighLevel | app.gohighlevel.com |
| Calendly | calendly.com |
| N8N Dashboard | dreamain8n.zeabur.app |
| Dream AI Dashboard | dashboard.dreamai.com |
| Support | support@dreamai.com |

---

*End of Quick Start Guide*
