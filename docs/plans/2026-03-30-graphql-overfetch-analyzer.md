# GraphQL Over-Fetch Analyzer Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a standalone Claude Code skill plugin that discovers GraphQL queries in a codebase, traces field usage in consuming code, and auto-fixes over-fetched fields.

**Architecture:** Single SKILL.md in a standalone plugin. The skill is rigid and discipline-enforcing — three phases (Discover → Trace → Fix) with hard gates between each. No supporting files, hooks, or external dependencies. All analysis is Claude's reasoning guided by the skill instructions.

**Tech Stack:** SKILL.md (markdown), plugin.json (JSON), CHANGELOG.md (markdown)

---

## File Structure

```
plugins/graphql-overfetch-analyzer/
├── .claude-plugin/
│   └── plugin.json        # Plugin metadata (name, description, version)
├── skills/
│   └── analyze-overfetch/
│       └── SKILL.md        # The skill — all three phases, hard gates, checklist
└── CHANGELOG.md            # Keep a Changelog format
```

---

### Task 1: Scaffold the Plugin

**Files:**
- Create: `plugins/graphql-overfetch-analyzer/.claude-plugin/plugin.json`
- Create: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md` (placeholder)
- Create: `plugins/graphql-overfetch-analyzer/CHANGELOG.md`

- [ ] **Step 1: Create plugin.json**

```json
{
  "name": "graphql-overfetch-analyzer",
  "description": "Analyze GraphQL queries for over-fetching of data",
  "version": "0.1.0"
}
```

Write this to `plugins/graphql-overfetch-analyzer/.claude-plugin/plugin.json`.

- [ ] **Step 2: Create placeholder SKILL.md**

```markdown
---
name: analyze-overfetch
description: Use when analyzing a codebase for GraphQL queries that fetch more data than is consumed by the application code
---

# Analyze Over-Fetch

Placeholder — skill content will be authored via TDD in subsequent tasks.
```

Write this to `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`.

- [ ] **Step 3: Create CHANGELOG.md**

```markdown
# Changelog

## [0.1.0] - 2026-03-30

### Added

- analyze-overfetch skill for discovering and fixing GraphQL query over-fetching
```

Write this to `plugins/graphql-overfetch-analyzer/CHANGELOG.md`.

- [ ] **Step 4: Validate the scaffold**

Run: `bin/validate-plugin plugins/graphql-overfetch-analyzer`

Expected: All checks pass (frontmatter valid, plugin.json valid, naming matches, semver valid, changelog present).

- [ ] **Step 5: Commit the scaffold**

```bash
git add plugins/graphql-overfetch-analyzer
git commit -m "feat(graphql-overfetch-analyzer): scaffold plugin with placeholder skill"
```

---

### Task 2: Write Baseline Test for the Skill (RED)

The baseline test verifies that without proper skill content, Claude would NOT follow the rigid three-phase pipeline. This is the RED phase of TDD — the test must fail before we write the skill.

**Files:**
- Test: manual verification — invoke the placeholder skill and confirm Claude does NOT produce phased output with hard gates

- [ ] **Step 1: Define the baseline test**

The baseline test scenario: Given a codebase with GraphQL queries that over-fetch data, invoke the `analyze-overfetch` skill. Without the real skill content, Claude should NOT:

1. Produce a structured three-phase pipeline (Discover → Trace → Fix)
2. Present a query catalog and wait for user validation before tracing
3. Present an over-fetch report and wait for user validation before fixing
4. Offer three fix scope options (fix all, fix per-query, report only)

Write the baseline test definition to a test notes section at the top of SKILL.md (temporary — will be removed after GREEN phase):

```markdown
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
```

- [ ] **Step 2: Run the baseline test**

Invoke the placeholder skill mentally or in a test session. Confirm that the placeholder content does NOT cause Claude to execute the eight behaviors listed above. The placeholder says nothing about phases, catalogs, tracing, or fix scopes — so Claude would improvise rather than following the spec's rigid pipeline.

Expected: FAIL — the placeholder skill does not produce the required behavior.

- [ ] **Step 3: Commit the baseline test**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "test(analyze-overfetch): add baseline test for three-phase pipeline"
```

---

### Task 3: Write the Skill — Phase 1: Discover (GREEN)

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Write the frontmatter, overview, iron law, and checklist skeleton**

Replace the entire SKILL.md with the following (keep the baseline test comment — it gets removed in Task 7):

````markdown
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
````

- [ ] **Step 2: Verify the Phase 1 content covers the spec**

Check against the spec's Phase 1 requirements:
- Schema discovery (5 steps) — covered in Step 1.1
- Query discovery (4 patterns) — covered in Step 1.2
- Catalog output (4 fields) — covered in Step 1.3
- Hard gate — covered in Phase 1 Gate

Expected: All Phase 1 spec requirements are addressed.

- [ ] **Step 3: Commit Phase 1**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "feat(analyze-overfetch): write Phase 1 Discover with schema/query discovery and catalog"
```

---

### Task 4: Write the Skill — Phase 2: Trace (GREEN continued)

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Append Phase 2 content to SKILL.md**

Add the following after the Phase 1 Gate section:

````markdown
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
````

- [ ] **Step 2: Verify Phase 2 content covers the spec**

Check against the spec's Phase 2 requirements:
- Tracing strategy (4 steps) — covered in Steps 2.1-2.4
- Conservative analysis (4 indeterminate scenarios) — covered in Step 2.3
- What gets flagged (3 items) — covered in Step 2.4 (Unused category)
- What does NOT get flagged (3 items) — conditional branches and type assertions count as used (implicit in Step 2.2), indeterminate not flagged (explicit in Step 2.3)
- Over-fetch report (4 fields) — covered in Step 2.5
- Hard gate — covered in Phase 2 Gate

Expected: All Phase 2 spec requirements are addressed.

- [ ] **Step 3: Commit Phase 2**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "feat(analyze-overfetch): write Phase 2 Trace with usage mapping and conservative analysis"
```

---

### Task 5: Write the Skill — Phase 3: Fix (GREEN continued)

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Append Phase 3 content to SKILL.md**

Add the following after the Phase 2 Gate section:

````markdown
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

## Out of Scope

This skill does NOT:
- Modify the GraphQL schema
- Update TypeScript generated types (re-run codegen instead)
- Analyze mutations or subscriptions (over-fetching is a query concern)
- Run external tools or scripts
````

- [ ] **Step 2: Verify Phase 3 content covers the spec**

Check against the spec's Phase 3 requirements:
- Fix workflow (4 steps) — covered in Steps 3.1-3.4
- Fix scope options (3 choices) — covered in Step 3.2
- Preserve formatting/comments/aliases — covered in Step 3.3
- Empty selection set removal — covered in Step 3.3
- Verify syntax — covered in Step 3.4
- Out of scope (3 items) — covered in Out of Scope section
- Post-fix guidance (3 reminders) — covered in Post-Fix Guidance

Expected: All Phase 3 spec requirements are addressed.

- [ ] **Step 3: Commit Phase 3**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "feat(analyze-overfetch): write Phase 3 Fix with auto-fix workflow and syntax verification"
```

---

### Task 6: Close Loopholes (REFACTOR)

Add discipline-enforcing sections that prevent Claude from rationalizing its way out of the rigid pipeline.

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Add Red Flags and Common Rationalizations**

Append the following before the Out of Scope section:

````markdown
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
````

- [ ] **Step 2: Add When to Use / When NOT to Use**

Insert after the overview paragraph, before the iron law:

````markdown
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
````

- [ ] **Step 3: Verify the complete skill covers all spec requirements**

Read the full SKILL.md and check every spec section:

1. Plugin identity (name, trigger, type) — frontmatter ✓
2. Three-phase pipeline — checklist + three phase sections ✓
3. Phase 1: Discover (schema, queries, catalog, gate) ✓
4. Phase 2: Trace (consumer, tracking, indirection, usage map, report, gate) ✓
5. Phase 3: Fix (report, scope, edits, verify, post-fix, out of scope) ✓
6. Rigid enforcement — iron law, red flags, rationalizations ✓
7. Framework-agnostic — no framework-specific assumptions in phases ✓
8. Conservative defaults — indeterminate handling ✓

- [ ] **Step 4: Validate the plugin**

Run: `bin/validate-plugin plugins/graphql-overfetch-analyzer`

Expected: All checks pass.

- [ ] **Step 5: Commit the loophole-closing additions**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "refactor(analyze-overfetch): add red flags, rationalizations, and usage guards"
```

---

### Task 7: Remove Baseline Test Comment

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Remove the baseline test HTML comment**

The `<!-- BASELINE TEST (remove after GREEN) ... -->` comment block was added in Task 2. Now that the skill is complete (GREEN), remove it. The comment starts with `<!-- BASELINE TEST` and ends with `-->`.

- [ ] **Step 2: Validate the plugin**

Run: `bin/validate-plugin plugins/graphql-overfetch-analyzer`

Expected: All checks pass.

- [ ] **Step 3: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "chore(analyze-overfetch): remove baseline test comment after GREEN"
```

---

### Task 8: Final Review

- [ ] **Step 1: Read the complete SKILL.md end to end**

Verify:
- No placeholder text, TODOs, or incomplete sections
- All phase gates are present and use `<HARD-GATE>` tags
- Iron law is in a code block
- Frontmatter `name` matches directory name (`analyze-overfetch`)
- Frontmatter `description` starts with "Use when"
- No `@` links or file path references to other skills

- [ ] **Step 2: Run final plugin validation**

Run: `bin/validate-plugin plugins/graphql-overfetch-analyzer`

Expected: All checks pass.

- [ ] **Step 3: Verify git status is clean**

Run: `git status`

Expected: Nothing unstaged or untracked in the plugin directory.
