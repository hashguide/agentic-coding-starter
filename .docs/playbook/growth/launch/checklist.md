---
kind: playbook
domain: growth
topics: [growth, launch, checklist]
authority: guidance
---

# Launch Checklist

This checklist assumes a hosted product with accounts and payments. For mobile, desktop, package, extension, marketplace, or enterprise releases, first adapt it using [distribution channels](distribution-channels.md) and mark genuinely inapplicable items explicitly rather than implementing SaaS features the product does not need.

## Phase 1 — Pre-Launch (One Week Out)

### Product
- [ ] Core workflow completes end-to-end without errors
- [ ] Onboarding flow tested with at least 3 real people (not team members) — see [design/layouts/onboarding.md](../../experience/layouts/onboarding.md)
- [ ] Payment flow tested with Stripe test cards — see [integrations/stripe/testing.md](../../integrations/providers/stripe/testing.md)
- [ ] Auth enumeration protection in place (login returns identical error whether email doesn't exist or password is wrong) — see [backend/error-messages.md](../../backend/runtime/error-messages.md)
- [ ] Account deletion hard-deletes PII and cancels Stripe subscription — see [how-to/setup-data-export-and-deletion.md](../../guides/setup-data-export-and-deletion.md)
- [ ] Data export flow tested end-to-end — export file received by email

### Infrastructure
- [ ] VPS provisioned and hardened — see [how-to/provision-and-harden-vps.md](../../guides/provision-and-harden-vps.md)
- [ ] SSL certificate issued and renewal automation verified — see [how-to/setup-ssl.md](../../guides/setup-ssl.md)
- [ ] CI pipeline green on main
- [ ] Staging deployment verified
- [ ] Automatic security updates configured — see [infrastructure/vps/system-setup/auto-updates.md](../../delivery/infrastructure/vps/system-setup/auto-updates.md)
- [ ] Server firewall configured (default deny) — see [infrastructure/vps/system-setup/firewall.md](../../delivery/infrastructure/vps/system-setup/firewall.md)

### Monitoring
- [ ] External uptime check configured and alerting — see [observability/uptime.md](../../observability/uptime.md)
- [ ] Error monitoring active (Sentry or equivalent) — see [observability/error-monitoring.md](../../observability/error-monitoring.md)
- [ ] Disk space alerts configured
- [ ] P0 alert goes to SMS, not only email

### Legal and Compliance
- [ ] Privacy policy live at stable URL, linked from footer and signup — see [how-to/setup-privacy-policy.md](../../guides/setup-privacy-policy.md)
- [ ] Terms of service live and linked — see [compliance/terms-of-service.md](../../compliance/terms-of-service.md)
- [ ] Cookie consent banner implemented (if using analytics) — see [how-to/setup-cookie-consent.md](../../guides/setup-cookie-consent.md)
- [ ] GDPR data subject rights implementable (export, deletion) if serving EU users — see [compliance/gdpr.md](../../compliance/gdpr.md)

### Security
- [ ] All secrets in environment variables, not in code — see [security/secrets.md](../../security/secrets.md)
- [ ] MFA enabled for administrators and publisher accounts; recovery tested — see [auth/mfa.md](../../access/authentication/mfa.md)
- [ ] Production and staging use separate credentials — see [environments/secrets-by-environment.md](../../delivery/environments/secrets-by-environment.md)
- [ ] Input validation on all API endpoints — see [security/input-validation.md](../../security/input-validation.md)
- [ ] HSTS header active — see [security/transport.md](../../security/transport.md)
- [ ] Webhook signature validation implemented — see [workers/webhooks.md](../../backend/async/webhooks.md)
- [ ] Rate limiting on auth endpoints — see [infrastructure/vps/nginx.md](../../delivery/infrastructure/vps/nginx.md)

### Email
- [ ] Transactional email domain authenticated (SPF/DKIM/DMARC) — see [email/transactional.md](../../integrations/email/transactional.md)
- [ ] Welcome email, password reset, and payment confirmation emails send correctly
- [ ] Email deliverability verified (score 9/10 on mail-tester.com) — see [how-to/setup-sendgrid.md](../../guides/setup-sendgrid.md)

### Stripe (Live Mode)
- [ ] Switched from test keys to live keys in production
- [ ] Stripe webhook endpoint registered with production URL and live webhook secret
- [ ] Webhook handler tested with a real payment
- [ ] Subscription creation and cancellation tested in live mode

---

## Phase 2 — Launch Day

### Go Live
- [ ] Production deployment executed, health endpoint returns 200
- [ ] Stripe is in live mode (not test mode)
- [ ] No active errors in Sentry
- [ ] Uptime check shows green

### Announce
- [ ] Product Hunt launch submitted at midnight PT (if doing PH launch) — see [marketing/launch-strategy.md](../marketing/launch-strategy.md)
- [ ] Relevant community posts published (Reddit, Hacker News, Discord) with genuine engagement, not spam
- [ ] Personal network notified (email, social)
- [ ] Waitlist email sent to pre-launch subscribers

### First Hours
- [ ] Monitor error rate in Sentry — address any new error types immediately
- [ ] Watch Stripe dashboard for first live payments
- [ ] Respond to all comments and feedback within the first 4 hours

---

## Phase 3 — First Week Post-Launch

### User Feedback
- [ ] In-app feedback widget installed — see [ops/feedback.md](../../operations/feedback.md)
- [ ] Churned trial users emailed with one question: "What made you decide not to continue?" — see [ops/feedback.md](../../operations/feedback.md)
- [ ] Support tickets categorized by topic — recurring themes = documentation gaps

### Metrics
- [ ] Activation metric tracked (first key action completed per new signup) — see [ops/metrics.md](../../operations/metrics.md)
- [ ] Trial-to-paid conversion tracked
- [ ] MRR calculation set up

### Technical
- [ ] Review first week of error logs — address top 3 errors by frequency
- [ ] Confirm all background jobs processed without failures
- [ ] Verify database backup is running
- [ ] Confirm log rotation is working (disk not filling)

---

## Related Documents

- [Marketing launch strategy](../marketing/launch-strategy.md)
- [Ops metrics](../../operations/metrics.md)
- [Ops feedback](../../operations/feedback.md)
- [Security checklist](../../security/checklist.md)
- [Distribution channels](distribution-channels.md)
