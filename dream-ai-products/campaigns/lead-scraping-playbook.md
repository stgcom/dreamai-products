# Dream AI Lead Scraping Playbook

> **Purpose:** Systematic guide for scraping, enriching, and scoring leads across Dream AI's target niches.
> **Last Updated:** 2026-03-18

---

## Table of Contents

1. [Target Sources by Niche](#1-target-sources-by-niche)
2. [Data Points to Capture Per Lead](#2-data-points-to-capture-per-lead)
3. [Lead Enrichment Process](#3-lead-enrichment-process)
4. [Scoring Signals](#4-scoring-signals)
5. [Output Format & CRM Schema](#5-output-format--crm-schema)
6. [Tools & Commands Reference](#6-tools--commands-reference)
7. [Daily Scraping Cadence](#7-daily-scraping-cadence)
8. [Ethical & Legal Notes](#8-ethical--legal-notes)

---

## 1. Target Sources by Niche

### 1A. Real Estate Agents & Brokerages

| Source | URL Pattern | What to Extract |
|--------|------------|-----------------|
| **Google Maps** | `https://www.google.com/maps/search/real+estate+agent+{city}+{state}` | Business name, phone, rating, review count, address, hours, website URL |
| **Zillow Agent Finder** | `https://www.zillow.com/professionals/real-estate-agent-reviews/{city}-{state}/` | Agent name, brokerage, listing count, review count, rating, phone, website |
| **Realtor.com Agent Directory** | `https://www.realtor.com/realestateagents/{city}_{state}` | Agent name, brokerage, specialties, review count, phone, website, license # |
| **LinkedIn** | `https://www.linkedin.com/search/results/people/?keywords=real%20estate%20agent%20{city}` | Owner name, title, connections, company, experience years |
| **Yelp** | `https://www.yelp.com/search?find_desc=real+estate+agent&find_loc={city}+{state}` | Business name, rating, review count, phone, price range |

**Google Maps scrape command (Real Estate):**
```bash
firecrawl scrape "https://www.google.com/maps/search/real+estate+agent+Austin+TX" \
  -o .firecrawl/gmaps-realestate-austin.md \
  --formats markdown
```

**Zillow scrape command:**
```bash
firecrawl scrape "https://www.zillow.com/professionals/real-estate-agent-reviews/austin-tx/" \
  -o .firecrawl/zillow-austin.md \
  --formats markdown
```

**LinkedIn (requires browser):**
```bash
firecrawl browser "https://www.linkedin.com/search/results/people/?keywords=real%20estate%20agent%20Austin%20TX" \
  --session linkedin-search \
  --action snapshot \
  -o .firecrawl/linkedin-re-austin.json
```

---

### 1B. MedSpa / Aesthetics Clinics

| Source | URL Pattern | What to Extract |
|--------|------------|-----------------|
| **Google Maps** | `https://www.google.com/maps/search/medspa+{city}+{state}` | Business name, phone, rating, review count, address, hours, website |
| **RealSelf** | `https://www.realself.com/find-a-doctor?location={city}%2C+{state}&procedure=medspa` | Provider name, procedure count, review count, rating, clinic name, location |
| **Yelp** | `https://www.yelp.com/search?find_desc=medspa&find_loc={city}+{state}` | Business name, rating, review count, phone, price range, website |
| **Instagram** | Manual or via `firecrawl scrape` on profile pages | Follower count, post frequency, booking link in bio, tagged businesses |
| **Google Business Profile** | Direct scrape of individual GBP pages | Hours, review count, Q&A section, posts, booking button presence |

**Google Maps scrape command (MedSpa):**
```bash
firecrawl scrape "https://www.google.com/maps/search/medspa+Scottsdale+AZ" \
  -o .firecrawl/gmaps-medspa-scottsdale.md
```

**RealSelf scrape command:**
```bash
firecrawl scrape "https://www.realself.com/find-a-doctor?location=Scottsdale%2C+AZ&procedure=medspa" \
  -o .firecrawl/realself-scottsdale.md
```

**Instagram profile scrape (note: JS-heavy, may need browser):**
```bash
firecrawl scrape "https://www.instagram.com/{business_username}/" \
  -o .firecrawl/ig-{business}.md
```

---

### 1C. HVAC Companies

| Source | URL Pattern | What to Extract |
|--------|------------|-----------------|
| **Google Maps** | `https://www.google.com/maps/search/HVAC+{city}+{state}` | Business name, phone, rating, review count, address, hours, website |
| **Angi (formerly Angie's List)** | `https://www.angi.com/companylist/{state}/{city}/heating-and-cooling.htm` | Company name, rating, review count, phone, years in business, license info |
| **HomeAdvisor** | `https://www.homeadvisor.com/c.{city}.{state}.heating-cooling.html` | Company name, rating, phone, project count, service area |
| **Yelp** | `https://www.yelp.com/search?find_desc=HVAC&find_loc={city}+{state}` | Business name, rating, review count, phone, website |
| **BBB** | `https://www.bbb.org/us/{state}/{city}/category/heating--air-conditioning/` | Accreditation status, rating, complaint count, years in business, phone |

**Google Maps scrape command (HVAC):**
```bash
firecrawl scrape "https://www.google.com/maps/search/HVAC+Houston+TX" \
  -o .firecrawl/gmaps-hvac-houston.md
```

**Angi scrape command:**
```bash
firecrawl scrape "https://www.angi.com/companylist/tx/houston/heating-and-cooling.htm" \
  -o .firecrawl/angi-houston.md
```

**BBB scrape command:**
```bash
firecrawl scrape "https://www.bbb.org/us/tx/houston/category/heating--air-conditioning/" \
  -o .firecrawl/bbb-hvac-houston.md
```

---

### Batch Scraping: Multi-City Approach

For scaling across cities, use parallel scraping with city lists:

```bash
# Example: Google Maps across 5 cities for HVAC
CITIES=("Houston+TX" "Dallas+TX" "Austin+TX" "San+Antonio+TX" "Phoenix+AZ")

for city in "${CITIES[@]}"; do
  firecrawl scrape "https://www.google.com/maps/search/HVAC+${city}" \
    -o ".firecrawl/gmaps-hvac-${city//+/-}.md" &
done
wait
```

---

## 2. Data Points to Capture Per Lead

### Primary Fields (Always Capture)

| Field | Source | Notes |
|-------|--------|-------|
| `business_name` | Maps, directories | Legal or DBA name |
| `owner_name` | LinkedIn, website about page | Decision-maker name |
| `phone` | Maps, directories, website | Primary business number |
| `email` | Website contact page, LinkedIn | Use enrichment scrape |
| `website` | Maps, directories | Full URL |
| `address` | Maps | Full street address |
| `city` | Maps | Normalized city name |
| `state` | Maps | 2-letter code |
| `zip` | Maps | 5-digit zip |

### Review & Reputation Fields

| Field | Source | Notes |
|-------|--------|-------|
| `google_rating` | Google Maps | 1-5 scale, 1 decimal |
| `google_review_count` | Google Maps | Total number of reviews |
| `yelp_rating` | Yelp | 1-5 scale |
| `yelp_review_count` | Yelp | Total |
| `bbb_rating` | BBB | A+ through F |
| `bbb_accredited` | BBB | Boolean |

### Social & Digital Presence

| Field | Source | Notes |
|-------|--------|-------|
| `instagram_url` | Website footer, Google Maps | Full profile URL |
| `instagram_followers` | IG profile scrape | Approximate count |
| `facebook_url` | Maps, website | Full profile URL |
| `linkedin_url` | LinkedIn search | Owner/company profile |
| `twitter_url` | Website | If present |

### Business Indicators

| Field | Source | Notes |
|-------|--------|-------|
| `years_in_business` | BBB, website | Integer |
| `estimated_employees` | LinkedIn | Range (1-10, 11-50, etc.) |
| `has_online_booking` | Website scrape | Boolean |
| `has_chat_widget` | Website scrape | Boolean |
| `has_ai_chatbot` | Website scrape | Boolean |
| `booking_platform` | Website scrape | e.g., Calendly, Acuity, Jane App |
| `cms_platform` | Website headers/source | WordPress, Squarespace, etc. |
| `has_sms_notifications` | Website/features | Boolean |
| `google_business_posts` | GBP scrape | Active posts? Boolean |
| `review_response_rate` | Google Maps | % of reviews with owner response |

---

## 3. Lead Enrichment Process

### Step 1: Initial Discovery Scrape
Run Google Maps search for niche + city. Extract all business listings from results.

```bash
firecrawl scrape "https://www.google.com/maps/search/{niche}+{city}+{state}" \
  -o ".firecrawl/discover-{niche}-{city}.md"
```

### Step 2: Website Tech Stack Scan
For each discovered website, scrape and analyze for:

```bash
firecrawl scrape "https://{lead_website}" \
  -o ".firecrawl/site-{business_name}.md" \
  --formats markdown,links
```

**What to detect from the scrape output:**

| Signal | What to Look For | Indicates |
|--------|-----------------|-----------|
| **Chat widget** | `livechat`, `intercom`, `drift`, `tawk`, `zendesk`, `crisp`, `hubspot chat` in page source/links | Has customer support automation |
| **AI chatbot** | `openai`, `chatgpt`, `botpress`, `dialogflow`, `ai assistant`, `virtual assistant` | Already using AI |
| **Online booking** | `calendly`, `acuity`, `square appointments`, `vagaro`, `magnoliamedspa`, `zenoti`, `boulevard`, `booking`, `schedule` in links | Has digital booking |
| **SMS platform** | `podium`, `birdeye`, `thryv`, `sms` signup forms | Using reputation mgmt tools |
| **CRM** | `hubspot`, `salesforce`, `zoho`, `pipedrive`, `gohighlevel` in forms/scripts | Has CRM |
| **CMS** | `wp-content`, `squarespace`, `wix`, `shopify` in source | Platform identification |
| **Review widgets** | `birdeye`, `podium`, `grade.us`, `whitespark` | Active review management |
| **AI/receptionist** | `smith.ai`, `patlive`, `answering`, `virtual receptionist` | Outsourced or AI reception |

**Agent-powered structured extraction:**
```bash
firecrawl agent "Extract the following from this business website: 1) Does it have a chat widget? 2) Does it offer online booking? 3) What booking system does it use? 4) Does it mention AI or automation? 5) What CMS does it use? 6) Find any social media links. 7) Find owner/CEO name. 8) Find contact email." \
  "https://{lead_website}" \
  -o ".firecrawl/enrich-{business_name}.json"
```

### Step 3: Google Business Profile Deep Scrape

```bash
firecrawl scrape "https://www.google.com/maps/place/{encoded_business_name}/{place_id}" \
  -o ".firecrawl/gbp-{business_name}.md"
```

**Extract from GBP:**
- Business hours (and if they vary by day)
- Total review count + average rating
- Response rate (count reviews with owner replies)
- Photos count
- Posts (active or stale)
- Q&A section
- "Request a quote" or "Book" button presence
- Attributes (wheelchair accessible, women-owned, etc.)

### Step 4: LinkedIn Owner Research

```bash
firecrawl browser "https://www.linkedin.com/search/results/people/?keywords={owner_name}+{business_name}" \
  --session linkedin-enrich \
  --action snapshot \
  -o ".firecrawl/linkedin-{business_name}.json"
```

**Extract:** Owner profile URL, title, company size, tenure, mutual connections, recent activity.

### Step 5: Assemble Enriched Lead Record

Combine all scrape results into a single lead record (see Section 5 for format).

---

## 4. Scoring Signals

### Scoring Model

Each lead gets a **priority score (1-100)** based on signals:

#### High Intent Signals (Add 20-25 points each)

| Signal | Detection Method | Score |
|--------|-----------------|-------|
| **No chat widget** | Website scrape: no livechat/intercom/drift/tawk found | +25 |
| **No online booking** | Website scrape: no calendly/acuity/square booking found | +25 |
| **Low review count** (< 20) | Google Maps scrape | +20 |
| **No review responses** | GBP scrape: zero owner replies to reviews | +20 |
| **No website at all** | Maps listing with no URL | +25 |
| **Using basic answering service** | Website mentions "call for appointment" only | +20 |

#### Medium Intent Signals (Add 10-15 points each)

| Signal | Detection Method | Score |
|--------|-----------------|-------|
| **Has booking but no AI** | Booking software detected, no AI chatbot | +15 |
| **Slow review responses** | Only some reviews have owner replies | +12 |
| **Generic website** | Wix/Squarespace template, no customization | +12 |
| **No social media** | No IG/FB links found | +12 |
| **No Google Business posts** | GBP scrape: no active posts | +10 |
| **Using legacy CRM** | Salesforce/HubSpot but no automation visible | +10 |

#### Low Intent Signals (Subtract 10-20 points)

| Signal | Detection Method | Score |
|--------|-----------------|-------|
| **Already has AI chatbot** | ChatGPT/Botpress/Dialogflow detected | -20 |
| **Using GoHighLevel** | GHL detected in scripts/forms | -15 |
| **Active review management** | Birdeye/Podium/Grade.us detected | -10 |
| **High review count** (> 100) with AI | Suggests sophisticated ops | -15 |
| **Enterprise booking platform** | Zenoti, Boulevard (MedSpa) | -10 |

### Priority Buckets

| Score Range | Priority | Action |
|-------------|----------|--------|
| **70-100** | 🔴 **High** | Outreach within 24h, personalized offer |
| **40-69** | 🟡 **Medium** | Add to nurture sequence, weekly follow-up |
| **1-39** | 🟢 **Low** | Add to long-term list, quarterly check-in |
| **< 1** | ⚪ **Skip** | Already well-serviced, not a fit |

### Niche-Specific Scoring Adjustments

**Real Estate:**
- Bonus +10 if: agent has < 50 Zillow reviews (indicates growth hunger)
- Bonus +10 if: no IDX website or basic IDX
- Penalty -15 if: Keller Williams / RE/MAX team (likely has corporate AI tools)

**MedSpa:**
- Bonus +10 if: Vagaro or MindBody user (open to tech upgrades)
- Bonus +15 if: < 50 Google reviews in competitive market (Scottsdale, Miami, LA)
- Penalty -20 if: using Zenoti or Boulevard (enterprise MedSpa software)

**HVAC:**
- Bonus +10 if: no service area defined on website
- Bonus +10 if: using basic scheduling (phone-only)
- Penalty -10 if: ServiceTitan or Housecall Pro user (already tech-forward)

---

## 5. Output Format & CRM Schema

### CSV Export Format

```csv
business_name,owner_name,phone,email,website,address,city,state,zip,google_rating,google_review_count,yelp_rating,yelp_review_count,bbb_rating,instagram_url,facebook_url,linkedin_url,has_online_booking,has_chat_widget,has_ai_chatbot,booking_platform,cms_platform,years_in_business,estimated_employees,review_response_rate,priority_score,priority_bucket,niche,source,scrape_date,notes
```

### Example Row

```csv
"Sunstone MedSpa","Dr. Jennifer Park","(480) 555-1234","jennifer@sunstonemedspa.com","https://sunstonemedspa.com","4505 E Chandler Blvd","Phoenix","AZ","85048",4.7,89,4.5,23,A+,"https://instagram.com/sunstonemedspa","https://facebook.com/sunstonemedspa","https://linkedin.com/in/drjenniferpark",true,false,false,Acuity Squarespace,WordPress,8,12,35,75,High,MedSpa,Google Maps,2026-03-18,"No AI, slow review responses"
```

### CRM Import Format (JSON)

```json
{
  "lead_id": "LEAD-2026-0318-001",
  "business": {
    "name": "Sunstone MedSpa",
    "owner": "Dr. Jennifer Park",
    "phone": "(480) 555-1234",
    "email": "jennifer@sunstonemedspa.com",
    "website": "https://sunstonemedspa.com",
    "address": {
      "street": "4505 E Chandler Blvd",
      "city": "Phoenix",
      "state": "AZ",
      "zip": "85048"
    }
  },
  "reputation": {
    "google_rating": 4.7,
    "google_review_count": 89,
    "yelp_rating": 4.5,
    "yelp_review_count": 23,
    "bbb_rating": "A+",
    "review_response_rate": 35
  },
  "social": {
    "instagram": "https://instagram.com/sunstonemedspa",
    "facebook": "https://facebook.com/sunstonemedspa",
    "linkedin": "https://linkedin.com/in/drjenniferpark"
  },
  "tech_stack": {
    "has_online_booking": true,
    "has_chat_widget": false,
    "has_ai_chatbot": false,
    "booking_platform": "Acuity Squarespace",
    "cms": "WordPress",
    "has_sms": false,
    "has_review_management": false
  },
  "business_indicators": {
    "years_in_business": 8,
    "estimated_employees": 12
  },
  "scoring": {
    "priority_score": 75,
    "priority_bucket": "High",
    "signals": {
      "high": ["no_chat_widget", "slow_review_responses"],
      "medium": ["has_booking_no_ai"],
      "low": []
    }
  },
  "meta": {
    "niche": "MedSpa",
    "source": "Google Maps",
    "scrape_date": "2026-03-18",
    "scrape_batch": "scottsdale-medspa-0318",
    "notes": "No AI, slow review responses, open to growth"
  }
}
```

### File Organization

```
.firecrawl/
├── discover-{niche}-{city}-{date}.md        # Initial discovery scrapes
├── gbp-{business-name}.md                   # Google Business Profile details
├── site-{business-name}.md                  # Website scrapes
├── enrich-{business-name}.json              # Agent-enriched structured data
├── linkedin-{business-name}.json            # LinkedIn owner data
├── leads-{niche}-{city}-{date}.csv          # Final lead CSV export
└── leads-{niche}-{city}-{date}-crm.json     # CRM import JSON
```

---

## 6. Tools & Commands Reference

### Firecrawl CLI Commands

```bash
# ─── DISCOVERY ───
# Search for businesses (Google Maps style)
firecrawl scrape "https://www.google.com/maps/search/{query}" -o .firecrawl/discover.md

# Search via firecrawl's native search
firecrawl search "top rated medspa Scottsdale AZ" --scrape --limit 10 -o .firecrawl/search.json --json

# ─── SCRAPING ───
# Single page scrape (static or JS-rendered)
firecrawl scrape "https://{url}" -o .firecrawl/page.md

# Multiple formats
firecrawl scrape "https://{url}" --format markdown,links -o .firecrawl/page.json

# ─── MAPPING ───
# Find all subpages on a business website
firecrawl map "https://{website}" --search "booking" -o .firecrawl/map.json

# ─── ENRICHMENT ───
# AI-powered structured extraction from a business page
firecrawl agent "Extract: business owner name, email, services offered, booking system, chat widget presence" \
  "https://{website}" -o .firecrawl/enrich.json

# ─── CRAWLING ───
# Bulk crawl directory pages (e.g., all Angi listings)
firecrawl crawl "https://www.angi.com/companylist/{state}/{city}/heating-and-cooling/" \
  -o .firecrawl/crawl-hvac-{city}/ --maxDepth 2

# ─── BROWSER (for JS-heavy / interactive pages) ───
# LinkedIn search
firecrawl browser "https://www.linkedin.com/search/results/people/?keywords={query}" \
  --session enrich --action snapshot -o .firecrawl/linkedin.json

# Instagram profiles
firecrawl browser "https://www.instagram.com/{username}/" \
  --session ig-check --action snapshot -o .firecrawl/ig.json
```

### Playwright (for Complex Sites)

For sites where Firecrawl browser isn't sufficient, use Playwright directly:

```bash
# Install if needed
npx playwright install chromium

# Google Maps search and extract listings (JavaScript)
node -e "
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto('https://www.google.com/maps/search/HVAC+Houston+TX');
  await page.waitForSelector('[role=\"feed\"]');
  // Scroll to load more results
  for (let i = 0; i < 5; i++) {
    await page.evaluate(() => document.querySelector('[role=\"feed\"]').scrollTop += 1000);
    await page.waitForTimeout(2000);
  }
  const results = await page.evaluate(() => {
    return [...document.querySelectorAll('[role=\"feed\"] > div')].map(el => ({
      name: el.querySelector('.fontHeadlineSmall')?.textContent,
      rating: el.querySelector('.MW4etd')?.textContent,
      reviews: el.querySelector('.UY7F9')?.textContent,
      phone: el.querySelector('[data-item-id=\"phone\"]')?.textContent,
      website: el.querySelector('[data-item-id=\"authority\"]')?.href,
    }));
  });
  console.log(JSON.stringify(results, null, 2));
  await browser.close();
})();
" > .firecrawl/playwright-gmaps-hvac-houston.json
```

### Batch Automation Script

```bash
#!/bin/bash
# dream-ai-scrape.sh — Daily lead scraping automation
# Usage: ./dream-ai-scrape.sh <niche> <city> <state>

NICHE=$1
CITY=$2
STATE=$3
DATE=$(date +%Y-%m-%d)
BATCH="${CITY// /-}-${NICHE}-${DATE}"

echo "🔥 Dream AI Lead Scraper — ${NICHE} in ${CITY}, ${STATE}"
echo "Batch: ${BATCH}"

mkdir -p ".firecrawl/${BATCH}"

# Step 1: Discovery from Google Maps
echo "[1/4] Discovery scrape..."
firecrawl scrape "https://www.google.com/maps/search/${NICHE}+${CITY}+${STATE}" \
  -o ".firecrawl/${BATCH}/discovery.md"

# Step 2: Secondary discovery from directories
echo "[2/4] Directory scraping..."
case $NICHE in
  realestate)
    firecrawl scrape "https://www.zillow.com/professionals/real-estate-agent-reviews/${CITY}-${STATE}/" \
      -o ".firecrawl/${BATCH}/zillow.md" &
    firecrawl scrape "https://www.realtor.com/realestateagents/${CITY}_${STATE}" \
      -o ".firecrawl/${BATCH}/realtor.md" &
    ;;
  medspa)
    firecrawl scrape "https://www.realself.com/find-a-doctor?location=${CITY}%2C+${STATE}&procedure=medspa" \
      -o ".firecrawl/${BATCH}/realself.md" &
    ;;
  hvac)
    firecrawl scrape "https://www.angi.com/companylist/${STATE}/${CITY}/heating-and-cooling.htm" \
      -o ".firecrawl/${BATCH}/angi.md" &
    firecrawl scrape "https://www.bbb.org/us/${STATE}/${CITY}/category/heating--air-conditioning/" \
      -o ".firecrawl/${BATCH}/bbb.md" &
    ;;
esac
wait

# Step 3: Yelp supplementary
echo "[3/4] Yelp scrape..."
firecrawl scrape "https://www.yelp.com/search?find_desc=${NICHE}&find_loc=${CITY}+${STATE}" \
  -o ".firecrawl/${BATCH}/yelp.md"

# Step 4: Compile initial lead list
echo "[4/4] Done. Parse results and run enrichment next."
echo "📁 Results in .firecrawl/${BATCH}/"
```

### Credit Monitoring

```bash
# Check remaining credits before large batch
firecrawl credit-usage --json --pretty

# Estimate cost: ~1 credit per page scrape
# Typical batch: 50-200 pages = 50-200 credits per city
```

---

## 7. Daily Scraping Cadence

### Daily Schedule

| Time (UTC) | Task | Scope |
|------------|------|-------|
| **06:00** | Discovery scrape — 3 cities × 1 niche | Google Maps + 1 directory per niche |
| **08:00** | Website enrichment — all new leads from discovery | Firecrawl agent extraction per site |
| **10:00** | GBP deep scrape — top 20 leads | Review analysis, response rates |
| **12:00** | LinkedIn enrichment — top 10 leads | Owner identification |
| **14:00** | Scoring & export | Compute scores, export CSV + CRM JSON |
| **16:00** | Deliver to CRM / outreach queue | Update Notion database or CRM |

### Weekly Cadence

| Day | Niche Focus | Cities |
|-----|-------------|--------|
| Monday | Real Estate | Cities 1-3 |
| Tuesday | MedSpa | Cities 1-3 |
| Wednesday | HVAC | Cities 1-3 |
| Thursday | Real Estate | Cities 4-6 |
| Friday | MedSpa | Cities 4-6 |
| Saturday | HVAC + catch-up | Cities 4-6 |
| Sunday | Review & clean data | — |

### Target Volume

| Metric | Daily Target | Weekly Target |
|--------|-------------|---------------|
| New leads discovered | 50-100 | 300-600 |
| Leads enriched (full profile) | 25-50 | 150-300 |
| High-priority leads identified | 5-10 | 30-60 |
| Ready for outreach | 5-10 | 30-60 |

### Cron Job (if running on server)

```bash
# Run daily at 06:00 UTC
0 6 * * * /home/node/.openclaw/workspace/dream-ai-products/campaigns/dream-ai-scrape.sh realestate Austin TX >> /var/log/dream-ai-scrape.log 2>&1
```

---

## 8. Ethical & Legal Notes

### Compliant Scraping Practices

1. **Respect `robots.txt`** — Check before crawling. Firecrawl handles this automatically.
2. **Rate limiting** — Don't exceed Firecrawl concurrency limits. Space requests with `sleep 2` between individual scrapes.
3. **Public data only** — Only scrape publicly available information (listings, public profiles, public reviews).
4. **No PII harvesting** — Don't scrape personal phone numbers or emails from private sources. Use business contacts only.
5. **Terms of Service** — LinkedIn and Instagram have strict ToS. Use browser automation sparingly and rotate sessions.
6. **CAN-SPAM compliance** — All outreach from scraped leads must include opt-out.
7. **GDPR awareness** — For leads in EU jurisdictions, ensure legitimate interest basis and provide data deletion path.
8. **Data retention** — Delete raw scrape files after 90 days. Keep only scored/enriched leads.

### What NOT to Scrape

- ❌ Personal social media profiles (non-business)
- ❌ Password-protected content
- ❌ Private Facebook groups
- ❌付费 directories behind login walls
- ❌ Email addresses from data breach databases
- ❌ Phone numbers listed as "do not call"

---

## Appendix: Quick Reference Commands

```bash
# Full pipeline for a single lead
firecrawl scrape "https://www.google.com/maps/search/medspa+Scottsdale+AZ" -o .firecrawl/discover.md
firecrawl agent "Extract: business name, owner, phone, email, website, rating, review count, services" "https://{lead_website}" -o .firecrawl/enrich.json
firecrawl browser "https://www.linkedin.com/search/results/people/?keywords={owner}+{business}" --session li --action snapshot -o .firecrawl/linkedin.json

# Parse CSV from JSON
jq -r '.data[] | [.business_name, .phone, .email, .website, .google_rating, .priority_score] | @csv' .firecrawl/enrich.json > leads.csv

# Count leads in a batch
wc -l .firecrawl/leads-*.csv

# Find high-priority leads
grep "High" .firecrawl/leads-*.csv
```

---

> **Next Steps:**
> 1. Set up Firecrawl API key in environment: `export FIRECRAWL_API_KEY=fc-...`
> 2. Run discovery scrape for first target city/niche
> 3. Review results and calibrate scoring thresholds
> 4. Connect CRM import to Notion database
> 5. Set up daily cron job for automated scraping
