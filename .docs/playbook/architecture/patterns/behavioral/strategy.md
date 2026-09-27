---
kind: playbook
domain: architecture
topics: [architecture, patterns, behavioral, strategy]
authority: guidance
---

# Strategy Pattern

The Strategy pattern defines a family of algorithms or behaviors, encapsulates each one, and makes them interchangeable. The caller works with an interface; the concrete implementation is selected at runtime based on configuration or context. It is the behavioral equivalent of the Factory pattern.

## Definition

A strategy is an interface that defines the expected behavior. Multiple concrete implementations (strategies) provide different implementations of that behavior. The caller holds a reference to the strategy interface and can work with any implementation without knowing which one it has.

## Code Example

Swappable storage backends behind a common interface:

```typescript
interface StorageBackend {
  upload(key: string, data: Buffer, contentType: string): Promise<string>;
  getSignedUrl(key: string, expiresInSeconds: number): Promise<string>;
  delete(key: string): Promise<void>;
}

class S3StorageBackend implements StorageBackend {
  private client: S3Client;
  private bucket: string;

  constructor() {
    this.client = new S3Client({ region: process.env.AWS_REGION! });
    this.bucket = process.env.S3_BUCKET!;
  }

  async upload(key: string, data: Buffer, contentType: string): Promise<string> {
    await this.client.send(new PutObjectCommand({
      Bucket: this.bucket,
      Key: key,
      Body: data,
      ContentType: contentType,
    }));
    return key;
  }

  async getSignedUrl(key: string, expiresInSeconds: number): Promise<string> {
    return getSignedUrl(this.client, new GetObjectCommand({ Bucket: this.bucket, Key: key }), {
      expiresIn: expiresInSeconds,
    });
  }

  async delete(key: string): Promise<void> {
    await this.client.send(new DeleteObjectCommand({ Bucket: this.bucket, Key: key }));
  }
}

class LocalStorageBackend implements StorageBackend {
  private baseDir: string;

  constructor() {
    this.baseDir = process.env.LOCAL_STORAGE_DIR ?? './uploads';
  }

  async upload(key: string, data: Buffer, contentType: string): Promise<string> {
    await fs.writeFile(path.join(this.baseDir, key), data);
    return key;
  }

  async getSignedUrl(key: string, expiresInSeconds: number): Promise<string> {
    // Local dev: return a direct URL (no signing needed)
    return `/uploads/${key}`;
  }

  async delete(key: string): Promise<void> {
    await fs.unlink(path.join(this.baseDir, key));
  }
}

// Select strategy based on environment
function createStorageBackend(): StorageBackend {
  return process.env.NODE_ENV === 'production'
    ? new S3StorageBackend()
    : new LocalStorageBackend();
}

const storage = createStorageBackend();
const key = await storage.upload(`exports/${userId}.zip`, fileBuffer, 'application/zip');
const url = await storage.getSignedUrl(key, 3600);
```

## Where This Applies in the Playbook

- **Payment providers** — Stripe, Paddle, Lemon Squeezy behind a `PaymentProvider` interface. See [payments/provider-options.md](../../../business/payments/provider-options.md).
- **Email providers** — Resend, SendGrid, Postmark behind an `EmailSender` interface. See [email/provider-options.md](../../../integrations/email/provider-options.md).
- **Storage backends** — S3, local disk, GCS behind a `StorageBackend` interface (example above).

## Related Documents

- [Adapter pattern](../structural/adapter.md)
- [Factory pattern](../creational/factory.md)
- [Payment provider options](../../../business/payments/provider-options.md)
- [Email provider options](../../../integrations/email/provider-options.md)
