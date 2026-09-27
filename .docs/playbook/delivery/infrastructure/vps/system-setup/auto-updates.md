---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, system, setup, auto, updates]
authority: guidance
---

# Automatic Security Updates

Unpatched packages are the most common initial compromise vector on VPS servers. Automate security updates without exception — waiting to apply patches manually means any window of delay is a window of exposure.

## Install and Configure unattended-upgrades

```bash
sudo apt install unattended-upgrades apt-listchanges -y
```

Enable automatic updates:

```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
# Select "Yes" when prompted
```

Configure the upgrade behavior:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Ensure these settings are active (uncomment or add):

```
// Apply security updates from all security origins
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

// Remove old unused packages after upgrade
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";

// Automatically reboot when required (e.g., after a kernel update)
// Set to the time of your lowest traffic window
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-WithUsers "false";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";

// Send email on upgrade failures
Unattended-Upgrade::Mail "ops@yourdomain.com";
Unattended-Upgrade::MailReport "on-change";

// Enable verbose logging
Unattended-Upgrade::Verbose "false";
Unattended-Upgrade::Debug "false";
```

## Configure Update Schedule

```bash
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

This runs daily: update the package list, download available security updates, and apply them.

## Reboot Policy

**Allow automatic reboots during a defined low-traffic window.** Kernel updates require a reboot to take effect — a server running an unpatched kernel after an update is still vulnerable.

Set `Automatic-Reboot-Time` to your application's lowest traffic period (e.g., 3:00 AM in your primary user timezone).

**If you cannot tolerate automatic reboots** (e.g., stateful workloads), use the `needrestart` package to detect when a reboot is required and alert instead:

```bash
sudo apt install needrestart -y

# Check after manual updates
sudo needrestart -r l  # list mode — shows what needs restarting
```

## Verify Configuration is Active

```bash
# Manually trigger a dry-run to confirm it works
sudo unattended-upgrade --dry-run --debug

# Check the last run log
sudo cat /var/log/unattended-upgrades/unattended-upgrades.log
```

## Security

Unpatched packages are the most common entry point for server compromises. CVEs for popular packages (OpenSSL, the Linux kernel, OpenSSH) are actively scanned for and exploited within hours to days of public disclosure. The patching window between disclosure and exploitation is short — automation removes human delay from the equation.

## Related Documents

- [VPS initial setup](initial-setup.md)
- [VPS Linux hardening](linux-hardening.md)
- [VPS monitoring agents](monitoring-agents.md)
