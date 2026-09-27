---
kind: playbook
domain: guides
topics: [guides, deploy, vps]
authority: guidance
---

# How to Deploy to a VPS

**Prerequisites:** A hardened VPS from [How to Provision and Harden a VPS](provision-and-harden-vps.md).

**What you'll have at the end:** A live application running on the VPS, served over HTTPS.

---

## Step 1 — Install Runtime Dependencies on the Server

```bash
# Node.js 20 LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version   # v20.x.x
npm --version    # 10.x.x

# PM2 (if using PM2 — skip if using Docker)
sudo npm install -g pm2

# Git
sudo apt install -y git
```

---

## Step 2 — Choose PM2 or Docker

**PM2:** Simpler, runs Node.js directly on the host. Easier to manage for a single app. See [VPS PM2](../delivery/infrastructure/vps/pm2.md).

**Docker:** More consistent environments, better dependency isolation. Higher operational overhead. See [VPS Docker](../delivery/infrastructure/vps/docker.md).

**This guide continues with PM2.** For Docker, follow the Docker guide then resume from Step 5.

---

## Step 3 — Clone the Application

```bash
# On the server, as the deploy user
mkdir -p /home/deploy/app
cd /home/deploy/app
git clone <your-repo-url> .
```

If the repo is private, configure an SSH deploy key or personal access token.

---

## Step 4 — Set Environment Variables

```bash
nano /home/deploy/app/.env.production
```

Add all required environment variables. Verify them with the application's validation schema at startup.

---

## Step 5 — Install Dependencies and Build

```bash
cd /home/deploy/app
npm ci --omit=dev
npm run build
npx prisma migrate deploy
```

---

## Step 6 — Configure Reverse Proxy (Nginx or Caddy)

**Option A: Nginx**

```bash
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/app
# Paste config from infrastructure/vps/nginx.md (without the SSL block for now)
sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**Option B: Caddy**

```bash
# Install Caddy (see infrastructure/vps/caddy.md)
sudo nano /etc/caddy/Caddyfile
# Paste config from caddy.md with your domain
sudo systemctl reload caddy
# Caddy handles SSL automatically — skip Step 7
```

---

## Step 7 — Issue SSL Certificate (Nginx Only)

Follow [How to Setup SSL](setup-ssl.md):

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Verify renewal works
sudo certbot renew --dry-run
```

---

## Step 8 — Start the Application with PM2

```bash
cd /home/deploy/app
pm2 start ecosystem.config.js --env production
pm2 status  # All workers should be "online"
pm2 logs app --lines 50  # Check for startup errors

# Enable auto-restart on reboot
pm2 startup
# Run the command pm2 outputs
pm2 save
```

---

## Step 9 — Final Verification

```bash
# Check the app responds
curl -I https://yourdomain.com
# Should return: HTTP/2 200

# Check the health endpoint
curl https://yourdomain.com/api/health
# Should return: {"status":"ok","db":"ok"}

# Check PM2 status
pm2 status

# Check Nginx error log for any issues
sudo tail -20 /var/log/nginx/error.log
```

Open the live URL in a browser. Test a full login/signup flow.

---

## Deploying Updates

```bash
# Automated deploy script (run from CI or manually on the server)
cd /home/deploy/app
git pull origin main
npm ci --omit=dev
npm run build
npx prisma migrate deploy
pm2 reload app --update-env
```

See [VPS Zero-Downtime](../delivery/infrastructure/vps/zero-downtime.md) for zero-downtime options.

---

## Related Documents

- [How to provision and harden a VPS](provision-and-harden-vps.md)
- [How to setup SSL](setup-ssl.md)
- [VPS PM2](../delivery/infrastructure/vps/pm2.md)
- [Infrastructure CI/CD](../delivery/infrastructure/ci-cd.md)
