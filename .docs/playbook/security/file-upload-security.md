---
kind: playbook
domain: security
topics: [file-uploads, object-storage, malware]
authority: guidance
---
# File Upload Security

Treat uploaded files and metadata as untrusted input.

## Define before implementation
- allowed content types and business purpose
- maximum file and aggregate sizes
- number/rate limits
- storage location and access model
- retention/deletion behavior
- whether files are rendered, transformed, parsed, executed, indexed, or sent to another service

## Validation
Validate server-side. Check file signature/magic bytes when type matters, sanitize or replace filenames, generate storage keys independently of user paths, and reject unsupported content. Client validation is convenience only.

## Storage and delivery
Prefer object storage or an isolated file service for untrusted uploads. Use private-by-default access for non-public files and time-bounded signed access where appropriate. Set safe response headers and content disposition for downloads.

## Processing
Run risky parsing/transcoding/scanning outside the web request and with constrained permissions/resources. Apply malware scanning or content inspection when threat, product surface, or compliance requires it. Protect image/document processors from resource exhaustion.

## Authorization
Every read, transform, delete, or share action must enforce ownership/tenant permissions server-side. Do not infer authorization from an opaque URL alone.

## Review
Test spoofed extensions/MIME types, oversized/decompression-bomb inputs, path traversal, SVG/HTML/scriptable content, parser failures, duplicate uploads, expired links, cross-tenant access, deletion/retention, and logging of sensitive metadata.
