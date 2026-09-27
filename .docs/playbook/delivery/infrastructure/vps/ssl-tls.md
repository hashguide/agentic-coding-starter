---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, ssl, tls]
authority: guidance
---

# SSL/TLS on VPS

Let's Encrypt provides free, automated TLS certificates. When using Nginx, certbot handles certificate issuance and installs renewal automation. When using Caddy, this is all automatic — no configuration needed (see [Caddy](caddy.md)).

## Certbot Setup for Nginx

```bash
sudo apt install certbot python3-certbot-nginx -y

# Issue certificate and auto-configure Nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Follow the prompts:
# - Enter email address for renewal notifications
# - Agree to terms of service
# - Choose whether to redirect HTTP → HTTPS (choose yes)
```

After success, certbot:
- Places certificates at `/etc/letsencrypt/live/yourdomain.com/`
- Modifies your Nginx site config to reference them
- Installs a systemd timer or cron job to auto-renew

## Verify Renewal Automation

This is the most important step: **verify renewal before the certificate expires**, not when it does.

```bash
# Check the renewal timer is active
sudo systemctl status certbot.timer

# Dry-run renewal to verify the process works
sudo certbot renew --dry-run

# Check when certificates expire
sudo certbot certificates
```

Let's Encrypt certificates expire in 90 days. Certbot renews them when ≤30 days remain. Renewal must succeed automatically — a failed renewal that goes unnoticed causes a full HTTPS outage.

Set up a renewal failure alert:

```bash
# Test a real renewal failure alert by checking the renewal log
cat /var/log/letsencrypt/letsencrypt.log | grep -i "error\|failed"
```

Configure email alerts in `/etc/letsencrypt/renewal/<domain>.conf`:

```ini
[renewalparams]
# ...existing params...
post_hook = systemctl reload nginx
```

## HSTS

HSTS (HTTP Strict Transport Security) tells browsers to always use HTTPS for your domain, even if the user types `http://`. Once set, browsers ignore HTTP entirely for the specified duration.

```nginx
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

- `max-age=63072000` — 2 years (required for preload list submission)
- `includeSubDomains` — all subdomains must also be HTTPS
- `preload` — opt into browser preload lists (browsers ship with your domain hardcoded as HTTPS-only)

**Before setting `preload`:** ensure every subdomain you control serves valid HTTPS. Removing a domain from the preload list takes months.

## OCSP Stapling

OCSP stapling reduces latency by having Nginx pre-fetch the certificate's validity response from the CA and attach it to the TLS handshake, rather than requiring the browser to query the CA separately:

```nginx
# In the server block
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
```

## Recommended Cipher Suite Restrictions

Certbot's `options-ssl-nginx.conf` (included automatically) already restricts to modern ciphers. To manually verify and tighten:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers off;  # TLS 1.3 handles cipher selection well; don't override
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
```

Disable TLS 1.0 and 1.1 — they have known vulnerabilities and are deprecated by all major browsers.

## Security

Verify renewal automation before it's needed — an expired certificate causes a full outage. Test the dry-run immediately after setup, not 80 days later when the first renewal window opens.

## Related Documents

- [VPS Nginx](nginx.md)
- [VPS Caddy](caddy.md)
- [Security transport](../../../security/transport.md)
