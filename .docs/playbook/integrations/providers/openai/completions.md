---
kind: playbook
domain: integrations
topics: [integrations, providers, openai, completions]
authority: guidance
---

# OpenAI Chat Completions

The Chat Completions API is the primary interface for generating text with GPT models. This document covers a basic completion call, key parameters, streaming, and token cost awareness.

## Basic Completion

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY! });

async function generateSummary(text: string): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [
      {
        role: 'system',
        content: 'You are a helpful assistant that summarizes text concisely.',
      },
      {
        role: 'user',
        content: `Summarize the following in 2-3 sentences:\n\n${text}`,
      },
    ],
    max_tokens: 200,
    temperature: 0.3,
  });

  return completion.choices[0].message.content ?? '';
}
```

## Key Parameters

| Parameter | Range | Effect |
|---|---|---|
| `temperature` | 0.0–2.0 | Controls randomness. 0 = deterministic (best for structured tasks, fact extraction). 1.0 = balanced. > 1.5 = very creative/random. |
| `max_tokens` | 1–model max | Maximum tokens in the response. Set this to avoid runaway costs. Budget 4× what you expect for safety. |
| `top_p` | 0.0–1.0 | Alternative to temperature. Use one or the other, not both. |
| `presence_penalty` | -2.0–2.0 | Positive values discourage repetition of already-used topics. |
| `frequency_penalty` | -2.0–2.0 | Positive values discourage repeating the same words/phrases. |

## Model Selection Tradeoffs

| Model | Capability | Cost (approximate) | Best for |
|---|---|---|---|
| `gpt-4o` | Best quality | ~$5/1M tokens | Complex reasoning, high-stakes generation |
| `gpt-4o-mini` | Very good quality | ~$0.15/1M tokens | Most production use cases — best value |
| `o1` | Best at math/coding | High | Mathematical reasoning, complex logic |
| `gpt-3.5-turbo` | Good | ~$0.50/1M tokens | High-volume, cost-sensitive tasks |

**Start with `gpt-4o-mini`.** It handles most production tasks at a fraction of `gpt-4o`'s cost.

## Streaming Response

For user-facing generation, stream the response so the user sees text appearing progressively rather than waiting for the full completion:

```typescript
// Server-Sent Events endpoint for streaming
app.post('/api/generate-stream', requireAuth, asyncHandler(async (req, res) => {
  const { prompt } = z.object({ prompt: z.string().max(2000) }).parse(req.body);

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  const stream = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [{ role: 'user', content: prompt }],
    max_tokens: 500,
    stream: true,
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content;
    if (content) {
      res.write(`data: ${JSON.stringify({ content })}\n\n`);
    }
  }

  res.write('data: [DONE]\n\n');
  res.end();
}));
```

## Token Cost Estimation

Token cost = (input tokens + output tokens) × price per token

Rough estimates:
- 1 token ≈ 4 characters or ¾ of a word
- A typical paragraph (100 words) ≈ 130 tokens
- A page of text (500 words) ≈ 650 tokens

Use `tiktoken` to count tokens before sending to estimate cost:

```typescript
import { encoding_for_model } from 'tiktoken';

function estimateTokens(text: string, model = 'gpt-4o-mini'): number {
  const enc = encoding_for_model(model);
  return enc.encode(text).length;
}
```

Always set a `max_tokens` limit. Without it, a long response can cost far more than expected.

## Related Documents

- [OpenAI overview](overview.md)
- [OpenAI assistants](assistants.md)
- [OpenAI testing](testing.md)
- [Anthropic messages](../anthropic/messages.md)
