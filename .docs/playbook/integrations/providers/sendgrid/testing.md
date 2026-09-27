---
kind: playbook
domain: integrations
topics: [integrations, providers, sendgrid, testing]
authority: guidance
---

# SendGrid Testing

Test email sending locally without delivering real messages using SendGrid's sandbox mode, test email addresses, and deliverability tools.

## Sandbox Mode

Sandbox mode processes the API call and validates the request but does not deliver the email. Use it in test environments and CI to verify sends without inbox noise.

```typescript
await sgMail.send({
  to: 'test@example.com',
  from: 'noreply@yourdomain.com',
  subject: 'Test email',
  text: 'Test content',
  mailSettings: {
    sandboxMode: {
      enable: process.env.NODE_ENV !== 'production',
    },
  },
});
```

Set this based on environment — automatically enabled in non-production environments.

## Test Email Addresses

For testing actual email delivery (not sandbox mode), use an email address you control. Options:
- **Personal email:** Direct and verifiable, but inbox fills up quickly
- **Mailosaur:** Testing email service that provides disposable inboxes with API access for automated assertions
- **Mailtrap:** Catches all outbound email in a test inbox without delivering to real recipients

For automated testing, Mailosaur is the most powerful — you can programmatically check that specific emails were received, inspect headers, and verify links.

## Deliverability Checking

Before enabling production sending, verify the setup:

**1. MX Toolbox SPF check:**
Visit mxtoolbox.com/spf.aspx and enter your sending domain. Confirm SendGrid is listed as an authorized sender.

**2. DKIM check:**
Visit mxtoolbox.com/dkim.aspx. Enter your selector (available in SendGrid domain authentication settings) and domain. Confirm the DKIM record is present and valid.

**3. Send a test email and check the headers:**
Send to an actual mailbox and view "Show original" or equivalent to see the Authentication-Results header. Look for:
- `dkim=pass`
- `spf=pass`
- `dmarc=pass`

**4. Spam score check:**
Use mail-tester.com — send one email to the address it provides and it will report a spam score with specific recommendations.

## Related Documents

- [SendGrid overview](overview.md)
- [SendGrid transactional](transactional.md)
- [Email transactional](../../email/transactional.md)
