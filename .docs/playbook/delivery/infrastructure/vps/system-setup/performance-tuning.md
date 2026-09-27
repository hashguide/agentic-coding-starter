---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, system, setup, performance, tuning]
authority: guidance
---

# VPS Performance Tuning

Default Linux kernel parameters are tuned for general-purpose use, not sustained web traffic. These settings allow a single VPS to absorb traffic spikes and handle significantly more concurrent connections without scaling events.

## TCP Stack Tuning

```bash
sudo nano /etc/sysctl.d/99-performance.conf
```

```ini
# ============================================================
# Connection Queue Size
# ============================================================
# Maximum number of queued connection requests awaiting acceptance
net.core.somaxconn = 65535

# Increase the backlog queue for SYN connections
net.ipv4.tcp_max_syn_backlog = 65535

# ============================================================
# Time-Wait Socket Reuse
# ============================================================
# Allow TIME_WAIT sockets to be reused for new connections
# Safe on servers; do not enable on NAT devices
net.ipv4.tcp_tw_reuse = 1

# Reduce the time a socket stays in TIME_WAIT state
net.ipv4.tcp_fin_timeout = 15

# ============================================================
# Socket Buffers
# ============================================================
# Increase receive and send buffer sizes for higher throughput
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 87380 134217728
net.ipv4.tcp_wmem = 4096 65536 134217728

# Enable automatic buffer tuning
net.ipv4.tcp_moderate_rcvbuf = 1

# ============================================================
# Connection Keepalive
# ============================================================
# Start sending keepalive probes after 60 seconds of idle
net.ipv4.tcp_keepalive_time = 60
# Interval between probes
net.ipv4.tcp_keepalive_intvl = 10
# Number of probes before dropping the connection
net.ipv4.tcp_keepalive_probes = 5
```

Apply:

```bash
sudo sysctl -p /etc/sysctl.d/99-performance.conf
```

## File Descriptor Limits

Node.js and Nginx each hold an open file descriptor per connection. Default limits (~1024) are too low for production traffic.

```bash
sudo nano /etc/security/limits.conf
```

```
# Append at the end
*    soft    nofile    65535
*    hard    nofile    65535
root soft    nofile    65535
root hard    nofile    65535
```

```bash
sudo nano /etc/pam.d/common-session
```

Add the line:

```
session required pam_limits.so
```

Verify after re-login:

```bash
ulimit -n
# Should output: 65535
```

## Swap File

A swap file provides emergency memory overflow capacity and prevents OOM kills during memory spikes. On an SSD-backed VPS, a small swap file is acceptable:

```bash
# Create a 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Tune swappiness — how aggressively the kernel swaps
# Low values (10-20) mean the kernel uses swap only when nearly out of RAM
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.d/99-performance.conf
```

## Nginx Worker and Connection Tuning

```nginx
# /etc/nginx/nginx.conf
worker_processes auto;           # One worker per CPU core
worker_rlimit_nofile 65535;

events {
    worker_connections 4096;     # Max connections per worker
    use epoll;                   # Linux-optimized event model
    multi_accept on;             # Accept multiple connections per epoll event
}
```

## Verify Under Load

After applying all tuning:

```bash
# Check current connection count
ss -s

# Check open file descriptors per process
cat /proc/$(pgrep node)/limits | grep "Max open files"

# Check effective sysctl values
sysctl net.core.somaxconn net.ipv4.tcp_tw_reuse
```

## Related Documents

- [VPS Linux hardening](linux-hardening.md)
- [VPS Nginx](../nginx.md)
- [VPS PM2](../pm2.md)
