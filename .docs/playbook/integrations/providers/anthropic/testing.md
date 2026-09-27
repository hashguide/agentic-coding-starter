---
kind: playbook
domain: integrations
topics: [integrations, providers, anthropic, testing]
authority: guidance
---

# Anthropic Testing

Testing Claude integrations requires avoiding real API calls in CI (cost, flakiness, latency) while validating prompt behavior and error handling systematically.

## Mocking in Unit Tests

Mock the Anthropic SDK to prevent real API calls in unit and integration tests:

```typescript
// src/__mocks__/@anthropic-ai/sdk.ts
export default class Anthropic {
  messages = {
    create: jest.fn().mockResolvedValue({
      id: 'msg_mock',
      type: 'message',
      role: 'assistant',
      content: [{ type: 'text', text: 'Mocked Claude response' }],
      model: 'claude-haiku-4-5-20251001',
      stop_reason: 'end_turn',
      usage: { input_tokens: 50, output_tokens: 20 },
    }),
    stream: jest.fn().mockReturnValue({
      on: (event: string, handler: Function) => {
        if (event === 'text') handler('Mocked ');
        if (event === 'text') handler('response');
        if (event === 'finalMessage') handler();
        return { on: jest.fn() };
      },
    }),
  };
}
```

```typescript
// In your test
jest.mock('@anthropic-ai/sdk');

test('generateSummary calls claude with correct params', async () => {
  const result = await generateSummary('Long text...');
  expect(result).toBe('Mocked Claude response');
});
```

## Prompt Testing

Prompt reliability is harder to unit-test than deterministic code — the same prompt can produce different outputs. Approaches:

**Property-based testing:** Assert properties of the output rather than exact strings.
```typescript
const summary = await generateSummary(longText);
expect(summary.length).toBeLessThan(500); // Within expected length
expect(summary.split('.').length).toBeGreaterThan(1); // Has multiple sentences
```

**Golden file testing:** Generate output against a fixed set of inputs and store it. On subsequent runs, flag significant deviations.

**Eval framework:** For production prompt reliability, build an evaluation suite with expected inputs and criteria. Run it against real API calls on a schedule, not in CI.

## Budget Guardrails for Development

```typescript
// In development, add a spend guard before calling the API
if (process.env.NODE_ENV === 'development') {
  const today = new Date().toISOString().split('T')[0];
  const todaySpend = await getDailySpend(today); // From your usage tracking
  if (todaySpend > Number(process.env.DAILY_AI_SPEND_LIMIT_USD ?? 1)) {
    throw new Error(`Daily AI spend limit reached. Set DAILY_AI_SPEND_LIMIT_USD to increase.`);
  }
}
```

Set low per-developer spending limits in development API keys in the Anthropic Console:
- Per-developer dev key: $5/month
- CI key: $0 (all tests mock)

## Testing Streaming

```typescript
test('streaming handler emits content events', async () => {
  const chunks: string[] = [];

  const response = await supertest(app)
    .post('/api/generate-stream')
    .set('Authorization', `Bearer ${testToken}`)
    .send({ prompt: 'Test prompt' });

  expect(response.status).toBe(200);
  expect(response.headers['content-type']).toContain('text/event-stream');
  // Parse SSE events from response.text
});
```

## Related Documents

- [Anthropic overview](overview.md)
- [Anthropic messages](messages.md)
- [OpenAI testing (comparison)](../openai/testing.md)
- [Testing strategy](../../../quality/testing/strategy.md)
