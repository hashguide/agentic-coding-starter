---
kind: playbook
domain: guides
topics: [guides, setup, twilio, sms]
authority: guidance
---

# How to Setup Twilio SMS

**What you'll have at the end:** A working, compliant SMS flow — messages send, delivery status is tracked, and STOP opt-outs are handled.

---

## Step 1 — Create a Twilio Account and Get Credentials

1. Sign up at twilio.com
2. Note the **Account SID** and **Auth Token** from the Console dashboard
3. These go in environment variables — never commit them

```
TWILIO_ACCOUNT_SID=ACxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token
TWILIO_PHONE_NUMBER=+1XXXXXXXXXX
```

---

## Step 2 — Provision a Phone Number

1. Twilio Console → **Phone Numbers → Manage → Buy a Number**
2. Select a number with SMS capability in your country
3. Note the phone number in E.164 format (`+1XXXXXXXXXX`)
4. Set the number as `TWILIO_PHONE_NUMBER` in your environment

---

## Step 3 — Implement the Send Function

```typescript
import twilio from 'twilio';

const client = twilio(process.env.TWILIO_ACCOUNT_SID!, process.env.TWILIO_AUTH_TOKEN!);

export async function sendSms(params: {
  to: string;     // E.164 format: +1XXXXXXXXXX
  message: string;
  userId: string;
}) {
  const message = await client.messages.create({
    body: params.message,
    from: process.env.TWILIO_PHONE_NUMBER!,
    to: params.to,
    statusCallback: `${process.env.APP_URL}/api/webhooks/twilio/status`,
  });

  // Log for audit and delivery tracking
  await db.smsLog.create({
    data: {
      userId: params.userId,
      sid: message.sid,
      to: params.to,
      body: params.message,
      status: message.status,
    },
  });

  return message.sid;
}
```

---

## Step 4 — Implement the Status Callback Webhook

```typescript
import twilio from 'twilio';

// POST /api/webhooks/twilio/status
app.post('/api/webhooks/twilio/status', express.urlencoded({ extended: false }), async (req, res) => {
  const signature = req.headers['x-twilio-signature'] as string;
  const url = `${process.env.APP_URL}/api/webhooks/twilio/status`;

  const isValid = twilio.validateRequest(
    process.env.TWILIO_AUTH_TOKEN!,
    signature,
    url,
    req.body
  );

  if (!isValid) {
    return res.status(403).send('Invalid signature');
  }

  const { MessageSid, MessageStatus } = req.body;

  await db.smsLog.updateMany({
    where: { sid: MessageSid },
    data: { status: MessageStatus, updatedAt: new Date() },
  });

  res.status(204).send();
});
```

---

## Step 5 — Implement Opt-Out Handling

STOP keyword handling is legally required (TCPA in the US). Twilio handles automatic opt-out replies, but you must also update your database to avoid sending to opted-out numbers.

```typescript
// POST /api/webhooks/twilio/inbound
app.post('/api/webhooks/twilio/inbound', express.urlencoded({ extended: false }), async (req, res) => {
  const signature = req.headers['x-twilio-signature'] as string;
  const isValid = twilio.validateRequest(
    process.env.TWILIO_AUTH_TOKEN!,
    signature,
    `${process.env.APP_URL}/api/webhooks/twilio/inbound`,
    req.body
  );

  if (!isValid) return res.status(403).send();

  const { From, Body } = req.body;
  const keyword = Body?.trim().toUpperCase();

  if (['STOP', 'UNSUBSCRIBE', 'CANCEL', 'END', 'QUIT'].includes(keyword)) {
    await db.user.updateMany({
      where: { phone: From },
      data: { smsOptOut: true, smsOptOutAt: new Date() },
    });
  } else if (['START', 'UNSTOP', 'YES'].includes(keyword)) {
    await db.user.updateMany({
      where: { phone: From },
      data: { smsOptOut: false },
    });
  }

  // TwiML response — can be empty or a brief confirmation
  res.set('Content-Type', 'text/xml');
  res.send('<Response></Response>');
});
```

---

## Step 6 — Configure Webhook URLs in Twilio Console

1. Twilio Console → Phone Numbers → your number → **Messaging Configuration**
2. Set **A message comes in** → Webhook → `https://yourdomain.com/api/webhooks/twilio/inbound`
3. Set **Status Callback URL** → `https://yourdomain.com/api/webhooks/twilio/status`

---

## Step 7 — Verify with a Test Message

```typescript
// Using Twilio's test credentials (ACtest... / safe to commit to tests)
const testClient = twilio('ACtest...', 'test_auth_token');
const message = await testClient.messages.create({
  to: '+15005550006',  // Magic number that always succeeds
  from: '+15005550001', // Test from number
  body: 'Test message',
});
console.log(message.status);  // Should be 'queued'
```

Then send a real message to your own phone number and verify it arrives.

---

## Verify

- [ ] Test message delivered to real phone
- [ ] Status callback updates `smsLog` in the database
- [ ] Sending "STOP" from your phone sets `smsOptOut: true`
- [ ] Webhook signature validation rejects invalid requests

---

## Related Documents

- [Integrations Twilio overview](../integrations/providers/twilio/overview.md)
- [Integrations Twilio SMS](../integrations/providers/twilio/sms.md)
- [Integrations Twilio testing](../integrations/providers/twilio/testing.md)
