---
kind: playbook
domain: guides
topics: [guides, setup, ssl]
authority: guidance
---

# How to Setup SSL

**What you'll have at the end:** Valid HTTPS certificates on your domain with automatic renewal verified.

---

## Option A: Nginx + Certbot

**Prerequisites:** Nginx installed, port 80 and 443 open in UFW, DNS A record pointing to the server.

```bash
# Install certbot with Nginx plugin
sudo apt install -y certbot python3-certbot-nginx

# Issue the certificate and auto-configure Nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Follow the prompts:
- Enter an email address for expiry notifications
- Agree to the terms of service
- Select "Redirect HTTP traffic to HTTPS" (recommended)

**Verify issuance:**

```bash
sudo certbot certificates
# Should show your domain with expiry date ~90 days out

curl -I https://yourdomain.com
# Should return HTTP/2 200 and include Strict-Transport-Security header
```

**Verify renewal automation:**

```bash
# Test the renewal process (dry run — no actual renewal)
sudo certbot renew --dry-run
# Should output: "Congratulations, all simulated renewals succeeded"
```

Check that the renewal timer is active:

```bash
sudo systemctl status certbot.timer
# Should show: active (waiting)
```

**Test a forced renewal (optional, but recommended):**

```bash
# Force renewal now, even if the cert is not near expiry
sudo certbot renew --force-renewal
sudo systemctl reload nginx
```

---

## Option B: Caddy (Zero Configuration)

Caddy handles SSL automatically — no certbot, no cron, no manual renewal steps.

**Prerequisites:** Caddy installed, DNS A record pointing to the server, port 80 and 443 open.

```bash
sudo nano /etc/caddy/Caddyfile
```

```caddyfile
yourdomain.com {
    reverse_proxy localhost:3000
}
```

```bash
sudo systemctl reload caddy

# Caddy requests the certificate automatically on first request
# Verify after ~30 seconds:
curl -I https://yourdomain.com
```

No further configuration needed. Caddy renews certificates automatically.

---

## Verify HSTS

HSTS tells browsers to always use HTTPS for your domain. Check the response header:

```bash
curl -I https://yourdomain.com | grep -i strict-transport
# Should return: strict-transport-security: max-age=63072000; includeSubDomains; preload
```

If missing, add this to your Nginx server block or Caddyfile:

**Nginx:**
```nginx
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

**Caddy:**
```caddyfile
header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
```

---

## Verify SSL Grade

Run an SSL Labs test to confirm the certificate and configuration are correct:

Visit `ssllabs.com/ssltest/analyze.html?d=yourdomain.com` and verify the grade is **A** or **A+**.

Common issues causing lower grades:
- TLS 1.0/1.1 enabled (should be disabled)
- Weak cipher suites
- Missing HSTS

---

## Related Documents

- [VPS Nginx](../delivery/infrastructure/vps/nginx.md)
- [VPS Caddy](../delivery/infrastructure/vps/caddy.md)
- [VPS SSL/TLS](../delivery/infrastructure/vps/ssl-tls.md)
- [Security transport](../security/transport.md)
