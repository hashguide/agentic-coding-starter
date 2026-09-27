---
kind: playbook
domain: integrations
topics: [integrations, providers, anthropic, mcp]
authority: guidance
---

# Model Context Protocol (MCP)

The Model Context Protocol (MCP) is an open standard that defines how AI models communicate with external tools and data sources. An MCP server exposes tools that Claude (or other MCP-compatible models) can call during a conversation, enabling actions like querying a database, calling an API, or reading files.

## What an MCP Server Is

An MCP server is a process that:
1. Declares a set of tools (functions with typed parameters and descriptions)
2. Receives tool call requests from a client (Claude, an agent, or an IDE)
3. Executes the tool and returns the result
4. The result is passed back to the model, which incorporates it into its response

The model does not call your database directly. It asks the MCP server to do so, and the MCP server returns structured results the model can reason about.

## Basic Tool Definition

```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  ListToolsRequestSchema,
  CallToolRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';

const server = new Server(
  { name: 'acme-tools', version: '1.0.0' },
  { capabilities: { tools: {} } }
);

// Declare available tools
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: 'get_user_by_email',
      description: 'Look up a user by their email address',
      inputSchema: {
        type: 'object',
        properties: {
          email: { type: 'string', description: 'The user email address' },
        },
        required: ['email'],
      },
    },
    {
      name: 'get_subscription_status',
      description: 'Get the current subscription status for a user',
      inputSchema: {
        type: 'object',
        properties: {
          userId: { type: 'string', description: 'The user ID' },
        },
        required: ['userId'],
      },
    },
  ],
}));

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  switch (request.params.name) {
    case 'get_user_by_email': {
      const { email } = request.params.arguments as { email: string };
      const user = await db.user.findUnique({
        where: { email },
        select: { id: true, name: true, createdAt: true, subscriptionStatus: true },
      });
      return {
        content: [{ type: 'text', text: JSON.stringify(user ?? { error: 'not_found' }) }],
      };
    }

    case 'get_subscription_status': {
      const { userId } = request.params.arguments as { userId: string };
      const user = await db.user.findUnique({
        where: { id: userId },
        select: { subscriptionStatus: true, currentPeriodEnd: true },
      });
      return {
        content: [{ type: 'text', text: JSON.stringify(user) }],
      };
    }

    default:
      throw new Error(`Unknown tool: ${request.params.name}`);
  }
});

// Start the server on stdio transport
const transport = new StdioServerTransport();
await server.connect(transport);
```

## Referencing MCP Servers in API Calls

When using the Anthropic API directly with MCP, the integration works through a client-side MCP client that bridges the API and the server. In Claude Code, MCP servers are configured in `.claude/settings.json` or globally. For API-based integrations, use the `@anthropic-ai/sdk` with tool use:

```typescript
// For API-based tool use (equivalent to MCP tool definitions)
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  tools: [
    {
      name: 'get_user_by_email',
      description: 'Look up a user by their email address',
      input_schema: {
        type: 'object' as const,
        properties: {
          email: { type: 'string' },
        },
        required: ['email'],
      },
    },
  ],
  messages: [{ role: 'user', content: 'What is the subscription status for user@example.com?' }],
});
```

## When to Use MCP

MCP is worth the setup when:
- Building a Claude Code extension or IDE integration that needs context from your codebase/database
- Creating an AI agent with persistent access to your application's data
- The same tool definitions need to be used across multiple AI integrations

For one-off API integrations with specific tools, inline tool definitions in the Messages API are simpler than a full MCP server.

## Related Documents

- [Anthropic overview](overview.md)
- [Anthropic messages](messages.md)
- [Anthropic testing](testing.md)
