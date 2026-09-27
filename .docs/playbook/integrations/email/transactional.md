---
kind: playbook
domain: integrations
topics: [integrations, email, transactional]
authority: guidance
---

# Transactional Emails

Transactional emails are triggered by user actions or system events. They are non-optional (unlike marketing emails) and must arrive reliably. Every production SaaS has a minimum required set of transactional emails that must be sent before launch.

## Minimum Required Email List at Launch

| Email | Trigger | Required fields |
|---|---|---|
| **Welcome / email verification** | Signup | Verification link (time-limited, single-use), product name |
| **Password reset** | Forgot password request | Reset link (time-limited, single-use), expiry notice |
| **Payment confirmation / receipt** | Successful payment | Invoice amount, plan name, next billing date, link to invoice |
| **Payment failed** | Failed charge | Amount, a direct link to update payment method, grace period end date |
| **Plan changed** | Upgrade, downgrade, or plan switch | New plan name, new price, effective date |
| **Data export ready** | Export job complete | Download link (time-limited), expiry date |
| **Account deletion confirmation** | Deletion job complete | Confirmation that all data has been removed |

## Template Structure

Every transactional email must include:
- **Subject line:** Specific and factual — "Your payment failed" not "Important account notice"
- **HTML body:** Branded, responsive template
- **Plain-text fallback:** Required for email clients that disable HTML and for spam filter scoring
- **Unsubscribe link:** Not required for strictly transactional (verification, password reset, payment receipt) — required for any notification-type email

```typescript
import { Resend } from 'resend';
import { WelcomeEmail } from '../emails/welcome';

const resend = new Resend(process.env.RESEND_API_KEY!);

export async function sendWelcomeEmail(user: { email: string; name: string; verificationToken: string }) {
  await resend.emails.send({
    from: 'Product <noreply@yourdomain.com>',
    to: user.email,
    subject: 'Verify your email address',
    react: WelcomeEmail({ name: user.name, verificationToken: user.verificationToken }),
    text: `Hi ${user.name},\n\nVerify your email: ${process.env.APP_URL}/verify?token=${user.verificationToken}\n\nThis link expires in 24 hours.`,
  });
}
```

## Domain Authentication

Your sending domain must be authenticated before sending. Unauthenticated email is treated as spam by most modern email clients.

**SPF (Sender Policy Framework):** A DNS TXT record listing the servers authorized to send email from your domain.

```
TXT @ "v=spf1 include:resend.com ~all"
```

**DKIM (DomainKeys Identified Mail):** A cryptographic signature added to outgoing email, verified against a public key in DNS.

Most providers generate the DKIM key pair; you add the public key to your DNS as a TXT record. Follow the provider-specific setup guide.

**DMARC (Domain-based Message Authentication, Reporting & Conformance):** Tells receiving servers what to do when SPF or DKIM fails.

```
TXT _dmarc "v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com"
```

Start with `p=none` (monitor only) and move to `p=quarantine` or `p=reject` after validating your SPF and DKIM are correctly set up.

All three records are required for reliable deliverability. Your email provider's setup guide will provide the exact records to create. Verify with MXToolbox or the provider's built-in verification tool.

## Plain-Text Fallback

Every HTML email must have a plain-text alternative. Many email clients prefer plain text; spam filters use the ratio of text to HTML as a spam signal. A well-written plain-text fallback can improve deliverability.

```typescript
const emailContent = {
  html: renderToString(WelcomeEmail({ ... })),
  text: `Welcome to ${PRODUCT_NAME}!\n\nVerify your email: ${verificationUrl}\n\nThis link expires in 24 hours.\n\nQuestions? Reply to this email.`,
};
```

## Related Documents

- [Email provider options](provider-options.md)
- [Email notifications](notifications.md)
- [SendGrid integration](../providers/sendgrid/overview.md)
- [Payments dunning](../../business/payments/dunning.md)
