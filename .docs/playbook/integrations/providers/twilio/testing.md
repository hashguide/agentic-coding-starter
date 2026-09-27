---
kind: playbook
domain: integrations
topics: [integrations, providers, twilio, testing]
authority: guidance
---

# Twilio Testing

Twilio provides test credentials that validate API calls without sending real messages, and "magic" phone numbers that simulate specific outcomes.

## Test Credentials

**Console → Account → Test credentials** (separate from your live credentials)

```
TWILIO_ACCOUNT_SID=ACtest...
TWILIO_AUTH_TOKEN=test_auth_token
```

With test credentials:
- API calls succeed without sending real messages
- You are not charged
- Messages appear in the test console, not in real carrier networks

## Magic Phone Numbers

When using test credentials, Twilio provides magic "to" phone numbers that simulate specific outcomes:

| Phone number | Simulated outcome |
|---|---|
| `+15005550001` | Number is invalid |
| `+15005550006` | Number is valid — message succeeds |
| `+15005550007` | Number is blocked (cannot receive SMS) |
| `+15005550008` | Number does not support SMS |
| `+15005550009` | Account cannot send to this number |

```typescript
// Test sending to a number that will succeed
await sendSms({
  to: '+15005550006',
  body: 'Test message',
  userId: testUser.id,
});

// Test an invalid number
await sendSms({
  to: '+15005550001',
  body: 'Test message',
  userId: testUser.id,
});
// Expect this to throw with Twilio error code 21211
```

## Local Webhook Testing with ngrok

For testing inbound SMS and status callbacks locally, expose your local server with a tunnel:

```bash
# Install ngrok
brew install ngrok

# Expose local server
ngrok http 3000
# Output: Forwarding https://abc123.ngrok.io -> http://localhost:3000
```

Configure the ngrok URL as the webhook in the Twilio console:
- Status Callback: `https://abc123.ngrok.io/webhooks/twilio/status`
- Inbound SMS: `https://abc123.ngrok.io/webhooks/twilio/inbound`

## Simulating Inbound Messages

The Twilio Console provides a "Test" button on your phone number configuration page to simulate an inbound SMS without a real device. You can also use the REST API:

```bash
# Send a test inbound SMS to your webhook
curl -X POST https://api.twilio.com/2010-04-01/Accounts/$ACCOUNT_SID/Messages.json \
  --data-urlencode "From=+15551234567" \
  --data-urlencode "To=+15559876543" \
  --data-urlencode "Body=STOP" \
  -u $ACCOUNT_SID:$AUTH_TOKEN
```

## Related Documents

- [Twilio overview](overview.md)
- [Twilio SMS](sms.md)
- [Twilio webhooks](webhooks.md)
