---
kind: playbook
domain: ai
topics: [ai, rag, retrieval, embeddings, context, grounding, and]
authority: guidance
---
# Retrieval and Context for AI Features

Use retrieval when the model needs private, current, domain-specific, or too-large knowledge that cannot be reliably supplied in the base prompt.

Choose lexical search, structured queries, vector/semantic retrieval, or hybrid retrieval from the information type and evaluation results. Enforce authorization before retrieved content reaches the model.

Define chunking/indexing, metadata, freshness, deletion, ranking, context budgeting, source attribution and fallback when retrieval is weak. Evaluate retrieval separately from generation so failures can be diagnosed.
