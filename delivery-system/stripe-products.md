# Stripe Product Setup Guide — Dream AI

> Complete product catalog configuration for all 7 entry-level digital products.

---

## Product Catalog

### Product 1: AI Starter Pack

| Field | Value |
|-------|-------|
| **Name** | AI Starter Pack |
| **Description** | The complete beginner's guide to using AI in your business — prompts, workflows, and tools that save 10+ hours/week |
| **Price** | $27 |
| **Stripe Price ID** | `price_ai_starter_pack` (replace with live ID after creation) |
| **Product Type** | `guide` |
| **Delivery URL** | `https://dreamai.com/dl/ai-starter-pack` |
| **Metadata** | `{"product_id": "ai-starter-pack", "product_type": "guide", "upsell": "ai-automations-pro"}` |
| **Email Sequence** | `guide-sequence` (5-step, starts immediately) |

---

### Product 2: AI Prompts Vault

| Field | Value |
|-------|-------|
| **Name** | AI Prompts Vault |
| **Description** | 100+ battle-tested AI prompts for marketing, sales, content, and operations — copy, paste, customize |
| **Price** | $17 |
| **Stripe Price ID** | `price_ai_prompts_vault` |
| **Product Type** | `template` |
| **Delivery URL** | `https://dreamai.com/dl/prompts-vault` |
| **Metadata** | `{"product_id": "ai-prompts-vault", "product_type": "template", "upsell": null}` |
| **Email Sequence** | `template-sequence` (4-step, starts immediately) |

---

### Product 3: Prompt Engineering Mastery

| Field | Value |
|-------|-------|
| **Name** | Prompt Engineering Mastery |
| **Description** | Go from zero to expert — structured frameworks for writing prompts that actually work, every time |
| **Price** | $37 |
| **Stripe Price ID** | `price_prompt_engineering_mastery` |
| **Product Type** | `course` |
| **Delivery URL** | `https://dreamai.com/learn/prompt-engineering` |
| **Metadata** | `{"product_id": "prompt-engineering-mastery", "product_type": "course", "upsell": "ai-automations-pro"}` |
| **Email Sequence** | `course-sequence` (5-step, starts immediately) |

---

### Product 4: Content Repurposing Playbook

| Field | Value |
|-------|-------|
| **Name** | Content Repurposing Playbook |
| **Description** | Turn 1 piece of content into 10+ assets — the exact system used by top creators |
| **Price** | $22 |
| **Stripe Price ID** | `price_content_repurposing_playbook` |
| **Product Type** | `template` |
| **Delivery URL** | `https://dreamai.com/dl/repurposing-playbook` |
| **Metadata** | `{"product_id": "content-repurposing-playbook", "product_type": "template", "upsell": null}` |
| **Email Sequence** | `template-sequence` (4-step, starts immediately) |

---

### Product 5: Solo Business AI Toolkit

| Field | Value |
|-------|-------|
| **Name** | Solo Business AI Toolkit |
| **Description** | AI-powered systems for one-person businesses — automate your lead gen, onboarding, and follow-up |
| **Price** | $32 |
| **Stripe Price ID** | `price_solo_business_toolkit` |
| **Product Type** | `toolkit` |
| **Delivery URL** | `https://dreamai.com/dl/solo-toolkit` |
| **Metadata** | `{"product_id": "solo-business-toolkit", "product_type": "toolkit", "upsell": "ai-automations-pro"}` |
| **Email Sequence** | `toolkit-sequence` (5-step, starts immediately) |

---

### Product 6: AI Automations Guide

| Field | Value |
|-------|-------|
| **Name** | AI Automations Guide |
| **Description** | Step-by-step guide to building no-code AI automations with n8n, Zapier, and Make |
| **Price** | $27 |
| **Stripe Price ID** | `price_ai_automations_guide` |
| **Product Type** | `guide` |
| **Delivery URL** | `https://dreamai.com/dl/automations-guide` |
| **Metadata** | `{"product_id": "ai-automations-guide", "product_type": "guide", "upsell": "ai-automations-pro"}` |
| **Email Sequence** | `guide-sequence` (5-step, starts immediately) |

---

### Product 7: AI Side Hustle Blueprint

| Field | Value |
|-------|-------|
| **Name** | AI Side Hustle Blueprint |
| **Description** | 7 proven ways to make money with AI — from freelancing to building your own AI-powered product |
| **Price** | $19 |
| **Stripe Price ID** | `price_ai_side_hustle_blueprint` |
| **Product Type** | `guide` |
| **Delivery URL** | `https://dreamai.com/dl/side-hustle-blueprint` |
| **Metadata** | `{"product_id": "ai-side-hustle-blueprint", "product_type": "guide", "upsell": "ai-starter-pack"}` |
| **Email Sequence** | `guide-sequence` (5-step, starts immediately) |

---

## Stripe Dashboard Setup

### Step 1: Create Products in Stripe

For each product above:

1. Go to **Stripe Dashboard → Products → + Add Product**
2. Enter: Name, Description
3. Set Price: one-time, in USD
4. Under **Metadata**, add the key-value pairs from above
5. Copy the generated `price_id` (e.g., `price_1AbCdEf...`)
6. Update the product record in PostgreSQL with the real `stripe_price_id`

### Step 2: Set Up Metadata Fields

Every Stripe product should include these metadata fields:

| Key | Value | Purpose |
|-----|-------|---------|
| `product_id` | (see catalog) | Maps to PostgreSQL `products.slug` |
| `product_type` | guide/template/course/toolkit | Determines email sequence path |
| `upsell` | (product_id or null) | Post-purchase upsell recommendation |

### Step 3: Configure Webhooks

1. Go to **Stripe Dashboard → Developers → Webhooks**
2. Click **+ Add Endpoint**
3. URL: `https://your-n8n-instance.com/webhook/stripe-checkout`
4. Select events:
   - ✅ `checkout.session.completed` — **required** (triggers delivery)
   - ✅ `checkout.session.expired` — optional (log abandoned carts)
   - ✅ `charge.refunded` — optional (revoke access, update CRM)
5. Copy the **Signing Secret** (`whsec_xxx`) → add to N8N credentials as `stripe-webhook-secret`

### Step 4: Create Checkout Sessions (Embedded)

Each product page should create a Stripe Checkout Session with the correct `price_id`:

```javascript
// Example: Frontend checkout button handler
const stripe = Stripe('pk_live_xxx');

async function checkout(priceId) {
  const response = await fetch('/api/create-checkout', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ price_id: priceId })
  });
  const session = await response.json();
  await stripe.redirectToCheckout({ sessionId: session.id });
}

// Usage on product pages:
// <button onclick="checkout('price_ai_starter_pack_live')">Buy Now — $27</button>
```

**Server-side (Node.js example):**

```javascript
const stripe = require('stripe')('sk_live_xxx');

app.post('/api/create-checkout', async (req, res) => {
  const { price_id } = req.body;

  const session = await stripe.checkout.sessions.create({
    mode: 'payment',
    payment_method_types: ['card'],
    line_items: [{ price: price_id, quantity: 1 }],
    success_url: 'https://dreamai.com/thank-you?session_id={CHECKOUT_SESSION_ID}',
    cancel_url: 'https://dreamai.com/cancel',
    metadata: {
      product_id: price_id.replace('price_', '').split('_').slice(1).join('_')
    }
  });

  res.json({ id: session.id });
});
```

---

## Test Mode vs Production Checklist

### Test Mode Setup

| Step | Action | Status |
|------|--------|--------|
| 1 | Use Stripe test keys (`sk_test_xxx`, `pk_test_xxx`) | ☐ |
| 2 | Create all 7 products in **Test Mode** | ☐ |
| 3 | Use N8N instance with test webhook URL | ☐ |
| 4 | Test webhook with Stripe CLI: `stripe listen --forward-to localhost:5678/webhook/stripe-checkout` | ☐ |
| 5 | Create test checkout sessions for each product | ☐ |
| 6 | Use Stripe test card: `4242 4242 4242 4242`, any future expiry, any CVC | ☐ |
| 7 | Verify webhook fires → N8N workflow executes | ☐ |
| 8 | Verify delivery email sent (check SendGrid) | ☐ |
| 9 | Verify customer row in PostgreSQL | ☐ |
| 10 | Verify Telegram notification received | ☐ |
| 11 | Test refund flow: `stripe refunds create` → verify CRM updated | ☐ |
| 12 | Test error path: invalid product_id → verify error alert | ☐ |

### Production Checklist

| Step | Action | Status |
|------|--------|--------|
| 1 | Switch to live keys (`sk_live_xxx`, `pk_live_xxx`) | ☐ |
| 2 | Create all 7 products in **Live Mode** | ☐ |
| 3 | Update `stripe_price_id` values in PostgreSQL with live IDs | ☐ |
| 4 | Update webhook endpoint URL to production N8N | ☐ |
| 5 | Replace webhook signing secret with live `whsec_xxx` | ☐ |
| 6 | Enable webhook events for production endpoint | ☐ |
| 7 | Do a real $1 purchase → verify full delivery flow | ☐ |
| 8 | Verify email, CRM, sequence, Telegram, sales log | ☐ |
| 9 | Refund the test purchase → verify cleanup | ☐ |
| 10 | Monitor N8N execution logs for first 24h | ☐ |
| 11 | Set up Stripe Radar rules for fraud prevention | ☐ |
| 12 | Enable Stripe Tax if selling in multiple jurisdictions | ☐ |

### Test Card Numbers

| Card | Result |
|------|--------|
| `4242 4242 4242 4242` | ✅ Success |
| `4000 0025 0000 3155` | ⚠️ Requires 3D Secure |
| `4000 0000 0000 9995` | ❌ Insufficient funds |
| `4000 0000 0000 0002` | ❌ Card declined |
| `5555 5555 5555 4444` | ✅ Mastercard success |

---

## Price Summary Table

| # | Product | Price | Type | Upsell |
|---|---------|-------|------|--------|
| 1 | AI Starter Pack | $27 | guide | AI Automations Pro |
| 2 | AI Prompts Vault | $17 | template | — |
| 3 | Prompt Engineering Mastery | $37 | course | AI Automations Pro |
| 4 | Content Repurposing Playbook | $22 | template | — |
| 5 | Solo Business AI Toolkit | $32 | toolkit | AI Automations Pro |
| 6 | AI Automations Guide | $27 | guide | AI Automations Pro |
| 7 | AI Side Hustle Blueprint | $19 | guide | AI Starter Pack |

**Total product count:** 7
**Price range:** $17 — $37
**Average price:** $25.86

---

## PostgreSQL Seed Data

Run after creating products in Stripe to populate the database:

```sql
INSERT INTO products (product_name, slug, product_type, price, delivery_url, download_type, description) VALUES
  ('AI Starter Pack', 'ai-starter-pack', 'guide', 27.00, 'https://dreamai.com/dl/ai-starter-pack', 'url', 'The complete beginners guide to using AI in your business'),
  ('AI Prompts Vault', 'ai-prompts-vault', 'template', 17.00, 'https://dreamai.com/dl/prompts-vault', 'url', '100+ battle-tested AI prompts for marketing, sales, content, and operations'),
  ('Prompt Engineering Mastery', 'prompt-engineering-mastery', 'course', 37.00, 'https://dreamai.com/learn/prompt-engineering', 'member_area', 'Go from zero to expert — structured frameworks for writing prompts'),
  ('Content Repurposing Playbook', 'content-repurposing-playbook', 'template', 22.00, 'https://dreamai.com/dl/repurposing-playbook', 'url', 'Turn 1 piece of content into 10+ assets'),
  ('Solo Business AI Toolkit', 'solo-business-toolkit', 'toolkit', 32.00, 'https://dreamai.com/dl/solo-toolkit', 'url', 'AI-powered systems for one-person businesses'),
  ('AI Automations Guide', 'ai-automations-guide', 'guide', 27.00, 'https://dreamai.com/dl/automations-guide', 'url', 'Step-by-step guide to building no-code AI automations'),
  ('AI Side Hustle Blueprint', 'ai-side-hustle-blueprint', 'guide', 19.00, 'https://dreamai.com/dl/side-hustle-blueprint', 'url', '7 proven ways to make money with AI');

-- After creating Stripe products, update with real IDs:
-- UPDATE products SET stripe_product_id = 'prod_xxx', stripe_price_id = 'price_xxx' WHERE slug = 'ai-starter-pack';
-- Repeat for each product.
```

---

*Stripe setup guide version 1.0 — Created 2026-03-18 by CATALYST (via SOVEREIGN subagent)*
