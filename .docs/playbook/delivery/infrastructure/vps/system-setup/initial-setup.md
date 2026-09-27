---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, system, setup, initial]
authority: guidance
---

# VPS Initial Setup

The first steps after provisioning a VPS establish a secure foundation. A fresh VPS with default settings is unsafe — root login is enabled, password authentication is active, and there is no firewall. This must be fixed before anything else runs on the server.

## Step 1 — Log in as Root

```bash
ssh root@<server-ip>
```

## Step 2 — Create a Non-Root Sudo User

```bash
adduser deploy
usermod -aG sudo deploy
```

Choose a username like `deploy` that signals its purpose. Avoid `admin` or `ubuntu` (common targets for brute force).

## Step 3 — Copy Your SSH Public Key to the New User

From your **local machine**:

```bash
ssh-copy-id deploy@<server-ip>
```

If `ssh-copy-id` is not available:

```bash
# On local machine — get your public key
cat ~/.ssh/id_ed25519.pub

# On server, as root
mkdir -p /home/deploy/.ssh
echo "<paste-your-public-key>" >> /home/deploy/.ssh/authorized_keys
chmod 700 /home/deploy/.ssh
chmod 600 /home/deploy/.ssh/authorized_keys
chown -R deploy:deploy /home/deploy/.ssh
```

## Step 4 — Verify SSH Access as Non-Root User

**Before disabling root login**, open a second terminal and verify login works:

```bash
ssh deploy@<server-ip>
sudo whoami  # Should print: root
```

Do not close your root session until this is confirmed.

## Step 5 — Disable Password Authentication and Root Login

Edit `/etc/ssh/sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

Set these values (add if not present, modify if existing):

```
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

## Step 6 — Verify Root Login is Disabled

From a third terminal (do not close your existing sessions yet):

```bash
ssh root@<server-ip>
# Should be refused or rejected with: Permission denied (publickey)
```

## Step 7 — Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

## Security

**A fresh VPS with root login and password authentication enabled is compromised within hours of provisioning.** Automated bots continuously scan public IP ranges and attempt credential stuffing on SSH port 22. Disabling password authentication (forcing key-based auth) and disabling root login removes the two most common attack vectors.

Do not proceed to any other setup step until these changes are in place and verified.

## Next Steps

Continue with:
1. [Firewall configuration](firewall.md)
2. [SSH hardening](ssh-hardening.md)
3. [Linux hardening](linux-hardening.md)

## Related Documents

- [VPS overview](../overview.md)
- [SSH hardening](ssh-hardening.md)
- [How to provision and harden a VPS](../../../../guides/provision-and-harden-vps.md)
