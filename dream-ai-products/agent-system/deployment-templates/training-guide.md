# Dream AI — Agent Training Guide

> **Version:** 1.0.0 | **Last Updated:** 2026-03-18
> **Purpose:** Train a Dream AI agent on a specific client's business so it responds accurately, on-brand, and with real expertise.
> **Audience:** AI Builders, Customer Success Managers, and technically inclined clients.

---

## Overview

Training an AI agent means giving it the right information so it can represent your client's business accurately. This guide covers:

1. **FAQ Import** — Get the client's most common questions into the knowledge base
2. **Script Customization** — Make the AI sound like the client, not a generic bot
3. **Knowledge Base Structuring** — Organize info for optimal retrieval
4. **Voice Selection** — Choose and configure the right voice (voice agents)
5. **Testing & Iteration** — Validate the training with real scenarios

---

## 1. FAQ Import Process

### Step 1: Gather Raw FAQs

**Sources to mine:**

| Source | How to Get It |
|--------|---------------|
| Website FAQ page | Scrape or copy-paste |
| Google Business Profile Q&A | Export from GBP dashboard |
| Support ticket history | Export from CRM/helpdesk |
| Sales team knowledge | Interview top 2 salespeople |
| Call recordings | Review top 20 call transcripts |
| Customer emails | Look for repeated questions |
| Competitor websites | What questions do competitors answer? |

**Interview questions for the client:**
1. "What are the top 10 questions customers ask on the phone?"
2. "What do people ask on your website / Google profile?"
3. "What questions come up during the sales process?"
4. "What questions do customers ask after purchasing?"
5. "What's the #1 question you wish people would stop asking?"

### Step 2: Structure FAQs for AI Consumption

Format each FAQ with clear question + answer pairs:

```markdown
## [Category Name]

### Q: [Exact customer question]
**A:** [Complete answer the AI should give]

### Q: [Exact customer question]
**A:** [Complete answer the AI should give]
```

**Example:**
```markdown
## Pricing & Payments

### Q: How much does a furnace installation cost?
**A:** Furnace installations typically range from $3,500 to $7,500 depending on the size of your home, the efficiency rating you choose, and whether any ductwork modifications are needed. We offer free in-home estimates where we can give you an exact price. Would you like to schedule one?

### Q: Do you offer financing?
**A:** Yes! We offer 0% financing for 18 months on installations over $3,000. We also have monthly payment plans starting at $99/month. Would you like me to connect you with our financing team to see what you qualify for?
```

### Step 3: Import into Knowledge Base

**Option A: Manual Entry (small KB, <50 FAQs)**
1. Open the knowledge base file (`[client]-kb.md`)
2. Add structured Q&A pairs under appropriate categories
3. Upload to the AI system (OpenAI Assistants, N8N, etc.)

**Option B: Bulk Import (large KB, 50+ FAQs)**
1. Create a CSV with columns: `category`, `question`, `answer`
2. Use the import script:

```python
import csv
import json

def convert_faq_csv_to_kb(csv_path, output_path):
    """Convert FAQ CSV to structured knowledge base markdown."""
    with open(csv_path, 'r') as f:
        reader = csv.DictReader(f)
        categories = {}
        
        for row in reader:
            cat = row['category']
            if cat not in categories:
                categories[cat] = []
            categories[cat].append({
                'q': row['question'],
                'a': row['answer']
            })
    
    with open(output_path, 'w') as f:
        f.write(f"# {client_name} - FAQ Knowledge Base\n\n")
        for cat, faqs in categories.items():
            f.write(f"## {cat}\n\n")
            for faq in faqs:
                f.write(f"### Q: {faq['q']}\n")
                f.write(f"**A:** {faq['a']}\n\n")
    
    print(f"Generated {len(categories)} categories, {sum(len(v) for v in categories.values())} FAQs")

# Usage
convert_faq_csv_to_kb('client_faq.csv', 'client-kb.md')
```

**Option C: API Import (OpenAI Assistants)**
```python
from openai import OpenAI

client = OpenAI()

# Upload FAQ file
file = client.files.create(
    open("client-faq-kb.md", "rb"),
    purpose="assistants"
)

# Add to assistant's file search
assistant = client.beta.assistants.update(
    assistant_id="asst_xxx",
    tool_resources={
        "file_search": {
            "vector_store_ids": ["vs_xxx"]
        }
    }
)
```

### Step 4: Validate FAQ Coverage

After import, run these test queries:
- [ ] All FAQ questions (do you get correct answers?)
- [ ] 5 random customer questions (not in FAQ — does it handle gracefully?)
- [ ] Ambiguous questions (does it ask for clarification?)
- [ ] Trick questions (does it avoid hallucinating?)

---

## 2. Script Customization Guide

### 2.1 Anatomy of a Good Script

```
[GREETING] → [DISCOVERY] → [VALUE] → [ACTION] → [CLOSE]

Example (HVAC):
1. GREETING: "Thank you for calling Arctic Air! This is your virtual assistant. How can I help you today?"
2. DISCOVERY: "I'd be happy to help with that. Can I get a bit more detail about what you're experiencing?"
3. VALUE: "Based on what you're describing, it sounds like we can help. We have same-week availability and offer free estimates."
4. ACTION: "I have an opening on Tuesday at 2pm or Wednesday at 10am. Which works better?"
5. CLOSE: "Great, you're all set for Tuesday at 2pm. You'll receive a confirmation text shortly. Is there anything else?"
```

### 2.2 Customization Points

| Element | What to Customize | How |
|---------|-------------------|-----|
| **Greeting** | Business name, AI name, warmth level | Update greeting script with client's preferences |
| **Discovery** | Qualification questions per niche | Use niche-specific questions from niches-kb.md |
| **Value Props** | Key differentiators, guarantees | Interview client for their unique selling points |
| **Call to Action** | What the AI should propose (booking, callback, etc.) | Define the desired outcome per call type |
| **Tone & Language** | Formal vs. casual, industry jargon | Client brand voice interview |
| **Error Handling** | What AI says when it doesn't know | Configure fallback responses |
| **Transfer Script** | How AI hands off to human | Define when and how to transfer |

### 2.3 Tone Calibration

| Client Preference | AI Tone Setting | Example Language |
|-------------------|----------------|------------------|
| **Corporate/Formal** | Professional, measured | "I'd be happy to assist you with that request." |
| **Friendly/Local** | Warm, conversational | "Oh no! Let's get that sorted for you right away." |
| **Premium/Luxury** | Confident, sophisticated | "That's an excellent choice. Let me arrange that for you." |
| **Medical/Professional** | Reassuring, clear | "I understand your concern. Let me get you the right information." |

**Tip:** Write 5 sample sentences in the client's preferred tone and have them approve before configuring the AI.

### 2.4 Script Templates by Scenario

#### Inbound Service Inquiry
```
AI: "Hi, thanks for calling [BUSINESS]! I'm [AI_NAME], your virtual assistant. How can I help you today?"

[Customer describes issue]

AI: "I see — [paraphrase the issue back]. We definitely handle that. Let me ask a few quick questions so I can help you best:
1. Can I get your name?
2. What's the best callback number?
3. When would be a good time for our team to reach you?"

[If booking]
AI: "I have availability [OPTION_1] or [OPTION_2]. Which works better?"
[If not booking]
AI: "Perfect, I've passed this along to our team. You'll hear from us within [TIMEFRAME]. Anything else I can help with?"
```

#### Pricing Inquiry
```
AI: "Great question! [SERVICE_NAME] typically ranges from [LOW] to [HIGH] depending on [FACTORS]. The best way to get an exact price is with a free [estimate/consultation]. I can book that for you right now — what day works best?"
```

#### Objection Handling
```
[Too expensive]
AI: "I understand budget matters. Our clients find that [VALUE PROPOSITION]. We also offer [FINANCING/OPTIONS] to make it more manageable. Would it help to get a specific quote for your situation?"

[Need to think about it]
AI: "Absolutely — take your time. What questions can I answer to help you decide? And just so you know, [CURRENT PROMOTION/URGENCY]. Would you like me to follow up in a few days?"
```

### 2.5 A/B Testing Scripts

For optimization, create 2 versions of key scripts:
- **Version A:** Direct approach ("Would you like to book?")
- **Version B:** Value approach ("Most of our clients see results within X. Want to get started?")

Track: booking rate, conversation length, customer satisfaction for each version. Keep the winner.

---

## 3. Knowledge Base Structuring

### 3.1 Optimal Structure for RAG

```
# [Client Name] Knowledge Base

## 1. Business Overview
- Who you are
- What you do
- Where you're located
- Hours of operation

## 2. Services / Products
### [Service 1]
- Description
- What's included
- Price range
- Who it's for
### [Service 2]
...

## 3. FAQ
### General
### Pricing
### Scheduling
### Policies
...

## 4. Policies
- Cancellation policy
- Payment terms
- Warranty / guarantees
- Privacy / data handling

## 5. Scripts & Responses
- Greeting variations
- Objection responses
- Closing variations

## 6. Competitive Positioning
- Why choose us over [competitor]
- Our guarantees
- Differentiators
```

### 3.2 Chunking Best Practices

| Chunk Type | Size | When to Use |
|-----------|------|-------------|
| **FAQ pairs** | 50–150 tokens each | Best for Q&A retrieval |
| **Service descriptions** | 150–300 tokens | For detailed inquiries |
| **Scripts** | 200–500 tokens per scenario | For conversation flow |
| **Policies** | 100–200 tokens each | For specific policy questions |
| **General info** | 300–500 tokens | For overview questions |

### 3.3 Metadata Tags

Tag each chunk with metadata for better retrieval:
```
[category: pricing]
[service: furnace-installation]
[audience: homeowner]
[urgency: standard]
```

---

## 4. Voice Selection (Voice Agents)

### 4.1 Voice Selection Criteria

| Factor | Options | Recommendation |
|--------|---------|----------------|
| **Gender** | Male / Female / Neutral | Match client preference; test both |
| **Age** | Young (20s) / Middle (30s-40s) / Mature (50s+) | Match target customer demographic |
| **Accent** | Neutral American / Regional / International | Neutral American for broad appeal; regional for local businesses |
| **Tone** | Warm / Professional / Energetic / Calm | Match brand personality |
| **Speed** | 0.7x – 1.3x | 0.9x–1.0x recommended; slower for older demographics |

### 4.2 Available Voice Models

| Provider | Best For | Notes |
|----------|----------|-------|
| **ElevenLabs** | Premium, natural voices | Largest library; custom voice cloning available |
| **OpenAI** | Reliable, good default | TTS and voice modes; fewer customization options |
| **Google Cloud TTS** | Enterprise, multi-language | WaveNet voices; strong multilingual |
| **Amazon Polly** | Cost-effective, reliable | Neural voices available; good for high volume |

### 4.3 Voice Testing Protocol

1. **Greeting test:** Play the business greeting in the selected voice
2. **Information test:** AI reads a pricing paragraph — is it clear?
3. **Emotional test:** AI delivers bad news (sold out) — is it empathetic?
4. **Speed test:** AI reads a complex instruction — is it easy to follow?
5. **Customer test:** If possible, have a real customer test and give feedback

### 4.4 Custom Voice Cloning (Premium Feature)

For clients wanting a truly unique voice:
1. Collect 5–10 minutes of the client's (or their best employee's) voice
2. Clean audio (remove background noise, pauses)
3. Submit to ElevenLabs or similar cloning service
4. Fine-tune and test
5. Deploy as the AI's voice

**Legal note:** Get written consent for voice cloning. Document that it's an AI representation.

---

## 5. Testing & Iteration Framework

### 5.1 Test Script (Run After Every Training Update)

```markdown
## Test: [Date]

### Scenario 1: Happy Path
- [ ] AI greets correctly
- [ ] AI collects required info (name, phone, email, reason)
- [ ] AI books appointment successfully
- [ ] Appointment appears in calendar
- [ ] Lead appears in CRM
- **Result:** PASS / FAIL
- **Notes:**

### Scenario 2: FAQ Accuracy (test 10 random FAQs)
- [ ] FAQ 1: [Q] → [Expected A] → [Actual A] → PASS/FAIL
- [ ] FAQ 2: ...
- **Accuracy:** X/10
- **Notes:**

### Scenario 3: Objection Handling (test 3 objections)
- [ ] "Too expensive" → [Response] → PASS/FAIL
- [ ] "Need to think about it" → [Response] → PASS/FAIL
- [ ] "I already have someone" → [Response] → PASS/FAIL
- **Result:** PASS / FAIL
- **Notes:**

### Scenario 4: Edge Cases
- [ ] AI doesn't know the answer → Transfers correctly?
- [ ] Customer is angry → De-escalates?
- [ ] Customer asks for human → Transfers immediately?
- [ ] After-hours call → Handled appropriately?
- **Result:** PASS / FAIL
- **Notes:**

### Scenario 5: Integration Check
- [ ] Calendar booking syncs
- [ ] CRM lead created
- [ ] Notifications sent
- [ ] Analytics logged
- **Result:** PASS / FAIL
- **Notes:**

### Overall Score: X/5 scenarios passed
### Action Items:
1.
2.
3.
```

### 5.2 Iteration Cycle

```
Week 1: Train → Test → Fix → Test (daily)
Week 2: Monitor real conversations → Identify patterns → Adjust
Week 3: Client feedback → Script refinements → A/B test
Week 4: Full review → Optimization report → Next month plan
Ongoing: Monthly review cycle (test, adjust, report)
```

### 5.3 Key Metrics to Track After Training

| Metric | Target | How to Measure |
|--------|--------|----------------|
| FAQ Accuracy | 95%+ | Test 20 random FAQs weekly |
| Booking Completion Rate | 80%+ | Conversations that end in a booking |
| Customer Satisfaction | 4.5+/5 | Post-call survey or dashboard rating |
| Transfer Rate | <20% | % of calls transferred to human |
| Average Handle Time | <3 min | Dashboard analytics |
| Lead Quality | 70%+ qualified | CRM data review |

---

## 6. Training Checklist (Per Client)

Complete before go-live:

- [ ] Raw FAQs gathered from client (min. 15)
- [ ] FAQs structured and imported into knowledge base
- [ ] Business overview written (who, what, where, when)
- [ ] Services/products documented with pricing
- [ ] Policies documented (cancellation, payment, warranty)
- [ ] Scripts customized (greeting, booking, objections, closing)
- [ ] Tone and personality configured
- [ ] Voice selected and tested (voice agents)
- [ ] Calendar configured with correct availability
- [ ] CRM integration tested (lead creation)
- [ ] All test scenarios passed (min. 4/5)
- [ ] Client approved final scripts (written approval)
- [ ] Training documentation delivered to client

---

## 7. Common Training Mistakes to Avoid

| Mistake | Why It's Bad | Fix |
|---------|-------------|-----|
| Too little information | AI hallucinates or gives generic answers | Minimum 15 FAQs + full business overview |
| Too much unstructured text | AI can't find the right answer; retrieval is poor | Use structured headers, categories, Q&A pairs |
| Generic scripts | AI sounds like every other chatbot | Customize with client's actual language, differentiators |
| No error handling | AI freezes or says "I don't know" | Build in graceful transfers and "let me check" responses |
| Not testing with real questions | Missed edge cases discovered post-launch | Test with real customer questions from call recordings |
| Set-and-forget training | AI becomes outdated as business changes | Monthly review cycle; update KB when business changes |
| Wrong voice for demographic | Customers feel disconnected | Match voice age/tone to client's target customer |

---

*End of Training Guide*
