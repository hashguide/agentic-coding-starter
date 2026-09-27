---
kind: playbook
domain: integrations
topics: [integrations, providers, sendgrid]
authority: guidance
---

# SendGrid Setup

SendGrid is a transactional and marketing email provider used by a large number of SaaS companies. This document covers account creation, API key generation, domain authentication, and IP warmup.

## Account Creation

1. Create a SendGrid account at sendgrid.com
2. Set a strong password and enable MFA immediately — your sending reputation is tied to this account
3. Complete the Sender Verification flow (Settings → Sender Authentication)

## API Key Generation

Create a restricted API key with the minimum required permissions:

**Settings → API Keys → Create API Key → Restricted access**

For transactional email sending:
- **Mail Send:** Full access
- Everything else: No access

```typescript
// Environment variable
SENDGRID_API_KEY=SG....

// Usage
import sgMail from '@sendgrid/mail';

sgMail.setApiKey(process.env.SENDGRID_API_KEY!);
```

**Never use the full account API key in application code.** A leaked restricted key with Mail Send only can be used to send email from your domain. A leaked full account key allows an attacker to delete your account, access templates, view contacts, and destroy your sending reputation.

## Domain Authentication (SPF and DKIM)

Unauthenticated email lands in spam. Authenticate your sending domain before sending any production email.

**Settings → Sender Authentication → Domain Authentication → Authenticate a domain**

1. Enter your domain (e.g., `example.com`)
2. SendGrid provides CNAME records to add to your DNS
3. Add the CNAME records at your DNS provider
4. Return to SendGrid and click "Verify"

Once verified, all email sent via SendGrid from your domain is automatically DKIM-signed and SPF-authorized. No additional configuration needed.

## Dedicated IP and IP Warmup

By default, new SendGrid accounts use shared IP addresses. Shared IPs work for most SaaS volumes. If you send high volume (> 50,000 emails/day) or need maximum deliverability control, add a dedicated IP.

**IP Warmup:** A new dedicated IP has no sending reputation. Email clients are skeptical of IP addresses with no history. Warm up by sending gradually increasing volumes over 4–6 weeks:
- Week 1: 100–500 emails/day
- Week 2: 500–2,000/day
- Week 3: 2,000–10,000/day
- Continue doubling weekly until full volume

SendGrid's Automated IP Warmup feature handles this automatically.

## Security

**Generate a restricted API key with send-only permissions.** Never use the full account key in application code. A leaked send-only key is recoverable — you rotate the key and nothing else is compromised. A leaked full account key gives the attacker control of your entire email infrastructure and sending reputation.

## Related Documents

- [SendGrid transactional](transactional.md)
- [SendGrid webhooks](webhooks.md)
- [SendGrid testing](testing.md)
- [Email provider options](../../email/provider-options.md)
