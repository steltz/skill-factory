# UI-Finisher Design Spec

**Date:** 2026-04-11
**Status:** Approved (design phase)
**Plugin:** `ui-finisher`
**Single skill:** `audit-ui-handlers`

## Overview

UI-Finisher is a static-analysis plugin that audits a single **page** — identified by its URL/route or a direct file path — for unfinished interactive elements: orphaned buttons, stubbed handlers, handlers missing required side effects. It proposes patches gated by human approval and pairs every patch with a manual QA checklist because the verifier is always a human.

The plugin is LLM-native: it uses Claude's existing tools (Glob, Read, Grep, Edit) and native code comprehension rather than shipping custom parsers or AST libraries. It is a sibling to `graphql-overfetch-analyzer` in structure and install pattern.

## Scope

### Inputs

- **Primary:** a page URL or route (e.g., `/dashboard`, `/settings/profile`, `https://example.com/account`) that the skill resolves to a file on disk via filesystem conventions (Next.js app/pages router, SvelteKit, Astro, Remix) or a bounded React Router fallback
- **Alternative:** a direct file path (e.g., `src/components/SettingsForm.tsx`) for users who want to skip URL resolution
- Full URLs have their origin stripped before resolution; query strings and hashes are discarded
- The skill never fetches the URL, runs the app, or launches a headless browser — resolution is pure disk lookup

### Outputs

- A structured audit report in the terminal, grouped by confidence tier
- Proposed diffs for Tier 1 and Tier 2 findings (never auto-applied)
- After user approval: applied patches plus a manual QA checklist

### Non-goals

- Running code or headless browser automation
- Mutating anything outside the audited file's reach (no touching `api/`, no creating new files)
- Replacing an existing test suite — this is a human-in-the-loop tool
- Repo-wide pattern discovery (explicitly forbidden; see Rigid Gates)

### Scope Boundary

The skill reads the target file plus its direct imports only. It never greps the whole repo. Prop-drilling trace depth is capped at 2 levels. Context that can't be resolved within this boundary results in a Tier 3 (flag-only) classification, not a proposal.

## Audit Pipeline

Six phases execute in order. Phase 0 resolves the user's input to a target file; Phases 1–5 run the audit against that file. Each phase produces a named artifact the next phase consumes.

### Phase 0 — Resolve

Translate the user's input (URL, route, or file path) into a single target file on disk. If the user gave a direct file path, this phase is a no-op.

Resolution order:

1. **Filesystem conventions (Glob/Read only).** Try known page-file conventions for Next.js app router (`app/foo/bar/page.tsx`), Next.js pages router (`pages/foo/bar.tsx`), SvelteKit (`src/routes/foo/bar/+page.svelte`), Astro (`src/pages/foo/bar.astro`), and Remix (`app/routes/foo.bar.tsx`). If exactly one convention file exists, that is the target.
2. **React Router fallback (one bounded Grep).** If Step 1 finds nothing and the repo uses React Router, a single Grep is permitted — restricted to route-registration file globs (`**/App.*`, `**/router*.*`, `**/routes*.*`) and a literal route-path pattern. From the hit, Read the file, find the `element={<X />}` binding, and resolve the component to a source file via its import.
3. **Ask.** If Steps 1 and 2 fail, stop and ask the user for a direct file path. Never guess.
4. **Ambiguity.** If Step 1 or 2 returns multiple candidates, stop and ask which one to audit.

Phase 0 is the only phase allowed to Grep outside the target file, and only within the bounded glob set above. All other Grep-outside-target-file activity in Phases 1–5 is a rigid-gate violation.

### Phase 1 — Harvest

Read the target file with the Read tool. Identify every interactive element by scanning for:

- Native JSX tags: `button`, `a`, `form`, `input`, `select`, `textarea`
- Custom components whose name matches `/Button|Link|Form|Input|Select|Dropdown|Menu$/` and has an interaction prop
- Elements inside `items.map(...)` loops (dynamic rendering)

Output: a list of `{element, line, attrs, handlerRef}` records.

### Phase 2 — Trace

For each record, locate the handler:

- Inline arrow: already in hand
- Named function in same file: Grep within the file for the definition
- Imported helper: Read the import source file, then Grep for the definition (one hop only)
- Prop-drilled from parent: trace up to 2 levels via Grep; beyond that, mark Tier 3 and stop

Output: each record annotated with `{handlerBody, handlerOrigin, traceDepth, traceable}`.

### Phase 3 — Classify

Apply the tiering rules (detailed in `tiering-rules.md`):

- **Tier 1 — definitely unfinished:** orphaned elements, empty bodies, only `console.log`/`alert`, contains `// TODO`/`// FIXME`/`stub`/`placeholder`
- **Tier 2 — likely unfinished:** state-only handler with an action-verb button label; form `onSubmit` that doesn't call any imported `api/*` function when such imports exist in the file
- **Tier 3 — ambiguous:** untraceable handler; toggle-y label with local state updates; prop-drilling beyond 2 levels
- **Clean:** handler fully traced and performs a non-trivial side effect

Output: each record tagged with a tier.

### Phase 4 — Propose

For every Tier 1 and Tier 2 record:

- Use only imports already present in the target file as available building blocks (**strict, imports-only scope**)
- If no usable imports exist, downgrade to Tier 3 with a "could look wider" note
- Generate a proposed handler body: try/catch + loading state + success/error branches when a mutation is involved; minimal state logic when local-only
- Produce a unified diff against the target file

Output: a list of proposed `{record, diff, rationale, confidence}` objects. **No file writes yet.**

### Phase 5 — Approve & Patch

Present the report (format below). On user approval, apply selected diffs via the Edit tool sequentially. If any Edit fails, stop and report. After successful patches, emit the manual QA checklist.

## Report Format

When Phase 4 completes, the skill emits a single structured report before any write:

```
UI-Finisher Audit: src/components/SettingsForm.tsx

[Tier 1 — will propose diffs]
  1. <button>Delete Account</button>  (line 47)
     Issue: no onClick prop, element is orphaned
     Proposal: add handleDelete; wires to existing deleteUser import
     Confidence: high

  2. handleSave  (line 82)
     Issue: body is only console.log('saved')
     Proposal: call updateUser (already imported), add isSaving state,
               use existing toast import for success/error
     Confidence: high

[Tier 2 — will propose diffs]
  3. onSubmit on <form>  (line 120)
     Issue: only calls setFormState; button labeled "Save Settings"
     Proposal: wrap in submit handler that calls updateProfile
     Confidence: medium
     Rationale: action-verb label + existing api/profile import

[Tier 3 — flagged, no proposal]
  4. <button>Toggle Filters</button>  (line 203)
     Status: handler updates local state only; label is toggle-y
     Note: left as-is (matches local UI mutation pattern)

  5. handleRefresh  (line 240)
     Status: untraceable — passed down from parent 3 levels
     Note: could look wider for context; re-run after widening scope

[Clean — not touching]
  - <a href="/help">Help</a>
  - <input type="text" ...>
```

Followed by the approval prompt:

```
Proposals: 3 (1, 2, 3). Apply which?
  a) all
  b) pick (e.g., "1,3")
  c) none
```

### Approval Rules

- Default is "none" if user doesn't answer affirmatively
- Each selected diff is applied with Edit sequentially
- If any Edit fails (file changed since read, string not unique), stop and report — no automatic recovery
- After successful patches, emit the QA checklist

### QA Checklist Format

One checkbox per patched handler, phrased as a human action plus a verification step:

```
Manual QA Checklist for SettingsForm.tsx:

[ ] Click "Delete Account" — verify handleDelete fires and deleteUser is called
[ ] Fill the form and click "Save Settings" — verify:
    - button shows loading state during request
    - success toast appears on 200
    - error toast appears on non-200
[ ] Verify no visual regression on modified lines
```

## Skill Structure

### Type

**Technique** — repeatable step-by-step process with no Iron Law in the discipline-skill sense. Five phases, decision points, bounded pipeline.

### Enforcement

Flexible within steps, **rigid at two gates**:

1. **No-write-before-approval gate:** the skill MUST NOT call Edit before the user has explicitly approved the proposal list. Phrased as a rigid rule even though the skill overall is a technique.
2. **No-repo-wide-grep gate:** the skill MUST NOT Grep outside the target file's direct imports. Context gaps downgrade findings to Tier 3.

Both gates get explicit Red Flag rationalization rows:
- "but the fix is obvious" → no, still need approval
- "but I bet there's a toast utility elsewhere" → no, stay in-file

### Coordination Pattern

**Generator-Verifier** (user as verifier). The skill generates proposals; the user verifies before any write. Same pattern as `brainstorming` and `writing-plans` review gates.

### Structural Markers (Technique)

- Step-by-step process (numbered checklist)
- When to Use / When NOT to Use
- Common Mistakes section
- Decision flowchart (graphviz dot) for tier classification
- Real-World Example (SettingsForm)
- Minimal Integration section (standalone, no dispatch)

### Word Count Target

~1000 words for SKILL.md. Comparable to `writing-plans` (~910) and `finishing-a-development-branch` (~680).

### Supporting Files

- **`tiering-rules.md`** — detailed tier classification heuristics with examples. Keeps SKILL.md scan-optimized. Same pattern as `test-driven-development` with `testing-anti-patterns.md`.
- **`report-template.md`** — the exact report format and QA checklist format, so SKILL.md can reference it without inlining 40+ lines.

## Plugin Packaging

### Directory Layout

```
plugins/ui-finisher/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── audit-ui-handlers/
│       ├── SKILL.md
│       ├── tiering-rules.md
│       └── report-template.md
└── CHANGELOG.md
```

No `hooks/`, no bin scripts. Pure skill content.

### plugin.json

```json
{
  "name": "ui-finisher",
  "description": "Audit a single page — by URL/route or file path — for unfinished interactive elements (orphaned buttons, stubbed handlers, missing side effects) and propose patches with human approval. Static analysis only.",
  "version": "0.1.0"
}
```

### SKILL.md Frontmatter

```yaml
---
name: audit-ui-handlers
description: Use when auditing a page — identified by its URL or route (e.g., /dashboard, /settings/profile) or by a direct file path — for unfinished interactive elements such as orphaned buttons, stubbed handlers, or handlers missing required side effects like API calls or state updates
---
```

Description starts with "Use when" and contains triggering conditions only.

### CHANGELOG.md

Initial `0.1.0` entry following Keep a Changelog format: "Initial release. Adds `audit-ui-handlers` skill."

### Install Path

```
claude plugin marketplace add /Users/nicholasstelter/Code/skill-factory
claude plugin install ui-finisher
```

### Versioning

Starts at `0.1.0`. Bumps to `0.2.0` on any additional skill. `1.0.0` when declared stable. Follows `sft-versioning-guide` rules.

## Validation Strategy

### Baseline Test (RED)

Construct a small fixture file with known Tier 1, Tier 2, and Tier 3 handlers. Run the skill on it without having loaded the full SKILL.md. Observe failures — missing findings, wrong tier assignments, premature writes — and record them.

### GREEN

Load the skill, re-run against the same fixture. Verify:
- Correct tier assignments for each seeded handler
- Correct proposal content using only in-file imports
- No write happens before approval
- QA checklist emitted only after successful patches

### Refactor

Close loopholes identified during RED in the skill body as rationalization rows or red flag entries.

## Technical Blind Spots

Acknowledged limitations the skill surfaces rather than hides:

- **Prop drilling beyond 2 levels** — Tier 3 with a trace-depth note. The skill does not guess.
- **Dynamic rendering correctness** — elements inside `items.map(...)` get found, but the skill does not attempt to verify that the correct `item.id` is threaded through the handler. That's human QA territory.
- **False positives on UI-only handlers** — Tier 3 covers toggle-y labels. A button labeled `Toggle Filters` that only calls `setOpen(x => !x)` is classified as clean/ambiguous, not broken.
- **No type checking** — the skill reasons about code, not types. Proposed diffs may need TypeScript fixes a human applies.

## Open Questions

None at design-approval time. Any new questions during planning will be added to the plan document.
