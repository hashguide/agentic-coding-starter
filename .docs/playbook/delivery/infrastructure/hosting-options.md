---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, hosting, options]
authority: guidance
---

# Hosting Options

Choosing where to host your SaaS affects operational complexity, cost trajectory, and how much infrastructure you own versus delegate. The right choice at launch is often wrong at scale — and that's fine, as long as you choose intentionally.

## Options Comparison

| Provider | Operational complexity | Cost at small scale | Cost at growth | Control/Flexibility |
|---|---|---|---|---|
| **Vercel** | Very low | Free tier generous | High (function invocations + bandwidth) | Low — opinionated, Next.js-centric |
| **Railway** | Low | Free tier + pay-as-you-go | Predictable, mid-range | Medium — Docker-based, flexible |
| **Fly.io** | Medium | Free allowance, then usage | Predictable, competitive | High — containers, global edge |
| **AWS** | High | Cheap at very low usage | Cheapest at high scale if well-configured | Very high — full control |
| **VPS (Hetzner/DigitalOcean)** | High | Fixed, cheapest per resource | Cheapest predictable cost at growth | Full — you own everything |

## When to Use Each

**Vercel:** Ideal for pure frontend/Next.js apps with minimal server-side work. Excellent developer experience. Becomes expensive when you have high function invocation volume or need persistent processes (background workers, long connections). Not suitable as a primary host if you need WebSockets, cron jobs, or persistent worker queues.

**Railway:** Good first deployment platform. Supports Dockerized apps, databases, Redis — everything you need for a full-stack SaaS. Lower operational overhead than a VPS. Fair pricing with no surprises. Use when you want managed infrastructure without committing to AWS or running your own servers.

**Fly.io:** Strong choice when you need global edge deployment or fine-grained container control. Slightly higher learning curve than Railway but more capable. Good for latency-sensitive workloads.

**AWS (ECS, Lambda, RDS, etc.):** Right choice at scale or when your team has existing AWS expertise. Wrong choice for a solo builder at launch — operational complexity is high and the default bill shock risk is real.

**VPS (Hetzner, DigitalOcean, Vultr, Linode):** Best cost efficiency for predictable workloads. Full control over the stack. Requires you to manage servers, security patches, deployments, and monitoring. Right for builders comfortable with server administration who want maximum control at minimum cost. See `infrastructure/vps/` for the full VPS setup sequence.

## Recommended Default

**Start on Railway or Fly.io.** Both support the full stack (app server, PostgreSQL, Redis, background workers), deploy from Docker or from source, and handle TLS automatically. The operational overhead is low enough for a solo builder and the cost at early scale is predictable.

**Migrate to a VPS or AWS** when your monthly cost on managed platforms becomes materially higher than the engineering time required to operate your own infrastructure. For most products, this threshold is somewhere between $500–$2000/month in hosting spend.

## Related Documents

- [VPS overview](vps/overview.md)
- [Infrastructure CI/CD](ci-cd.md)
- [Non-functional requirements](../../product/non-functional-requirements.md)
