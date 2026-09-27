---
kind: playbook
domain: access
topics: [access, models, license, based]
authority: guidance
---

# License-Based Access

Digital products sold as a one-time purchase — templates, plugins, tools, downloadable software — don't need ongoing user accounts. Access is granted by a license key, validated at the point of use, without a persistent subscription or session.

## License Key Generation

License keys must be:
- **Cryptographically random:** Generated from a CSPRNG, not sequential or guessable
- **Cryptographically signed:** The server can verify authenticity without looking up the key in a database (for offline validation) or with a fast database lookup (for online validation)
- **Non-sequential:** An attacker cannot guess the next valid key from observed valid keys

```typescript
import { createHmac, randomBytes } from 'crypto';

const LICENSE_SECRET = process.env.LICENSE_SIGNING_SECRET!; // At least 32 bytes, never rotated

// Generate a signed license key
function generateLicenseKey(params: {
  orderId: string;
  productId: string;
  email: string;
}): string {
  // Random prefix — prevents brute force
  const random = randomBytes(8).toString('hex');

  // Payload — what we're signing
  const payload = `${params.orderId}:${params.productId}:${params.email}:${random}`;

  // HMAC signature
  const signature = createHmac('sha256', LICENSE_SECRET)
    .update(payload)
    .digest('hex')
    .substring(0, 16); // Use first 16 chars for readability

  // Format: XXXX-XXXX-XXXX-XXXX
  const combined = (random + signature).toUpperCase();
  return combined.match(/.{1,4}/g)!.join('-');
}

// Validate a license key (online — lookup in DB)
async function validateLicenseKey(key: string): Promise<{ valid: boolean; orderId?: string }> {
  const normalized = key.replace(/-/g, '').toLowerCase();

  const license = await db.license.findUnique({
    where: { key: normalized },
    select: { orderId: true, revokedAt: true, activationCount: true, maxActivations: true },
  });

  if (!license || license.revokedAt) return { valid: false };
  if (license.activationCount >= license.maxActivations) return { valid: false };

  return { valid: true, orderId: license.orderId };
}
```

## Offline vs. Online Validation

| Validation | How it works | When to use |
|---|---|---|
| **Online** | Key validated against a database on every use | Web apps, SaaS tools, APIs |
| **Offline** | Key is self-verifying via cryptographic signature | Desktop apps, CLI tools, environments without internet |

For offline validation, the license key itself encodes the permissions (product, tier, expiry) and includes a signature the application verifies locally using a public key embedded in the binary.

## Security

**License keys must be cryptographically signed and non-guessable.** Sequential or short keys (8 hex characters ≈ 4 billion possibilities) are trivially enumerated by an automated brute-force attack against the validation endpoint.

**Rate-limit the validation endpoint.** Even with strong keys, a brute-force attack that can make 1,000,000 validation requests per minute will eventually find valid keys. Rate-limit validation requests per IP: 10 attempts per minute is generous for legitimate users and devastating for brute-force attackers.

**Revoke keys when orders are refunded.** A fully automated refund flow must include key revocation: mark the license `revokedAt` in the database; the next validation returns `valid: false`.

```typescript
async function revokeKey(orderId: string): Promise<void> {
  await db.license.updateMany({
    where: { orderId },
    data: { revokedAt: new Date() },
  });
}
```

## Related Documents

- [Entitlement verification](entitlement-verification.md)
- [Access model selection](access-model-selection.md)
- [Payments provider options](../../business/payments/provider-options.md)
