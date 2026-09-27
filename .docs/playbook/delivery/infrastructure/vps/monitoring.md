---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, monitoring]
authority: guidance
---

# VPS Health Monitoring

Server-level monitoring covers disk space, memory, CPU, and uptime. These checks run independently of application-level error monitoring — a server can be healthy at the application layer while disk is filling up or memory is under pressure.

## Disk Space Alerts

A full disk causes crashes across the entire stack — PostgreSQL stops accepting writes, Nginx cannot write logs, Node.js cannot create temp files. Alert before disk fills, not when it does.

**Alert threshold: 75% full — warning. 90% — critical.**

### Option 1: Netdata (if already installed)

Netdata includes disk space monitoring with configurable alerts out of the box. No additional configuration required beyond installation (see [monitoring agents](system-setup/monitoring-agents.md)).

### Option 2: Cron-based email alert

```bash
crontab -e
```

```
# Check disk every 30 minutes; email if over 80%
*/30 * * * * df -h / | awk 'NR==2{gsub(/%/,"",$5); if($5+0 > 80) print "Disk at " $5 "% on " ENVIRON["HOSTNAME"]}' | \
  grep -q "Disk" && \
  df -h | mail -s "[ALERT] Disk space warning on $(hostname)" ops@yourdomain.com
```

### Automatic Cleanup

Prevent disk from filling with log accumulation:

```bash
# Rotate PM2 logs (see pm2.md)
# Rotate Nginx logs
sudo logrotate -f /etc/logrotate.d/nginx

# Remove old Docker images and stopped containers
docker system prune -f
```

## Uptime Monitoring

Uptime checks must run **from outside your own infrastructure**. A check running on the same server as the app cannot detect a network outage or a server-level crash.

**Recommended:** Betterstack (formerly Uptime Robot), Better Uptime, Freshping, or UptimeRobot.

### What to Monitor

| Check | URL | Alert on |
|---|---|---|
| Homepage | `https://yourdomain.com` | Non-200 response |
| Health endpoint | `https://yourdomain.com/api/health` | Non-200, or response takes >5s |
| Login API | `https://yourdomain.com/api/auth/login` | Non-200 for a known-good request |

The health endpoint should check actual system dependencies:

```typescript
// GET /api/health
app.get('/api/health', async (req, res) => {
  try {
    await db.$queryRaw`SELECT 1`;
    res.json({ status: 'ok', db: 'ok' });
  } catch (error) {
    res.status(503).json({ status: 'error', db: 'unreachable' });
  }
});
```

### Alert Channels

- Email: baseline; acceptable for non-critical products
- SMS/phone call: required for P0 (complete outage) alerts — email is missed while sleeping

Configure the uptime checker to:
1. Retry 2x before alerting (reduces false positives from transient network issues)
2. Send SMS for consecutive failures (true outage, not a blip)
3. Send an "up" notification when the service recovers

## Memory Monitoring

High memory usage causes OOM kills and erratic restarts:

```bash
# Quick check
free -h

# Watch memory over time
watch -n 5 free -h

# Check what's consuming memory
ps aux --sort=-%mem | head -20
```

PM2 can restart processes that exceed a memory threshold:

```javascript
// ecosystem.config.js
max_memory_restart: '512M',  // Restart the process if it exceeds 512MB
```

## Related Documents

- [VPS monitoring agents](system-setup/monitoring-agents.md)
- [Observability uptime](../../../observability/uptime.md)
- [Observability logging](../../../observability/logging.md)
