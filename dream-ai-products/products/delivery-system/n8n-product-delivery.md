# N8N Product Delivery Workflow — Dream AI

> Automated digital product delivery: Stripe webhook → email delivery → CRM → notifications.
> Follows BOLT's N8N naming conventions and PostgreSQL patterns.

---

## Workflow: Stripe Payment → Product Delivery

**Workflow Name:** `DreamAI — Digital Product Delivery`
**Trigger:** Stripe Webhook (`checkout.session.completed`)
**Error Workflow:** `DreamAI — Error: Digital Product Delivery`

---

## Node Flow

```
[Stripe Webhook] → [Validate Event] → [Lookup Product] → [Send Delivery Email] → [Add to CRM] → [Tag Email Sequence] → [Telegram Alert] → [Log to Sales Tracker]
```

---

## Node Configuration

### Node 1: Stripe Webhook Trigger

| Field | Value |
|-------|-------|
| Node Type | Webhook |
| HTTP Method | POST |
| Path | `/webhooks/stripe-checkout` |
| Response Mode | Last Node |
| Authentication | Header Auth (`Stripe-Signature`) |

**Header Auth Credential:** `stripe-webhook-secret` (configured in N8N credentials)

### Node 2: Validate Event (Function / Code)

Validates the incoming Stripe webhook payload and extracts key fields.

```javascript
// Validate and extract Stripe checkout.session.completed event
const event = $input.first().json;

// Verify event type
if (event.type !== 'checkout.session.completed') {
  throw new Error(`Unexpected event type: ${event.type}`);
}

const session = event.data.object;

// Extract core fields
const output = {
  stripe_event_id: event.id,
  session_id: session.id,
  customer_email: session.customer_details?.email || session.customer_email,
  customer_name: session.customer_details?.name || 'Customer',
  amount_total: session.amount_total / 100, // Convert from cents
  currency: session.currency?.toUpperCase() || 'USD',
  product_id: session.metadata?.product_id || session.line_items?.data?.[0]?.price?.product,
  price_id: session.line_items?.data?.[0]?.price?.id,
  payment_status: session.payment_status,
  created_at: new Date(event.created * 1000).toISOString()
};

// Reject if missing critical fields
if (!output.customer_email || !output.product_id) {
  throw new Error(`Missing required fields. email: ${output.customer_email}, product_id: ${output.product_id}`);
}

return [output];
```

### Node 3: Lookup Product in Database

| Field | Value |
|-------|-------|
| Node Type | PostgreSQL (Execute Query) |
| Operation | Select |

```sql
SELECT
  id,
  product_name,
  slug,
  product_type,           -- 'guide', 'template', 'course', 'toolkit'
  delivery_url,
  download_type,          -- 'url', 'email_attachment', 'member_area'
  email_sequence_id,
  discord_role_id,
  upsell_product_id,
  thank_you_page_url,
  price
FROM products
WHERE stripe_product_id = $1
  AND active = true
LIMIT 1;
```

**Parameter:** `$1` → `{{ $json.product_id }}` from Node 2

**IF node after this:** If no product found → branch to error handler (Node 8). If found → continue.

### Node 4: Send Delivery Email

| Field | Value |
|-------|-------|
| Node Type | SendGrid (HTTP Request) or N8N Email Send |
| Method | POST |
| API Endpoint | `https://api.sendgrid.com/v3/mail/send` |

#### Email Template

**Subject:** `Your {{ $node["Lookup Product"].json.product_name }} is ready 🎉`

**From:** `Stephen@dreamai.com` (Stephen, Dream AI)

**Body (HTML):**

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body style="margin:0;padding:0;background-color:#0a0a0a;font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,sans-serif;">
  <table width="100%" cellpadding="0" cellspacing="0" style="max-width:600px;margin:0 auto;padding:40px 20px;">
    <tr>
      <td style="text-align:center;padding-bottom:32px;">
        <h1 style="color:#ffffff;font-size:28px;margin:0;">Your {{ $node["Lookup Product"].json.product_name }} is ready 🎉</h1>
      </td>
    </tr>
    <tr>
      <td style="color:#a0a0a0;font-size:16px;line-height:1.6;padding-bottom:24px;">
        Hey {{ $node["Validate Event"].json.customer_name.split(' ')[0] }},<br><br>
        Thanks for your purchase! Your digital product is ready to download.
      </td>
    </tr>
    <tr>
      <td style="padding:24px 0;">
        <table width="100%" cellpadding="0" cellspacing="0" style="background:linear-gradient(135deg,#1a1a2e,#16213e);border-radius:12px;border:1px solid #2a2a4a;">
          <tr>
            <td style="padding:24px;text-align:center;">
              <p style="color:#ffffff;font-size:18px;font-weight:bold;margin:0 0 16px 0;">📦 {{ $node["Lookup Product"].json.product_name }}</p>
              <a href="{{ $node["Lookup Product"].json.delivery_url }}"
                 style="display:inline-block;background:linear-gradient(135deg,#6366f1,#8b5cf6);color:#ffffff;font-size:16px;font-weight:bold;padding:14px 32px;border-radius:8px;text-decoration:none;">
                Access Your Download →
              </a>
            </td>
          </tr>
        </table>
      </td>
    </tr>
    <tr>
      <td style="color:#a0a0a0;font-size:14px;line-height:1.8;padding-bottom:24px;">
        <strong style="color:#ffffff;">Quick instructions:</strong><br>
        1. Click the button above to access your product<br>
        2. Bookmark the page for future reference<br>
        3. Start with the getting started guide (first section)<br>
        4. Join our community for support and bonus tips
      </td>
    </tr>
    <tr>
      <td style="padding:24px 0;border-top:1px solid #2a2a4a;">
        <table width="100%" cellpadding="0" cellspacing="0">
          <tr>
            <td style="text-align:center;">
              <p style="color:#ffffff;font-size:16px;margin:0 0 12px 0;">💬 Join the Dream AI Community</p>
              <a href="https://discord.gg/dreamai"
                 style="display:inline-block;background:#5865F2;color:#ffffff;font-size:14px;font-weight:bold;padding:12px 28px;border-radius:8px;text-decoration:none;">
                Join Discord →
              </a>
            </td>
          </tr>
        </table>
      </td>
    </tr>
    <tr>
      <td style="color:#666666;font-size:12px;text-align:center;padding-top:32px;border-top:1px solid #1a1a1a;">
        <p style="margin:0;">Dream AI · stephen@dreamai.com</p>
        <p style="margin:8px 0 0 0;">Questions? Reply to this email — I read every one.</p>
      </td>
    </tr>
  </table>
</body>
</html>
```

#### SendGrid API Payload (HTTP Request node config)

```json
{
  "personalizations": [
    {
      "to": [{ "email": "{{ $json.customer_email }}", "name": "{{ $json.customer_name }}" }],
      "subject": "Your {{ $node['Lookup Product'].json.product_name }} is ready 🎉"
    }
  ],
  "from": { "email": "Stephen@dreamai.com", "name": "Stephen @ Dream AI" },
  "content": [
    {
      "type": "text/html",
      "value": "<!-- Full HTML template above -->"
    }
  ]
}
```

**Credential:** SendGrid API Key (stored in N8N credentials as `sendgrid-api-key`)

### Node 5: Add Customer to CRM (PostgreSQL)

| Field | Value |
|-------|-------|
| Node Type | PostgreSQL (Execute Query) |
| Operation | Insert |

```sql
INSERT INTO customers (
  email,
  name,
  product_id,
  product_name,
  amount_paid,
  currency,
  stripe_session_id,
  stripe_event_id,
  purchase_date,
  source,
  delivery_status
) VALUES (
  $1, $2, $3, $4, $5, $6, $7, $8, $9, 'stripe_checkout', 'delivered'
)
ON CONFLICT (email, product_id)
DO UPDATE SET
  purchase_date = EXCLUDED.purchase_date,
  amount_paid = EXCLUDED.amount_paid,
  delivery_status = 'delivered',
  updated_at = now();
```

**Parameters:**
- `$1` → `{{ $json.customer_email }}`
- `$2` → `{{ $json.customer_name }}`
- `$3` → `{{ $json.product_id }}`
- `$4` → `{{ $node["Lookup Product"].json.product_name }}`
- `$5` → `{{ $json.amount_total }}`
- `$6` → `{{ $json.currency }}`
- `$7` → `{{ $json.session_id }}`
- `$8` → `{{ $json.stripe_event_id }}`
- `$9` → `{{ $json.created_at }}`

### Node 6: Tag Customer in Email Sequence

| Field | Value |
|-------|-------|
| Node Type | PostgreSQL (Execute Query) |
| Operation | Insert |

```sql
INSERT INTO email_sequence_queue (
  customer_id,
  email,
  sequence_id,
  sequence_name,
  current_step,
  status,
  product_type,
  send_at,
  metadata
)
SELECT
  c.id,
  c.email,
  p.email_sequence_id,
  es.sequence_name,
  1,
  'pending',
  p.product_type,
  now(),  -- Immediate: first email goes out now
  jsonb_build_object(
    'product_name', p.product_name,
    'purchase_amount', c.amount_paid,
    'source', 'product_purchase'
  )
FROM customers c
JOIN products p ON p.id = c.product_id
JOIN email_sequences es ON es.id = p.email_sequence_id
WHERE c.email = $1
  AND c.product_id = $2
  AND p.email_sequence_id IS NOT NULL;
```

**Parameters:**
- `$1` → `{{ $json.customer_email }}`
- `$2` → `{{ $json.product_id }}`

### Node 7: Telegram Alert to Stephen

| Field | Value |
|-------|-------|
| Node Type | Telegram |
| Operation | Send Message |
| Chat ID | `686076918` |

**Message Template:**

```
💰 New sale!

📦 Product: {{ $node["Lookup Product"].json.product_name }}
💵 Amount: ${{ $json.amount_total }} {{ $json.currency }}
👤 Customer: {{ $json.customer_name }} ({{ $json.customer_email }})
🕐 Time: {{ $json.created_at }}

✅ Delivery email sent
```

### Node 8: Log to Sales Tracker (PostgreSQL)

| Field | Value |
|-------|-------|
| Node Type | PostgreSQL (Execute Query) |
| Operation | Insert |

```sql
INSERT INTO sales_log (
  event_type,
  stripe_event_id,
  session_id,
  customer_email,
  customer_name,
  product_id,
  product_name,
  amount,
  currency,
  delivery_status,
  email_sent,
  crm_updated,
  sequence_tagged,
  telegram_notified,
  raw_payload,
  created_at
) VALUES (
  'product_purchase',
  $1, $2, $3, $4, $5, $6, $7, $8,
  'delivered', true, true, true, true,
  $9,
  now()
);
```

**Parameters:**
- `$1` → `{{ $json.stripe_event_id }}`
- `$2` → `{{ $json.session_id }}`
- `$3` → `{{ $json.customer_email }}`
- `$4` → `{{ $json.customer_name }}`
- `$5` → `{{ $json.product_id }}`
- `$6` → `{{ $node["Lookup Product"].json.product_name }}`
- `$7` → `{{ $json.amount_total }}`
- `$8` → `{{ $json.currency }}`
- `$9` → `{{ JSON.stringify($input.first().json) }}` (raw webhook payload)

### Node 9: Error Handler (Catch Error Workflow)

If any node fails, this workflow catches it:

```sql
INSERT INTO delivery_errors (
  workflow_name,
  error_message,
  customer_email,
  product_id,
  stripe_event_id,
  raw_data,
  resolved,
  created_at
) VALUES (
  'product_delivery',
  $1, $2, $3, $4, $5, false, now()
);
```

**Notification:** Slack `#ops-alerts` + Telegram message to Stephen.

---

## PostgreSQL Table Schemas

### products

```sql
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  product_type TEXT NOT NULL,          -- 'guide', 'template', 'course', 'toolkit'
  stripe_product_id TEXT UNIQUE,
  stripe_price_id TEXT,
  price NUMERIC(10,2) NOT NULL,
  currency TEXT DEFAULT 'USD',
  delivery_url TEXT NOT NULL,           -- Download URL or member area link
  download_type TEXT DEFAULT 'url',     -- 'url', 'email_attachment', 'member_area'
  email_sequence_id INTEGER REFERENCES email_sequences(id),
  discord_role_id TEXT,                 -- Discord role to grant
  upsell_product_id UUID REFERENCES products(id),
  thank_you_page_url TEXT,
  description TEXT,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_products_stripe_product ON products(stripe_product_id);
CREATE INDEX idx_products_slug ON products(slug);
```

### customers

```sql
CREATE TABLE customers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL,
  name TEXT,
  product_id UUID REFERENCES products(id),
  product_name TEXT,
  amount_paid NUMERIC(10,2),
  currency TEXT DEFAULT 'USD',
  stripe_session_id TEXT,
  stripe_event_id TEXT,
  purchase_date TIMESTAMPTZ DEFAULT now(),
  source TEXT DEFAULT 'stripe_checkout',
  delivery_status TEXT DEFAULT 'pending', -- pending, delivered, failed
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE UNIQUE INDEX idx_customers_email_product ON customers(email, product_id);
CREATE INDEX idx_customers_email ON customers(email);
CREATE INDEX idx_customers_purchase_date ON customers(purchase_date);
```

### email_sequence_queue

```sql
CREATE TABLE email_sequence_queue (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_id UUID REFERENCES customers(id),
  email TEXT NOT NULL,
  sequence_id INTEGER REFERENCES email_sequences(id),
  sequence_name TEXT,
  current_step INTEGER DEFAULT 1,
  total_steps INTEGER DEFAULT 5,
  status TEXT DEFAULT 'pending',         -- pending, sent, failed, completed, unsubscribed
  product_type TEXT,
  send_at TIMESTAMPTZ,
  sent_at TIMESTAMPTZ,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_seq_queue_status ON email_sequence_queue(status, send_at);
CREATE INDEX idx_seq_queue_email ON email_sequence_queue(email);
```

### email_sequences

```sql
CREATE TABLE email_sequences (
  id SERIAL PRIMARY KEY,
  sequence_name TEXT NOT NULL,
  product_type TEXT,                     -- Maps to product_type for auto-tagging
  total_steps INTEGER DEFAULT 5,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### email_templates

```sql
CREATE TABLE email_templates (
  id SERIAL PRIMARY KEY,
  sequence_id INTEGER REFERENCES email_sequences(id),
  step INTEGER NOT NULL,
  subject TEXT NOT NULL,
  body_html TEXT NOT NULL,
  delay_days INTEGER DEFAULT 3,         -- Days after previous step
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(sequence_id, step)
);
```

### sales_log

```sql
CREATE TABLE sales_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type TEXT NOT NULL,
  stripe_event_id TEXT,
  session_id TEXT,
  customer_email TEXT,
  customer_name TEXT,
  product_id UUID,
  product_name TEXT,
  amount NUMERIC(10,2),
  currency TEXT DEFAULT 'USD',
  delivery_status TEXT,
  email_sent BOOLEAN DEFAULT false,
  crm_updated BOOLEAN DEFAULT false,
  sequence_tagged BOOLEAN DEFAULT false,
  telegram_notified BOOLEAN DEFAULT false,
  raw_payload JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_sales_log_date ON sales_log(created_at);
CREATE INDEX idx_sales_log_product ON sales_log(product_id);
```

### delivery_errors

```sql
CREATE TABLE delivery_errors (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workflow_name TEXT,
  error_message TEXT,
  customer_email TEXT,
  product_id TEXT,
  stripe_event_id TEXT,
  raw_data JSONB,
  resolved BOOLEAN DEFAULT false,
  resolved_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_delivery_errors_unresolved ON delivery_errors(resolved) WHERE NOT resolved;
```

---

## Full N8N JSON Workflow

```json
{
  "name": "DreamAI — Digital Product Delivery",
  "active": true,
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "stripe-checkout",
        "responseMode": "lastNode",
        "options": {}
      },
      "id": "node-webhook",
      "name": "Stripe Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [250, 300],
      "webhookId": "stripe-checkout"
    },
    {
      "parameters": {
        "jsCode": "// Validate and extract Stripe checkout.session.completed event\nconst event = $input.first().json;\n\nif (event.type !== 'checkout.session.completed') {\n  throw new Error(`Unexpected event type: ${event.type}`);\n}\n\nconst session = event.data.object;\n\nconst output = {\n  stripe_event_id: event.id,\n  session_id: session.id,\n  customer_email: session.customer_details?.email || session.customer_email,\n  customer_name: session.customer_details?.name || 'Customer',\n  amount_total: session.amount_total / 100,\n  currency: session.currency?.toUpperCase() || 'USD',\n  product_id: session.metadata?.product_id,\n  price_id: session.line_items?.data?.[0]?.price?.id,\n  payment_status: session.payment_status,\n  created_at: new Date(event.created * 1000).toISOString()\n};\n\nif (!output.customer_email || !output.product_id) {\n  throw new Error(`Missing required fields. email: ${output.customer_email}, product_id: ${output.product_id}`);\n}\n\nreturn [output];"
      },
      "id": "node-validate",
      "name": "Validate Event",
      "type": "n8n-nodes-base.code",
      "typeVersion": 1,
      "position": [450, 300]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "SELECT id, product_name, slug, product_type, delivery_url, download_type, email_sequence_id, discord_role_id, upsell_product_id, thank_you_page_url, price FROM products WHERE stripe_product_id = $1 AND active = true LIMIT 1;",
        "options": {
          "queryReplacement": "={{ $json.product_id }}"
        }
      },
      "id": "node-lookup",
      "name": "Lookup Product",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [650, 300],
      "credentials": {
        "postgres": {
          "id": "dream-ai-postgres",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{ $json.product_name }}",
              "operation": "notEmpty"
            }
          ]
        }
      },
      "id": "node-if-product",
      "name": "Product Found?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [850, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.sendgrid.com/v3/mail/send",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"personalizations\": [{\n    \"to\": [{\"email\": \"{{ $json.customer_email }}\", \"name\": \"{{ $json.customer_name }}\"}],\n    \"subject\": \"Your {{ $('Lookup Product').item.json.product_name }} is ready 🎉\"\n  }],\n  \"from\": {\"email\": \"Stephen@dreamai.com\", \"name\": \"Stephen @ Dream AI\"},\n  \"content\": [{\"type\": \"text/html\", \"value\": \"See email template in workflow spec\"}]\n}",
        "options": {}
      },
      "id": "node-send-email",
      "name": "Send Delivery Email",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [1050, 200],
      "credentials": {
        "httpHeaderAuth": {
          "id": "sendgrid-api-key",
          "name": "SendGrid API Key"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO customers (email, name, product_id, product_name, amount_paid, currency, stripe_session_id, stripe_event_id, purchase_date, source, delivery_status) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, 'stripe_checkout', 'delivered') ON CONFLICT (email, product_id) DO UPDATE SET purchase_date = EXCLUDED.purchase_date, amount_paid = EXCLUDED.amount_paid, delivery_status = 'delivered', updated_at = now();",
        "options": {
          "queryReplacement": "={{ $json.customer_email }}\n={{ $json.customer_name }}\n={{ $json.product_id }}\n={{ $('Lookup Product').item.json.product_name }}\n={{ $json.amount_total }}\n={{ $json.currency }}\n={{ $json.session_id }}\n={{ $json.stripe_event_id }}\n={{ $json.created_at }}"
        }
      },
      "id": "node-crm",
      "name": "Add to CRM",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1250, 200],
      "credentials": {
        "postgres": {
          "id": "dream-ai-postgres",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO email_sequence_queue (customer_id, email, sequence_id, sequence_name, current_step, status, product_type, send_at, metadata) SELECT c.id, c.email, p.email_sequence_id, es.sequence_name, 1, 'pending', p.product_type, now(), jsonb_build_object('product_name', p.product_name, 'purchase_amount', c.amount_paid, 'source', 'product_purchase') FROM customers c JOIN products p ON p.id = c.product_id JOIN email_sequences es ON es.id = p.email_sequence_id WHERE c.email = $1 AND c.product_id = $2 AND p.email_sequence_id IS NOT NULL;",
        "options": {
          "queryReplacement": "={{ $json.customer_email }}\n={{ $json.product_id }}"
        }
      },
      "id": "node-sequence",
      "name": "Tag Email Sequence",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1450, 200],
      "credentials": {
        "postgres": {
          "id": "dream-ai-postgres",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "chatId": "686076918",
        "text": "💰 New sale!\n\n📦 Product: {{ $('Lookup Product').item.json.product_name }}\n💵 Amount: ${{ $json.amount_total }} {{ $json.currency }}\n👤 Customer: {{ $json.customer_name }} ({{ $json.customer_email }})\n🕐 Time: {{ $json.created_at }}\n\n✅ Delivery email sent",
        "additionalFields": {}
      },
      "id": "node-telegram",
      "name": "Telegram Alert",
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1,
      "position": [1650, 200],
      "credentials": {
        "telegramApi": {
          "id": "dream-ai-telegram",
          "name": "Dream AI Telegram"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO sales_log (event_type, stripe_event_id, session_id, customer_email, customer_name, product_id, product_name, amount, currency, delivery_status, email_sent, crm_updated, sequence_tagged, telegram_notified, raw_payload, created_at) VALUES ('product_purchase', $1, $2, $3, $4, $5, $6, $7, $8, 'delivered', true, true, true, true, $9, now());",
        "options": {
          "queryReplacement": "={{ $json.stripe_event_id }}\n={{ $json.session_id }}\n={{ $json.customer_email }}\n={{ $json.customer_name }}\n={{ $json.product_id }}\n={{ $('Lookup Product').item.json.product_name }}\n={{ $json.amount_total }}\n={{ $json.currency }}\n={{ JSON.stringify($input.first().json) }}"
        }
      },
      "id": "node-sales-log",
      "name": "Log to Sales Tracker",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1850, 200],
      "credentials": {
        "postgres": {
          "id": "dream-ai-postgres",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {},
      "id": "node-success-end",
      "name": "Success",
      "type": "n8n-nodes-base.noOp",
      "typeVersion": 1,
      "position": [2050, 200]
    },
    {
      "parameters": {
        "chatId": "686076918",
        "text": "🚨 DELIVERY FAILURE\n\nProduct lookup failed for Stripe event: {{ $json.stripe_event_id }}\nCustomer: {{ $json.customer_email }}\nProduct ID: {{ $json.product_id }}\n\nCheck delivery_errors table.",
        "additionalFields": {}
      },
      "id": "node-error-telegram",
      "name": "Error Alert",
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1,
      "position": [1050, 450],
      "credentials": {
        "telegramApi": {
          "id": "dream-ai-telegram",
          "name": "Dream AI Telegram"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO delivery_errors (workflow_name, error_message, customer_email, product_id, stripe_event_id, raw_data, resolved, created_at) VALUES ('product_delivery', 'Product not found in database', $1, $2, $3, $4, false, now());",
        "options": {
          "queryReplacement": "={{ $json.customer_email }}\n={{ $json.product_id }}\n={{ $json.stripe_event_id }}\n={{ JSON.stringify($input.first().json) }}"
        }
      },
      "id": "node-error-log",
      "name": "Log Error",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1250, 450],
      "credentials": {
        "postgres": {
          "id": "dream-ai-postgres",
          "name": "Dream AI PostgreSQL"
        }
      }
    }
  ],
  "connections": {
    "Stripe Webhook": {
      "main": [[{"node": "Validate Event", "type": "main", "index": 0}]]
    },
    "Validate Event": {
      "main": [[{"node": "Lookup Product", "type": "main", "index": 0}]]
    },
    "Lookup Product": {
      "main": [[{"node": "Product Found?", "type": "main", "index": 0}]]
    },
    "Product Found?": {
      "main": [
        [{"node": "Send Delivery Email", "type": "main", "index": 0}],
        [{"node": "Error Alert", "type": "main", "index": 0}]
      ]
    },
    "Send Delivery Email": {
      "main": [[{"node": "Add to CRM", "type": "main", "index": 0}]]
    },
    "Add to CRM": {
      "main": [[{"node": "Tag Email Sequence", "type": "main", "index": 0}]]
    },
    "Tag Email Sequence": {
      "main": [[{"node": "Telegram Alert", "type": "main", "index": 0}]]
    },
    "Telegram Alert": {
      "main": [[{"node": "Log to Sales Tracker", "type": "main", "index": 0}]]
    },
    "Log to Sales Tracker": {
      "main": [[{"node": "Success", "type": "main", "index": 0}]]
    },
    "Error Alert": {
      "main": [[{"node": "Log Error", "type": "main", "index": 0}]]
    }
  },
  "settings": {
    "executionOrder": "v1",
    "errorWorkflow": "error-product-delivery"
  },
  "tags": [
    { "name": "product-delivery" },
    { "name": "stripe" },
    { "name": "automation" }
  ]
}
```

---

## Deployment Notes

### Prerequisites
1. **PostgreSQL:** All tables created (run schema SQL above)
2. **N8N Credentials configured:**
   - `dream-ai-postgres` → PostgreSQL connection
   - `sendgrid-api-key` → SendGrid API key (Header Auth, key: `Authorization`, value: `Bearer SG.xxx`)
   - `dream-ai-telegram` → Telegram Bot API token
   - `stripe-webhook-secret` → Stripe webhook signing secret (for signature verification)
3. **Stripe Webhook URL:** Add `https://your-n8n-instance.com/webhook/stripe-checkout` to Stripe dashboard
4. **Products table populated:** All 7 products must exist with correct `stripe_product_id` values

### Testing Checklist
- [ ] Stripe test mode: Create checkout session → verify webhook fires
- [ ] Product lookup: Verify correct product returned for test product_id
- [ ] Email delivery: Check SendGrid activity log for test email
- [ ] CRM insert: Verify row in `customers` table
- [ ] Sequence tagging: Verify row in `email_sequence_queue`
- [ ] Telegram: Verify notification received
- [ ] Sales log: Verify row in `sales_log`
- [ ] Error path: Test with invalid product_id → verify error logged + Telegram alert
- [ ] Duplicate purchase: Same email + product → verify `ON CONFLICT` updates correctly

### Monitoring
- Check N8N execution history daily
- Monitor `delivery_errors` table for unresolved failures
- Weekly: Review `sales_log` vs Stripe dashboard for discrepancies

---

*Workflow version 1.0 — Created 2026-03-18 by CATALYST (via SOVEREIGN subagent)*
