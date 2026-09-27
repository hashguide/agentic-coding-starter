---
kind: playbook
domain: backend
topics: [pagination, cursor, offset, filtering, sorting, list-api, backend, architecture]
authority: guidance
---
# Pagination, Filtering, and Sorting

## Offset/page pagination
Use when datasets are moderate, users benefit from page numbers or random access, and concurrent inserts do not make position instability costly.

## Cursor pagination
Use for large or frequently changing ordered datasets, feeds, event histories, or APIs where stable traversal and efficient deep pagination matter. Define deterministic sort keys and cursor semantics.

## No pagination / client filtering
Use when bounded datasets are small enough to load safely and the product benefits from instant local interaction.

Filters and sorts should be explicit contracts with allow-listed fields/operators, predictable defaults, stable ordering, permission-aware queries, and indexes matched to common access paths.
