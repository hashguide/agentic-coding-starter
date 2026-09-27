---
kind: playbook
domain: integrations
topics: [integrations, providers, sendgrid, transactional]
authority: guidance
---

# SendGrid Transactional Email

This document covers creating dynamic templates, sending email via the API, and configuring unsubscribe handling for non-transactional email categories.

## Creating a Dynamic Template

Dynamic templates use Handlebars syntax for variable substitution.

**Email API → Dynamic Templates → Create a Dynamic Template**

1. Name the template (e.g., "Welcome Email")
2. Add a version and design it with the Visual Editor or code editor
3. Note the Template ID (`d-...`)

Template with Handlebars variables:

```html
<!-- Template body -->
<h1>Welcome, {{name}}!</h1>
<p>Thanks for joining {{productName}}. Click the link below to verify your email.</p>
<a href="{{verificationUrl}}">Verify my email</a>
<p>This link expires in 24 hours.</p>
```

## Sending via the API

```typescript
import sgMail from '@sendgrid/mail';

sgMail.setApiKey(process.env.SENDGRID_API_KEY!);

async function sendWelcomeEmail(params: {
  to: string;
  name: string;
  verificationUrl: string;
}) {
  await sgMail.send({
    to: params.to,
    from: {
      email: 'noreply@yourdomain.com',
      name: 'Product Name',
    },
    templateId: 'd-your-template-id-here',
    dynamicTemplateData: {
      name: params.name,
      productName: 'Product Name',
      verificationUrl: params.verificationUrl,
    },
    // Always include plain text for clients that prefer it
    text: `Welcome, ${params.name}!\n\nVerify your email: ${params.verificationUrl}\n\nThis link expires in 24 hours.`,
  });
}
```

## Unsubscribe Handling

SendGrid provides two mechanisms:

**Unsubscribe Groups:** For multi-category email (transactional vs. notifications vs. marketing), create separate groups and assign each sent email to the appropriate group. Users can unsubscribe from individual groups without affecting others.

**Email API → Unsubscribes → Unsubscribe Groups → Create**

```typescript
await sgMail.send({
  to: user.email,
  from: 'noreply@yourdomain.com',
  templateId: 'd-notification-template',
  dynamicTemplateData: { ... },
  asm: {
    groupId: 12345, // Your unsubscribe group ID
    groupsToDisplay: [12345], // Groups shown in the preference center
  },
});
```

**Global Unsubscribe:** Added automatically by SendGrid. One-click opt-out from all non-transactional email.

**Respecting Opt-Outs:** When SendGrid receives an unsubscribe, it suppresses future sends to that address. Check suppression lists via the API before attempting to re-add suppressed addresses.

## Related Documents

- [SendGrid overview](overview.md)
- [SendGrid webhooks](webhooks.md)
- [Email transactional](../../email/transactional.md)
- [Email notifications](../../email/notifications.md)
