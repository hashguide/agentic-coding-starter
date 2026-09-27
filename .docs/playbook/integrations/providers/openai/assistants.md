---
kind: playbook
domain: integrations
topics: [integrations, providers, openai, assistants]
authority: guidance
---

# OpenAI Assistants API

The Assistants API provides a stateful, multi-step conversation model with built-in tools (code interpreter, file search) and persistent thread history. It is more complex than Chat Completions and is worth the added overhead only for specific use cases.

## Thread / Run / Message Model

| Concept | What it is |
|---|---|
| **Assistant** | A named, configured AI with specific instructions, a model, and tools enabled |
| **Thread** | A persistent conversation history — a sequence of messages |
| **Message** | A single user or assistant turn in a thread |
| **Run** | Executing the assistant against a thread — produces assistant messages in response to the latest user message |

A Thread is analogous to a conversation session. A Run is a single "think and respond" step within that conversation.

## Creating an Assistant

```typescript
const assistant = await openai.beta.assistants.create({
  name: 'Support Assistant',
  instructions: `
    You are a helpful support assistant for Product Name.
    Answer questions based on the documentation provided.
    If you don't know the answer, say so — do not make up information.
  `,
  model: 'gpt-4o-mini',
  tools: [{ type: 'file_search' }],
});

// Save assistant.id to your database or environment config
// Do not recreate the assistant on every request
```

## Creating a Thread and Running the Assistant

```typescript
async function chat(assistantId: string, userMessage: string, threadId?: string) {
  // Create a new thread or continue an existing one
  const thread = threadId
    ? await openai.beta.threads.retrieve(threadId)
    : await openai.beta.threads.create();

  // Add the user's message to the thread
  await openai.beta.threads.messages.create(thread.id, {
    role: 'user',
    content: userMessage,
  });

  // Run the assistant and wait for completion
  const run = await openai.beta.threads.runs.createAndPoll(thread.id, {
    assistant_id: assistantId,
  });

  if (run.status !== 'completed') {
    throw new Error(`Run ended with status: ${run.status}`);
  }

  // Retrieve the assistant's latest response
  const messages = await openai.beta.threads.messages.list(thread.id, { limit: 1 });
  const lastMessage = messages.data[0];
  const content = lastMessage.content[0];

  return {
    threadId: thread.id,
    response: content.type === 'text' ? content.text.value : '',
  };
}
```

## When Assistants API is Worth the Complexity

**Use Assistants API when:**
- You need persistent, resumable conversation history across sessions
- You need built-in code interpreter (runs code, generates charts)
- You need file search / RAG over a document corpus without building your own retrieval pipeline
- Your users interact with a conversational agent across multiple sessions

**Use Chat Completions when:**
- Each request is independent (no conversation history needed)
- You want maximum control over the context window
- You're building a simple generation endpoint
- You want lower latency (Completions are faster than Assistants for single-turn tasks)

The Assistants API adds state management complexity and cost (thread storage). Plain Chat Completions are almost always simpler and cheaper for stateless generation tasks.

## Related Documents

- [OpenAI overview](overview.md)
- [OpenAI completions](completions.md)
- [OpenAI testing](testing.md)
