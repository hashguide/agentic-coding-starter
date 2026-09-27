---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, system, setup, firewall]
authority: guidance
---

# VPS Firewall Configuration

UFW (Uncomplicated Firewall) is the standard tool for managing iptables rules on Ubuntu/Debian. The configuration principle is default deny on inbound traffic, then allowlist only the ports your application requires.

## Every Open Port Is Attack Surface

An open port not required by your application is a vector for exploitation. The goal is the minimum set of open ports that allows your application to function.

## Required Open Ports for a Typical SaaS

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH (management access) |
| 80 | TCP | HTTP (redirect to HTTPS) |
| 443 | TCP | HTTPS (application traffic) |

If you change your SSH port (see [SSH hardening](ssh-hardening.md)), allow that port instead of 22.

## Configuration Commands

```bash
# Confirm UFW is installed
sudo apt install ufw -y

# Set default policies: deny all inbound, allow all outbound
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (must do this BEFORE enabling the firewall or you'll lock yourself out)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable the firewall
sudo ufw enable

# Verify the rules
sudo ufw status verbose
```

Expected output:

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
```

## Rate Limiting on SSH

UFW supports rate limiting to block IPs that attempt too many connections in a short window:

```bash
# Limit SSH connections — blocks IPs that make 6+ connections in 30 seconds
sudo ufw limit 22/tcp
```

This is a lightweight layer of protection. Install fail2ban (see [SSH hardening](ssh-hardening.md)) for more comprehensive brute force protection.

## Application-Specific Ports

If your stack has services that must communicate between processes on the same server (PostgreSQL on port 5432, Redis on port 6379), do NOT open those ports to the public internet. They should only be reachable locally (`127.0.0.1`) or within a private network.

```bash
# Never run these — they expose the database to the public internet
# sudo ufw allow 5432/tcp  ← WRONG
# sudo ufw allow 6379/tcp  ← WRONG
```

If you need database access for administration from your workstation, use an SSH tunnel instead:

```bash
# Local port 5433 → remote PostgreSQL port 5432 via SSH tunnel
ssh -L 5433:localhost:5432 deploy@<server-ip> -N
```

## Security

Default deny everything, then allowlist only what's needed. Every open port is attack surface. A database port open to the public internet is a critical vulnerability regardless of what authentication the database itself requires.

## Related Documents

- [VPS initial setup](initial-setup.md)
- [VPS SSH hardening](ssh-hardening.md)
- [VPS Nginx](../nginx.md)
