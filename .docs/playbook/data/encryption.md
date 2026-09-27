---
kind: playbook
domain: data
topics: [data, encryption]
authority: guidance
---

# Database Encryption

Database encryption operates at three levels: disk (at-rest encryption), transport (TLS for connections), and field (application-level encryption for specific columns). Each level addresses a different attack vector.

## At-Rest Encryption

Encrypts the database files on disk. If an attacker obtains the physical storage media or a backup file, they cannot read the data without the encryption key.

**Managed databases:** Most managed Postgres providers (Neon, Supabase, RDS, Railway) enable at-rest encryption by default, typically using AES-256. Verify in your provider's security documentation.

**Self-hosted:** Enable Linux filesystem-level encryption (LUKS) or use Postgres's native transparent data encryption if your version supports it. As of 2024, standard Postgres does not include built-in TDE — use pgCrypto for field-level encryption or a patched distribution.

```bash
# Verify LUKS encryption on a self-hosted VPS (if enabled at provision time)
lsblk -o NAME,FSTYPE,MOUNTPOINT,SIZE
# Look for dm-crypt in the output
```

## In-Transit TLS

All database connections must use TLS. An unencrypted connection allows anyone on the same network path to read all queries and responses.

```typescript
// Prisma — enforce TLS in DATABASE_URL
// postgresql://user:pass@host:5432/db?sslmode=require

// Direct pg connection
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: {
    rejectUnauthorized: true, // Verify the server's TLS certificate
    // ca: fs.readFileSync('/path/to/ca.crt').toString(), // For self-signed certs
  },
});
```

**Never use `sslmode=disable` or `rejectUnauthorized: false` in production.** These bypass TLS verification and allow man-in-the-middle attacks.

## Field-Level Encryption

For highly sensitive columns (SSNs, government IDs, financial account numbers, medical data), encrypt at the application layer before writing to the database. Even if someone reads the database rows directly, they cannot access the plaintext.

```typescript
import { createCipheriv, createDecipheriv, randomBytes } from 'crypto';

const ALGORITHM = 'aes-256-gcm';
const KEY = Buffer.from(process.env.FIELD_ENCRYPTION_KEY!, 'hex'); // 32 bytes, hex-encoded

function encryptField(plaintext: string): string {
  const iv = randomBytes(12); // 96-bit IV for GCM
  const cipher = createCipheriv(ALGORITHM, KEY, iv);
  const encrypted = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);
  const authTag = cipher.getAuthTag();
  // Store IV + authTag + ciphertext together
  return Buffer.concat([iv, authTag, encrypted]).toString('base64');
}

function decryptField(encryptedBase64: string): string {
  const buf = Buffer.from(encryptedBase64, 'base64');
  const iv = buf.subarray(0, 12);
  const authTag = buf.subarray(12, 28);
  const encrypted = buf.subarray(28);
  const decipher = createDecipheriv(ALGORITHM, KEY, iv);
  decipher.setAuthTag(authTag);
  return decipher.update(encrypted) + decipher.final('utf8');
}

// Usage
await db.user.update({
  where: { id: userId },
  data: {
    taxId: encryptField(req.body.taxId), // Stored encrypted in DB
  },
});

const user = await db.user.findUnique({ where: { id: userId } });
const taxId = decryptField(user.taxId); // Decrypted in application
```

## Security — Key Management

**Encryption without key management is false security.** If the encryption key is stored alongside the encrypted data (same database, same server, same environment variable file), encrypting is theater — anyone who can read the data can also read the key.

**Key management requirements:**
1. **Key separation:** The field encryption key must be stored separately from the encrypted data. Use a secrets manager (AWS Secrets Manager, HashiCorp Vault, Doppler) — not a `.env` file on the same server.
2. **Key rotation policy:** Define how often keys rotate (annually at minimum) and what happens to data encrypted with old keys (re-encrypt on access, or batch re-encrypt).
3. **Envelope encryption:** A master key (stored in a KMS like AWS KMS) encrypts a data encryption key (DEK). The DEK encrypts the actual data. Rotating the master key does not require re-encrypting all data — only the DEK is re-wrapped.
4. **Test rotation before launch:** Run the key rotation process in staging before it becomes a production emergency.

## Related Documents

- [Database backups](backups.md)
- [Security secrets](../security/secrets.md)
- [Security data privacy](../security/data-privacy.md)
- [Compliance GDPR](../compliance/gdpr.md)
