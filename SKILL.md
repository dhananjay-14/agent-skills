---
description: MongoDB/Mongoose query optimization and database performance best practices
globs: "**/*.ts,**/*.js"
alwaysApply: false
---

# MongoDB / Mongoose Performance Rules

## Query Optimization

- Always use projections — never fetch entire documents when only a subset of fields is needed. Identify the minimum fields required and select only those.
- Use lean/plain-object reads by default for read-only operations. Only hydrate full ORM documents when document methods or lifecycle hooks are actually needed.
- Use index hints for queries where the database planner may choose a suboptimal index, especially on compound indexes with high cardinality fields.
- Before adding a new query pattern, verify that a supporting index exists. If not, flag the gap and propose the appropriate index.

## Batch Operations

- Never query inside a loop. Always collect identifiers first, then fetch in a single batched query.
- Use parallel execution (e.g. Promise.all) for independent async operations instead of awaiting them sequentially.
- Use bulk insert/update APIs for multi-document writes instead of looping over single-document operations.

## Caching

- Cache data that is frequently read and rarely changed. Choose TTL based on how much staleness is acceptable for that data.
- Invalidate or clear cache entries on any mutation that affects cached data.
- Cache keys must be specific enough to avoid collisions — include all relevant context (user, org, filters, pagination).
- For list endpoints, cache by the full query signature, not just the resource type.
- Never cache data that changes on every request or is heavily written to.

## Pagination

- All list endpoints must be paginated. Never allow unbounded fetches.
- Enforce a maximum page size on the server side regardless of what the client requests.
- For large collections or deep pages, prefer cursor/keyset pagination over offset-based pagination. Offset pagination degrades as depth increases.

## Write Performance

- Minimize update payloads — only update fields that actually changed, not the entire document.
- For fire-and-forget writes (analytics, counters, cache warming), do not block the main flow. Run them in the background and suppress errors explicitly.

## Aggregation Pipelines

- Use aggregation pipelines for complex reporting or transformation logic instead of fetching raw data and processing it in application code.
- Place filter stages as early as possible in the pipeline to reduce the working set before expensive operations.
- Drop unnecessary fields before joins or grouping stages.
- Use disk-spill options only for genuinely large datasets where memory limits are a real concern — prefer index-covered queries first.

## General Rules

- Never query without a filter — always include at least one indexed field as a condition.
- Avoid pattern/regex matching on non-indexed fields in production queries.
- Use exact document counts with filters when accuracy matters; avoid estimated counts for filtered queries.
- For existence checks, fetch only the document ID rather than the full document.
- Avoid eager relation loading (populate/join) by default — only load related data when it is explicitly needed for the response.