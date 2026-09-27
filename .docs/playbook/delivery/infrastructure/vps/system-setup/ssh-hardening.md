---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, system, setup, ssh, hardening]
authority: guidance
---

# SSH Hardening

After disabling password authentication and root login in [initial setup](initial-setup.md), additional sshd_config hardening and fail2ban installation further reduce the attack surface on SSH.

## sshd_config Directives

Edit `/etc/ssh/sshd_config` and ensure these values are set:

```bash
sudo nano /etc/ssh/sshd_config
```

```
# Disable password-based authentication (should already be set from initial-setup)
PasswordAuthentication no
PermitRootLogin no

# Limit authentication attempts per connection
MaxAuthTries 3

# Limit concurrent unauthenticated connections
MaxStartups 10:30:60

# Set a short timeout for incomplete logins
LoginGraceTime 30

# Disable unused authentication methods
PermitEmptyPasswords no
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no

# Restrict which users can log in via SSH
AllowUsers deploy

# Use only modern, secure key exchange algorithms
KexAlgorithms curve25519-sha256,diffie-hellman-group16-sha512

# Disable X11 forwarding if not needed
X11Forwarding no

# Disable TCP forwarding if not needed (remove if you use SSH tunnels)
# AllowTcpForwarding no
```

Apply the changes:

```bash
sudo systemctl restart sshd
```

## Optional: Non-Standard SSH Port

Moving SSH from port 22 to a non-standard port (e.g., 2222) reduces log noise from automated scanners. It does not meaningfully increase security against targeted attackers but reduces background noise.

If you change the port:
1. Update UFW to allow the new port first: `sudo ufw allow 2222/tcp`
2. Remove the old port: `sudo ufw delete allow 22/tcp`
3. Update your `~/.ssh/config` or always use `ssh -p 2222 deploy@<server-ip>`

```
# /etc/ssh/sshd_config
Port 2222
```

## Install and Configure fail2ban

fail2ban monitors log files and bans IP addresses that show signs of brute force activity (repeated failed authentication attempts). It is essential on any publicly-accessible SSH port.

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Create a local configuration override (never edit the default `.conf` files directly — they are overwritten on package updates):

```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
# Ban for 1 hour
bantime = 3600
# Look back 10 minutes
findtime = 600
# Allow 5 failures before banning
maxretry = 5
# Use systemd backend
backend = systemd

[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
maxretry = 3
bantime = 86400
```

```bash
sudo systemctl restart fail2ban

# Verify fail2ban is running and the sshd jail is active
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

## Verify the Configuration

After all changes:

```bash
# Check sshd config for syntax errors
sudo sshd -t

# Confirm SSH is listening on the expected port
sudo ss -tlnp | grep sshd

# Test login in a new terminal before closing existing sessions
ssh -p <port> deploy@<server-ip>
```

## Security

fail2ban is essential on any publicly-accessible SSH port. Without it, bots continuously attempt credential stuffing attacks — generating thousands of failed authentication attempts per day. Even with key-based auth enabled, this background noise consumes log space, makes anomaly detection harder, and puts unnecessary load on the SSH daemon.

## Related Documents

- [VPS initial setup](initial-setup.md)
- [VPS firewall](firewall.md)
- [VPS Linux hardening](linux-hardening.md)
