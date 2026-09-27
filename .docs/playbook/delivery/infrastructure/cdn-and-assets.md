---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, cdn, and, assets]
authority: guidance
---

# CDN and Static Assets

Serving static assets (JavaScript bundles, CSS, images, fonts) through a CDN reduces latency, offloads bandwidth from your origin server, and improves Core Web Vitals scores. This is non-optional for production SaaS.

## The Core Rule

All static assets are served via CDN, not from your origin server. Your origin handles API requests and server-rendered pages. Assets are cached at the edge.

## CDN Options

| CDN | How to use | Best for |
|---|---|---|
| **Vercel / Netlify built-in** | Automatic with hosting platform | Apps already on these platforms |
| **Cloudflare** | Proxy your domain; caches automatically | Any origin; add WAF for free |
| **AWS CloudFront** | Integrated with S3 and ALB | AWS-hosted apps |
| **Bunny.net** | Pull zone pointing at your origin | Cost-efficient alternative |

**Recommended:** Cloudflare. It's free for most traffic volumes, adds DDoS protection and WAF, and requires only a DNS change. No CDN-specific code in your app.

## Content-Hash Fingerprinting

Cache-bust assets by embedding a content hash in the filename, not via query strings (which some CDNs and proxies strip).

Modern build tools (Vite, Next.js, webpack) do this automatically:

```
/assets/main-4a7f3b2c.js   ← cache indefinitely
/assets/main.js             ← never use this in production
```

With content-hash filenames, set `Cache-Control: public, max-age=31536000, immutable` — the asset is permanently cacheable because its URL changes when its content changes.

```typescript
// Express: serve build output with long cache headers
import express from 'express';
import path from 'path';

app.use(
  '/assets',
  express.static(path.join(__dirname, 'dist/assets'), {
    maxAge: '1y',
    immutable: true,
  })
);
```

## Cache Header Recommendations by Asset Type

| Asset type | Cache-Control | Notes |
|---|---|---|
| JS/CSS with content hash | `public, max-age=31536000, immutable` | Permanent — URL changes on update |
| Images with content hash | `public, max-age=31536000, immutable` | Permanent |
| Fonts | `public, max-age=31536000, immutable` | Fonts rarely change |
| HTML pages | `no-cache` or `max-age=0, must-revalidate` | Always fetch latest |
| API responses | `no-store` | Never cache sensitive data |
| Service worker (`sw.js`) | `max-age=0, no-cache` | Must always be current |

## Compression

Enable Brotli and gzip compression at the CDN layer or origin:

**Nginx:**
```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml;
gzip_comp_level 6;

brotli on;
brotli_comp_level 6;
brotli_types text/plain text/css application/json application/javascript text/xml;
```

**Cloudflare:** Enable Brotli in the dashboard under **Speed → Optimization → Content Optimization**. Cloudflare applies Brotli automatically to eligible content types.

Compression reduces JS bundle transfer sizes by 60–80%, directly improving LCP and Time to Interactive.

## User-Uploaded Files

Store user-uploaded files in object storage (S3, Cloudflare R2, DigitalOcean Spaces), not on the application server's filesystem. Serve them via a CDN or signed URL:

```typescript
import { S3Client, PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3 = new S3Client({ region: process.env.AWS_REGION! });

// Upload
await s3.send(new PutObjectCommand({
  Bucket: process.env.S3_BUCKET!,
  Key: `uploads/${userId}/${filename}`,
  Body: fileBuffer,
  ContentType: mimeType,
}));

// Generate time-limited signed URL for download
const url = await getSignedUrl(
  s3,
  new GetObjectCommand({ Bucket: process.env.S3_BUCKET!, Key: key }),
  { expiresIn: 3600 }
);
```

## Related Documents

- [Infrastructure hosting options](hosting-options.md)
- [VPS Nginx configuration](vps/nginx.md)
- [Marketing SEO](../../growth/marketing/seo.md)
