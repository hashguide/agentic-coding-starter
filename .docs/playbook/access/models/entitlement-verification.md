---
kind: playbook
domain: access
topics: [access, models, entitlement, verification]
authority: guidance
---

# Entitlement Verification

Entitlement verification answers "did this person legitimately purchase access to this resource?" after the purchase has occurred. This covers signed download links, re-delivery flows, and device/activation binding for digital products.

## Signed, Expiring Download URLs

A permanent, unsigned download URL will be shared, indexed, and accessed by people who didn't pay. All download links must be time-limited and cryptographically signed.

```typescript
import { createHmac } from 'crypto';

const DOWNLOAD_SECRET = process.env.DOWNLOAD_SIGNING_SECRET!;

// Generate a signed, expiring download URL
function generateDownloadUrl(params: {
  orderId: string;
  productId: string;
  expiresInSeconds: number;
}): string {
  const expiresAt = Math.floor(Date.now() / 1000) + params.expiresInSeconds;
  const payload = `${params.orderId}:${params.productId}:${expiresAt}`;

  const signature = createHmac('sha256', DOWNLOAD_SECRET)
    .update(payload)
    .digest('hex');

  const url = new URL(`${process.env.APP_URL}/downloads/${params.productId}`);
  url.searchParams.set('order', params.orderId);
  url.searchParams.set('expires', expiresAt.toString());
  url.searchParams.set('sig', signature);

  return url.toString();
}

// Validate a download URL
function validateDownloadUrl(params: {
  orderId: string;
  productId: string;
  expiresAt: number;
  signature: string;
}): boolean {
  if (params.expiresAt < Math.floor(Date.now() / 1000)) return false; // Expired

  const payload = `${params.orderId}:${params.productId}:${params.expiresAt}`;
  const expectedSig = createHmac('sha256', DOWNLOAD_SECRET)
    .update(payload)
    .digest('hex');

  // Constant-time comparison to prevent timing attacks
  return crypto.timingSafeEqual(
    Buffer.from(params.signature),
    Buffer.from(expectedSig),
  );
}

// Download handler
app.get('/downloads/:productId', async (req, res) => {
  const { order: orderId, expires, sig } = req.query;

  const valid = validateDownloadUrl({
    orderId: orderId as string,
    productId: req.params.productId,
    expiresAt: parseInt(expires as string),
    signature: sig as string,
  });

  if (!valid) return res.status(403).json({ code: 'FORBIDDEN' });

  // Stream the file to the response
  const fileUrl = await getProductFileUrl(req.params.productId);
  res.redirect(fileUrl);
});
```

**Typical expiry:** 24–72 hours for initial download links; shorter (15 minutes) for re-generated links.

## Re-Delivery Flow

Users lose their download links. A re-delivery flow verifies purchase without requiring an active session.

```typescript
// POST /api/redelivery — verify by email + order ID
app.post('/api/redelivery', asyncHandler(async (req, res) => {
  const { email, orderId } = z.object({
    email: z.string().email(),
    orderId: z.string(),
  }).parse(req.body);

  const order = await db.order.findFirst({
    where: { id: orderId, customerEmail: email },
    include: { products: true },
  });

  // Always return the same response to prevent order ID enumeration
  if (!order || order.refundedAt) {
    // Wait the same amount of time as a successful lookup to prevent timing attacks
    await new Promise((r) => setTimeout(r, 50));
    res.json({ message: 'If that order exists, download links have been sent.' });
    return;
  }

  // Send fresh download links by email
  await emailService.sendTransactional({
    to: email,
    subject: 'Your download links',
    html: redeliveryEmailTemplate({ order }),
    text: 'Download links re-sent.',
  });

  res.json({ message: 'If that order exists, download links have been sent.' });
}));
```

## Device/Activation Binding

For software products, limit the number of simultaneous activations to reduce unauthorized redistribution while not being overly restrictive to legitimate users.

```typescript
async function activateLicense(key: string, deviceId: string): Promise<'activated' | 'limit_reached'> {
  const license = await db.license.findUnique({
    where: { key },
    include: { activations: true },
  });

  if (!license || license.revokedAt) return 'limit_reached';

  // Already activated on this device
  const existingActivation = license.activations.find((a) => a.deviceId === deviceId);
  if (existingActivation) return 'activated';

  // Check activation limit
  if (license.activations.length >= license.maxActivations) return 'limit_reached';

  await db.licenseActivation.create({
    data: { licenseId: license.id, deviceId, activatedAt: new Date() },
  });

  return 'activated';
}
```

**Recommendation:** Set `maxActivations` generously for individual buyers (2–5 devices) — over-restriction generates more support tickets than under-restriction generates lost sales.

## Security

**Permanent, unsigned download URLs will be shared and abused.** A link posted to a forum, shared on Discord, or indexed by a search engine is effectively a public download for the entire internet. Time-limited, signed URLs ensure that even a shared link expires within hours, not years.

## Related Documents

- [License-based access](license-based-access.md)
- [Access model selection](access-model-selection.md)
- [Payments provider options](../../business/payments/provider-options.md)
