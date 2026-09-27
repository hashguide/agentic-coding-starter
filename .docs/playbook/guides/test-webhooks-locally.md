---
kind: playbook
domain: guides
topics: [guides, test, webhooks, locally]
authority: guidance
---

# How to Test Webhooks Locally

**What you'll have at the end:** Inbound webhook events from external providers (Stripe, SendGrid, Twilio) delivered to your local development server and verified end-to-end.

---

## Option A: Stripe CLI (For Stripe Webhooks)

The Stripe CLI is the fastest path for Stripe-specific webhooks. It authenticates as your Stripe account and forwards events directly.

**Step 1 — Install:**

```bash
# macOS
brew install stripe/stripe-cli/stripe

# Linux
curl -s https://packages.stripe.dev/api/security/keypair/stripe-cli-gpg/public | gpg --dearmor | sudo tee /usr/share/keyrings/stripe.gpg
echo "deb [signed-by=/usr/share/keyrings/stripe.gpg] https://packages.stripe.dev/stripe-cli-debian-local stable main" | sudo tee /etc/apt/sources.list.d/stripe.list
sudo apt update && sudo apt install stripe
```

**Step 2 — Authenticate:**

```bash
stripe login
# Opens browser to authenticate with your Stripe account
```

**Step 3 — Forward events to your local server:**

```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```

The CLI outputs a webhook signing secret like `whsec_...`. Use this value as `STRIPE_WEBHOOK_SECRET` in your local `.env`:

```
STRIPE_WEBHOOK_SECRET=whsec_test_...
```

**Step 4 — Trigger test events:**

In a second terminal, trigger events to test your handler:

```bash
# Simulate a subscription created event
stripe trigger customer.subscription.created

# Simulate a payment failure
stripe trigger invoice.payment_failed

# Simulate checkout session completion
stripe trigger checkout.session.completed
```

**Step 5 — Verify:**

Check your local server logs for incoming webhook events. Confirm your handler processes them correctly (e.g., database record updated, email queued).

---

## Option B: ngrok (For Any Provider)

ngrok creates a public HTTPS tunnel to your local server. Paste the ngrok URL as the webhook destination in any provider's console.

**Step 1 — Install:**

```bash
# macOS
brew install ngrok/ngrok/ngrok

# Or download from ngrok.com
```

**Step 2 — Authenticate (free account):**

```bash
ngrok config add-authtoken <your-ngrok-authtoken>
```

**Step 3 — Start the tunnel:**

```bash
# Forward ngrok HTTPS to local port 3000
ngrok http 3000
```

ngrok outputs a public URL like `https://abc123.ngrok.io`.

**Step 4 — Configure the provider:**

Paste the ngrok URL as the webhook endpoint in the provider's dashboard:
- **SendGrid:** Settings → Mail Settings → Event Webhook → `https://abc123.ngrok.io/api/webhooks/sendgrid`
- **Twilio:** Phone number settings → Status Callback → `https://abc123.ngrok.io/api/webhooks/twilio`
- **GitHub:** Repository settings → Webhooks → `https://abc123.ngrok.io/api/webhooks/github`

**Step 5 — Trigger and verify:**

Perform the action that triggers the webhook (send a test email, trigger a status change, etc.) and verify it arrives in your local server logs.

ngrok also provides a web inspector at `http://localhost:4040` showing all incoming requests and allowing replay.

---

## Verifying Signature Validation

Always verify your signature validation logic works correctly:

```bash
# Send a request with an invalid signature and confirm your endpoint returns 400
curl -X POST http://localhost:3000/api/webhooks/stripe \
  -H "Content-Type: application/json" \
  -H "Stripe-Signature: invalid" \
  -d '{"type":"test"}'
# Should return: 400 or 401
```

---

## Related Documents

- [Workers webhooks](../backend/async/webhooks.md)
- [Stripe webhooks](../integrations/providers/stripe/webhooks.md)
- [Twilio webhooks](../integrations/providers/twilio/webhooks.md)
- [SendGrid webhooks](../integrations/providers/sendgrid/webhooks.md)
