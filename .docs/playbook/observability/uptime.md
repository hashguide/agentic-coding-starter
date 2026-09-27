---
kind: playbook
domain: observability
topics: [observability, uptime]
authority: guidance
---

# Uptime Monitoring

Uptime monitoring checks from outside your infrastructure that the application is reachable and responding correctly. If your server, network, or DNS fails, your internal monitoring goes down with it — external checks are the only reliable signal.

## External Uptime Checks Are Non-Optional

The check must come from outside your own infrastructure:
- A monitoring service on the same VPS cannot detect a VPS failure
- A check behind the same CDN cannot detect a CDN outage
- An internal health check cannot detect DNS resolution failures

Use an external provider: **Betterstack**, **UptimeRobot**, **Freshping**, or **Better Uptime**.

## Which Critical Paths to Monitor

Monitor the minimum set of endpoints that confirms the full stack is healthy:

| Check | URL | What it verifies |
|---|---|---|
| **Homepage** | `https://yourdomain.com` | DNS, CDN, server reachable |
| **Health endpoint** | `https://yourdomain.com/api/health` | App running + DB connected |
| **Login page** | `https://yourdomain.com/login` | Auth routes reachable |

The health endpoint must check actual dependencies:

```typescript
// GET /api/health
app.get('/api/health', async (req, res) => {
  try {
    await db.$queryRaw`SELECT 1`;
    res.json({ status: 'ok', db: 'ok', ts: new Date().toISOString() });
  } catch (err) {
    logger.error({ err }, 'Health check failed: database unreachable');
    res.status(503).json({ status: 'error', db: 'unreachable' });
  }
});
```

Do not expose internal system information (memory usage, active connections) in the health endpoint — use a separate, authenticated endpoint for that.

## Check Configuration

- **Interval:** Every 1 minute
- **Retry before alert:** 2 consecutive failures before triggering an alert (reduces false positives from transient network issues)
- **Alert channels:** See below

## Alert Channels by Severity

| Outage duration | Channel |
|---|---|
| 2 consecutive failures (2 minutes) | Slack / email — potential issue |
| 5+ consecutive failures (5 minutes) | SMS + email — confirmed P0 outage |
| Recovery | Slack / email — "service recovered" notification |

**P0 outages must trigger SMS or phone call alerts.** Email is missed at 3 AM. If you have paying customers, on-call means being reachable, even when asleep.

## Status Page

A public status page lets users check whether an issue is on your end before contacting support. It also reduces support ticket volume during incidents.

Options:
- **Betterstack / UptimeRobot** — includes a hosted status page with your custom domain
- **Instatus** — dedicated status page product
- **statuspage.io** — Atlassian product, more enterprise-oriented

Configure the status page to update automatically when your uptime checks detect a failure.

## Recovery Notification

Always send a "back online" notification through the same channels. A team that learns of an outage from a customer — not from monitoring — is a team without effective monitoring.

## Related Documents

- [Observability error monitoring](error-monitoring.md)
- [VPS monitoring](../delivery/infrastructure/vps/monitoring.md)
- [Environments production](../delivery/environments/production.md)
