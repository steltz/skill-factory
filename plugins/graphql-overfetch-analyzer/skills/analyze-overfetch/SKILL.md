---
name: analyze-overfetch
description: Use when analyzing a codebase for GraphQL queries that fetch more data than is consumed by the application code
---

# Analyze Over-Fetch

Discover GraphQL queries in a codebase, trace how their results are consumed, identify fields that are fetched but never used, and auto-fix the queries.

## When to Use

- Codebase has GraphQL queries and you want to find unused fields
- Performance optimization — reducing payload size
- Code cleanup — removing dead query fields after feature removal
- Pre-migration audit — understanding what data is actually consumed

## When NOT to Use

- No GraphQL in the codebase
- Schema-only repos with no query consumers
- Queries are dynamically generated at runtime (the skill traces static code)
- User wants to analyze subscriptions or mutations (out of scope)

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

## Phase 2: Trace

For each query in the validated catalog, trace how the result is consumed in application code.

### Step 2.1: Find the Consumer

Starting from where the query is invoked, identify the variable that holds the response data:

- JS/TS: `const { data } = useQuery(GET_USER)` → trace `data`
- Python: `result = client.execute(query)` → trace `result`
- Go: `resp, err := client.Query(ctx, &query, vars)` → trace `resp` or `query`

If the query is defined in a standalone `.graphql` file, search for imports or references to that file to find the invocation site.

### Step 2.2: Track Field Access

Follow the response variable through the consuming code. Record every field access:

- **Destructuring:** `const { user: { name, email } } = data` → `user.name`, `user.email` used
- **Dot notation:** `data.user.name` → `user.name` used
- **Bracket notation:** `data['user']['name']` → `user.name` used
- **Spread:** `{ ...data.user }` into a typed target → all fields of `user` used
- **Function arguments:** `renderProfile(data.user)` → follow into `renderProfile` one level deep
- **Map/iteration:** `data.posts.map(p => p.title)` → `posts.title` used

### Step 2.3: Handle Indirection

Follow the response variable one level deep into called functions or child components. If the data disappears into any of these patterns, mark ALL fields on that path as **indeterminate** (not unused):

- `JSON.stringify(data)` — opaque serialization
- `console.log(data)` or logging utilities
- `Object.keys(data.user)` — dynamic access
- `data[variableName]` — computed property access
- Spread into an untyped target: `{ ...data.user }` with no type annotation
- Reflection or metaprogramming

**Indeterminate means "we can't tell" — never flag indeterminate fields as unused.**

### Step 2.4: Build the Usage Map

For each query, produce:

| Category | Fields |
|----------|--------|
| **Fetched** | All fields selected in the query |
| **Used** | Fields with confirmed access in consuming code |
| **Unused** | Fetched minus Used minus Indeterminate |
| **Indeterminate** | Fields where usage could not be determined |

### Step 2.5: Compile the Over-Fetch Report

For each query that has unused fields:

```
Query: GetUser (src/queries/user.graphql:3)
Consumer: src/components/UserProfile.tsx:15
Unused fields: user.createdAt, user.updatedAt, user.posts.body
Indeterminate fields: user.metadata (spread into untyped object)
```

### Phase 2 Gate

<HARD-GATE>
STOP. Present the over-fetch report and ask:

> "Here are the over-fetching findings for N queries. For each query I've listed:
> - Fields confirmed unused
> - Fields marked indeterminate (conservatively kept)
>
> Please review. Are any findings incorrect? Confirm to proceed to fixes."

Do NOT proceed to Phase 3 until the user confirms.
</HARD-GATE>

## Phase 3: Fix

Apply fixes to eliminate over-fetched fields from queries.

### Step 3.1: Present Report Summary

Present the over-fetch report grouped by severity (most unused fields first):

```
1. GetUserProfile — 5 unused fields (src/queries/user.graphql:3)
2. GetPostList — 3 unused fields (src/queries/posts.graphql:1)
3. GetSettings — 1 unused field (src/queries/settings.graphql:8)
```

### Step 3.2: User Selects Fix Scope

Ask the user:

> "How would you like to proceed?
> 1. **Fix all** — Remove unused fields from all queries
> 2. **Fix per-query** — Walk through each query, confirm individually
> 3. **Report only** — No changes, keep the report as documentation"

Wait for the user's choice before proceeding.

### Step 3.3: Apply Edits

For each query the user has confirmed:

1. Open the file containing the query definition
2. Remove each unused field from the selection set
3. If removing a field leaves an empty selection set on a nested object, remove the entire nested selection (e.g., if `posts { body }` had `body` removed and no other fields remain, remove the `posts` selection entirely)
4. Preserve formatting, comments, and field aliases
5. Do NOT modify indeterminate fields

### Step 3.4: Verify Syntax

After applying edits, re-read each modified query and verify:

- The query still parses as valid GraphQL (balanced braces, no trailing commas, no empty selection sets)
- No fields were accidentally removed that were marked as used or indeterminate
- If any syntax issue is found, fix it immediately

### Post-Fix Guidance

After all fixes are applied, tell the user:

> "Fixes applied. Follow-up actions:
> - Re-run GraphQL codegen if your project uses it (e.g., `graphql-codegen`, `relay-compiler`)
> - Run your test suite to verify nothing broke
> - Check if removed fields should also be removed from shared fragments used by other queries"

## Red Flags

These thoughts mean STOP — you are about to skip a phase:

- "The queries are simple, I can skip the catalog"
- "I already know which fields are unused"
- "The user seems to want a quick answer"
- "There's only one query, I don't need the full pipeline"
- "I'll trace and fix at the same time to save time"
- "The schema is obvious, I don't need to parse it"
- "I'll just remove fields that look unused"

## Common Rationalizations

| Rationalization | Reality |
|-----------------|---------|
| "Only one query, skip the catalog" | One query still needs schema validation and user confirmation |
| "Fields are obviously unused" | Obvious to you is not obvious — trace the code |
| "Indeterminate means probably unused" | Indeterminate means you don't know. Leave it. |
| "I'll combine Trace and Fix" | Combining phases skips the user validation gate |
| "The user said to just fix it" | The user approved the pipeline. Follow it. |
| "This field can't possibly be used" | Prove it by finding the consumer, or mark indeterminate |

## Out of Scope

This skill does NOT:
- Modify the GraphQL schema
- Update TypeScript generated types (re-run codegen instead)
- Analyze mutations or subscriptions (over-fetching is a query concern)
- Run external tools or scripts
