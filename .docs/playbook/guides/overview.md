---
kind: playbook
domain: guides
topics: [guides]
authority: guidance
---

# How-To Guides

The how-to directory contains complete, sequential procedures for common SaaS setup tasks. Each guide starts from zero and ends at a verified, working result.

## Reference Docs vs. How-To Guides

The rest of this library consists of **reference documents**: they explain options, tradeoffs, and decision criteria. They help you understand what to choose and why.

How-to guides are different. They assume you've already made the decision and need to execute it. A how-to guide:

- Gives exact commands, in order
- Skips options and explanations — if you're reading a how-to, you're building, not deciding
- Ends with a verification step so you know it worked
- Produces a tested, production-ready outcome when followed from start to finish

## When to Use Which

| You want to... | Use |
|---|---|
| Understand options for hosting | [Infrastructure hosting options](../delivery/infrastructure/hosting-options.md) |
| Set up a VPS from scratch | [How-to: Provision and harden VPS](provision-and-harden-vps.md) |
| Understand OAuth provider tradeoffs | [OAuth overview](../integrations/providers/oauth/overview.md) |
| Wire up Google sign-in | [How-to: Setup OAuth Google](setup-oauth-google.md) |

## How-To Guides in This Directory

| Guide | What you'll have at the end |
|---|---|
| [Provision and harden VPS](provision-and-harden-vps.md) | A secured, ready-to-deploy server |
| [Deploy to VPS](deploy-vps.md) | A live, running application |
| [Setup SSL](setup-ssl.md) | Valid HTTPS certificates with renewal verified |
| [Setup cookie consent](setup-cookie-consent.md) | Compliant cookie consent flow |
| [Setup privacy policy](setup-privacy-policy.md) | A hosted, linked privacy policy |
| [Setup env validation](setup-env-validation.md) | Fail-fast env var validation at startup |
| [Test webhooks locally](test-webhooks-locally.md) | Working local webhook tunnel |
| [Test workers locally](test-workers-locally.md) | Local queue processing verified |
| [Test third-party integrations](test-third-party-integrations.md) | Integration testing strategy applied |
| [Setup Stripe subscriptions](setup-stripe-subscriptions.md) | End-to-end subscription flow |
| [Setup Stripe Connect](setup-stripe-connect.md) | Working marketplace payment flow |
| [Setup SendGrid](setup-sendgrid.md) | Verified, deliverable transactional email |
| [Setup Twilio SMS](setup-twilio-sms.md) | Working compliant SMS flow |
| [Setup OpenAI](setup-openai.md) | Working, cost-controlled OpenAI integration |
| [Setup Anthropic](setup-anthropic.md) | Working Anthropic integration |
| [Setup OAuth — Google](setup-oauth-google.md) | Working Google sign-in |
| [Setup OAuth — GitHub](setup-oauth-github.md) | Working GitHub sign-in |
| [Setup admin panel](setup-admin-panel.md) | Secured, functioning admin panel |
| [Setup notifications](setup-notifications.md) | Full in-app notification system |
| [Setup data export and deletion](setup-data-export-and-deletion.md) | Compliant export and deletion flows |
