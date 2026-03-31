---
name: analyze-overfetch
description: Use when analyzing a codebase for GraphQL queries that fetch more data than is consumed by the application code
---

<!-- BASELINE TEST (remove after GREEN)
Test: Invoke this skill against a codebase with GraphQL queries and fragments.
FAIL criteria (v0.1.0 skill fails if Claude does ANY of these without being told):
1. Discovers fragments as first-class catalog items alongside queries
2. Builds a fragment dependency graph (queries → fragments → nested fragments)
3. Presents fragment consumer counts in the catalog
4. Traces fragment fields through multiple consumers (multi-consumer tracking)
5. Produces a cross-query audit with severity rankings (High/Medium/Low)
6. Recommends fragment splitting for partially-used fragments
7. Offers "Split fragments" as a fix scope option
8. Runs a verification checklist with 8 specific checkbox items
9. Has a "When Stuck" table with 4 problem/solution rows
10. Uses a 4-phase pipeline: Inventory → Trace → Audit → Fix
-->

# Analyze Over-Fetch

Discover GraphQL queries and fragments in a codebase, trace how their results are consumed, audit findings across operations, and auto-fix over-fetched fields.

Violating the letter of the rules is violating the spirit of the rules.

## When to Use

- Codebase has GraphQL queries and you want to find unused fields
- Performance optimization — reducing payload size
- Code cleanup — removing dead query fields after feature removal
- Pre-migration audit — understanding what data is actually consumed
- Fragment analysis — finding fields in shared fragments that no consumer uses

## When NOT to Use

- No GraphQL in the codebase
- Schema-only repos with no query consumers
- Queries are dynamically generated at runtime (the skill traces static code)
- User wants to analyze subscriptions or mutations (out of scope)

```
NO FIELD MAY BE REMOVED WITHOUT A TRACED CONSUMER AND AN AUDITED FINDING.
Inventory → Trace → Audit → Fix. No phase skipped. No phase reordered.
```

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Phase 1: Inventory** — find schema, queries, fragments; build catalog
2. **Phase 1 Gate** — present catalog, wait for user validation
3. **Phase 2: Trace** — trace field consumption per operation and fragment
4. **Phase 2 Gate** — present usage maps, wait for user validation
5. **Phase 3: Audit** — cross-reference findings, rank severity, identify fragment splits
6. **Phase 3 Gate** — present findings report, user selects fix scope
7. **Phase 4: Fix** — apply edits, verify syntax
8. **Post-fix verification** — run verification checklist

## Phase 1: Inventory

Locate all GraphQL artifacts in the codebase and build a query + fragment catalog.

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

### Step 1.3: Find All Fragments

Search for GraphQL fragments alongside queries:

1. `fragment` definitions in `.graphql` / `.gql` files
2. Inline `fragment` definitions in tagged template literals
3. For each fragment, record: name, target type, selected fields, file/line
4. For each query, record which fragments it spreads (`...FragmentName`)
5. Build a dependency graph: queries → fragments → nested fragments (fragments that spread other fragments)

### Step 1.4: Build the Catalog

**Operations:**

For each discovered query, record:

| Field | Value |
|-------|-------|
| Operation name | The named query (e.g., `GetUser`) or `<anonymous>` |
| File | Exact file path and line number |
| Selected fields | Full list including nested selections and spread fragments |
| Schema source | Local path or user-provided location |

**Fragments:**

For each discovered fragment, record:

| Field | Value |
|-------|-------|
| Fragment name | e.g., `UserFields` |
| Target type | The type the fragment is defined on (e.g., `User`) |
| File | Exact file path and line number |
| Selected fields | Full list of fields in this fragment |
| Consumer count | Number of operations that spread this fragment |

Present the catalog to the user as two numbered lists: Operations and Fragments.

### Phase 1 Gate

<HARD-GATE>
STOP. Present the catalog and ask:

> "Here are the N queries and M fragments I found. Please review:
> 1. Are any queries or fragments missing?
> 2. Should any be excluded from analysis?
>
> Confirm to proceed to tracing."

Do NOT proceed to Phase 2 until the user confirms.
</HARD-GATE>
