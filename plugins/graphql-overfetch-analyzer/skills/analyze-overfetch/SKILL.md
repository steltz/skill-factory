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

Placeholder — skill content will be authored via TDD in subsequent tasks.
