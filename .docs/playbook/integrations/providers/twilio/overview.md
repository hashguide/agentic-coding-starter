---
kind: playbook
domain: integrations
topics: [integrations, providers, twilio]
authority: guidance
---

# Twilio Setup

Twilio provides SMS, voice, and messaging services. This document covers account creation, credential location, phone number provisioning, and subaccount strategy.

## Account Creation

1. Create a Twilio account at twilio.com
2. Verify your phone number during signup
3. Complete identity verification if required for your country

The Console dashboard shows your **Account SID** and **Auth Token** immediately after signup.

## Credentials

**Console → Account → General settings → API credentials**

```
Account SID:  ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Auth Token:   [hidden — click to reveal]
```

The Auth Token is the master credential for your account. Treat it like a private key:
- Store it only in environment variables / secrets manager
- Never log it
- Rotate it immediately if ever exposed

```
TWILIO_ACCOUNT_SID=ACxxx...
TWILIO_AUTH_TOKEN=your_auth_token
TWILIO_PHONE_NUMBER=+15551234567
```

## Phone Number Provisioning

**Console → Phone Numbers → Manage → Buy a number**

Select a number with the capabilities your application requires:
- **SMS-capable:** Required for sending and receiving SMS
- **Voice-capable:** Required for voice calls
- **MMS-capable:** Required if sending media messages

Choose a number in your target country. US numbers: $1/month. International: varies.

For higher SMS delivery rates in the US, consider a **Toll-Free number** (more deliverable than local numbers for A2P messaging) or a **10DLC-registered local number** (required for commercial SMS in the US — see SMS compliance).

## Subaccounts

Subaccounts provide isolated environments under your master account — separate credentials, phone numbers, and billing. Use subaccounts for:
- **Environments:** A subaccount per environment (production, staging) keeps credentials cleanly separated
- **Multi-tenant isolation:** Each tenant gets a subaccount with its own phone number (for marketplace-style products)

**Console → Account → Subaccounts → Create new subaccount**

Each subaccount has its own Account SID and Auth Token.

## Security

**Rotate the Auth Token immediately if ever exposed.** The Auth Token authenticates every API call and every inbound webhook request. A leaked token gives an attacker the ability to send SMS from your numbers, access call logs, and modify phone number configurations.

**Validate X-Twilio-Signature on every inbound webhook request.** Without validation, any actor can POST to your webhook endpoints and trigger SMS sending or other actions.

## Related Documents

- [Twilio SMS](sms.md)
- [Twilio webhooks](webhooks.md)
- [Twilio testing](testing.md)
