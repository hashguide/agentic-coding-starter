---
kind: playbook
domain: architecture
topics: [architecture, patterns, structural, adapter]
authority: guidance
---

# Adapter Pattern

The Adapter pattern wraps an incompatible interface to make it match the interface callers expect. It is a structural bridge: the underlying implementation stays unchanged, and the adapter translates between the expected interface and the actual one.

## Definition

An adapter is a class or function that sits between a caller and an incompatible dependency, translating the caller's calls into the interface the dependency understands, and translating the dependency's responses back into what the caller expects.

## Code Example

Adapting two different email providers to a common internal interface:

```typescript
// The internal interface callers work with
interface EmailSender {
  sendTransactional(params: {
    to: string;
    subject: string;
    html: string;
    text: string;
  }): Promise<{ messageId: string }>;
}

// Adapter for Resend
class ResendAdapter implements EmailSender {
  private client: Resend;

  constructor() {
    this.client = new Resend(process.env.RESEND_API_KEY!);
  }

  async sendTransactional(params: {
    to: string;
    subject: string;
    html: string;
    text: string;
  }): Promise<{ messageId: string }> {
    const result = await this.client.emails.send({
      from: 'noreply@yourdomain.com',
      to: params.to,
      subject: params.subject,
      html: params.html,
      text: params.text,
    });

    if (result.error) {
      throw new Error(`Resend error: ${result.error.message}`);
    }

    return { messageId: result.data!.id };
  }
}

// Adapter for SendGrid
class SendGridAdapter implements EmailSender {
  private client: sgMail.MailService;

  constructor() {
    this.client = sgMail;
    this.client.setApiKey(process.env.SENDGRID_API_KEY!);
  }

  async sendTransactional(params: {
    to: string;
    subject: string;
    html: string;
    text: string;
  }): Promise<{ messageId: string }> {
    const [response] = await this.client.send({
      to: params.to,
      from: 'noreply@yourdomain.com',
      subject: params.subject,
      html: params.html,
      text: params.text,
    });

    return { messageId: response.headers['x-message-id'] as string };
  }
}

// Callers never know which provider is in use
const emailSender: EmailSender = new ResendAdapter();
await emailSender.sendTransactional({ to, subject, html, text });
```

## When This Is the Right Tool

- You need to swap between multiple third-party providers without changing callers
- A third-party library has an interface that doesn't match your internal conventions
- You want to mock an external dependency in tests by swapping the adapter

## When to Use the Provider's SDK Directly

If you will always use a single provider and have no need for swapping, the adapter adds indirection without benefit. Use the SDK directly until you have a concrete reason to abstract it.

The adapter becomes worth adding when: you've already changed providers once, or when you want to test email sending without hitting a real email API.

## Related Documents

- [Strategy pattern](../behavioral/strategy.md)
- [Factory pattern](../creational/factory.md)
- [Email provider options](../../../integrations/email/provider-options.md)
