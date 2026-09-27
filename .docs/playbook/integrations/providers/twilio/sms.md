---
kind: playbook
domain: integrations
topics: [integrations, providers, twilio, sms]
authority: guidance
---

# Twilio SMS

This document covers sending SMS messages, opt-in compliance requirements for US messaging, opt-out handling, and delivery status tracking.

## Sending an SMS

```typescript
import twilio from 'twilio';

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID!,
  process.env.TWILIO_AUTH_TOKEN!,
);

async function sendSms(params: {
  to: string;     // E.164 format: +15551234567
  body: string;
  userId: string; // For audit logging
}) {
  const message = await client.messages.create({
    body: params.body,
    from: process.env.TWILIO_PHONE_NUMBER!,
    to: params.to,
    statusCallback: `${process.env.APP_URL}/webhooks/twilio/status`,
  });

  // Log the send for audit trail and delivery tracking
  await db.smsLog.create({
    data: {
      userId: params.userId,
      messageSid: message.sid,
      to: params.to,
      status: message.status,
      sentAt: new Date(),
    },
  });

  return message;
}
```

## TCPA / Opt-In Compliance (US)

Sending commercial SMS to US numbers without prior written consent is a violation of the Telephone Consumer Protection Act (TCPA). Penalties: $500–$1,500 per message.

**Required before sending any marketing or transactional SMS:**
- **Opt-in collection:** User explicitly consents to receive SMS from you. The opt-in must be:
  - Voluntary (not buried in terms)
  - Specific to SMS (not just agreeing to terms of service)
  - Recorded with timestamp and phone number
- **Opt-in confirmation message:** After opt-in, send: "[Product Name]: You've opted in to receive messages. Msg & data rates may apply. Reply STOP to unsubscribe."
- **Program disclosure:** Include in all messages or at opt-in: message frequency, "Msg & data rates may apply"

**Transactional vs. promotional:** Transactional messages (verification codes, order confirmations) have lower compliance requirements than promotional messages. Verify the classification with your legal counsel.

## STOP / Opt-Out Handling

Twilio automatically handles STOP, UNSTOP, HELP, and other standard keywords at the carrier level — you do not need to implement keyword parsing yourself. However, you must:
- Honor opt-outs in your database to prevent sending from your application
- Log opt-out events

When Twilio receives a STOP reply, it:
1. Blocks future messages from your number to that phone number at the carrier level
2. Sends your webhook a message status event with the body "STOP"

Your application must update the user's messaging preference:

```typescript
// In your inbound SMS webhook handler
if (req.body.Body?.toUpperCase() === 'STOP') {
  await db.user.updateMany({
    where: { phone: req.body.From },
    data: { smsOptOut: true, smsOptOutAt: new Date() },
  });
}
```

## Delivery Status Tracking

The `statusCallback` URL in the send request receives delivery status updates as Twilio receives them from the carrier:

- `queued` → `sent` → `delivered` (success path)
- `queued` → `sent` → `failed` / `undelivered` (failure path)

Update your database with each status event:

```typescript
await db.smsLog.update({
  where: { messageSid: req.body.MessageSid },
  data: {
    status: req.body.MessageStatus,
    errorCode: req.body.ErrorCode || null,
    updatedAt: new Date(),
  },
});
```

## Related Documents

- [Twilio overview](overview.md)
- [Twilio webhooks](webhooks.md)
- [Twilio testing](testing.md)
