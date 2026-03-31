---
name: analyze-overfetch
description: Use when analyzing a codebase for GraphQL queries that fetch more data than is consumed by the application code
---

# Analyze Over-Fetch

Discover GraphQL queries and fragments in a codebase, trace how their results are consumed, audit findings across operations, and auto-fix over-fetched fields.

Violating the letter of the rules is violating the spirit of the rules.

## When to Use

- Codebase has GraphQL queries and you want to find unused fields
- Performance optimization — reducing payload size
- Code cleanup — removing dead query fields after feature removal
- Pre-migration audit — understanding what data is actually consumed
- Fragment analysis — finding fields in shared fragments that no consumer uses
- Route-specific audit — analyzing only the queries used by a single page or route

## When NOT to Use

- No GraphQL in the codebase
- Schema-only repos with no query consumers
- Queries are dynamically generated at runtime (the skill traces static code)
- User wants to analyze subscriptions or mutations (out of scope)
- Route is server-rendered with no client-side GraphQL queries (nothing to trace)

```
NO FIELD MAY BE REMOVED WITHOUT A TRACED CONSUMER AND AN AUDITED FINDING.
Inventory → Trace → Audit → Fix. No phase skipped. No phase reordered.
```

## Preamble: Scope Selection

Before starting the pipeline, ask the user:

> "What scope should this analysis cover?
> 1. **Full project** — analyze all GraphQL queries in the codebase
> 2. **Single route** — analyze only the queries used by a specific page or route"

If the user selects **full project**, skip all steps and annotations marked `[SCOPED]` throughout the pipeline. The full-project path is the default and works exactly as before.

If the user selects **single route**, record the target route (e.g., `/users/:id`) and proceed to Step 1.2 after Step 1.1.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Phase 1: Inventory** — find schema, queries, fragments; build catalog
2. **Phase 1 Gate** — present catalog, wait for user validation
3. **Phase 2: Trace** — trace field consumption per operation and fragment
4. **Phase 2 Gate** — present usage maps, wait for user validation
5. **Phase 3: Audit** — cross-reference findings, rank severity, identify query and fragment splits
6. **Phase 3 Gate** — present findings report, user selects fix scope
7. **Phase 4: Fix** — apply field removals, query splits, fragment splits, verify syntax
8. **Phase 4: Verify** — run codegen, run type checks, fix any errors introduced by the refactor
9. **Post-fix verification** — run verification checklist

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

### Step 1.2: Resolve Route Scope `[SCOPED]`

Map the target route to its component set. Consult `route-resolution-patterns.md` for framework-specific conventions.

**Framework detection** — check in this order, stop at the first match:

1. **Next.js App Router** — `app/` directory with `page.tsx` files
2. **Next.js Pages Router** — `pages/` directory with page files
3. **React Router** — `<Routes>`, `createBrowserRouter`, `<Switch>` in source
4. **Vue Router** — `createRouter`, `new VueRouter`, route config arrays
5. **Angular Router** — `RouterModule.forRoot`, `provideRouter`, route configs
6. **Manual fallback** — ask the user to identify the entry component(s) for the route

**Import tree traversal:**

1. From the entry component(s), follow imported child components one level deep
2. Include layout/wrapper components that render the route (parent routes, layouts)
3. Record the complete component set:

| Field | Value |
|-------|-------|
| Target route | The route path the user specified |
| Framework | Detected framework and router type |
| Entry files | Top-level page/route component(s) |
| Full file list | All components in scope (entry + layouts + one-level imports) |

<HARD-GATE>
STOP. Present the component set and ask:

> "I detected [framework] and resolved route `[path]` to these N components:
> [numbered list of files]
>
> Are any components missing or incorrectly included?
> Confirm to proceed to query discovery within this scope."

Do NOT proceed to Step 1.3 until the user confirms.
</HARD-GATE>

### Step 1.3: Find All Query Operations

Search for GraphQL queries using these patterns:

**Standalone files:**
- Glob for `**/*.graphql` and `**/*.gql`
- Read each file; extract operations that start with `query` (catalog but skip `mutation` and `subscription` — they are out of scope for tracing and fixing)

**Inline in code (any language):**
- Grep for tagged template literals: `` gql` ``, `` graphql` ``
- Grep for multi-line strings containing `query \w+` or `query {`
- Grep for framework invocations: `useQuery(`, `client.query(`, `client.execute(`, `graphql(` — then read the referenced query definition

**`[SCOPED]`:** Search only within the confirmed component set from Step 1.2 and their direct imports. Do not search files outside the scope boundary.

### Step 1.4: Find All Fragments

Search for GraphQL fragments alongside queries:

1. `fragment` definitions in `.graphql` / `.gql` files
2. Inline `fragment` definitions in tagged template literals
3. For each fragment, record: name, target type, selected fields, file/line
4. For each query, record which fragments it spreads (`...FragmentName`)
5. Build a dependency graph: queries → fragments → nested fragments (fragments that spread other fragments)

**`[SCOPED]`:** Start from scoped queries discovered in Step 1.3, but follow the fragment dependency graph exhaustively — including fragments defined outside the scope boundary. A scoped query may spread a fragment defined in a shared utilities file; that fragment and its nested fragments must be included. Additionally, for each discovered fragment, search the entire codebase for all consumers (not just scoped ones). Record out-of-scope consumers for boundary fragment detection in Step 1.5.

### Step 1.5: Build the Catalog

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
| Boundary | `[SCOPED]` Yes/No — whether any consumer is outside the scope boundary |

Present the catalog to the user as two numbered lists: Operations and Fragments.

**`[SCOPED]`:** Add scope metadata to each catalog entry. Flag **boundary fragments** — fragments that have at least one consumer outside the confirmed component set. Boundary fragments require conservative handling in later phases because not all consumers can be traced.

### Phase 1 Gate

<HARD-GATE>
STOP. Present the catalog and ask:

> "Here are the N queries and M fragments I found. Please review:
> 1. Are any queries or fragments missing?
> 2. Should any be excluded from analysis?
>
> `[SCOPED]` Note: M of these fragments are **boundary fragments** — they have consumers outside the scoped route. Findings for boundary fragment fields will be preliminary and require a full-project analysis to confirm.
>
> Confirm to proceed to tracing."

Do NOT proceed to Phase 2 until the user confirms.
</HARD-GATE>

## Phase 2: Trace

For each operation and fragment in the validated catalog, trace how the result is consumed in application code. Consult `tracing-patterns.md` for language-specific patterns when encountering unfamiliar consumer code.

**`[SCOPED]`:** Trace only within the confirmed component set. For boundary fragments, out-of-scope consumers are NOT traced — their fields are treated as **indeterminate** (not unused). This ensures scoped analysis never produces false "safe to remove" findings for shared fragments.

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

**`[SCOPED]` Boundary fragment handling:** For boundary fragments, record per-consumer usage only for in-scope consumers. Out-of-scope consumers contribute an indeterminate mark to every field in the fragment. The trace report must show the out-of-scope consumer count.

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

[SCOPED] Boundary Fragment: UserFields (src/fragments/user.graphql:1)
In-scope consumers: GetUser (traced)
Out-of-scope consumers: 2 (NOT traced — fields indeterminate)
Used by in-scope: id, name, email
Unused by in-scope: createdAt, lastLoginIp, avatar, role
Indeterminate (out-of-scope): ALL fields (2 untraced consumers)
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

**`[SCOPED]` Boundary fragment restriction:** A boundary fragment field can only be marked "safe to remove" if ALL consumers (including out-of-scope) have been traced. Since out-of-scope consumers are not traced in scoped mode, boundary fragment fields are never "safe to remove" — they are classified as **Info** severity instead (see Step 3.5).

### Step 3.3: Shared Query Splitting Recommendations

For each query imported by multiple files, check whether each consumer uses a different subset of fields:

1. From the trace results, group consumers by query operation name
2. For each multi-consumer query, compare the per-consumer "used" field sets
3. If consumers use materially different subsets, recommend splitting the query into per-page variants

**Example finding:**

```
Shared Query: GetUser (src/queries/user.graphql:3)
Consumers: 3 files import this query
  - UserProfile.tsx uses: id, name, email, avatar, bio
  - UserListItem.tsx uses: id, name
  - UserSettings.tsx uses: id, name, email, preferences
Recommendation: Split into GetUserProfile, GetUserListItem, GetUserSettings
```

**Split threshold:** Recommend splitting when any consumer uses fewer than 60% of the query's selected fields, OR when the query selects 5+ fields that no single consumer uses entirely. These are guidelines — present the per-consumer usage breakdown and let the user decide.

**`[SCOPED]`:** In scoped mode, only in-scope consumers are traced. Flag queries that have out-of-scope consumers as needing full-project analysis before splitting.

### Step 3.4: Fragment Splitting Recommendations

For each fragment with fields used by SOME consumers but not others:
- Fragment name
- Fields used by all consumers (base fragment)
- Fields used by specific consumers only (consumer-specific fragments)
- Recommend splitting: `UserFields` → `UserFieldsBase` + `UserFieldsProfile` + `UserFieldsAdmin`

**`[SCOPED]`:** Fragment splitting recommendations for boundary fragments are preliminary. Out-of-scope consumers may use fields differently. Recommend a full-project re-run before applying boundary fragment splits.

### Step 3.5: Severity Ranking

Rank all findings:

| Severity | Criteria | Action |
|----------|----------|--------|
| **High** | Field unused by all consumers, easy removal | Remove field from query or fragment |
| **High** | Shared query with consumers using materially different field subsets | Recommend per-page query split |
| **Medium** | Fragment field used by some consumers but not others | Recommend fragment split |
| **Low** | Single unused field in a small query, low payload impact | Remove field (optional) |
| **Info** | `[SCOPED]` Boundary fragment field unused by in-scope consumers, but out-of-scope consumers not traced | No action — requires full-project analysis to confirm |

### Step 3.6: Compile the Audit Report

Present findings grouped by severity (high first):

```
HIGH SEVERITY:
1. GetUserProfile — 5 unused fields (src/queries/user.graphql:3)
2. Fragment UserFields — 2 fields unused by all 3 consumers
3. Shared Query GetUser — 3 consumers use different field subsets
   - UserProfile.tsx: 5/8 fields (63%)
   - UserListItem.tsx: 2/8 fields (25%)
   - UserSettings.tsx: 4/8 fields (50%)
   Recommendation: Split into GetUserProfile, GetUserListItem, GetUserSettings

MEDIUM SEVERITY:
4. Fragment UserFields — avatar used by 1/3 consumers, role used by 1/3 consumers
   Recommendation: Split into UserFieldsBase + UserFieldsProfile + UserFieldsAdmin

LOW SEVERITY:
5. GetSettings — 1 unused field (src/queries/settings.graphql:8)
```

### Phase 3 Gate

<HARD-GATE>
STOP. Present the audit report and ask:

> "Here are the findings ranked by severity. How would you like to proceed?
> 1. **Fix all** — Remove all unused fields, split shared queries, and split recommended fragments
> 2. **Fix per-finding** — Walk through each finding, confirm individually
> 3. **Split queries/fragments** — Only apply query and fragment splitting recommendations
> 4. **Report only** — No changes, keep the report as documentation
>
> `[SCOPED]` Note: Info-severity findings for boundary fragments are preliminary — they reflect only in-scope consumers. A full-project analysis is recommended before acting on boundary fragment findings."

Do NOT proceed to Phase 4 until the user selects a fix scope.
</HARD-GATE>

## Phase 4: Fix

Apply user-approved edits to query and fragment definitions.

**`[SCOPED]` Boundary fragment safety rule:** Do NOT edit boundary fragment definitions unless the user explicitly overrides after a warning that out-of-scope consumers have not been traced. Query-level field removals (fields selected directly in a query, not via a fragment spread) are always safe regardless of scope.

### Step 4.1: Apply Field Removals

For each finding the user has confirmed:

1. Open the file containing the query or fragment definition
2. Remove each unused field from the selection set
3. If removing a field leaves an empty selection set on a nested object, remove the entire nested selection (e.g., if `posts { body }` had `body` removed and no other fields remain, remove the `posts` selection entirely)
4. Preserve formatting, comments, and field aliases
5. Do NOT modify indeterminate fields

### Step 4.2: Apply Shared Query Splits

For each shared query splitting the user has confirmed:

1. Create a new query file (or add to the consumer's file) for each per-page variant
2. Copy the original query, then remove fields not used by that consumer
3. Update each consumer's import to reference its new per-page query
4. If no consumer still uses the original shared query, delete it
5. Verify each new query selects exactly the fields its consumer uses (no fields lost, no fields gained)

**Naming convention:** Append the page or consumer context to the original name — e.g., `GetUser` → `GetUserForProfile`, `GetUserForListItem`, `GetUserForSettings`. Present the proposed names to the user before applying.

### Step 4.3: Apply Fragment Splits

For each fragment splitting the user has confirmed:

1. Create the base fragment containing fields used by all consumers
2. Create consumer-specific fragments containing fields used only by those consumers
3. Update each query to spread the base fragment plus the relevant consumer-specific fragment
4. Remove the original fragment definition
5. Verify each query still selects the same fields it did before (no fields lost, no fields added)

### Step 4.4: Verify Syntax

After applying all edits, re-read each modified file and verify:

- The query or fragment still parses as valid GraphQL (balanced braces, no trailing commas, no empty selection sets)
- No fields were accidentally removed that were marked as used or indeterminate
- Fragment spreads reference fragments that still exist
- If any syntax issue is found, fix it immediately

### Step 4.5: Run Codegen

After all edits are applied and syntax is verified, detect and run the project's GraphQL code generation tool.

**Detection order** — check in this order, stop at the first match:

1. **package.json scripts** — look for scripts named `codegen`, `generate`, `gql:generate`, `graphql:codegen`, or scripts that invoke `graphql-codegen`, `relay-compiler`, `graphql-code-generator`, `gql.tada`, `genql`
2. **Config files** — `codegen.ts`, `codegen.yml`, `codegen.json`, `.graphqlrc.yml`, `.graphqlrc.json`, `relay.config.js`, `relay.config.json`
3. **Build tool plugins** — check `vite.config.*`, `next.config.*`, `webpack.config.*` for GraphQL codegen plugins

If no codegen tool is detected, ask the user:

> "I didn't detect a GraphQL code generation tool. Does this project use one? If so, what command runs it?"

If the user confirms there is no codegen, skip to Step 4.6.

**Run the codegen command** and capture its output. If it fails:

1. Read the error output — codegen failures after field removal typically mean a consumer still references a removed field or type
2. This is a **critical signal** — it means a field was removed that is still referenced somewhere
3. Undo the offending field removal, re-check the trace for that field, and re-run codegen
4. Do NOT proceed to type checking until codegen passes cleanly

### Step 4.6: Run Type Checks

After codegen passes (or was skipped), run the project's type checker to catch any type errors introduced by the refactor.

**Detection order:**

1. **TypeScript** — check for `tsconfig.json`. Run `npx tsc --noEmit` (or the project's type-check script from package.json, e.g., `typecheck`, `type-check`, `tsc`)
2. **Flow** — check for `.flowconfig`. Run `npx flow check`
3. **Python (mypy/pyright)** — check for `mypy.ini`, `pyrightconfig.json`, `pyproject.toml` with `[tool.mypy]` or `[tool.pyright]`. Run the detected tool
4. **Go** — check for `go.mod`. Run `go build ./...`

If no type checker is detected, ask the user:

> "I didn't detect a type checker. Does this project use one? If so, what command runs it?"

If the user confirms there is no type checker, skip to Post-Fix Summary.

**Run the type check command** and capture its output. If it fails:

1. Read the error output — type errors after field removal typically mean consuming code still references a removed field
2. For each type error, determine whether:
   - A field was removed that a consumer still accesses → undo the removal, update the trace
   - Generated types are stale → re-run codegen first (Step 4.5)
   - The type error is pre-existing (existed before this refactor) → note it but do not fix unrelated errors
3. Re-run type checks after each fix until they pass cleanly or only pre-existing errors remain

### Post-Fix Summary

After codegen and type checks pass (or were skipped), tell the user:

> "Fixes applied and verified. Summary:
> - [N] fields removed from [M] queries/fragments
> - [N] queries split into per-page variants (if applicable)
> - [N] fragments split (if applicable)
> - Codegen: [passed / skipped / N/A]
> - Type check: [passed / skipped / N/A]
>
> Follow-up actions:
> - Run your test suite to verify nothing broke
> - If fragments or queries were split, update any documentation that references the original names"

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
- "I know which components this route renders, no need to check the router"
- "This fragment is only used by this route" (without searching for other consumers)
- "I'll skip the component set gate — the user already told me the route"
- "The out-of-scope consumers probably don't use this field either"
- "I can edit this boundary fragment — it's just one field"
- "Every page needs the same query — no point splitting"
- "Codegen passed last time, I can skip it after this change"
- "The type errors are probably pre-existing, I'll ignore them"
- "I'll run codegen later, let me just finish the edits first"

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
| "This fragment is probably only used here" | Prove it by searching the entire codebase for consumers. Boundary detection exists for this reason. |
| "Out-of-scope consumers won't be affected" | You haven't traced them. Treat their fields as indeterminate. |
| "I can infer the route's components from the file structure" | Check the router config. File structure doesn't reveal route groups, parallel routes, or lazy loading. |
| "The component set gate is redundant since the user named the route" | Routes resolve to components through framework conventions. The user confirms the resolution, not the route name. |
| "All pages need the same data, no point splitting the query" | If consumers use different field subsets, they're fetching unnecessary data. Present the per-consumer breakdown and let the user decide. |
| "Codegen is optional, the edits look correct" | Codegen catches references to removed fields that tracing missed. It's a safety net, not busywork. |
| "Type errors are unrelated to my changes" | Verify each error. Type errors after field removal are the exact signal that a consumer still references a removed field. |
| "I'll run codegen and type checks at the end" | Run codegen immediately after edits. Type errors compound — catching them early prevents cascading fixes. |

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
- [ ] Shared queries with divergent consumer usage were flagged for splitting
- [ ] Codegen ran successfully after all edits (or confirmed not applicable)
- [ ] Type checks passed after all edits (or confirmed not applicable)
- [ ] `[SCOPED]` Route was resolved via framework router config, not guessed from file structure
- [ ] `[SCOPED]` User confirmed the component set before query discovery
- [ ] `[SCOPED]` All boundary fragments were identified and flagged
- [ ] `[SCOPED]` No boundary fragment definitions were edited without explicit user override

## When Stuck

| Problem | Solution |
|---------|----------|
| Schema not found locally | Ask user for file path, introspection URL, or registry ID |
| Trace hits dynamic access pattern | Mark all fields on that path as indeterminate |
| Fragment has no discoverable consumers | Ask user — it may be dead code, or consumed by another repo |
| Modified query fails syntax check | Undo the edit, re-examine the original, fix manually |
| `[SCOPED]` Framework not detected | Ask user to identify the router and entry component(s) manually |
| `[SCOPED]` Lazy-loaded components not resolved | Follow the dynamic import to the target file; if it's a code-split bundle, ask the user for the component path |
| `[SCOPED]` Fragment consumed by out-of-scope code | Mark it as a boundary fragment; treat all its fields as indeterminate for out-of-scope consumers |
| Codegen command not found | Check package.json scripts, then config files (codegen.ts, codegen.yml). Ask the user if nothing is found. |
| Codegen fails after field removal | A consumer still references the removed field or type. Read the error, undo the offending removal, re-check the trace. |
| Type errors after edits | Distinguish new errors (caused by this refactor) from pre-existing ones. Fix new errors by undoing the removal or updating the consumer. |
| Shared query has too many consumers to split | Present the per-consumer usage table and let the user decide which consumers warrant their own query. Not every consumer needs a split. |

## Out of Scope

This skill does NOT:
- Modify the GraphQL schema
- Analyze mutations or subscriptions (over-fetching is a query concern)
- Fix pre-existing type errors unrelated to the refactor
- Run the test suite (tell the user to run it as a follow-up)
