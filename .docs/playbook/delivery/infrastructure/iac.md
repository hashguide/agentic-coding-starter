---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, iac]
authority: guidance
---

# Infrastructure as Code

Infrastructure as Code (IaC) means every piece of infrastructure — servers, databases, DNS records, firewall rules, CDN configuration — is defined in version-controlled files, not configured manually through a web console.

**The rule:** No manual production infrastructure changes through a web console. If it's not in code, it doesn't exist reliably.

## Why This Matters

Manual console changes:
- Are invisible to the team (no diff, no review, no audit trail)
- Cannot be reproduced reliably when rebuilding (disaster recovery, new environments)
- Create drift between environments — staging differs from production in unknown ways
- Disappear with team member turnover ("only Sarah knew how that was configured")

## Options

| Tool | Language/syntax | Learning curve | Ecosystem maturity | Best for |
|---|---|---|---|---|
| **Terraform** | HCL (HashiCorp Config Language) | Medium | Very high — provider for everything | Teams with existing Terraform knowledge, any provider |
| **Pulumi** | TypeScript / Python / Go | Medium (if you know the language) | High | Developers who prefer real programming languages over DSLs |
| **SST (Ion)** | TypeScript | Low (for AWS) | Medium — AWS/Cloudflare focused | AWS-centric apps; excellent DX for Node.js/TypeScript teams |
| **Kamal** | YAML | Low | Medium — Docker-focused | VPS deployments; not full infrastructure provisioning |

## Recommended Default

**Terraform** for teams with cloud experience or complex multi-service infrastructure. **SST** for AWS-focused Node.js/TypeScript teams who want the lowest cognitive overhead. **Pulumi** if you dislike DSLs and want to write real TypeScript.

For a solo builder on a single VPS, a bash-based provisioning script (captured in the repo) is a pragmatic starting point before committing to a full IaC tool.

## Basic Terraform Example

```hcl
# main.tf — DigitalOcean droplet
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }

  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "digitalocean" {
  token = var.do_token
}

resource "digitalocean_droplet" "app" {
  name   = "saas-app-prod"
  region = "nyc3"
  size   = "s-2vcpu-4gb"
  image  = "ubuntu-22-04-x64"
  ssh_keys = [data.digitalocean_ssh_key.default.id]

  tags = ["app", "production"]
}

resource "digitalocean_firewall" "app" {
  name = "saas-app-firewall"
  droplet_ids = [digitalocean_droplet.app.id]

  inbound_rule {
    protocol   = "tcp"
    port_range = "22"
    source_addresses = ["0.0.0.0/0"]
  }

  inbound_rule {
    protocol   = "tcp"
    port_range = "443"
    source_addresses = ["0.0.0.0/0"]
  }

  outbound_rule {
    protocol              = "tcp"
    port_range            = "all"
    destination_addresses = ["0.0.0.0/0"]
  }
}

variable "do_token" {}
```

## State Management

IaC tools maintain a state file tracking what's deployed. Store state remotely (S3, Terraform Cloud, GCS), never locally — a local state file lost means losing the ability to manage existing infrastructure.

Always lock state during operations to prevent concurrent modifications from two team members running `terraform apply` simultaneously.

## Related Documents

- [Infrastructure hosting options](hosting-options.md)
- [VPS overview](vps/overview.md)
- [Environments production](../environments/production.md)
