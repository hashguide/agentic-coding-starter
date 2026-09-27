---
kind: playbook
domain: integrations
topics: [integrations, providers, openai]
authority: guidance
---

# OpenAI Setup

OpenAI provides LLM APIs (GPT-4, GPT-4o, o1) for text generation, code assistance, embeddings, and multimodal tasks. This document covers account setup, API key generation, usage limits, and the server-only security rule.

## Account Setup

1. Create an account at platform.openai.com
2. Navigate to **API Keys** → **Create new secret key**
3. Name the key (e.g., "production-app" or "development")
4. Copy the key immediately — OpenAI shows it only once

## Usage Limits and Budget Alerts

Set spending limits before building production features:

**Settings → Billing → Usage limits**

- **Monthly spend limit:** Hard limit — API requests fail once this is reached
- **Email notification threshold:** Alert you before reaching the hard limit

For development: set a low hard limit ($10–20) to prevent runaway costs from bugs or prompt injection attacks.

For production: set a limit based on your expected volume × average cost per request × 2× safety factor.

## Environment Configuration

```
OPENAI_API_KEY=sk-...

# Store model names in config, not scattered through code
OPENAI_DEFAULT_MODEL=gpt-4o-mini
OPENAI_FALLBACK_MODEL=gpt-3.5-turbo
```

```typescript
import OpenAI from 'openai';

export const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!,
});
```

## Security

**OpenAI API keys exposed in client-side code are trivially extractable and will be abused.** Any key embedded in a browser bundle, mobile app binary, or served as a JavaScript variable is publicly accessible. Attackers actively scan GitHub and package archives for exposed API keys and will use them immediately.

All OpenAI API calls must be made server-side:
- The key lives only in server-side environment variables
- Client code calls your API endpoints, which proxy to OpenAI
- Never set the key as a `NEXT_PUBLIC_` variable or expose it in any client bundle

```typescript
// Safe: server-side route calls OpenAI
app.post('/api/generate', requireAuth, asyncHandler(async (req, res) => {
  const completion = await openai.chat.completions.create({ ... });
  res.json({ content: completion.choices[0].message.content });
}));

// Unsafe: never do this in client code
const openai = new OpenAI({ apiKey: 'sk-...' }); // In browser = exposed
```

## Related Documents

- [OpenAI completions](completions.md)
- [OpenAI assistants](assistants.md)
- [OpenAI testing](testing.md)
- [Anthropic setup](../anthropic/overview.md)
