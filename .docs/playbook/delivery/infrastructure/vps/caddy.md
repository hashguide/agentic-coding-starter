---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, caddy]
authority: guidance
---

# Caddy Reverse Proxy

Caddy is a modern web server that handles HTTPS automatically via Let's Encrypt with zero manual certificate configuration. Compared to Nginx, Caddy's configuration is dramatically simpler for the standard reverse proxy + auto-HTTPS use case.

## Basic Caddyfile

```caddyfile
# /etc/caddy/Caddyfile

yourdomain.com {
    # Automatic HTTPS via Let's Encrypt — no certbot, no renewal cron
    # Caddy handles certificate issuance and renewal automatically

    # Reverse proxy all traffic to the Node.js app
    reverse_proxy localhost:3000

    # Security headers
    header {
        X-Frame-Options "SAMEORIGIN"
        X-Content-Type-Options "nosniff"
        Referrer-Policy "strict-origin-when-cross-origin"
        Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
        -Server  # Remove the Server header entirely
    }

    # Rate limiting (requires caddy-ratelimit module or configure at app level)

    # Log access
    log {
        output file /var/log/caddy/access.log
    }
}
```

## Installation

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

## Start and Enable

```bash
sudo systemctl enable caddy
sudo systemctl start caddy

# After editing Caddyfile
sudo systemctl reload caddy

# Verify it's working
curl -I https://yourdomain.com
```

## How Automatic HTTPS Works

Caddy uses the ACME protocol to request a certificate from Let's Encrypt automatically when it first receives a request for your domain. It:

1. Verifies domain ownership via HTTP-01 or TLS-ALPN-01 challenge
2. Obtains the certificate
3. Serves HTTPS immediately
4. Renews the certificate automatically before it expires (typically 30 days before the 90-day expiry)

No certbot, no cron job, no renewal failure risk from misconfigured automation.

## Caddy vs. Nginx

| Factor | Caddy | Nginx |
|---|---|---|
| **HTTPS setup** | Zero configuration — automatic | Manual certbot + renewal cron |
| **Config syntax** | Simple, readable | Verbose, error-prone |
| **Rate limiting** | Basic built-in; advanced via plugin | Mature, built-in |
| **Performance** | Excellent | Excellent |
| **Ecosystem** | Smaller, growing | Very large, extensive docs |
| **Advanced routing** | Good | More options |

**When to choose Caddy:** You want automatic HTTPS without managing Let's Encrypt renewal, and your routing requirements are straightforward (standard reverse proxy).

**When to choose Nginx:** You need advanced routing logic, an existing team familiar with Nginx, mature rate limiting DSL, or Nginx-specific features like `ngx_http_image_filter_module`.

## Related Documents

- [VPS Nginx](nginx.md)
- [VPS SSL/TLS](ssl-tls.md)
- [Security transport](../../../security/transport.md)
