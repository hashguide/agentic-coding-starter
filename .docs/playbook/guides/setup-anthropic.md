---
kind: playbook
domain: guides
topics: [guides, setup, anthropic]
authority: guidance
---

# How to Setup Anthropic

**What you'll have at the end:** A working Anthropic integration with Claude, including streaming and optional MCP configuration.

---

## Step 1 — Create an API Key

1. Sign in at console.anthropic.com
2. Navigate to **API Keys → Create Key**
3. Name the key (e.g., "Production" or "Development - yourname")
4. Set an optional usage limit if available on your tier
5. Copy the key immediately

```
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_DEFAULT_MODEL=claude-haiku-4-5-20251001
```

The key is server-side only. Never put it in a `NEXT_PUBLIC_` variable or any client-accessible location.

---

## Step 2 — Set Cost Guardrails

Anthropic's Console allows spending limits per API key (availability depends on tier). At minimum:

- Set a per-developer dev key limit: $5–10/month
- Set CI key to $0 (CI tests should mock the API)
- Track `usage.input_tokens` and `usage.output_tokens` in your application logs

---

## Step 3 — Make a First Successful Call

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY! });

async function testCall() {
  const message = await anthropic.messages.create({
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 100,
    system: 'You are a helpful assistant.',
    messages: [
      { role: 'user', content: 'Say "hello world" and nothing else.' },
    ],
  });

  const content = message.content[0];
  console.log(content.type === 'text' ? content.text : '');
  console.log('Tokens used:', message.usage);
}

testCall();
```

```bash
ANTHROPIC_API_KEY=sk-ant-... npx ts-node src/test-anthropic.ts
```

Expected output: `hello world` and token usage.

---

## Step 4 — Implement Streaming

```typescript
app.post('/api/generate', requireAuth, async (req, res) => {
  const { prompt } = z.object({ prompt: z.string().max(2000) }).parse(req.body);

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');

  const stream = anthropic.messages.stream({
    model: process.env.ANTHROPIC_DEFAULT_MODEL!,
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

  stream.on('error', (err) => {
    console.error('Anthropic stream error:', err);
    res.write('data: [ERROR]\n\n');
    res.end();
  });
});
```

---

## Step 5 — (Optional) Configure an MCP Server

MCP (Model Context Protocol) exposes tools your application's data to Claude. See [integrations/anthropic/mcp.md](../integrations/providers/anthropic/mcp.md) for the full implementation.

For Claude Code integrations, add the MCP server to `.claude/settings.json`:

```json
{
  "mcpServers": {
    "my-app-tools": {
      "command": "node",
      "args": ["dist/mcp-server.js"]
    }
  }
}
```

---

## Verify

- [ ] API key is server-side only
- [ ] First test call succeeds
- [ ] Streaming endpoint delivers content incrementally (verify in browser Network tab)
- [ ] Token usage is logged for each call
- [ ] CI uses mocked Anthropic (no actual API calls in CI)

---

## Related Documents

- [Integrations Anthropic overview](../integrations/providers/anthropic/overview.md)
- [Integrations Anthropic messages](../integrations/providers/anthropic/messages.md)
- [Integrations Anthropic MCP](../integrations/providers/anthropic/mcp.md)
- [Integrations Anthropic testing](../integrations/providers/anthropic/testing.md)
