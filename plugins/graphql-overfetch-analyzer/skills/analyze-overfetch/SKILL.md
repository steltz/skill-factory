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
