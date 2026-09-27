---
kind: playbook
domain: guides
topics: [guides, setup, openai]
authority: guidance
---

# How to Setup OpenAI

**What you'll have at the end:** A working, cost-controlled OpenAI integration with streaming and rate limit handling.

---

## Step 1 — Create an API Key

1. Sign in at platform.openai.com
2. Navigate to **API Keys → Create new secret key**
3. Name the key (e.g., "Production" or "Development")
4. Set a usage limit: in the key settings or via **Billing → Usage limits**
5. Copy the key immediately

```
OPENAI_API_KEY=sk-...
```

The key must never appear in client-side code or environment variables accessible by the frontend. See [integrations/openai/overview.md](../integrations/providers/openai/overview.md).

---

## Step 2 — Set a Usage Budget Alert

1. platform.openai.com → **Billing → Usage limits**
2. Set a **Soft limit** at 80% of your budget (triggers an email)
3. Set a **Hard limit** at 100% of your budget (cuts off API calls)

For development: $20–50/month hard limit per developer.
For production: set based on projected usage + 2x safety buffer.

---

## Step 3 — Make a First Successful Call

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY! });

async function testCall() {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    max_tokens: 100,
    messages: [
      { role: 'system', content: 'You are a helpful assistant.' },
      { role: 'user', content: 'Say "hello world" and nothing else.' },
    ],
  });

  console.log(completion.choices[0].message.content);
  console.log('Tokens used:', completion.usage);
}

testCall();
```

Run with:

```bash
OPENAI_API_KEY=sk-... npx ts-node src/test-openai.ts
```

Expected output: `hello world` followed by token usage.

---

## Step 4 — Implement Streaming for User-Facing Generation

Streaming makes the UI feel faster — content appears as it generates rather than all at once after a delay.

```typescript
// Express SSE endpoint
app.post('/api/generate', requireAuth, async (req, res) => {
  const { prompt } = z.object({ prompt: z.string().max(2000) }).parse(req.body);

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  const stream = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    max_tokens: 500,
    stream: true,
    messages: [{ role: 'user', content: prompt }],
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content;
    if (content) {
      res.write(`data: ${JSON.stringify({ content })}\n\n`);
    }
  }

  res.write('data: [DONE]\n\n');
  res.end();
});
```

---

## Step 5 — Handle Rate Limit Errors

OpenAI returns HTTP 429 when rate limits are exceeded. Handle it gracefully:

```typescript
import OpenAI from 'openai';

async function callWithRetry(params: OpenAI.ChatCompletionCreateParamsNonStreaming) {
  let lastError: unknown;

  for (let attempt = 0; attempt < 3; attempt++) {
    try {
      return await openai.chat.completions.create(params);
    } catch (err) {
      if (err instanceof OpenAI.RateLimitError) {
        const waitMs = 1000 * Math.pow(2, attempt); // Exponential backoff
        await new Promise((resolve) => setTimeout(resolve, waitMs));
        lastError = err;
      } else {
        throw err; // Don't retry non-rate-limit errors
      }
    }
  }

  throw lastError;
}
```

---

## Verify

- [ ] API key is server-side only (not in any frontend bundle or NEXT_PUBLIC_ variable)
- [ ] Budget limits are set in the OpenAI dashboard
- [ ] First test call succeeds and returns a response
- [ ] Streaming endpoint delivers content incrementally (verify in browser Network tab)
- [ ] Rate limit error triggers retry with backoff (test with a simulated 429)

---

## Related Documents

- [Integrations OpenAI overview](../integrations/providers/openai/overview.md)
- [Integrations OpenAI completions](../integrations/providers/openai/completions.md)
- [Integrations OpenAI testing](../integrations/providers/openai/testing.md)
