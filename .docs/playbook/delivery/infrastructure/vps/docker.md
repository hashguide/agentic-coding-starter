---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, docker]
authority: guidance
---

# Docker on VPS

Running the application in Docker containers on a VPS provides consistent environments, simplified dependency management, and a clear deployment artifact. The tradeoff is added operational overhead compared to running Node.js directly with PM2.

## Multi-Stage Dockerfile

A multi-stage build keeps the production image small by discarding dev dependencies and build tooling:

```dockerfile
# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine AS production

WORKDIR /app

# Create a non-root user before running the app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma

# Run database migrations at startup
# (alternatively, run migrations as a separate pre-deploy job)
RUN npx prisma generate

# Switch to non-root user
USER appuser

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

## docker-compose.yml

```yaml
version: '3.9'

services:
  app:
    build: .
    restart: unless-stopped
    ports:
      - '3000:3000'
    environment:
      NODE_ENV: production
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: redis://redis:6379
    depends_on:
      - redis
    networks:
      - internal

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - internal
    # Redis is NOT exposed to the host — only accessible within the internal network
    # ports:
    #   - '6379:6379'  ← Never do this in production

volumes:
  redis_data:

networks:
  internal:
    driver: bridge
```

## Container Registry

Build the image in CI and push to a registry. Pull from the registry on the server instead of building on the server (avoids installing build tools on production VPS):

```bash
# GitHub Container Registry
docker build -t ghcr.io/org/app:$GIT_SHA .
docker push ghcr.io/org/app:$GIT_SHA

# On VPS deploy
docker pull ghcr.io/org/app:$GIT_SHA
docker-compose up -d --no-deps --pull always app
```

## Deployment on VPS

```bash
# Pull and restart without downtime (Docker Compose rolling update)
docker-compose pull app
docker-compose up -d --no-deps app

# View logs
docker-compose logs -f app

# Check running containers
docker-compose ps
```

## Read-Only Root Filesystem

Where the application allows it, run containers with a read-only root filesystem to limit what an attacker can modify if they achieve code execution:

```yaml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp              # Allow writes to /tmp only
```

This requires the application to write nothing to the filesystem except through explicit tmpfs mounts or mounted volumes.

## Security

Containers running as root are a privilege escalation risk. If a container process is compromised, root inside the container maps to root on the host if the container runtime is not properly isolated. Always specify a non-root `USER` in the Dockerfile.

The `USER appuser` directive above is non-negotiable in any production Dockerfile.

## Related Documents

- [VPS PM2](pm2.md)
- [VPS Nginx](nginx.md)
- [Infrastructure CI/CD](../ci-cd.md)
- [VPS zero-downtime deployment](zero-downtime.md)
