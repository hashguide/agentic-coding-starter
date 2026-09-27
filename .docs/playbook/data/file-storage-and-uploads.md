---
kind: playbook
domain: data
topics: [files, uploads, object-storage, signed-url, processing, data, file, storage, and]
authority: guidance
---
# File Storage and Uploads

## Application-server upload
Use for small files and modest traffic when server-side inspection or transformation is required before storage. Consider memory/disk limits and request timeouts.

## Direct object-storage upload
Use for large files or scale-sensitive uploads. The application issues a short-lived signed upload authorization; clients upload directly; the server records metadata and processing state.

## Local/device storage
Use when the product is local-first or files should remain on-device. Define backup, sync, export and migration behavior.

For every file flow define allowed types, maximum size, content/signature validation, ownership, access controls, storage key strategy, retention/deletion, malware/content scanning needs, processing jobs, preview generation, download authorization and orphan cleanup.
