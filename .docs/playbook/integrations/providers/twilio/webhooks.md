---
kind: playbook
domain: integrations
topics: [integrations, providers, twilio, webhooks]
authority: guidance
---

# Twilio Webhooks

Twilio sends HTTP POST requests to your application for message status updates, inbound SMS, and voice events. All inbound webhook requests must be validated using the X-Twilio-Signature header.

## Status Callback Webhooks

Configure the status callback URL when sending a message (see [sms.md](sms.md)). Twilio POSTs to this URL when the message status changes.

```typescript
app.post('/webhooks/twilio/status', express.urlencoded({ extended: false }), async (req, res) => {
  if (!validateTwilioRequest(req)) {
    return res.status(403).send('Forbidden');
  }

  const { MessageSid, MessageStatus, ErrorCode } = req.body;

  await db.smsLog.update({
    where: { messageSid: MessageSid },
    data: { status: MessageStatus, errorCode: ErrorCode || null },
  });

  res.sendStatus(200);
});
```

## Inbound SMS Webhooks

Configure the inbound SMS URL on your phone number:
**Console → Phone Numbers → Manage → Your number → Messaging → "A Message Comes In"**

```typescript
app.post('/webhooks/twilio/inbound', express.urlencoded({ extended: false }), async (req, res) => {
  if (!validateTwilioRequest(req)) {
    return res.status(403).send('Forbidden');
  }

  const { From, Body, MessageSid } = req.body;

  // Log the inbound message
  await db.inboundSmsLog.create({
    data: { from: From, body: Body, messageSid: MessageSid, receivedAt: new Date() },
  });

  // Handle opt-out keywords
  if (['STOP', 'STOPALL', 'UNSUBSCRIBE', 'CANCEL', 'END', 'QUIT'].includes(Body.trim().toUpperCase())) {
    await db.user.updateMany({
      where: { phone: From },
      data: { smsOptOut: true },
    });
  }

  // Twilio expects a TwiML response (can be empty to send no reply)
  res.type('text/xml').send('<Response></Response>');
});
```

## Signature Validation

```typescript
import twilio from 'twilio';

function validateTwilioRequest(req: express.Request): boolean {
  const twilioSignature = req.headers['x-twilio-signature'] as string;
  const authToken = process.env.TWILIO_AUTH_TOKEN!;
  const url = `${process.env.APP_URL}${req.path}`;

  return twilio.validateRequest(authToken, twilioSignature, url, req.body);
}
```

`twilio.validateRequest()` computes an HMAC-SHA1 of the full URL + sorted POST parameters using your Auth Token and compares it to the `X-Twilio-Signature` header.

**This must be on every inbound endpoint.** Without it, any actor can POST arbitrary payloads to your webhook URLs, triggering message processing, opt-out handling, or other business logic.

## Security

**X-Twilio-Signature validation is mandatory.** Without it, any actor can POST arbitrary payloads to your webhook endpoint. A fabricated inbound "STOP" message could opt out all your users. A fabricated status callback could incorrectly mark messages as delivered or failed.

## Related Documents

- [Twilio overview](overview.md)
- [Twilio SMS](sms.md)
- [Workers webhooks (general pattern)](../../../backend/async/webhooks.md)
