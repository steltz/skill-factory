# GraphQL Over-Fetch Analyzer — Design Spec

## Overview

A standalone Claude Code skill plugin that discovers GraphQL queries in a codebase, traces how query results are consumed, identifies fields that are fetched but never used, and auto-fixes queries by removing the over-fetched fields.

## Plugin Identity

- **Plugin name:** `graphql-overfetch-analyzer`
- **Skill name:** `analyze-overfetch`
- **Trigger:** "Use when analyzing a codebase for GraphQL queries that fetch more data than is consumed by the application code"
- **Skill type:** Rigid — three phases execute in strict order with hard gates between them

## Architecture: Three-Phase Pipeline

```
Discover → Trace → Fix
```

Each phase produces output that the user validates before the next phase begins. No phase may be skipped or reordered.

## Phase 1: Discover

Locate all GraphQL artifacts in the codebase and build a query catalog.

### Schema Discovery

1. Search for `.graphql` and `.gql` files containing type definitions (`type`, `input`, `interface`, `enum`)
2. Look for introspection JSON files (`schema.json`, `introspection-result.json`)
3. Check for SDL exports in code (common in JS/Python GraphQL servers)
4. If no schema is found locally, ask the user to provide the schema location — a file path (possibly in another repo), a URL to an introspection endpoint, or a schema registry identifier
5. Parse the schema into a type map for reference in later phases

### Query Discovery

1. `.graphql` / `.gql` files containing `query` operations (mutations and subscriptions are discovered for completeness but excluded from tracing and fixing)
2. Tagged template literals: `` gql`...` ``, `` graphql`...` `` in JS/TS
3. String variables assigned GraphQL query strings (Python, Go, Ruby, etc.)
4. Framework-specific invocation patterns: `useQuery()`, `client.query()`, `execute()` calls that reference query definitions

### Catalog Output

For each discovered query:
- File path and line number
- Operation name
- Full set of selected fields (including nested selections)
- Schema source notation (local path or user-provided location)

### Hard Gate

Present the catalog to the user for validation before proceeding to Phase 2. The user may identify missed queries or flag false positives.

## Phase 2: Trace

For each query in the validated catalog, trace how the result is consumed in application code.

### Tracing Strategy

1. **Find the consumer** — Starting from where the query is invoked, identify the variable that holds the response data
2. **Track field access** — Follow that variable through the code: destructuring, dot notation, bracket notation, spread operators, function arguments, serialization targets
3. **Handle indirection** — If the result is passed to another function or component, follow it one level deep. If it disappears into a generic utility (`JSON.stringify`, logging, etc.), mark all fields as "indeterminate" rather than false-flagging
4. **Build usage map** — For each query: fields fetched vs. fields accessed. The difference is the over-fetch set.

### Conservative Analysis

When the skill cannot confidently determine whether a field is used, it marks the field as "indeterminate" rather than flagging it. Scenarios that produce indeterminate results:
- Dynamic property access (`data[variable]`)
- Reflection or metaprogramming
- Serialization to an opaque sink
- Spread into an untyped target

### What Gets Flagged

- Fields fetched but never accessed anywhere in consuming code
- Nested object selections where the parent is accessed but specific child fields are not
- Relation fields (lists of objects) where only a subset of selected fields is used

### What Does NOT Get Flagged

- Fields used in conditional branches (still counts as used)
- Fields passed to TypeScript interfaces or type assertions (still counts as used)
- Indeterminate fields (conservative — no flag)

### Over-Fetch Report

For each query with over-fetched fields:
- Query name and file/line location
- Consumer file/line location
- List of unused fields
- List of indeterminate fields (for transparency)

### Hard Gate

Present the over-fetch report to the user for validation before proceeding to Phase 3.

## Phase 3: Fix

Apply fixes to eliminate over-fetched fields from queries.

### Fix Workflow

1. **Present report** — Structured summary grouped by severity (most unused fields first)
2. **User selects fix scope:**
   - **Fix all** — Apply all suggested removals
   - **Fix per-query** — Walk through each query, confirm individually
   - **Report only** — Skip fixes, keep the report as documentation
3. **Apply edits** — Remove unused fields from query definitions. Preserve formatting, comments, and aliases. If removing a field leaves an empty selection set on a nested object, remove the entire nested selection.
4. **Verify** — Re-scan modified queries to confirm valid GraphQL syntax. Correct any formatting issues (trailing commas, empty braces).

### Out of Scope

- Schema modifications
- TypeScript generated type updates (user should re-run codegen)
- Mutation or subscription analysis (over-fetching is a query concern)

### Post-Fix Guidance

After applying fixes, remind the user to:
- Re-run GraphQL codegen if applicable
- Run the project's test suite
- Consider whether removed fields should also be removed from shared fragments

## Plugin Structure

```
plugins/graphql-overfetch-analyzer/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── analyze-overfetch/
│       └── SKILL.md
└── CHANGELOG.md
```

No supporting files, hooks, or external dependencies. The analysis logic is Claude's reasoning guided by the SKILL.md instructions.

## Skill Enforcement

- **Rigid skill** — phases execute in strict order
- **Checklist-driven** — numbered checklist in SKILL.md with phases as top-level items and sub-steps within; Claude creates tasks from the checklist
- **Hard gates** — user validation required between each phase transition
- **Framework-agnostic** — works with any GraphQL client (Apollo, urql, Relay, graphql-request, Python, Go, etc.)
- **Conservative defaults** — indeterminate fields are not flagged; false positives are avoided at the cost of potential false negatives
