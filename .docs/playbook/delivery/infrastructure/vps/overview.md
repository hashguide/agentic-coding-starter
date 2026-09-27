---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps]
authority: guidance
---

# VPS Provider Options

A Virtual Private Server (VPS) gives you a dedicated Linux environment that you configure and operate yourself. Compared to platform-as-a-service hosting, a VPS is significantly cheaper for stable workloads but requires more operational skill.

## Provider Comparison

| Provider | Entry price (approx) | Performance/dollar | Datacenter locations | Ease of use |
|---|---|---|---|---|
| **Hetzner** | €4–6/mo (EU) | Excellent — highest perf/dollar | EU (Germany, Finland), US (Virginia) | Good — clean console, simple pricing |
| **DigitalOcean** | $6/mo | Good | US, EU, APAC, broad | Excellent — best DX, extensive docs |
| **Vultr** | $6/mo | Good | US, EU, APAC, broad | Good — similar to DigitalOcean |
| **Linode (Akamai)** | $5/mo | Good | US, EU, APAC | Good — reliable, developer-friendly |

## Recommended Default

**Hetzner for EU-based products or cost-sensitive workloads.** Their CX22 (2 vCPU, 4 GB RAM) starts at €4/month — roughly the same as DigitalOcean's 1 GB tier. The value proposition is unmatched for a solo builder.

**DigitalOcean for US-based products or if you want the best developer experience.** Their documentation, managed databases, and Spaces (object storage) make the full stack easy. Slightly more expensive than Hetzner.

Both offer one-click backups, floating IPs, and monitoring integrations. Choose based on primary datacenter location for your users.

## Entry Server Sizing

A 2 vCPU / 4 GB RAM instance handles the early stage of most SaaS products (Node.js app, PostgreSQL, Redis, Nginx, PM2). Size up when:
- CPU usage consistently exceeds 70% during normal traffic
- Memory usage exceeds 80% (OOM kills)
- Disk I/O becomes a bottleneck

## Related Documents

- [VPS initial setup](system-setup/initial-setup.md)
- [VPS firewall](system-setup/firewall.md)
- [VPS SSH hardening](system-setup/ssh-hardening.md)
- [Infrastructure hosting options](../hosting-options.md)
- [How to provision and harden a VPS](../../../guides/provision-and-harden-vps.md)
