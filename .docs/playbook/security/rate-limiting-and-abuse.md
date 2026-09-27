---
kind: playbook
domain: security
topics: [rate-limiting, abuse, quotas]
authority: guidance
---
# Rate Limiting and Abuse Controls

Protect scarce resources and high-risk actions according to the actual abuse boundary.

## Identify abuse surfaces
Typical examples include authentication, account creation, password recovery, public APIs, scraping, messaging, AI/model calls, file processing, webhooks, expensive searches, exports, invitations, and third-party quota consumption.

## Limit dimensions
Choose one or more keys that match the risk:
- IP/network for unauthenticated abuse
- account/user for individual consumption
- tenant/workspace for shared quotas
- API key/client for integrations
- resource/recipient for spam or hot-spot protection

## Control types
- fixed/sliding/token-bucket limits for request rate
- concurrency limits for expensive in-flight work
- usage quotas for bounded plan/provider consumption
- progressive delays or temporary lockouts for repeated risky attempts
- challenge/verification where automation abuse is the problem

## Define behavior
Specify thresholds, windows, burst allowance, response/error contract, retry timing, user messaging, trusted-service handling, observability, and administrative override/recovery.

## Review
Check distributed consistency where multiple instances enforce limits, proxy/IP handling, accidental shared-IP impact, bypass via multiple accounts/keys, denial-of-service amplification, and whether limits align with provider quotas and paid entitlements.

Record the protected action, limiting key, algorithm, threshold rationale, user experience, and monitoring in project security or feature technical design.
