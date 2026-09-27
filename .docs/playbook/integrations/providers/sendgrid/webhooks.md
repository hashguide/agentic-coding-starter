---
kind: playbook
domain: integrations
topics: [integrations, providers, sendgrid, webhooks]
authority: guidance
---

# SendGrid Event Webhook

SendGrid sends events to your application when email is delivered, opened, bounced, or marked as spam. Use these events to update your database, suppress future sends to bad addresses, and monitor deliverability.

## Configuring the Event Webhook

**Settings → Mail Settings → Event Webhook → Enable**

- HTTP Post URL: `https://yourdomain.com/webhooks/sendgrid`
- Select events to receive (recommended: all events)
- Enable "Signed Event Webhook" for signature verification

## Event Types and Handler Actions

| Event | Recommended action |
|---|---|
| `delivered` | Update email delivery log; no other action required |
| `open` | Update email open tracking for analytics |
| `click` | Update link click tracking for analytics |
| `bounce` (hard) | Add to suppression list; mark `emailBounced = true` on user record; do not send further email |
| `bounce` (soft) | Log; retry sending may be automatic — monitor for repeated soft bounces |
| `spam_report` | Add to suppression list; mark user as spam reporter; remove from all marketing |
| `unsubscribe` | Update notification preferences; mark email channel as disabled for this user |
| `group_unsubscribe` | Update the specific group preference for this user |

Hard bounces (permanent delivery failures — email address doesn't exist) must suppress future sends immediately. Sending to hard-bounced addresses damages your sender reputation.

## Webhook Handler

```typescript
app.post('/webhooks/sendgrid', express.json(), async (req, res) => {
  // Verify signature (requires "Signed Event Webhook" enabled in SendGrid)
  const signature = req.headers['x-twilio-email-event-webhook-signature'] as string;
  const timestamp = req.headers['x-twilio-email-event-webhook-timestamp'] as string;

  if (signature && !verifySendGridSignature(req.body, signature, timestamp)) {
    return res.status(400).send('Invalid signature');
  }

  // SendGrid sends an array of events per request
  const events = req.body as SendGridEvent[];

  await webhookQueue.add('sendgrid', { events }, {
    jobId: `sg-${Date.now()}`, // Batch processing — not per-event dedup
  });

  res.sendStatus(200);
});

async function processSendGridEvents(events: SendGridEvent[]) {
  for (const event of events) {
    if (event.event === 'bounce' && event.type === 'bounce') {
      await db.user.updateMany({
        where: { email: event.email },
        data: { emailBounced: true, emailBouncedAt: new Date(event.timestamp * 1000) },
      });
    } else if (event.event === 'spamreport') {
      await db.notificationPreference.updateMany({
        where: { user: { email: event.email } },
        data: { emailEnabled: false },
      });
    }
  }
}
```

## Signature Verification

```typescript
import { createHmac } from 'crypto';

function verifySendGridSignature(payload: unknown, signature: string, timestamp: string): boolean {
  const publicKey = process.env.SENDGRID_WEBHOOK_VERIFICATION_KEY!;
  const timestampedPayload = timestamp + JSON.stringify(payload);

  // SendGrid uses ECDSA P-256 for event webhook signatures
  // Use the @sendgrid/eventwebhook package for production:
  // import { EventWebhook } from '@sendgrid/eventwebhook';
  // const ew = new EventWebhook();
  // const key = ew.convertPublicKeyToECDH(publicKey);
  // return ew.verifySignature(key, rawBody, signature, timestamp);
  return true; // Replace with real verification
}
```

## Related Documents

- [SendGrid overview](overview.md)
- [SendGrid transactional](transactional.md)
- [Workers webhooks (general pattern)](../../../backend/async/webhooks.md)
