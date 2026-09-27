---
kind: playbook
domain: integrations
topics: [integrations, providers]
authority: guidance
---

# Integrations Overview

Each provider in this directory follows the same four-document pattern: setup/overview, core feature documents, webhooks, and testing. When integrating any provider, read exactly those documents for that provider — nothing else.

## Standard Four-Document Pattern

| Document | Purpose |
|---|---|
| `overview.md` | Account creation, API key generation, environment configuration, security setup |
| `[feature].md` | Core feature implementation (subscriptions, sending email, SMS, completions) |
| `webhooks.md` | Inbound event handling, signature verification, event processing |
| `testing.md` | Test mode, mock credentials, local development, CI strategy |

## Setup, Test, Secure, Monitor Sequence

For every integration, work through this sequence:

1. **Setup:** Follow the `overview.md` to create the account, generate API keys, configure environments (test vs. live), and add keys to the environment config
2. **Implement:** Follow the feature documents to implement the core functionality against test/sandbox credentials
3. **Secure:** Verify signature verification is in place for all inbound webhooks; verify API keys are server-only
4. **Test:** Follow the `testing.md` to verify happy paths, error paths, and webhook events end-to-end in test mode
5. **Monitor:** Add the integration's key events to your error monitoring (unexpected webhook failures, API error rates) before going live

## Provider Directories

- [stripe/](stripe/overview.md) — Payments, subscriptions, Connect
- [sendgrid/](sendgrid/overview.md) — Transactional and notification email
- [twilio/](twilio/overview.md) — SMS messaging
- [openai/](openai/overview.md) — LLM completions and assistants
- [anthropic/](anthropic/overview.md) — Claude API and MCP
- [oauth/](oauth/overview.md) — OAuth 2.0 (Google, GitHub)

## Related Documents

- [Workers webhooks](../../backend/async/webhooks.md)
- [Backend env config](../../backend/runtime/env-config.md)
- [Security secrets](../../security/secrets.md)
