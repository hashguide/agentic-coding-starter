---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, nginx]
authority: guidance
---

# Nginx Reverse Proxy

Nginx sits in front of the Node.js application server, handling SSL termination, request routing, rate limiting, and static asset serving. It is not optional — exposing a Node.js server directly on port 443 bypasses every security header and performance optimization that Nginx provides.

## Basic Reverse Proxy Configuration

```nginx
# /etc/nginx/sites-available/app
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    # Redirect all HTTP traffic to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL certificate paths (managed by certbot)
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # Hide Nginx version from error pages and Server header
    server_tokens off;

    # Limit request body size — prevents trivial request body abuse
    client_max_body_size 10m;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header X-XSS-Protection "0" always;  # Disabled — modern browsers ignore it; CSP is the replacement
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

    # Rate limiting zone (defined in http block — see below)
    limit_req zone=api burst=20 nodelay;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Serve static assets directly from Nginx with long cache headers
    location /assets/ {
        alias /home/deploy/app/dist/assets/;
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
}
```

## HTTP Block Rate Limiting

Define rate limit zones in the main `nginx.conf` http block:

```nginx
# /etc/nginx/nginx.conf — inside the http {} block
http {
    # ...existing config...

    # Rate limit: 10 requests/second per IP, 10MB zone for tracking
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

    # Stricter limit for auth endpoints
    limit_req_zone $binary_remote_addr zone=auth:10m rate=1r/s;
}
```

Apply the stricter auth limit to login/signup routes:

```nginx
location ~ ^/api/auth/ {
    limit_req zone=auth burst=5 nodelay;
    proxy_pass http://127.0.0.1:3000;
    # ...proxy headers...
}
```

## Enable and Verify

```bash
sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled/
sudo nginx -t  # Test configuration syntax
sudo systemctl reload nginx
```

## Security

`server_tokens off` removes the Nginx version number from HTTP response headers and default error pages. Attackers use version information to identify known CVEs. Concealing it adds friction.

`client_max_body_size` defaults to 1m and controls the maximum accepted request body. Without this, any client can send an arbitrarily large request body, consuming memory and bandwidth. Set it to the minimum required for your largest legitimate upload.

## Related Documents

- [VPS SSL/TLS](ssl-tls.md)
- [VPS Caddy](caddy.md)
- [VPS PM2](pm2.md)
- [Security transport](../../../security/transport.md)
