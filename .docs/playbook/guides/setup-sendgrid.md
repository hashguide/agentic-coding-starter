---
kind: playbook
domain: guides
topics: [guides, setup, sendgrid]
authority: guidance
---

# How to Setup SendGrid

**What you'll have at the end:** Verified, deliverable transactional email from your own domain using SendGrid.

---

## Step 1 — Create a SendGrid Account and API Key

1. Sign up at sendgrid.com
2. Navigate to **Settings → API Keys → Create API Key**
3. Name the key (e.g., "Production Transactional")
4. Choose **Restricted Access**
5. Enable **Mail Send → Full Access** only
6. Copy the key immediately

```
SENDGRID_API_KEY=SG....
EMAIL_FROM=noreply@yourdomain.com
```

---

## Step 2 — Verify Your Sending Domain

SendGrid requires DNS records to authenticate your domain and improve deliverability.

1. **Settings → Sender Authentication → Authenticate Your Domain**
2. Enter your domain
3. Choose your DNS provider
4. SendGrid provides 3 CNAME records — add them to your DNS

```
em1234.yourdomain.com    CNAME   u12345678.wl.sendgrid.net
s1._domainkey.yourdomain.com   CNAME   s1.domainkey.u12345678.wl.sendgrid.net
s2._domainkey.yourdomain.com   CNAME   s2.domainkey.u12345678.wl.sendgrid.net
```

5. Click **Verify** in SendGrid after adding the records (propagation can take 5–30 minutes)

---

## Step 3 — Create a Dynamic Template

1. **Email API → Dynamic Templates → Create a Dynamic Template**
2. Add a version (design editor or code editor)
3. Use Handlebars syntax for dynamic values: `{{name}}`, `{{ctaUrl}}`
4. Note the **Template ID** (`d-...`)

---

## Step 4 — Implement the Send Call

```typescript
import sgMail from '@sendgrid/mail';

sgMail.setApiKey(process.env.SENDGRID_API_KEY!);

export async function sendWelcomeEmail(user: { email: string; name: string }) {
  await sgMail.send({
    to: user.email,
    from: {
      email: process.env.EMAIL_FROM!,
      name: 'Your Product',
    },
    templateId: process.env.SENDGRID_WELCOME_TEMPLATE_ID!,
    dynamicTemplateData: {
      name: user.name,
      ctaUrl: `${process.env.APP_URL}/dashboard`,
    },
    // Required for GDPR/CAN-SPAM compliance — always include plain text
    text: `Welcome to Your Product, ${user.name}! Get started: ${process.env.APP_URL}/dashboard`,
  });
}
```

---

## Step 5 — Configure the Event Webhook

The event webhook lets you track bounces, opens, unsubscribes, and spam reports.

1. **Settings → Mail Settings → Event Webhook**
2. Set HTTP POST URL: `https://yourdomain.com/api/webhooks/sendgrid`
3. Enable events: Bounced, Spam Report, Unsubscribe, Delivered
4. Test the connection

Handle the webhook to suppress bounced addresses (see [SendGrid webhooks](../integrations/providers/sendgrid/webhooks.md)).

---

## Step 6 — Send and Verify a Real Test Email

```bash
# Use the SendGrid sandbox mode to test without delivering
# (in your test code)
await sgMail.send({
  ...emailData,
  mailSettings: { sandboxMode: { enable: process.env.NODE_ENV === 'test' } },
});

# For a real deliverability test, send to a test inbox service
# like Mailosaur or use your own email address
```

**Deliverability checklist:**

```bash
# Verify SPF record
nslookup -type=TXT yourdomain.com | grep spf

# Check DKIM via MXToolbox
# Visit: mxtoolbox.com/dkim.aspx and enter s1._domainkey.yourdomain.com

# Full deliverability score
# Visit: mail-tester.com — send a real email and get a score
```

Target: score of 9/10 or higher. Common issues:
- Missing or invalid DKIM — check your CNAME records propagated correctly
- No unsubscribe link in marketing emails — required by CAN-SPAM
- Sending from a domain with no SPF record

---

## Verify

- [ ] API key created with Mail Send only (no other permissions)
- [ ] Domain authentication verified (green checkmarks in SendGrid)
- [ ] Test email lands in inbox, not spam
- [ ] Event webhook receives events
- [ ] mail-tester.com score is 9/10 or higher

---

## Related Documents

- [Integrations SendGrid overview](../integrations/providers/sendgrid/overview.md)
- [Integrations SendGrid transactional](../integrations/providers/sendgrid/transactional.md)
- [Email transactional](../integrations/email/transactional.md)
