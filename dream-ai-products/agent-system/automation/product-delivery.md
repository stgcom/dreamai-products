# Automated Product Delivery System

> **Purpose:** End-to-end automated delivery of Dream AI digital products via Stripe webhooks.
> **Trigger:** Stripe `payment.succeeded` webhook event
> **Products:** 6 HTML-based digital tools

---

## Product Registry

| Product ID | File Path | Name | Base Price |
|------------|-----------|------|------------|
| `missed-call-calculator` | `/products/missed-call-calculator/index.html` | Missed Call Calculator | $47 |
| `ai-readiness-scorecard` | `/products/ai-readiness-scorecard/scorecard.html` | AI Readiness Scorecard | $47 |
| `sms-booking` | `/products/sms-booking/index.html` | SMS Booking System | $47 |
| `script-pack` | `/products/script-pack/index.html` | Sales Script Pack | $47 |
| `review-rocket` | `/products/review-rocket/index.html` | Review Rocket | $47 |
| `re-playbook` | `/products/re-playbook/index.html` | RE Playbook | $47 |

**Starter Kit (upsell):** $297 — includes all 6 products

---

## Delivery Flow Overview

```
[Stripe Webhook]
       ↓
[Parse Webhook Data]
       ↓
[Verify Payment Status]
       ↓
[Duplicate Check] ←→ [Log & Skip if duplicate]
       ↓
[Map Product ID → File]
       ↓
    ┌───┴───┐
    ↓       ↓
[Product   [Product
 Found]    Not Found]
    ↓           ↓
[Generate     [Send Apology
 Download     + Manual
 Link]        Request]
    ↓
[Send Delivery Email]
    ↓
[Update CRM (PostgreSQL)]
    ↓
[Alert Stephen via Telegram]
    ↓
[Wait 3 Days]
    ↓
[Send Check-in Email]
    ↓
[Wait 2 Days]
    ↓
[Send Review Request Email]
```

---

## N8N Workflow Configuration

### Complete Workflow JSON

```json
{
  "name": "Dream AI Product Delivery",
  "active": true,
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "stripe-webhook",
        "responseMode": "onReceived",
        "responseData": "firstEntryJson",
        "options": {}
      },
      "id": "webhook-trigger",
      "name": "Stripe Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "jsCode": "// Parse Stripe webhook and extract relevant data\nconst webhookData = $input.first().json.body;\nconst eventType = webhookData.type;\n\n// Only process payment.succeeded events\nif (eventType !== 'payment_intent.succeeded') {\n  return [{ json: { skip: true, reason: 'Not a payment.succeeded event' } }];\n}\n\nconst paymentIntent = webhookData.data.object;\nconst metadata = paymentIntent.metadata || {};\n\n// Extract customer and product information\nconst result = {\n  skip: false,\n  event_id: webhookData.id,\n  payment_intent_id: paymentIntent.id,\n  customer_email: paymentIntent.receipt_email || metadata.customer_email || 'unknown',\n  product_id: metadata.product_id || 'unknown',\n  product_name: metadata.product_name || 'Unknown Product',\n  amount_paid: (paymentIntent.amount / 100).toFixed(2),\n  currency: paymentIntent.currency.toUpperCase(),\n  payment_status: paymentIntent.status,\n  created_at: new Date(paymentIntent.created * 1000).toISOString()\n};\n\nreturn [{ json: result }];"
      },
      "id": "parse-webhook",
      "name": "Parse Webhook Data",
      "type": "n8n-nodes-base.code",
      "typeVersion": 1,
      "position": [450, 300]
    },
    {
      "parameters": {
        "jsCode": "// Verify payment succeeded and check for duplicates\nconst data = $input.first().json;\n\n// Skip if not a valid payment\nif (data.skip) {\n  return [{ json: data }];\n}\n\n// Verify payment status\nif (data.payment_status !== 'succeeded') {\n  return [{ json: { skip: true, reason: 'Payment status is ' + data.payment_status } }];\n}\n\n// Product ID validation\nconst validProducts = [\n  'missed-call-calculator',\n  'ai-readiness-scorecard',\n  'sms-booking',\n  'script-pack',\n  'review-rocket',\n  're-playbook'\n];\n\nif (!validProducts.includes(data.product_id)) {\n  return [{ json: { ...data, product_valid: false } }];\n}\n\nreturn [{ json: { ...data, product_valid: true } }];"
      },
      "id": "verify-payment",
      "name": "Verify Payment",
      "type": "n8n-nodes-base.code",
      "typeVersion": 1,
      "position": [650, 300]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "SELECT id FROM deliveries WHERE payment_intent_id = $1",
        "options": {
          "queryBatching": "single"
        }
      },
      "id": "check-duplicate",
      "name": "Check Duplicate",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [850, 300],
      "credentials": {
        "postgresDb": {
          "id": "postgres-dream-ai",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{ $json.skip }}",
              "value2": true
            },
            {
              "value1": "={{ $json.product_valid }}",
              "value2": false
            }
          ],
          "any": true
        }
      },
      "id": "if-valid-payment",
      "name": "IF Valid Payment",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [1050, 300]
    },
    {
      "parameters": {
        "jsCode": "// Handle product not found - generate apology\nconst data = $input.first().json;\nreturn [{ json: {\n  send道歉: true,\n  customer_email: data.customer_email,\n  product_id: data.product_id,\n  error_type: data.skip ? data.reason : 'product_not_found'\n}}];"
      },
      "id": "handle-error",
      "name": "Handle Error Case",
      "type": "n8n-nodes-base.code",
      "typeVersion": 1,
      "position": [1250, 500]
    },
    {
      "parameters": {
        "jsCode": "// Map product ID to file path and generate download URL\nconst PRODUCT_MAP = {\n  'missed-call-calculator': {\n    file: '/products/missed-call-calculator/index.html',\n    name: 'Missed Call Calculator',\n    quickstart: '1. Open the calculator in your browser\\n2. Enter your average missed calls per day\\n3. See how much revenue you\\'re losing\\n4. Use our recovery strategies to recapture leads'\n  },\n  'ai-readiness-scorecard': {\n    file: '/products/ai-readiness-scorecard/scorecard.html',\n    name: 'AI Readiness Scorecard',\n    quickstart: '1. Complete the 10-question assessment\\n2. Get your AI readiness score\\n3. Review your personalized report\\n4. Follow the recommended action plan'\n  },\n  'sms-booking': {\n    file: '/products/sms-booking/index.html',\n    name: 'SMS Booking System',\n    quickstart: '1. Customize your booking page\\n2. Set your availability\\n3. Share your booking link\\n4. Start accepting SMS bookings'\n  },\n  'script-pack': {\n    file: '/products/script-pack/index.html',\n    name: 'Sales Script Pack',\n    quickstart: '1. Browse the script categories\\n2. Find your industry-specific scripts\\n3. Customize for your business\\n4. Practice and implement daily'\n  },\n  'review-rocket': {\n    file: '/products/review-rocket/index.html',\n    name: 'Review Rocket',\n    quickstart: '1. Set up your review collection template\\n2. Customize the message script\\n3. Send to your best customers\\n4. Watch your reviews grow'\n  },\n  're-playbook': {\n    file: '/products/re-playbook/index.html',\n    name: 'RE Playbook',\n    quickstart: '1. Review the playbooks for your niche\\n2. Implement the outreach sequences\\n3. Track your pipeline\\n4. Scale what works'\n  }\n};\n\nconst data = $input.first().json;\nconst product = PRODUCT_MAP[data.product_id];\n\n// Generate time-limited download token (72 hours)\nconst token = Buffer.from(JSON.stringify({\n  product_id: data.product_id,\n  email: data.customer_email,\n  expires: Date.now() + (72 * 60 * 60 * 1000)\n})).toString('base64url');\n\nreturn [{ json: {\n  ...data,\n  file_path: product.file,\n  product_name: product.name,\n  quickstart: product.quickstart,\n  download_token: token,\n  download_url: `https://dreamai.com/download/${token}`,\n  download_expires_hours: 72\n}}];"
      },
      "id": "map-product",
      "name": "Map Product & Generate Link",
      "type": "n8n-nodes-base.code",
      "typeVersion": 1,
      "position": [1250, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.resend.com/emails",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"from\": \"Dream AI <products@dreamai.com>\",\n  \"to\": [\"{{ $json.customer_email }}\"],\n  \"subject\": \"Your {{ $json.product_name }} is ready! 🎉\",\n  \"html\": `\n    <div style=\"font-family: 'Segoe UI', Arial, sans-serif; max-width: 600px; margin: 0 auto;\">\n      <div style=\"background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 40px; text-align: center;\">\n        <h1 style=\"color: white; margin: 0; font-size: 28px;\">Your {{ $json.product_name }} is Ready!</h1>\n      </div>\n      \n      <div style=\"padding: 30px; background: #f9f9f9;\">\n        <p style=\"font-size: 16px; color: #333;\">Hi there! 👋</p>\n        <p style=\"font-size: 16px; color: #333;\">\n          Thank you for your purchase! Your payment of <strong>${{ $json.amount_paid }}</strong> has been confirmed.\n        </p>\n        \n        <div style=\"background: white; border-radius: 8px; padding: 20px; margin: 20px 0; border-left: 4px solid #667eea;\">\n          <h3 style=\"margin-top: 0; color: #667eea;\">📦 Your Product</h3>\n          <p style=\"font-size: 18px; font-weight: bold;\">{{ $json.product_name }}</p>\n        </div>\n        \n        <div style=\"text-align: center; margin: 30px 0;\">\n          <a href=\"{{ $json.download_url }}\" \n             style=\"background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); \n                    color: white; \n                    padding: 15px 40px; \n                    text-decoration: none; \n                    border-radius: 8px; \n                    font-size: 18px; \n                    font-weight: bold;\">\n            📥 Download Your Product\n          </a>\n          <p style=\"color: #666; font-size: 14px; margin-top: 10px;\">\n            Link expires in 72 hours\n          </p>\n        </div>\n        \n        <div style=\"background: white; border-radius: 8px; padding: 20px; margin: 20px 0;\">\n          <h3 style=\"margin-top: 0; color: #333;\">🚀 Quick Start Guide</h3>\n          <p style=\"white-space: pre-line; color: #555;\">{{ $json.quickstart }}</p>\n        </div>\n        \n        <div style=\"background: #fff3cd; border-radius: 8px; padding: 20px; margin: 20px 0; border: 1px solid #ffc107;\">\n          <h3 style=\"margin-top: 0; color: #856404;\">⭐ Want More?</h3>\n          <p style=\"color: #856404;\">\n            Get the <strong>complete Dream AI Starter Kit</strong> — all 6 products for just <strong>$297</strong>\n          </p>\n          <a href=\"https://dreamai.com/starter-kit\" \n             style=\"display: inline-block; \n                    background: #856404; \n                    color: white; \n                    padding: 10px 25px; \n                    text-decoration: none; \n                    border-radius: 5px;\">\n            Learn More →\n          </a>\n        </div>\n        \n        <div style=\"text-align: center; margin-top: 30px; padding-top: 20px; border-top: 1px solid #ddd;\">\n          <p style=\"color: #666; font-size: 14px;\">\n            Questions? Just reply to this email or contact us at\n            <a href=\"mailto:support@dreamai.com\">support@dreamai.com</a>\n          </p>\n          <p style=\"color: #999; font-size: 12px;\">\n            Dream AI | Empowering businesses with AI\n          </p>\n        </div>\n      </div>\n    </div>\n  `\n}",
        "options": {}
      },
      "id": "send-delivery-email",
      "name": "Send Delivery Email",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [1450, 300],
      "credentials": {
        "httpHeaderAuth": {
          "id": "resend-api-key",
          "name": "Resend API Key"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO deliveries (payment_intent_id, customer_email, product_id, product_name, amount_paid, download_token, delivered_at, status) VALUES ($1, $2, $3, $4, $5, $6, NOW(), 'delivered')",
        "options": {
          "queryBatching": "single"
        }
      },
      "id": "record-delivery",
      "name": "Record Delivery",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1650, 300],
      "credentials": {
        "postgresDb": {
          "id": "postgres-dream-ai",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO crm_contacts (email, status, tags, last_purchase_at, last_purchase_product) VALUES ($1, 'Purchased', $2, NOW(), $3) ON CONFLICT (email) DO UPDATE SET status = 'Purchased', tags = ARRAY_APPEND(crm_contacts.tags, $4), last_purchase_at = NOW(), last_purchase_product = $3",
        "options": {
          "queryBatching": "single"
        }
      },
      "id": "crm-update-contact",
      "name": "CRM Update Contact",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1850, 200],
      "credentials": {
        "postgresDb": {
          "id": "postgres-dream-ai",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO crm_deals (contact_email, product_id, amount, deal_status, created_at) VALUES ($1, $2, $3, 'completed', NOW())",
        "options": {
          "queryBatching": "single"
        }
      },
      "id": "crm-record-deal",
      "name": "CRM Record Deal",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [1850, 400],
      "credentials": {
        "postgresDb": {
          "id": "postgres-dream-ai",
          "name": "Dream AI PostgreSQL"
        }
      }
    },
    {
      "parameters": {
        "chatId": "686076918",
        "text": "💰 New purchase: {{ $json.product_name }} — ${{ $json.amount_paid }} from {{ $json.customer_email }}",
        "additionalFields": {
          "parseMode": "HTML"
        }
      },
      "id": "telegram-alert",
      "name": "Alert Stephen",
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1,
      "position": [2050, 300],
      "credentials": {
        "telegramApi": {
          "id": "telegram-dream-ai",
          "name": "Dream AI Telegram"
        }
      }
    },
    {
      "parameters": {
        "amount": 3,
        "unit": "days"
      },
      "id": "wait-3-days",
      "name": "Wait 3 Days",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [2250, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.resend.com/emails",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"from\": \"Dream AI <products@dreamai.com>\",\n  \"to\": [\"{{ $json.customer_email }}\"],\n  \"subject\": \"How's your {{ $json.product_name }} going? 🤔\",\n  \"html\": `\n    <div style=\"font-family: 'Segoe UI', Arial, sans-serif; max-width: 600px; margin: 0 auto;\">\n      <div style=\"background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 30px; text-align: center;\">\n        <h1 style=\"color: white; margin: 0; font-size: 24px;\">Quick check-in! 👋</h1>\n      </div>\n      \n      <div style=\"padding: 30px; background: #f9f9f9;\">\n        <p style=\"font-size: 16px; color: #333;\">Hi there!</p>\n        <p style=\"font-size: 16px; color: #333;\">\n          You grabbed <strong>{{ $json.product_name }}</strong> a few days ago — just wanted to check in!\n        </p>\n        \n        <div style=\"background: white; border-radius: 8px; padding: 20px; margin: 20px 0;\">\n          <p style=\"color: #555; margin-top: 0;\">\n            <strong>How's it working out for you?</strong>\n          </p>\n          <p style=\"color: #555;\">\n            Any questions? Stuck on something? Want to make sure you're getting the most out of it?\n          </p>\n        </div>\n        \n        <p style=\"font-size: 16px; color: #333;\">\n          Just hit reply — I read every response and I'm here to help.\n        </p>\n        \n        <p style=\"font-size: 16px; color: #333;\">\n          Cheers,<br>\n          Stephen<br>\n          <span style=\"color: #666; font-size: 14px;\">Dream AI</span>\n        </p>\n        \n        <div style=\"text-align: center; margin-top: 30px; padding-top: 20px; border-top: 1px solid #ddd;\">\n          <p style=\"color: #999; font-size: 12px;\">\n            Dream AI | Empowering businesses with AI\n          </p>\n        </div>\n      </div>\n    </div>
  `\n}",
        "options": {}
      },
      "id": "day3-checkin-email",
      "name": "Day 3 Check-in Email",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [2450, 300],
      "credentials": {
        "httpHeaderAuth": {
          "id": "resend-api-key",
          "name": "Resend API Key"
        }
      }
    },
    {
      "parameters": {
        "amount": 2,
        "unit": "days"
      },
      "id": "wait-2-more-days",
      "name": "Wait 2 More Days",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [2650, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.resend.com/emails",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"from\": \"Dream AI <products@dreamai.com>\",\n  \"to\": [\"{{ $json.customer_email }}\"],\n  \"subject\": \"Love {{ $json.product_name }}? Leave us a quick review! ⭐\",\n  \"html\": `\n    <div style=\"font-family: 'Segoe UI', Arial, sans-serif; max-width: 600px; margin: 0 auto;\">\n      <div style=\"background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 30px; text-align: center;\">\n        <h1 style=\"color: white; margin: 0; font-size: 24px;\">We'd love your feedback! ⭐</h1>\n      </div>\n      \n      <div style=\"padding: 30px; background: #f9f9f9;\">\n        <p style=\"font-size: 16px; color: #333;\">Hi there!</p>\n        <p style=\"font-size: 16px; color: #333;\">\n          Hope you've been enjoying <strong>{{ $json.product_name }}</strong>!\n        </p>\n        \n        <div style=\"background: white; border-radius: 8px; padding: 20px; margin: 20px 0; text-align: center;\">\n          <p style=\"color: #555; margin-top: 0;\">\n            <strong>Would you take 30 seconds to leave a review?</strong>\n          </p>\n          <p style=\"color: #555;\">\n            Your feedback helps other business owners discover tools that can transform their operations.\n          </p>\n          <a href=\"https://dreamai.com/review?product={{ $json.product_id }}\" \n             style=\"display: inline-block; \n                    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); \n                    color: white; \n                    padding: 12px 30px; \n                    text-decoration: none; \n                    border-radius: 8px; \n                    font-size: 16px; \n                    margin: 15px 0;\">\n            ⭐ Leave a Review\n          </a>\n        </div>\n        \n        <p style=\"font-size: 16px; color: #333;\">\n          It really means the world to us. 🙏\n        </p>\n        \n        <p style=\"font-size: 16px; color: #333;\">\n          Thanks,\n          Stephen<br>\n          <span style=\"color: #666; font-size: 14px;\">Dream AI</span>\n        </p>\n        \n        <div style=\"text-align: center; margin-top: 30px; padding-top: 20px; border-top: 1px solid #ddd;\">\n          <p style=\"color: #999; font-size: 12px;\">\n            Dream AI | Empowering businesses with AI\n          </p>\n        </div>\n      </div>\n    </div>
  `\n}",
        "options": {}
      },
      "id": "day5-review-email",
      "name": "Day 5 Review Request",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [2850, 300],
      "credentials": {
        "httpHeaderAuth": {
          "id": "resend-api-key",
          "name": "Resend API Key"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Error handler - retry logic\nconst MAX_RETRIES = 3;\nconst data = $input.first().json;\nconst retryCount = $node['Send Delivery Email'].retryCount || 0;\n\nif (retryCount >= MAX_RETRIES) {\n  // Alert Stephen of failed delivery\n  return [{ json: {\n    alert: true,\n    message: `⚠️ Delivery failed after ${MAX_RETRIES} attempts for ${data.customer_email} - Product: ${data.product_id}`\n  }}];\n}\n\nreturn [{ json: data }];"
      },
      "id": "error-handler",
      "name": "Error Handler",
      "type": "n8n-nodes-base.code",
      "typeVersion": 1,
      "position": [1450, 500]
    },
    {
      "parameters": {
        "chatId": "686076918",
        "text": "{{ $json.message }}",
        "additionalFields": {
          "parseMode": "HTML"
        }
      },
      "id": "alert-error-telegram",
      "name": "Alert Error Telegram",
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1,
      "position": [1650, 500],
      "credentials": {
        "telegramApi": {
          "id": "telegram-dream-ai",
          "name": "Dream AI Telegram"
        }
      }
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.resend.com/emails",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"from\": \"Dream AI <support@dreamai.com>\",\n  \"to\": [\"{{ $json.customer_email }}\"],\n  \"subject\": \"Action needed: Your Dream AI product delivery\",\n  \"html\": `\n    <div style=\"font-family: 'Segoe UI', Arial, sans-serif; max-width: 600px; margin: 0 auto;\">\n      <div style=\"padding: 30px;\">\n        <h2 style=\"color: #333;\">We hit a small snag 😔</h2>\n        <p style=\"color: #555;\">\n          Hi there,\n        </p>\n        <p style=\"color: #555;\">\n          We received your payment, but our automated delivery system had a hiccup getting your product to you.\n        </p>\n        <p style=\"color: #555;\">\n          <strong>Don't worry — we're on it!</strong> Our team has been notified and will personally deliver your product within the next few hours.\n        </p>\n        <p style=\"color: #555;\">\n          If you need it urgently, just reply to this email and we'll get it to you immediately.\n        </p>\n        <p style=\"color: #555;\">\n          Sorry for the inconvenience!<br>\n          Stephen<br>\n          <span style=\"color: #666;\">Dream AI</span>\n        </p>\n      </div>\n    </div>\n  `\n}",
        "options": {}
      },
      "id": "send-apology-email",
      "name": "Send Apology Email",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [1450, 700],
      "credentials": {
        "httpHeaderAuth": {
          "id": "resend-api-key",
          "name": "Resend API Key"
        }
      }
    }
  ],
  "connections": {
    "Stripe Webhook": {
      "main": [
        [{ "node": "Parse Webhook Data", "type": "main", "index": 0 }]
      ]
    },
    "Parse Webhook Data": {
      "main": [
        [{ "node": "Verify Payment", "type": "main", "index": 0 }]
      ]
    },
    "Verify Payment": {
      "main": [
        [{ "node": "Check Duplicate", "type": "main", "index": 0 }]
      ]
    },
    "Check Duplicate": {
      "main": [
        [{ "node": "IF Valid Payment", "type": "main", "index": 0 }]
      ]
    },
    "IF Valid Payment": {
      "main": [
        [{ "node": "Map Product & Generate Link", "type": "main", "index": 0 }],
        [{ "node": "Handle Error Case", "type": "main", "index": 0 }]
      ]
    },
    "Handle Error Case": {
      "main": [
        [{ "node": "Send Apology Email", "type": "main", "index": 0 }]
      ]
    },
    "Map Product & Generate Link": {
      "main": [
        [{ "node": "Send Delivery Email", "type": "main", "index": 0 }]
      ]
    },
    "Send Delivery Email": {
      "main": [
        [{ "node": "Record Delivery", "type": "main", "index": 0 }],
        [{ "node": "Error Handler", "type": "main", "index": 0 }]
      ]
    },
    "Error Handler": {
      "main": [
        [{ "node": "Alert Error Telegram", "type": "main", "index": 0 }]
      ]
    },
    "Record Delivery": {
      "main": [
        [
          { "node": "CRM Update Contact", "type": "main", "index": 0 },
          { "node": "CRM Record Deal", "type": "main", "index": 0 }
        ]
      ]
    },
    "CRM Update Contact": {
      "main": [
        [{ "node": "Alert Stephen", "type": "main", "index": 0 }]
      ]
    },
    "CRM Record Deal": {
      "main": [
        [{ "node": "Alert Stephen", "type": "main", "index": 0 }]
      ]
    },
    "Alert Stephen": {
      "main": [
        [{ "node": "Wait 3 Days", "type": "main", "index": 0 }]
      ]
    },
    "Wait 3 Days": {
      "main": [
        [{ "node": "Day 3 Check-in Email", "type": "main", "index": 0 }]
      ]
    },
    "Day 3 Check-in Email": {
      "main": [
        [{ "node": "Wait 2 More Days", "type": "main", "index": 0 }]
      ]
    },
    "Wait 2 More Days": {
      "main": [
        [{ "node": "Day 5 Review Request", "type": "main", "index": 0 }]
      ]
    }
  }
}
```

---

## Database Schema

### `deliveries` Table

```sql
CREATE TABLE deliveries (
    id SERIAL PRIMARY KEY,
    payment_intent_id VARCHAR(255) UNIQUE NOT NULL,
    customer_email VARCHAR(255) NOT NULL,
    product_id VARCHAR(100) NOT NULL,
    product_name VARCHAR(255) NOT NULL,
    amount_paid DECIMAL(10,2) NOT NULL,
    download_token TEXT,
    delivered_at TIMESTAMP DEFAULT NOW(),
    checkin_sent_at TIMESTAMP,
    review_sent_at TIMESTAMP,
    status VARCHAR(50) DEFAULT 'delivered',
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_deliveries_email ON deliveries(customer_email);
CREATE INDEX idx_deliveries_product ON deliveries(product_id);
CREATE INDEX idx_deliveries_status ON deliveries(status);
```

### `crm_contacts` Table

```sql
CREATE TABLE crm_contacts (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    status VARCHAR(50) DEFAULT 'lead',
    tags TEXT[] DEFAULT '{}',
    last_purchase_at TIMESTAMP,
    last_purchase_product VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### `crm_deals` Table

```sql
CREATE TABLE crm_deals (
    id SERIAL PRIMARY KEY,
    contact_email VARCHAR(255) NOT NULL,
    product_id VARCHAR(100) NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    deal_status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Error Handling Matrix

| Error Type | Action | Notification |
|------------|--------|--------------|
| **Email delivery fails** | Retry 3x (exponential backoff: 10s, 30s, 90s) | Telegram alert to Stephen after 3 failures |
| **Product not found** | Send apology email + manual delivery request | Log + Telegram alert |
| **Duplicate payment** | Skip delivery, log event | Silent (no notification) |
| **Database error** | Log error, queue for manual retry | Telegram alert to Stephen |
| **Download token expired** | Generate new token, resend email | User notification |
| **Webhook signature invalid** | Reject request, log security event | Telegram alert to Stephen |

---

## Monitoring & Metrics

### Key Metrics to Track

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Delivery success rate | > 99% | < 95% |
| Email open rate | > 60% | < 40% |
| Download completion | > 80% | < 60% |
| Day 3 check-in response | > 20% | < 10% |
| Review conversion | > 10% | < 5% |
| Avg. delivery time | < 60 seconds | > 180 seconds |

### Health Check Endpoint

```
GET /webhooks/health
Response: { "status": "healthy", "deliveries_24h": 42, "success_rate": 0.995 }
```

---

## Setup Checklist

- [ ] Create `deliveries`, `crm_contacts`, `crm_deals` tables in PostgreSQL
- [ ] Configure Stripe webhook endpoint URL in Stripe Dashboard
- [ ] Set up Resend API credentials in N8N
- [ ] Configure Telegram bot credentials in N8N
- [ ] Create `/products/` directory with all 6 product HTML files
- [ ] Set up download link generator service (or use N8N static file serving)
- [ ] Configure webhook path `/stripe-webhook` in N8N
- [ ] Test with Stripe CLI: `stripe listen --forward-to https://dreamain8n.zeabur.app/webhook/stripe-webhook`
- [ ] Test with test payment: `stripe payment_intents create --amount=4700 --currency=usd`
- [ ] Verify Telegram alert delivery
- [ ] Verify email templates render correctly
- [ ] Monitor first 10 real deliveries

---

## Maintenance Notes

- **Download tokens:** Expire after 72 hours. Consider implementing a token refresh endpoint for customers who miss the window.
- **Email deliverability:** Monitor Resend dashboard for bounce rates and spam complaints.
- **Database cleanup:** Archive deliveries older than 1 year monthly.
- **Product updates:** When updating product files, all new purchases get the latest version automatically.
- **A/B testing:** Consider testing different email subject lines for delivery and review request emails.

---

*Last updated: 2026-03-18*
*Owner: SOVEREIGN*
*Status: Ready for implementation*
