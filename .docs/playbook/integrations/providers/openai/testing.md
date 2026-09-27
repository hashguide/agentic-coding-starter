---
kind: playbook
domain: integrations
topics: [integrations, providers, openai, testing]
authority: guidance
---

# OpenAI Testing

Testing code that calls OpenAI requires a strategy for avoiding real API calls in CI (which costs money, introduces flakiness, and is slow), while still validating the integration in an environment that approximates production.

## Mocking in Unit Tests

For unit tests, mock the OpenAI client entirely. Do not make real API calls in CI.

```typescript
// src/__mocks__/openai.ts
export default class OpenAI {
  chat = {
    completions: {
      create: jest.fn().mockResolvedValue({
        choices: [
          {
            message: { content: 'Mocked completion response', role: 'assistant' },
            finish_reason: 'stop',
            index: 0,
          },
        ],
        usage: { prompt_tokens: 50, completion_tokens: 20, total_tokens: 70 },
      }),
    },
  };
}
```

```typescript
// In your test
jest.mock('openai');
import OpenAI from 'openai';

test('generateSummary returns expected output', async () => {
  const result = await generateSummary('A long text to summarize...');
  expect(result).toBe('Mocked completion response');
  expect((new OpenAI()).chat.completions.create).toHaveBeenCalledWith(
    expect.objectContaining({ model: 'gpt-4o-mini' })
  );
});
```

## Budget Guardrails for Development

Set a low hard spending limit in the OpenAI dashboard for development API keys:
- Development key: $10/month hard limit
- CI key: $5/month hard limit (ideally zero, if all tests mock)

Use separate API keys per environment so a CI misconfiguration that makes real API calls doesn't impact your production key's rate limits.

## Model Fallback Strategy

Production OpenAI integrations must handle rate limits and model unavailability:

```typescript
async function robustCompletion(messages: OpenAI.Chat.ChatCompletionMessageParam[]) {
  const models = ['gpt-4o-mini', 'gpt-3.5-turbo']; // Primary → fallback

  for (const model of models) {
    try {
      return await openai.chat.completions.create({
        model,
        messages,
        max_tokens: 500,
      });
    } catch (err) {
      if (err instanceof OpenAI.APIError) {
        if (err.status === 429) {
          // Rate limited — try fallback model
          console.warn(`Rate limited on ${model}, trying next model`);
          continue;
        }
        if (err.status === 503) {
          // Model unavailable — try fallback
          console.warn(`${model} unavailable, trying next model`);
          continue;
        }
      }
      throw err; // Unknown error — don't swallow
    }
  }

  throw new Error('All models failed');
}
```

## Integration Tests Against Real API

For integration tests that must validate the real API call:
- Use a dedicated integration test environment with its own API key
- Run integration tests separately from the main CI suite (not on every PR, but on merge to main or on a schedule)
- Cache responses where the same prompt will produce the same result

## Related Documents

- [OpenAI overview](overview.md)
- [OpenAI completions](completions.md)
- [Testing strategy](../../../quality/testing/strategy.md)
