---
kind: playbook
domain: guides
topics: [guides, provision, and, harden, vps]
authority: guidance
---

# How to Provision and Harden a VPS

**What you'll have at the end:** A fresh VPS that is secured, hardened, and ready for application deployment.

**Time estimate:** 30–45 minutes on first run.

---

## Step 1 — Provision the Server

1. Log in to your VPS provider (Hetzner, DigitalOcean, etc.)
2. Create a new server:
   - Ubuntu 22.04 LTS
   - 2 vCPU / 4 GB RAM minimum
   - Select the datacenter region closest to your users
3. Add your SSH public key during provisioning (or paste it in)
4. Note the server's IP address

**Verify:** `ping <server-ip>` responds.

---

## Step 2 — Initial Setup

Follow [infrastructure/vps/system-setup/initial-setup.md](../delivery/infrastructure/vps/system-setup/initial-setup.md):

```bash
# Log in as root
ssh root@<server-ip>

# Create non-root sudo user
adduser deploy
usermod -aG sudo deploy

# Copy SSH key to new user (from your local machine)
ssh-copy-id deploy@<server-ip>

# Open second terminal and verify login works
ssh deploy@<server-ip>
sudo whoami  # Must print: root

# Disable password auth and root login
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
# Set: PermitRootLogin no
sudo systemctl restart sshd
```

**Verify:** `ssh root@<server-ip>` is refused. `ssh deploy@<server-ip>` succeeds.

---

## Step 3 — Firewall

Follow [infrastructure/vps/system-setup/firewall.md](../delivery/infrastructure/vps/system-setup/firewall.md):

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw limit 22/tcp
sudo ufw enable
sudo ufw status verbose
```

**Verify:** Output shows ports 22, 80, 443 open. All others denied.

---

## Step 4 — SSH Hardening

Follow [infrastructure/vps/system-setup/ssh-hardening.md](../delivery/infrastructure/vps/system-setup/ssh-hardening.md):

```bash
sudo nano /etc/ssh/sshd_config
# Set: MaxAuthTries 3
# Set: LoginGraceTime 30
# Set: AllowUsers deploy
# Set: X11Forwarding no
sudo systemctl restart sshd

# Install fail2ban
sudo apt install fail2ban -y
sudo nano /etc/fail2ban/jail.local
# Add sshd jail config (see ssh-hardening.md)
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

**Verify:** `sudo fail2ban-client status sshd` shows the jail is active.

---

## Step 5 — Linux Hardening

Follow [infrastructure/vps/system-setup/linux-hardening.md](../delivery/infrastructure/vps/system-setup/linux-hardening.md):

```bash
sudo nano /etc/sysctl.d/99-hardening.conf
# Paste all sysctl settings from linux-hardening.md
sudo sysctl -p /etc/sysctl.d/99-hardening.conf
```

**Verify:** `sudo sysctl net.ipv4.tcp_syncookies` outputs `1`.

---

## Step 6 — Performance Tuning

Follow [infrastructure/vps/system-setup/performance-tuning.md](../delivery/infrastructure/vps/system-setup/performance-tuning.md):

```bash
sudo nano /etc/sysctl.d/99-performance.conf
# Paste TCP tuning settings
sudo sysctl -p /etc/sysctl.d/99-performance.conf

# File descriptor limits
sudo nano /etc/security/limits.conf
# Add: * soft nofile 65535 / * hard nofile 65535

# Swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Verify:** `swapon --show` shows the 2G swap active.

---

## Step 7 — Automatic Security Updates

Follow [infrastructure/vps/system-setup/auto-updates.md](../delivery/infrastructure/vps/system-setup/auto-updates.md):

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
# Enable Automatic-Reboot true, set low-traffic reboot time

# Dry-run test
sudo unattended-upgrade --dry-run --debug
```

**Verify:** Dry-run completes without errors. Reboot time is configured.

---

## Step 8 — Monitoring Agent

Follow [infrastructure/vps/system-setup/monitoring-agents.md](../delivery/infrastructure/vps/system-setup/monitoring-agents.md):

```bash
curl https://get.netdata.cloud/kickstart.sh > /tmp/netdata-kickstart.sh
sudo bash /tmp/netdata-kickstart.sh --nightly-channel
sudo systemctl enable netdata
sudo systemctl start netdata
```

**Verify:** `sudo systemctl status netdata` shows active. Dashboard accessible via SSH tunnel on port 19999.

---

## Final Verification

```bash
# All ports except 22, 80, 443 should be closed
sudo ufw status

# SSH hardening
sudo sshd -t  # No output = no config errors
sudo fail2ban-client status sshd

# Performance tuning
swapon --show
ulimit -n  # Should be 65535 after re-login

# Auto-updates
sudo systemctl status unattended-upgrades
```

The server is now ready for application deployment. Continue with [How to Deploy to VPS](deploy-vps.md).

---

## Related Documents

- [VPS overview](../delivery/infrastructure/vps/overview.md)
- [How to deploy VPS](deploy-vps.md)
