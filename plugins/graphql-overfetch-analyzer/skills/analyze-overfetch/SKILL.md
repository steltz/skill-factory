---
name: analyze-overfetch
description: Use when analyzing a codebase for GraphQL queries that fetch more data than is consumed by the application code
---

<!-- BASELINE TEST (remove after GREEN)
Test: Invoke this skill against a codebase with GraphQL queries.
FAIL criteria (placeholder skill fails if Claude does ANY of these without being told):
1. Runs a structured Discover → Trace → Fix pipeline
2. Builds a query catalog with file paths and field lists
3. Presents a hard gate asking user to validate catalog before tracing
4. Traces field usage in consuming code and builds a usage map
5. Classifies fields as unused vs indeterminate
6. Presents an over-fetch report with hard gate before fixing
7. Offers fix scope selection (fix all / fix per-query / report only)
8. Removes unused fields from queries and verifies valid GraphQL syntax
-->

# Analyze Over-Fetch

Discover GraphQL queries in a codebase, trace how their results are consumed, identify fields that are fetched but never used, and auto-fix the queries.

```
NO PHASE MAY BE SKIPPED OR REORDERED.
Discover → Trace → Fix. Hard gate between each.
```

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Phase 1: Discover** — find the schema and all query operations, build a catalog
2. **Phase 1 Gate** — present the catalog to the user, wait for validation
3. **Phase 2: Trace** — for each query, trace field usage in consuming code
4. **Phase 2 Gate** — present the over-fetch report to the user, wait for validation
5. **Phase 3: Fix** — user selects fix scope, apply edits, verify syntax
6. **Post-fix guidance** — remind user of follow-up actions

## Phase 1: Discover

Locate all GraphQL artifacts in the codebase and build a query catalog.

### Step 1.1: Find the Schema

Search in this order. Stop at the first match:

1. `.graphql` and `.gql` files containing type definitions (`type Query`, `type Mutation`, `type <Name>`, `input`, `interface`, `enum`)
2. Introspection JSON files (`schema.json`, `introspection-result.json`, `*.introspection.json`)
3. SDL exports in code — look for `buildSchema()`, `makeExecutableSchema()`, `typeDefs` variables

If no schema is found locally, ask the user:

> "I couldn't find a GraphQL schema in this codebase. Please provide one of:
> - A file path (can be in another repo)
> - A URL to an introspection endpoint
> - A schema registry identifier"

Do NOT proceed without a schema.

### Step 1.2: Find All Query Operations

Search for GraphQL queries using these patterns:

**Standalone files:**
- Glob for `**/*.graphql` and `**/*.gql`
- Read each file; extract operations that start with `query` (catalog but skip `mutation` and `subscription` — they are out of scope for tracing and fixing)

**Inline in code (any language):**
- Grep for tagged template literals: `` gql` ``, `` graphql` ``
- Grep for multi-line strings containing `query \w+` or `query {`
- Grep for framework invocations: `useQuery(`, `client.query(`, `client.execute(`, `graphql(` — then read the referenced query definition

### Step 1.3: Build the Catalog

For each discovered query, record:

| Field | Value |
|-------|-------|
| Operation name | The named query (e.g., `GetUser`) or `<anonymous>` |
| File | Exact file path and line number |
| Selected fields | Full list including nested selections (e.g., `user { id name posts { title } }` → `user.id`, `user.name`, `user.posts.title`) |
| Schema source | Local path or user-provided location |

Present the catalog to the user as a numbered list.

### Phase 1 Gate

<HARD-GATE>
STOP. Present the catalog and ask:

> "Here are the N queries I found. Please review:
> 1. Are any queries missing?
> 2. Should any be excluded from analysis?
>
> Confirm to proceed to tracing."

Do NOT proceed to Phase 2 until the user confirms.
</HARD-GATE>
