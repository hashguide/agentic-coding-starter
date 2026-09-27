---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, zero, downtime]
authority: guidance
---

# Zero-Downtime Deployment

On a single VPS, zero-downtime deployment requires either a rolling process restart or a blue-green switch between two running instances. Both approaches prevent users from hitting a period where no application process is accepting requests.

## Option 1: PM2 Cluster-Mode Rolling Restart (Recommended for Most)

In cluster mode, PM2 runs multiple Node.js processes. A reload restarts them one at a time — while one worker restarts, the others continue handling requests.

```bash
# Zero-downtime reload: workers restart one by one
pm2 reload app --update-env

# Verify all workers restarted successfully
pm2 status
pm2 logs app --lines 50
```

This requires:
- PM2 cluster mode (`exec_mode: 'cluster'`, `instances: 'max'` or a specific count)
- At least 2 worker processes (a single worker has a brief gap during restart)

**Deployment script:**

```bash
#!/bin/bash
set -e

cd /home/deploy/app

# Pull latest code
git pull origin main

# Install any new dependencies
npm ci --omit=dev

# Run database migrations
npx prisma migrate deploy

# Build the app
npm run build

# Zero-downtime restart
pm2 reload app --update-env

echo "Deploy complete"
```

## Option 2: Blue-Green Deployment

Blue-green runs two complete application instances (blue and green) and switches Nginx's upstream between them. The old instance continues serving requests until the new one is healthy.

```
Traffic → Nginx → Blue (active)
                   Green (idle)

Deploy:
1. Pull new code to Green
2. Start/restart Green
3. Healthcheck Green
4. Switch Nginx upstream to Green
5. Blue is now idle; stop Blue
```

**Nginx upstream configuration:**

```nginx
# /etc/nginx/conf.d/upstream.conf
upstream app_blue {
    server 127.0.0.1:3001;
}

upstream app_green {
    server 127.0.0.1:3002;
}
```

```nginx
# /etc/nginx/sites-available/app
server {
    # ...
    location / {
        # Switch this between app_blue and app_green
        proxy_pass http://app_green;
    }
}
```

**Blue-green deploy script:**

```bash
#!/bin/bash
set -e

ACTIVE=$(cat /etc/nginx/active-color 2>/dev/null || echo "blue")
if [ "$ACTIVE" = "blue" ]; then
    NEXT="green"
    NEXT_PORT=3002
else
    NEXT="blue"
    NEXT_PORT=3001
fi

echo "Deploying to $NEXT (port $NEXT_PORT)"

# Pull code and build
cd /home/deploy/app
git pull origin main
npm ci --omit=dev
npm run build

# Run migrations before switching
DATABASE_URL=$DATABASE_URL npx prisma migrate deploy

# Start the next instance
PORT=$NEXT_PORT pm2 start ecosystem.config.js --name "app-$NEXT" --env production

# Healthcheck
for i in {1..10}; do
    if curl -sf http://127.0.0.1:$NEXT_PORT/api/health; then
        break
    fi
    echo "Waiting for $NEXT to be healthy..."
    sleep 2
done

# Switch Nginx upstream
sed -i "s/proxy_pass http:\/\/app_$ACTIVE/proxy_pass http:\/\/app_$NEXT/" /etc/nginx/sites-available/app
sudo nginx -t && sudo systemctl reload nginx

# Record new active color
echo $NEXT | sudo tee /etc/nginx/active-color

# Stop old instance
pm2 stop "app-$ACTIVE"
pm2 delete "app-$ACTIVE"

echo "Switched to $NEXT"
```

## Choosing Between Them

| Factor | PM2 Rolling Reload | Blue-Green |
|---|---|---|
| Complexity | Low | Medium |
| Requires 2x resources | No | Yes |
| True zero-downtime | Yes (with ≥2 workers) | Yes |
| Instant rollback | No | Yes — reload the old instance |
| Database migration safety | Requires backward-compatible migrations | Same |

**Default recommendation:** PM2 rolling reload for single-VPS deployments. Blue-green when instant rollback is a hard requirement or when you have spare capacity.

## Related Documents

- [VPS PM2](pm2.md)
- [VPS Nginx](nginx.md)
- [Infrastructure CI/CD](../ci-cd.md)
