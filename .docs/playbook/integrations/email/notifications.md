---
kind: playbook
domain: integrations
topics: [integrations, email, notifications]
authority: guidance
---

# Email Notifications

Not all email is the same. The category of an email determines whether the user can unsubscribe from it and how it must be managed. Treating notification email as marketing (or vice versa) creates both legal and user experience problems.

## Email Category Distinctions

| Category | Definition | User can unsubscribe? | Examples |
|---|---|---|---|
| **Transactional** | Triggered directly by the user's own action or a critical account event | No — required for account function | Verification, password reset, payment receipt, deletion confirmation |
| **Product notification** | Triggered by other users' actions or system events relevant to the user's account | Yes | Comment added, teammate joined, file shared, export ready |
| **Marketing** | Promotional content, newsletters, feature announcements not triggered by a specific user action | Yes (legally required) | Product updates newsletter, "see what's new" emails, upsell campaigns |

## Relationship Between In-App and Email Notifications

In-app notifications (the bell icon + panel — see [design/interaction/notifications.md](../../experience/interaction/notifications.md)) and email notifications are independent delivery channels for the same event. A user should be able to:

- Receive the in-app notification but not the email
- Receive the email but not the in-app notification (unusual but valid)
- Receive both
- Receive neither (for non-critical notification types)

These two channels are toggled independently per notification type in the user's notification preferences.

## Notification Preference Center

The settings page Notifications section provides per-category, per-channel control:

```
Notifications

                     In-app     Email
──────────────────────────────────────
Comments             [✓]        [✓]
Task assigned        [✓]        [ ]
Team member joined   [✓]        [ ]
Billing updates      [✓]        [✓]
Security alerts      [✓]        [✓]  ← cannot be disabled
Weekly digest        [—]        [ ]  ← email only, no in-app equivalent
```

Security alerts (new login device, password changed, payment method changed) are non-disableable — they are safety communications, not preferences.

See [design/layouts/settings.md](../../experience/layouts/settings.md) for the full settings page design pattern.

## Unsubscribe Requirements

All non-transactional email (product notifications and marketing) must include an unsubscribe link. This is a legal requirement under CAN-SPAM (US), GDPR (EU), and similar regulations.

The unsubscribe must:
- Work in one click — no "confirm unsubscribe" page, no requiring the user to log in
- Take effect within 10 business days (CAN-SPAM requirement; in practice, implement immediately)
- Respect the category: unsubscribing from "weekly digest" must not unsubscribe from payment failure notifications

```typescript
// One-click unsubscribe endpoint
app.post('/api/email/unsubscribe', asyncHandler(async (req, res) => {
  const { token } = z.object({ token: z.string() }).parse(req.body);

  // Verify and decode the unsubscribe token (signed, contains userId + category)
  const payload = verifyUnsubscribeToken(token);

  await db.notificationPreference.upsert({
    where: { userId_category: { userId: payload.userId, category: payload.category } },
    create: { userId: payload.userId, category: payload.category, emailEnabled: false },
    update: { emailEnabled: false },
  });

  res.json({ message: 'You have been unsubscribed.' });
}));

// Generate a signed unsubscribe token for email links
function generateUnsubscribeToken(userId: string, category: string): string {
  const payload = `${userId}:${category}`;
  const sig = createHmac('sha256', process.env.UNSUBSCRIBE_SECRET!).update(payload).digest('hex');
  return Buffer.from(`${payload}:${sig}`).toString('base64url');
}
```

## List-Unsubscribe Header

Add the `List-Unsubscribe` header to all non-transactional emails. Gmail and Outlook use this header to offer a one-click unsubscribe in the email client UI:

```typescript
await resend.emails.send({
  from: 'Product <noreply@yourdomain.com>',
  to: user.email,
  subject: 'A comment was added to your project',
  headers: {
    'List-Unsubscribe': `<${process.env.APP_URL}/api/email/unsubscribe?token=${token}>`,
    'List-Unsubscribe-Post': 'List-Unsubscribe=One-Click',
  },
  // ...
});
```

## Related Documents

- [In-app notifications design](../../experience/interaction/notifications.md)
- [Settings layout (notification preferences)](../../experience/layouts/settings.md)
- [Transactional emails](transactional.md)
- [Email provider options](provider-options.md)
