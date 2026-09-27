---
kind: playbook
domain: integrations
topics: [integrations, providers, anthropic, messages]
authority: guidance
---

# Anthropic Messages API

The Messages API is the primary interface for generating text with Claude models. This document covers a basic call, system prompts, streaming, and model selection.

## Basic Messages Call

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY! });

async function generateSummary(text: string): Promise<string> {
  const message = await anthropic.messages.create({
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 300,
    system: 'You are a concise assistant. Summarize the provided text in 2-3 sentences.',
    messages: [
      { role: 'user', content: `Summarize this:\n\n${text}` },
    ],
  });

  const content = message.content[0];
  return content.type === 'text' ? content.text : '';
}
```

## System Prompts

The `system` parameter in Claude's API is a top-level parameter (not a message with `role: 'system'`). It sets persistent instructions that apply to the entire conversation:

```typescript
const message = await anthropic.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  system: `
    You are a customer support agent for Acme Inc.
    Always respond professionally and empathetically.
    If you cannot answer a question, say so and offer to escalate.
    Never reveal internal pricing or discount structures.
  `,
  messages: [
    { role: 'user', content: userMessage },
  ],
});
```

## Streaming Response

For user-facing generation, stream the response to avoid making users wait for the full completion:

```typescript
// Server-Sent Events endpoint
app.post('/api/generate-stream', requireAuth, asyncHandler(async (req, res) => {
  const { prompt } = z.object({ prompt: z.string().max(5000) }).parse(req.body);

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');

  const stream = anthropic.messages.stream({
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 500,
    messages: [{ role: 'user', content: prompt }],
  });

  stream.on('text', (text) => {
    res.write(`data: ${JSON.stringify({ content: text })}\n\n`);
  });

  stream.on('finalMessage', () => {
    res.write('data: [DONE]\n\n');
    res.end();
  });
}));
```

## Model Selection

| Model | Capability | Cost (approx) | Best for |
|---|---|---|---|
| `claude-haiku-4-5-20251001` | Fast, capable | Lowest | High-volume, latency-sensitive tasks |
| `claude-sonnet-4-6` | Strong capability | Mid | Balanced quality/cost — most production tasks |
| `claude-opus-4-8` | Highest capability | Highest | Complex reasoning, analysis, long documents |

**Start with `claude-haiku-4-5-20251001`** for most tasks. It is fast and inexpensive. Upgrade to Sonnet when quality needs to improve, and to Opus only when Sonnet is insufficient.

## Token Cost Awareness

Monitor usage in production to prevent cost surprises:

```typescript
const message = await anthropic.messages.create({ ... });

// Log usage for cost tracking
console.log({
  inputTokens: message.usage.input_tokens,
  outputTokens: message.usage.output_tokens,
  estimatedCostUsd: (
    message.usage.input_tokens * 0.000001 + // Haiku input price
    message.usage.output_tokens * 0.000005   // Haiku output price
  ).toFixed(6),
});
```

Always set `max_tokens` to a reasonable ceiling. Without it, a response can run indefinitely.

## Related Documents

- [Anthropic overview](overview.md)
- [Anthropic MCP](mcp.md)
- [Anthropic testing](testing.md)
- [OpenAI completions (comparison)](../openai/completions.md)
