---
kind: playbook
domain: integrations
topics: [integrations, providers, anthropic]
authority: guidance
---

# Anthropic Setup

Anthropic provides the Claude model family (Claude Opus, Sonnet, Haiku) for text generation, code assistance, and analysis. This document covers account setup, API key generation, rate limits, and the server-only security rule.

## Account Setup

1. Create an account at console.anthropic.com
2. Navigate to **API Keys** → **Create Key**
3. Name the key and set an optional expiry date
4. Copy the key immediately

## Rate Limits and Tiers

Anthropic uses a tier-based rate limit system:

| Tier | How to reach | Limits |
|---|---|---|
| **Free** | Default on new account | Low request/token limits; suitable for development |
| **Build** | Add a credit card + meet spend thresholds | Higher limits for most production workloads |
| **Scale** | Enterprise volume | Custom limits via Anthropic Sales |

Check current limits at: **Console → Rate limits**

Rate limits apply per model family:
- Input tokens per minute (TPM)
- Output tokens per minute
- Requests per minute (RPM)

## Environment Configuration

```
ANTHROPIC_API_KEY=sk-ant-...

# Store model names in config
ANTHROPIC_DEFAULT_MODEL=claude-haiku-4-5-20251001
ANTHROPIC_LARGE_MODEL=claude-sonnet-4-6
```

```typescript
import Anthropic from '@anthropic-ai/sdk';

export const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});
```

## Security

**Anthropic API keys must never appear in client bundles, `NEXT_PUBLIC_` variables, or any client-accessible location.** An API key in client-side code is fully exposed to anyone who opens browser DevTools or inspects the JavaScript bundle.

All Anthropic API calls are server-side:
- The key lives only in server-side environment variables
- Clients call your server endpoints; your server calls Anthropic
- In Next.js: never put `ANTHROPIC_API_KEY` in a `NEXT_PUBLIC_` variable
- In any React app: never import the `@anthropic-ai/sdk` in component files

```typescript
// Safe: server-side API route
// app/api/generate/route.ts (Next.js)
export async function POST(req: Request) {
  const { prompt } = await req.json();
  const message = await anthropic.messages.create({
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 500,
    messages: [{ role: 'user', content: prompt }],
  });
  return Response.json({ content: message.content[0] });
}
```

## Related Documents

- [Anthropic messages](messages.md)
- [Anthropic MCP](mcp.md)
- [Anthropic testing](testing.md)
- [OpenAI setup](../openai/overview.md)
