---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, system, setup, linux, hardening]
authority: guidance
---

# Linux Hardening

The default Linux kernel parameters are not tuned for security. These sysctl settings close well-known network-level attack vectors without affecting normal application operation.

## Apply sysctl Hardening

Create a dedicated file for security sysctl settings (do not modify `/etc/sysctl.conf` directly):

```bash
sudo nano /etc/sysctl.d/99-hardening.conf
```

```ini
# ============================================================
# IP Forwarding
# ============================================================
# Disable IPv4 packet forwarding — this server is not a router
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# ============================================================
# SYN Flood Protection
# ============================================================
# Enable SYN cookies — prevents SYN flood DoS attacks by
# validating the three-way handshake before allocating memory
net.ipv4.tcp_syncookies = 1

# Increase the backlog queue for connections waiting to complete
net.ipv4.tcp_max_syn_backlog = 2048

# Reduce SYN retries for failed connections
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2

# ============================================================
# ICMP and Redirect Restrictions
# ============================================================
# Ignore ICMP redirects — these can be used to manipulate routing tables
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Don't send ICMP redirects (this server is not a router)
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Ignore bogus ICMP error responses
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Log martian packets (packets with impossible source addresses)
net.ipv4.conf.all.log_martians = 1

# ============================================================
# Source Address Verification (Spoofing Prevention)
# ============================================================
# Strict reverse path filter — drops packets whose source address
# doesn't route back through the interface they arrived on
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# ============================================================
# Shared Memory Protection
# ============================================================
# Restrict access to /proc — prevents information leakage between users
kernel.dmesg_restrict = 1
kernel.perf_event_paranoid = 3

# ============================================================
# Core Dump Restrictions
# ============================================================
# Disable core dumps for SUID binaries — prevents leaking privileged memory
fs.suid_dumpable = 0

# ============================================================
# Randomize Memory Layout
# ============================================================
# Enable Address Space Layout Randomization (ASLR) — makes memory exploits harder
kernel.randomize_va_space = 2
```

Apply the settings:

```bash
sudo sysctl -p /etc/sysctl.d/99-hardening.conf

# Verify they took effect
sudo sysctl net.ipv4.tcp_syncookies
# Should output: net.ipv4.tcp_syncookies = 1
```

## What Each Setting Closes

| Setting | Attack vector closed |
|---|---|
| `ip_forward = 0` | Prevents the server from routing packets between networks |
| `tcp_syncookies = 1` | SYN flood DoS attacks that exhaust connection queue memory |
| `accept_redirects = 0` | ICMP redirect attacks that reroute traffic through attacker-controlled hosts |
| `rp_filter = 1` | IP spoofing — packets arriving with forged source addresses |
| `log_martians = 1` | Makes spoofed traffic visible in logs |
| `randomize_va_space = 2` | Buffer overflow / heap spray exploits targeting fixed memory addresses |
| `dmesg_restrict = 1` | Information leakage via kernel logs readable by unprivileged users |

## Restrict /tmp Execution

If `/tmp` is a separate partition or you use a tmpfs, mount it with `noexec` to prevent attackers from dropping and executing binaries there:

```bash
# Check if /tmp is a separate partition
mount | grep /tmp

# If it is, add noexec to /etc/fstab:
# tmpfs /tmp tmpfs defaults,rw,nosuid,nodev,noexec,relatime 0 0

# Apply without rebooting:
sudo mount -o remount,noexec /tmp
```

## Verify Settings Persist on Reboot

```bash
sudo reboot
# After reconnecting:
sudo sysctl net.ipv4.tcp_syncookies net.ipv4.conf.all.accept_redirects kernel.randomize_va_space
```

## Related Documents

- [VPS initial setup](initial-setup.md)
- [VPS SSH hardening](ssh-hardening.md)
- [VPS performance tuning](performance-tuning.md)
