---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, pm2]
authority: guidance
---

# PM2 Process Management

PM2 is a process manager for Node.js applications. It handles auto-restart on crash, cluster mode for multi-core utilization, log rotation, and startup configuration so the app restarts automatically after a VPS reboot.

## Installation

```bash
npm install -g pm2
```

## Basic ecosystem.config.js

Define your application in an ecosystem config file committed to the repository:

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'app',
      script: './dist/server.js',

      // Cluster mode: run one instance per CPU core
      instances: 'max',
      exec_mode: 'cluster',

      // Auto-restart on crash
      autorestart: true,
      watch: false,           // Never watch in production — triggers constant restarts
      max_memory_restart: '512M',

      // Environment variables
      env_production: {
        NODE_ENV: 'production',
        PORT: 3000,
      },

      // Log configuration
      out_file: '/var/log/app/out.log',
      error_file: '/var/log/app/error.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      merge_logs: true,       // Merge cluster instance logs into one file
    },
  ],
};
```

## Starting and Managing the App

```bash
# Start the app in production mode
pm2 start ecosystem.config.js --env production

# Zero-downtime restart (for deploys)
pm2 reload app --update-env

# Hard restart (applies ecosystem config changes)
pm2 restart app --update-env

# Stop
pm2 stop app

# View live logs
pm2 logs app

# Status dashboard
pm2 status
```

## Startup on Reboot

```bash
# Generate the startup command for the current OS and user
pm2 startup

# Run the command that pm2 outputs (it will look like this)
sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u deploy --hp /home/deploy

# Save current process list so it's restored on reboot
pm2 save
```

Verify auto-restart works:

```bash
sudo reboot
# After reconnecting:
pm2 status  # App should be running
```

## Log Rotation

Without log rotation, app logs grow without bound and eventually fill the disk.

```bash
pm2 install pm2-logrotate

# Configure log rotation
pm2 set pm2-logrotate:max_size 50M
pm2 set pm2-logrotate:retain 7       # Keep 7 rotated log files
pm2 set pm2-logrotate:compress true  # Gzip rotated logs
pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'  # Daily at midnight
```

## Cluster Mode

Cluster mode runs one Node.js process per CPU core. Node.js is single-threaded per process; cluster mode uses all available cores without requiring any changes to application code.

```bash
# Check how many cores are being used
pm2 status
# "instances" column shows the number of running workers

# Scale up or down without restarting
pm2 scale app 4     # Run exactly 4 instances
pm2 scale app max   # Run one per core
```

All cluster instances share the same PM2 process ID and are managed as a unit via `pm2 reload app`.

## Related Documents

- [VPS zero-downtime deployment](zero-downtime.md)
- [Infrastructure CI/CD](../ci-cd.md)
- [VPS Nginx](nginx.md)
