---
kind: playbook
domain: backend
topics: [api, rest, rpc, graphql, contracts, versioning, backend, architecture, design]
authority: guidance
---
# API Design

Choose an API style from the consumers, contract lifetime, data access patterns, and organizational boundaries.

## REST / resource-oriented HTTP
Use when interoperability, public consumption, conventional HTTP semantics, caching, and broad tooling matter. Define resource boundaries, status/error contracts, pagination, idempotency, and versioning.

Fits public APIs, heterogeneous clients, service boundaries, and teams that benefit from an explicit network contract.

## Typed RPC
Use when the client and server are controlled together and end-to-end type sharing materially improves development speed. It fits monorepos and tightly coordinated applications well.

Consider coupling, non-native clients, public API plans, transport semantics, and how contracts are versioned if consumers become independent.

## GraphQL
Use when clients have substantially different data-shape needs, graph traversal is central, or a schema-driven query layer solves repeated over/under-fetching problems.

Plan schema governance, authorization at field/object boundaries, query cost controls, caching, N+1 prevention, persisted operations where useful, and observability.

## Event / message contracts
Use asynchronous commands/events when consumers do not require an immediate request-response result or when work crosses durable background boundaries. Define delivery semantics, schemas, ordering, retries and idempotency.

## Decision questions
- Who consumes the contract and who controls those consumers?
- Must third parties use it?
- How independently do client and server evolve?
- Are requests mostly CRUD/resource operations, action commands, or graph-shaped reads?
- What caching and realtime behavior is required?
- What compatibility lifetime is expected?

Record the selected API styles, contract ownership, error model, versioning strategy, pagination/filtering conventions, authentication, and idempotency rules in project or feature technical design.
