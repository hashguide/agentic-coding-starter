---
kind: playbook
domain: architecture
topics: [web, mobile, desktop, cli, api, library, product-surface, architecture, product, surfaces]
authority: guidance
---
# Product Surface Selection

A product may expose one or several delivery surfaces. Add each because it serves a user or integration need.

## Web
Fits broad distribution, shared/collaborative state, rapid deployment and linkable content. Consider browser capabilities, connectivity, SEO/public routes and session security.

## Mobile native/hybrid
Fits camera/location/notifications, high-frequency personal use, offline needs or app-store distribution. Consider permissions, store review, background execution, device storage and release cadence.

## Desktop
Fits local files, OS integration, heavy workflows, offline/private processing or persistent local services. Consider packaging, updates, signing, local security and cross-platform behavior.

## CLI
Fits developer/ops workflows, automation, composability and headless environments. Design stable commands, exit codes, noninteractive mode, config precedence and machine-readable output.

## API / SDK / library
Fits programmable capability. Prioritize contract stability, versioning, authentication, quotas, examples, compatibility and developer experience.

Define a primary surface for the first release and the shared domain/service boundaries needed by any secondary surfaces.
