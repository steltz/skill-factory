# GraphQL Over-Fetch Analyzer v2 — Design Spec

## Overview

A full overhaul of the `graphql-overfetch-analyzer` plugin (v0.1.0 → 1.0.0). Redesigns the `analyze-overfetch` skill as a proper discipline skill with a 4-phase pipeline, fragment analysis, stronger enforcement mechanisms, and a supporting reference file.

## Plugin Identity

- **Plugin name:** `graphql-overfetch-analyzer`
- **Skill name:** `analyze-overfetch`
- **Trigger:** "Use when analyzing a codebase for GraphQL queries that fetch more data than is consumed by the application code"
- **Skill type:** Discipline (rigid enforcement)
- **Model after:** `test-driven-development` — Iron Law, rationalizations table, red flags, verification checklist
- **Version:** 1.0.0 (major — breaking pipeline change, scope expansion)

## Iron Law

```
NO FIELD MAY BE REMOVED WITHOUT A TRACED CONSUMER AND AN AUDITED FINDING.
Inventory → Trace → Audit → Fix. No phase skipped. No phase reordered.
```

## Architecture: Four-Phase Pipeline

| Phase | Purpose | Output | Gate |
|-------|---------|--------|------|
| **Inventory** | Find schema, queries, and fragments; build a complete catalog | Numbered catalog of operations + fragments with all selected fields | User validates catalog completeness |
| **Trace** | For each operation/fragment, trace field consumption in application code | Per-operation usage map (used / unused / indeterminate) | User validates trace accuracy |
| **Audit** | Cross-reference findings across queries and fragments; severity ranking; fragment splitting recommendations | Structured findings report with severity levels | User validates findings and selects fix scope |
| **Fix** | Apply user-approved edits to query/fragment definitions | Modified files + verification results | Verification checklist confirms correctness |

Each gate is a `<HARD-GATE>` — the agent must stop and wait for user confirmation before proceeding.

## Phase 1: Inventory

Locate all GraphQL artifacts in the codebase and build a query + fragment catalog.

### Schema Discovery

1. `.graphql` and `.gql` files containing type definitions (`type`, `input`, `interface`, `enum`)
2. Introspection JSON files (`schema.json`, `introspection-result.json`)
3. SDL exports in code (`buildSchema()`, `makeExecutableSchema()`, `typeDefs`)
4. If not found locally, ask the user for a file path, introspection URL, or registry ID

### Query Discovery

1. `.graphql` / `.gql` files containing `query` operations
2. Tagged template literals: `` gql`...` ``, `` graphql`...` `` in JS/TS
3. String variables containing GraphQL query strings (Python, Go, Ruby, etc.)
4. Framework invocations: `useQuery()`, `client.query()`, `execute()` referencing query definitions

### Fragment Discovery

1. `fragment` definitions in `.graphql` / `.gql` files
2. Inline fragment definitions in tagged template literals
3. For each fragment: name, target type, selected fields, file/line
4. For each query: which fragments it spreads (`...FragmentName`)
5. Build a dependency graph: queries → fragments → nested fragments

### Catalog Output

Two sections:

**Operations:**
- File path and line number
- Operation name
- Full set of selected fields (including nested selections and spread fragments)
- Schema source notation

**Fragments:**
- File path and line number
- Fragment name and target type
- Selected fields
- Consumer count (how many operations reference this fragment)

### Hard Gate

Present the catalog to the user. Ask if any queries or fragments are missing or should be excluded.

## Phase 2: Trace

For each operation and fragment in the validated catalog, trace field consumption in application code.

### Tracing Strategy

1. **Find the consumer** — identify the variable holding the response data
2. **Track field access** — destructuring, dot notation, bracket notation, spread, function arguments, iteration
3. **Handle indirection** — follow one level deep into called functions/components. If data disappears into opaque sinks, mark as indeterminate.
4. **Fragment-aware tracing** — when a query spreads a fragment, trace the fragment's fields through the query's consumer
5. **Multi-consumer tracking** — fragments spread by multiple queries get per-consumer tracking. A fragment field is "used" if any consumer accesses it.
6. **Build usage map** — per operation: fields fetched vs. fields accessed vs. indeterminate. Include a `consumers` column for fragment fields.

### Conservative Analysis

Mark as indeterminate (never flag as unused):
- Dynamic property access (`data[variable]`)
- Reflection or metaprogramming
- Serialization to an opaque sink (`JSON.stringify`, logging)
- Spread into an untyped target
- `Object.keys()` or similar dynamic access

### Hard Gate

Present usage maps to the user. Ask if any findings look incorrect.

## Phase 3: Audit

Cross-reference trace results across all operations and fragments. Produce a structured findings report.

### Audit Steps

1. **Per-query findings** — unused fields in each query's own selection
2. **Per-fragment findings** — fields unused across all consumers of a fragment
3. **Fragment splitting recommendations** — when a fragment has fields used by some consumers but not others, recommend splitting into base + consumer-specific fragments
4. **Severity ranking:**
   - **High** — field unused by all consumers, easy removal
   - **Medium** — fragment field used by some consumers but not others (splitting opportunity)
   - **Low** — single unused field in a small query (low payload impact)

### Hard Gate

Present the findings report. User selects fix scope:
- **Fix all** — apply all suggested removals
- **Fix per-finding** — walk through each finding, confirm individually
- **Split fragments** — for medium-severity findings, split fragments
- **Report only** — no changes, keep the report

## Phase 4: Fix

Apply user-approved edits.

1. Remove unused fields from query/fragment definitions
2. Split fragments if selected by user
3. If removing a field leaves an empty selection set, remove the entire nested selection
4. Preserve formatting, comments, and field aliases
5. Do NOT modify indeterminate fields
6. Re-read each modified file and verify valid GraphQL syntax
7. Confirm no used or indeterminate fields were removed

### Post-Fix Guidance

- Re-run GraphQL codegen if applicable
- Run the project's test suite
- Check if removed fields should also be removed from other shared fragments

## Enforcement Mechanisms

### Checklist (8 items, task-tracked)

1. **Phase 1: Inventory** — find schema, queries, fragments; build catalog
2. **Phase 1 Gate** — present catalog, wait for user validation
3. **Phase 2: Trace** — trace field consumption per operation and fragment
4. **Phase 2 Gate** — present usage maps, wait for user validation
5. **Phase 3: Audit** — cross-reference findings, rank severity, identify fragment splits
6. **Phase 3 Gate** — present findings report, user selects fix scope
7. **Phase 4: Fix** — apply edits, verify syntax
8. **Post-fix verification** — run verification checklist

### Red Flags (13 items)

1. "The queries are simple, I can skip the catalog"
2. "I already know which fields are unused"
3. "There's only one query, I don't need the full pipeline"
4. "I'll trace and fix at the same time"
5. "The schema is obvious, I don't need to find it"
6. "This fragment is only used once, no need to track consumers"
7. "I'll just remove fields that look unused"
8. "The user wants a quick answer, I'll skip the audit"
9. "Indeterminate probably means unused"
10. "I can split this fragment without checking all consumers"
11. "The trace was thorough enough — I don't need the audit phase"
12. "This field name sounds like it's unused"
13. "I'll fix the fragments later, let me just fix the queries now"

### Common Rationalizations (11 rows)

| Rationalization | Reality |
|-----------------|---------|
| "Only one query, skip the catalog" | One query still needs schema validation and user confirmation |
| "Fields are obviously unused" | Obvious to you is not obvious — trace the code |
| "Indeterminate means probably unused" | Indeterminate means you don't know. Leave it. |
| "I'll combine Trace and Fix" | Combining phases skips the user validation gate |
| "The user said to just fix it" | The user approved the pipeline. Follow it. |
| "This field can't possibly be used" | Prove it by finding the consumer, or mark indeterminate |
| "Fragments are just inlined queries" | Fragments have multiple consumers. Removing a field from a fragment affects every query that spreads it. |
| "One consumer doesn't use this fragment field, so remove it" | All consumers must be checked. One unused consumer doesn't make the field unused. |
| "I can skip the audit for simple codebases" | Simple codebases still benefit from severity ranking and the user review gate |
| "Fragment splitting is obvious, no need to present options" | The user decides whether to split. Present the recommendation and wait. |
| "Post-fix verification is just re-reading files" | Verification confirms no used or indeterminate fields were removed. It's a safety check, not busywork. |

### Verification Checklist (8 items)

- [ ] Every query and fragment in the codebase was cataloged
- [ ] User validated the catalog before tracing began
- [ ] Every operation's field usage was traced through consuming code
- [ ] User validated the trace results before audit began
- [ ] Audit report includes severity rankings and fragment analysis
- [ ] User selected fix scope before any edits were applied
- [ ] All modified files contain valid GraphQL syntax
- [ ] No used or indeterminate fields were removed

### When Stuck (4 rows)

| Problem | Solution |
|---------|----------|
| Schema not found locally | Ask user for file path, introspection URL, or registry ID |
| Trace hits dynamic access pattern | Mark all fields on that path as indeterminate |
| Fragment has no discoverable consumers | Ask user — it may be dead code, or consumed by another repo |
| Modified query fails syntax check | Undo the edit, re-examine the original, fix manually |

## Supporting File: `tracing-patterns.md`

Language-specific field-access patterns reference (~150 lines). Consulted during Phase 2 when the agent encounters unfamiliar consumer patterns.

| Section | Covers |
|---------|--------|
| **JavaScript/TypeScript** | Destructuring, hooks (`useQuery`, `useSuspenseQuery`), render props, HOCs, Apollo/urql/Relay patterns |
| **Python** | `client.execute()`, Strawberry/Ariadne/graphene consumer patterns, dataclass mapping |
| **Go** | Struct-based query variables, `shurcooL/graphql` and `Khan/genqlient` patterns |
| **General** | Spread operators, dynamic access, serialization sinks, cross-file import tracing |

## Plugin Structure

```
plugins/graphql-overfetch-analyzer/
├── .claude-plugin/
│   └── plugin.json          # version: 1.0.0
├── skills/
│   └── analyze-overfetch/
│       ├── SKILL.md          # Complete rewrite
│       └── tracing-patterns.md  # NEW: language-specific reference
└── CHANGELOG.md              # 1.0.0 entry
```

## Out of Scope

- Schema modifications
- TypeScript generated type updates (re-run codegen)
- Mutation or subscription analysis
- External tool execution
