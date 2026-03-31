# GraphQL Over-Fetch Analyzer v2 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite the `analyze-overfetch` skill from a 3-phase technique to a 4-phase discipline skill with fragment analysis, stronger enforcement, and a supporting reference file.

**Architecture:** Complete rewrite of SKILL.md with 4-phase rigid pipeline (Inventory → Trace → Audit → Fix). New `tracing-patterns.md` supporting file for language-specific reference. Discipline-enforcing structure modeled after `test-driven-development`: Iron Law, rationalizations table, red flags, verification checklist, When Stuck table.

**Tech Stack:** SKILL.md (markdown), tracing-patterns.md (markdown), plugin.json (JSON), CHANGELOG.md (markdown)

---

## File Structure

```
plugins/graphql-overfetch-analyzer/
├── .claude-plugin/
│   └── plugin.json                    # Modify: bump version 0.1.0 → 1.0.0
├── skills/
│   └── analyze-overfetch/
│       ├── SKILL.md                   # Modify: complete rewrite
│       └── tracing-patterns.md        # Create: language-specific tracing reference
└── CHANGELOG.md                       # Modify: add 1.0.0 entry
```

---

### Task 1: Write Baseline Test (RED)

The baseline test verifies that without the v2 skill content, Claude would NOT follow the new 4-phase pipeline with fragment analysis. The existing v0.1.0 skill has a 3-phase pipeline without fragment support — this serves as the failing state.

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Add baseline test comment to existing SKILL.md**

Add this comment block at the top of SKILL.md, immediately after the frontmatter closing `---`:

```markdown
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
```

- [ ] **Step 2: Verify baseline test fails**

Read the current v0.1.0 SKILL.md body. Confirm it does NOT contain:
- Fragment discovery or fragment catalog sections
- A 4-phase pipeline (it uses 3-phase: Discover → Trace → Fix)
- An Audit phase with severity rankings
- Fragment splitting recommendations
- A verification checklist with checkbox items
- A When Stuck table

Expected: FAIL — the v0.1.0 skill does not produce any of the 10 behaviors.

- [ ] **Step 3: Commit the baseline test**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "test(analyze-overfetch): add v2 baseline test for 4-phase pipeline with fragment analysis"
```

---

### Task 2: Rewrite Skill — Frontmatter, Overview, Iron Law, Checklist (GREEN)

Replace the entire SKILL.md with the v2 structure. This task covers everything above Phase 1.

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Replace SKILL.md with v2 skeleton**

Replace the entire file contents with:

````markdown
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
````

- [ ] **Step 2: Verify skeleton structure**

Read the file. Confirm:
- Frontmatter `name` matches directory name (`analyze-overfetch`)
- Frontmatter `description` starts with "Use when"
- Baseline test comment is present
- Iron Law is in a code block with the new 4-phase text
- Checklist has 8 items matching the spec
- Anti-rationalization phrase is present

- [ ] **Step 3: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "feat(analyze-overfetch): rewrite skeleton with 4-phase pipeline and Iron Law"
```

---

### Task 3: Write Phase 1 — Inventory (GREEN continued)

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Append Phase 1 content after the checklist**

Add the following after the checklist section:

````markdown
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
````

- [ ] **Step 2: Verify Phase 1 covers spec requirements**

Check against spec:
- Schema discovery (3 patterns + user fallback) — Step 1.1 ✓
- Query discovery (4 patterns) — Step 1.2 ✓
- Fragment discovery (5 items including dependency graph) — Step 1.3 ✓
- Catalog output: Operations (4 fields) and Fragments (5 fields) — Step 1.4 ✓
- Hard gate with user validation — Phase 1 Gate ✓

- [ ] **Step 3: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "feat(analyze-overfetch): write Phase 1 Inventory with fragment discovery and catalog"
```

---

### Task 4: Write Phase 2 — Trace (GREEN continued)

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Append Phase 2 content after Phase 1 Gate**

Add the following after the Phase 1 Gate section:

````markdown
## Phase 2: Trace

For each operation and fragment in the validated catalog, trace how the result is consumed in application code. Consult `tracing-patterns.md` for language-specific patterns when encountering unfamiliar consumer code.

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

### Step 2.4: Fragment-Aware Tracing

When a query spreads a fragment, trace the fragment's fields through the query's consumer:

1. Identify which fields in the consumer come from the spread fragment
2. Track access to those fields using the same rules as Step 2.2
3. Record which consumer (operation) contributed each "used" mark

**Multi-consumer tracking:** Fragments spread by multiple queries get per-consumer tracking. A fragment field is "used" if *any* consumer of that fragment accesses it. Track usage per-consumer for the audit phase.

### Step 2.5: Build the Usage Map

For each operation and fragment, produce:

| Category | Fields |
|----------|--------|
| **Fetched** | All fields selected in the query or fragment |
| **Used** | Fields with confirmed access in consuming code |
| **Unused** | Fetched minus Used minus Indeterminate |
| **Indeterminate** | Fields where usage could not be determined |
| **Consumers** | (fragments only) Which operations contributed each "used" mark |

### Step 2.6: Compile the Trace Report

For each operation or fragment that has unused fields:

```
Query: GetUser (src/queries/user.graphql:3)
Consumer: src/components/UserProfile.tsx:15
Unused fields: user.createdAt, user.updatedAt, user.posts.body
Indeterminate fields: user.metadata (spread into untyped object)

Fragment: UserFields (src/fragments/user.graphql:1)
Consumers: GetUser, GetUserList, GetUserProfile
Used by all: id, name, email
Used by some: avatar (GetUserProfile only), role (GetUser only)
Unused by all: createdAt, lastLoginIp
Indeterminate: metadata (GetUserList spreads into untyped object)
```

### Phase 2 Gate

<HARD-GATE>
STOP. Present the trace report and ask:

> "Here are the tracing results for N operations and M fragments. For each I've listed:
> - Fields confirmed unused
> - Fields marked indeterminate (conservatively kept)
> - Fragment fields with per-consumer usage breakdown
>
> Please review. Are any findings incorrect? Confirm to proceed to audit."

Do NOT proceed to Phase 3 until the user confirms.
</HARD-GATE>
````

- [ ] **Step 2: Verify Phase 2 covers spec requirements**

Check against spec:
- Find consumer — Step 2.1 ✓
- Track field access (6 patterns) — Step 2.2 ✓
- Handle indirection (6 patterns) — Step 2.3 ✓
- Fragment-aware tracing with multi-consumer tracking — Step 2.4 ✓
- Usage map with consumers column — Step 2.5 ✓
- Conservative analysis (indeterminate handling) — Step 2.3 ✓
- Hard gate — Phase 2 Gate ✓
- Reference to tracing-patterns.md — Phase 2 intro ✓

- [ ] **Step 3: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "feat(analyze-overfetch): write Phase 2 Trace with fragment-aware multi-consumer tracking"
```

---

### Task 5: Write Phase 3 — Audit (GREEN continued)

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Append Phase 3 content after Phase 2 Gate**

Add the following after the Phase 2 Gate section:

````markdown
## Phase 3: Audit

Cross-reference trace results across all operations and fragments. Produce a structured findings report with severity rankings.

### Step 3.1: Per-Query Findings

For each query with unused fields in its own selection (not from fragments), record:
- Query name and file/line
- Consumer file/line
- List of unused fields
- List of indeterminate fields

### Step 3.2: Per-Fragment Findings

For each fragment with fields unused across ALL consumers:
- Fragment name and file/line
- List of fields unused by every consumer
- These are safe to remove from the fragment definition

### Step 3.3: Fragment Splitting Recommendations

For each fragment with fields used by SOME consumers but not others:
- Fragment name
- Fields used by all consumers (base fragment)
- Fields used by specific consumers only (consumer-specific fragments)
- Recommend splitting: `UserFields` → `UserFieldsBase` + `UserFieldsProfile` + `UserFieldsAdmin`

### Step 3.4: Severity Ranking

Rank all findings:

| Severity | Criteria | Action |
|----------|----------|--------|
| **High** | Field unused by all consumers, easy removal | Remove field from query or fragment |
| **Medium** | Fragment field used by some consumers but not others | Recommend fragment split |
| **Low** | Single unused field in a small query, low payload impact | Remove field (optional) |

### Step 3.5: Compile the Audit Report

Present findings grouped by severity (high first):

```
HIGH SEVERITY:
1. GetUserProfile — 5 unused fields (src/queries/user.graphql:3)
2. Fragment UserFields — 2 fields unused by all 3 consumers

MEDIUM SEVERITY:
3. Fragment UserFields — avatar used by 1/3 consumers, role used by 1/3 consumers
   Recommendation: Split into UserFieldsBase + UserFieldsProfile + UserFieldsAdmin

LOW SEVERITY:
4. GetSettings — 1 unused field (src/queries/settings.graphql:8)
```

### Phase 3 Gate

<HARD-GATE>
STOP. Present the audit report and ask:

> "Here are the findings ranked by severity. How would you like to proceed?
> 1. **Fix all** — Remove all unused fields and split recommended fragments
> 2. **Fix per-finding** — Walk through each finding, confirm individually
> 3. **Split fragments** — Only apply fragment splitting recommendations
> 4. **Report only** — No changes, keep the report as documentation"

Do NOT proceed to Phase 4 until the user selects a fix scope.
</HARD-GATE>
````

- [ ] **Step 2: Verify Phase 3 covers spec requirements**

Check against spec:
- Per-query findings — Step 3.1 ✓
- Per-fragment findings (fields unused across all consumers) — Step 3.2 ✓
- Fragment splitting recommendations — Step 3.3 ✓
- Severity ranking (High/Medium/Low) — Step 3.4 ✓
- Audit report grouped by severity — Step 3.5 ✓
- Hard gate with 4 fix scope options — Phase 3 Gate ✓

- [ ] **Step 3: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "feat(analyze-overfetch): write Phase 3 Audit with severity ranking and fragment splitting"
```

---

### Task 6: Write Phase 4 — Fix (GREEN continued)

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Append Phase 4 content after Phase 3 Gate**

Add the following after the Phase 3 Gate section:

````markdown
## Phase 4: Fix

Apply user-approved edits to query and fragment definitions.

### Step 4.1: Apply Field Removals

For each finding the user has confirmed:

1. Open the file containing the query or fragment definition
2. Remove each unused field from the selection set
3. If removing a field leaves an empty selection set on a nested object, remove the entire nested selection (e.g., if `posts { body }` had `body` removed and no other fields remain, remove the `posts` selection entirely)
4. Preserve formatting, comments, and field aliases
5. Do NOT modify indeterminate fields

### Step 4.2: Apply Fragment Splits

For each fragment splitting the user has confirmed:

1. Create the base fragment containing fields used by all consumers
2. Create consumer-specific fragments containing fields used only by those consumers
3. Update each query to spread the base fragment plus the relevant consumer-specific fragment
4. Remove the original fragment definition
5. Verify each query still selects the same fields it did before (no fields lost, no fields added)

### Step 4.3: Verify Syntax

After applying all edits, re-read each modified file and verify:

- The query or fragment still parses as valid GraphQL (balanced braces, no trailing commas, no empty selection sets)
- No fields were accidentally removed that were marked as used or indeterminate
- Fragment spreads reference fragments that still exist
- If any syntax issue is found, fix it immediately

### Post-Fix Guidance

After all fixes are applied, tell the user:

> "Fixes applied. Follow-up actions:
> - Re-run GraphQL codegen if your project uses it (e.g., `graphql-codegen`, `relay-compiler`)
> - Run your test suite to verify nothing broke
> - Check if removed fields should also be removed from other shared fragments
> - If fragments were split, update any documentation that references the original fragment names"
````

- [ ] **Step 2: Verify Phase 4 covers spec requirements**

Check against spec:
- Remove unused fields — Step 4.1 ✓
- Split fragments if selected — Step 4.2 ✓
- Empty selection set removal — Step 4.1 item 3 ✓
- Preserve formatting/comments/aliases — Step 4.1 item 4 ✓
- Do NOT modify indeterminate — Step 4.1 item 5 ✓
- Verify syntax — Step 4.3 ✓
- Fragment spread validity after splits — Step 4.3 ✓
- Post-fix guidance (4 items) — Post-Fix Guidance ✓

- [ ] **Step 3: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "feat(analyze-overfetch): write Phase 4 Fix with fragment splitting and syntax verification"
```

---

### Task 7: Close Loopholes (REFACTOR)

Add discipline-enforcing sections that prevent Claude from rationalizing its way out of the pipeline.

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Append enforcement sections after Post-Fix Guidance**

Add the following after the Post-Fix Guidance section:

````markdown
## Red Flags

These thoughts mean STOP — you are about to violate the Iron Law:

- "The queries are simple, I can skip the catalog"
- "I already know which fields are unused"
- "There's only one query, I don't need the full pipeline"
- "I'll trace and fix at the same time"
- "The schema is obvious, I don't need to find it"
- "This fragment is only used once, no need to track consumers"
- "I'll just remove fields that look unused"
- "The user wants a quick answer, I'll skip the audit"
- "Indeterminate probably means unused"
- "I can split this fragment without checking all consumers"
- "The trace was thorough enough — I don't need the audit phase"
- "This field name sounds like it's unused"
- "I'll fix the fragments later, let me just fix the queries now"

## Common Rationalizations

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

## Verification Checklist

Before declaring work complete, verify each item:

- [ ] Every query and fragment in the codebase was cataloged
- [ ] User validated the catalog before tracing began
- [ ] Every operation's field usage was traced through consuming code
- [ ] User validated the trace results before audit began
- [ ] Audit report includes severity rankings and fragment analysis
- [ ] User selected fix scope before any edits were applied
- [ ] All modified files contain valid GraphQL syntax
- [ ] No used or indeterminate fields were removed

## When Stuck

| Problem | Solution |
|---------|----------|
| Schema not found locally | Ask user for file path, introspection URL, or registry ID |
| Trace hits dynamic access pattern | Mark all fields on that path as indeterminate |
| Fragment has no discoverable consumers | Ask user — it may be dead code, or consumed by another repo |
| Modified query fails syntax check | Undo the edit, re-examine the original, fix manually |

## Out of Scope

This skill does NOT:
- Modify the GraphQL schema
- Update TypeScript generated types (re-run codegen instead)
- Analyze mutations or subscriptions (over-fetching is a query concern)
- Run external tools or scripts
````

- [ ] **Step 2: Verify enforcement sections cover spec requirements**

Check against spec:
- Red Flags (13 items) — ✓ all 13 present
- Common Rationalizations (11 rows) — ✓ all 11 present
- Verification Checklist (8 items) — ✓ all 8 present
- When Stuck (4 rows) — ✓ all 4 present
- Out of Scope (4 items) — ✓ all 4 present

- [ ] **Step 3: Validate the plugin**

Run: `bin/validate-plugin plugins/graphql-overfetch-analyzer`

Expected: All checks pass.

- [ ] **Step 4: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "refactor(analyze-overfetch): add enforcement sections — red flags, rationalizations, verification checklist"
```

---

### Task 8: Create Supporting File — tracing-patterns.md (GREEN continued)

**Files:**
- Create: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/tracing-patterns.md`

- [ ] **Step 1: Write the tracing patterns reference file**

Create `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/tracing-patterns.md` with this content:

````markdown
# Tracing Patterns Reference

Language-specific field-access patterns for Phase 2 (Trace). Consult this reference when encountering unfamiliar consumer code patterns.

## JavaScript / TypeScript

### Hook-Based Consumers (React)

```typescript
// Apollo Client
const { data, loading } = useQuery(GET_USER);
// trace: data

// Apollo useSuspenseQuery
const { data } = useSuspenseQuery(GET_USER);
// trace: data

// urql
const [result] = useQuery({ query: GET_USER });
// trace: result.data

// Relay
const data = useLazyLoadQuery(GetUserQuery, {});
// trace: data
// Relay also uses useFragment — trace the fragment ref
const user = useFragment(UserFragment, userRef);
// trace: user (all fragment fields potentially used)
```

### Render Props and HOCs

```typescript
// Apollo render prop (legacy)
<Query query={GET_USER}>
  {({ data }) => <Profile user={data.user} />}
</Query>
// trace: data in the render function, follow into <Profile>

// graphql() HOC (legacy)
const Enhanced = graphql(GET_USER)(Component);
// trace: props.data in Component
```

### Direct Client Calls

```typescript
// Apollo client.query
const { data } = await client.query({ query: GET_USER });
// trace: data

// graphql-request
const data = await request(endpoint, GET_USER);
// trace: data (top-level fields are query root fields)

// fetch + manual parsing
const res = await fetch('/graphql', { body: JSON.stringify({ query }) });
const { data } = await res.json();
// trace: data
```

### Fragment Patterns (JS/TS)

```typescript
// Colocated fragment with useFragment (Relay, Apollo 3.8+)
const user = useFragment(UserFieldsFragmentDoc, data.user);
// trace: user — all fields of UserFields are potentially accessed

// Fragment spread in query definition
const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      ...UserFields
      extraField
    }
  }
  ${USER_FIELDS_FRAGMENT}
`;
// trace: data.user.extraField directly, UserFields fields via fragment tracing
```

## Python

### Client Libraries

```python
# graphql-core / Ariadne
result = graphql_sync(schema, query)
# trace: result.data

# gql (graphql-transport)
result = client.execute(query)
# trace: result (dict — field access via result["user"]["name"])

# Strawberry (async)
result = await schema.execute(query)
# trace: result.data

# sgqlc
op = Operation(Query)
op.user(id=1)
result = endpoint(op)
# trace: result.user (typed access)
```

### Field Access Patterns

```python
# Dict access
data["user"]["name"]  # → user.name used

# .get() with default
data.get("user", {}).get("name")  # → user.name used

# Unpacking
user = data["user"]
name, email = user["name"], user["email"]
# → user.name, user.email used

# Dataclass mapping
@dataclass
class User:
    name: str
    email: str
user = User(**data["user"])
# → if only name and email in dataclass, mark other user fields as unused
# → if **kwargs or extra fields accepted, mark as indeterminate
```

## Go

### Client Libraries

```go
// shurcooL/graphql
var query struct {
    User struct {
        Name  string
        Email string
    } `graphql:"user(id: $id)"`
}
err := client.Query(ctx, &query, vars)
// trace: query.User.Name, query.User.Email
// Go struct fields map directly to GraphQL fields — unused struct fields = unused query fields

// Khan/genqlient
resp, err := getUser(ctx, client, id)
// trace: resp.User.Name, resp.User.Email (generated types)

// machinebox/graphql
req := graphql.NewRequest(query)
var resp struct { User UserData }
err := client.Run(ctx, req, &resp)
// trace: resp.User fields
```

### Struct-Based Tracing

In Go, the struct definition IS the usage map. Fields defined in the struct are "used" by definition (Go will not compile with unused struct fields if they are unexported and accessed). Check which struct fields are actually accessed in subsequent code — a field present in the struct but never read after unmarshaling is still fetched but unused.

## General Patterns

### Spread Operators

```typescript
// Typed spread — all fields used
const profile: UserProfile = { ...data.user };
// → all fields of UserProfile type are used

// Untyped spread — indeterminate
const obj = { ...data.user };
// → mark all user fields as indeterminate (can't determine which are accessed)

// Partial spread with override — indeterminate
const merged = { ...data.user, name: "override" };
// → mark all user fields as indeterminate (spread is opaque)
```

### Serialization Sinks (Always Indeterminate)

These patterns make ALL fields on the path indeterminate:

- `JSON.stringify(data)` / `json.dumps(data)` / `json.Marshal(data)`
- `console.log(data)` / `print(data)` / `fmt.Println(data)`
- `localStorage.setItem("cache", JSON.stringify(data))`
- `sendToAnalytics(data)` (any opaque function call)
- `Object.keys(data.user)` / `Object.values(data.user)`
- `for (const key in data.user)` / `for key in data["user"]:`

### Cross-File Import Tracing

When the query consumer imports a function or component and passes data to it:

1. Follow the import to the target file
2. Read the function/component signature
3. Trace field access within that function (one level deep only)
4. If the function re-exports or passes data further, mark remaining fields as indeterminate
````

- [ ] **Step 2: Verify supporting file covers spec requirements**

Check against spec's tracing-patterns.md requirements:
- JavaScript/TypeScript: hooks, render props, HOCs, Apollo/urql/Relay — ✓
- Python: client.execute(), Strawberry/Ariadne/graphene, dataclass mapping — ✓
- Go: struct-based query variables, shurcooL/graphql, Khan/genqlient — ✓
- General: spread operators, dynamic access, serialization sinks, cross-file import tracing — ✓

- [ ] **Step 3: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/tracing-patterns.md
git commit -m "feat(analyze-overfetch): add tracing-patterns.md supporting file for language-specific patterns"
```

---

### Task 9: Remove Baseline Test Comment

**Files:**
- Modify: `plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md`

- [ ] **Step 1: Remove the baseline test HTML comment**

The `<!-- BASELINE TEST (remove after GREEN) ... -->` comment block was added in Task 1. Now that the skill is complete (GREEN + REFACTOR), remove it. The comment starts with `<!-- BASELINE TEST` and ends with `-->`.

- [ ] **Step 2: Validate the plugin**

Run: `bin/validate-plugin plugins/graphql-overfetch-analyzer`

Expected: All checks pass.

- [ ] **Step 3: Commit**

```bash
git add plugins/graphql-overfetch-analyzer/skills/analyze-overfetch/SKILL.md
git commit -m "chore(analyze-overfetch): remove baseline test comment after GREEN"
```

---

### Task 10: Final Review

- [ ] **Step 1: Read the complete SKILL.md end to end**

Verify:
- No placeholder text, TODOs, or incomplete sections
- All 4 phase gates are present and use `<HARD-GATE>` tags
- Iron Law is in a code block with the 4-phase pipeline text
- Frontmatter `name` matches directory name (`analyze-overfetch`)
- Frontmatter `description` starts with "Use when"
- Anti-rationalization phrase is present
- All 13 red flags are listed
- Common Rationalizations table has 11 rows
- Verification Checklist has 8 items
- When Stuck table has 4 rows
- No `@` links or file path references to other skills (tracing-patterns.md reference is OK — it's within the same skill)

- [ ] **Step 2: Read tracing-patterns.md end to end**

Verify:
- 4 language sections (JS/TS, Python, Go, General)
- Code examples are correct and complete
- No placeholder text

- [ ] **Step 3: Run final plugin validation**

Run: `bin/validate-plugin plugins/graphql-overfetch-analyzer`

Expected: All checks pass.

- [ ] **Step 4: Verify git status is clean**

Run: `git status`

Expected: Nothing unstaged or untracked in the plugin directory.
