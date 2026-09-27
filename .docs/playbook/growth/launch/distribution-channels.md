---
kind: playbook
domain: growth
topics: [growth, launch, distribution, channels]
authority: guidance
---

# Distribution Channels

Launch is the process of making a product obtainable, not merely publishing a website. The right channel depends on the product type: app stores, package registries, marketplaces, direct downloads, enterprise deployment, physical fulfillment, and the open web each impose different release and support obligations.

## Channel Comparison

| Channel | Best for | Main advantage | Main constraint |
|---|---|---|---|
| Direct web access | Web apps, content, communities, stores | Immediate releases and direct customer relationship | You own discovery, hosting, and support |
| Mobile app stores | Native mobile apps | Trusted installation and built-in discovery/payment | Review rules, fees, signing, staged rollout |
| Desktop stores | Consumer desktop apps and games | Updates and purchase trust | Platform-specific packaging and policies |
| Direct download | Desktop apps, utilities, games | Pricing and customer relationship remain under your control | Code signing, update security, reputation warnings |
| Package registry | Libraries, SDKs, CLIs | Fits developer workflows | Semantic versioning and supply-chain security |
| Browser extension marketplace | Browser-integrated tools | Permission and update mechanism provided | Review delays and strict permission disclosure |
| Third-party marketplace | Plugins, templates, integrations | Audience already has intent | Platform dependency and revenue share |
| Managed enterprise rollout | Internal and B2B software | Controlled access and configuration | Procurement, identity, device management, support |

## Release Artifact Checklist

For every channel, define:

- The signed, reproducible artifact and where it is retained
- Version number, release notes, and compatibility range
- Review or approval lead time
- Staged rollout, rollback, and emergency removal procedure
- Update mechanism and minimum supported version
- Ownership of the publisher account and recovery credentials
- Support URL, privacy disclosures, permissions, and data-use labels

Do not schedule a public announcement for the same moment you submit to a channel with unpredictable review time. Obtain approval first or plan a deliberate buffer.

## Multi-Channel Releases

Choose one canonical version and release record, then produce channel-specific artifacts from it. A mobile app, desktop client, public API, and web backend may deploy independently; publish a compatibility matrix and preserve backward-compatible server behavior while older clients remain installed.

Track channel-specific activation and retention, not just downloads. A registry install, app-store download, or marketplace listing view is not proof that the user reached the core value.

## Recommended Default

Begin with the channel where the target user already expects to acquire this kind of product. Add a second channel only when it provides distinct reach or required platform capability. Preserve a direct website as the canonical source for documentation, support, policies, and status even when a third party controls installation.

## Security

- Require MFA and least-privilege access on publisher accounts.
- Store signing keys outside the repository and rotate them through a documented process.
- Pin CI actions and protect release branches and tags.
- Publish checksums or signatures for direct downloads.
- Test update authenticity and rollback before the first production release.
- Treat package, extension, and app-store metadata as production assets subject to review.

## Related Documents

- [Product type selection](../../product/product-type-selection.md)
- [Release strategy](../../delivery/version-control/release-strategy.md)
- [CI/CD](../../delivery/infrastructure/ci-cd.md)
- [Launch checklist](checklist.md)
- [Developer documentation](../../quality/documentation/dev-docs.md)
