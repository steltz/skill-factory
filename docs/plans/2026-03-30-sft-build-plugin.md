# sft-build-plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add `sft-build-plugin` skill to skill-factory-toolkit — a thin orchestrator that pre-loads sft skills at each phase boundary of the superpowers workflow chain.

**Architecture:** Single SKILL.md added to the existing skill-factory-toolkit plugin, plus updates to CLAUDE.md, SessionStart hook, plugin.json version, and CHANGELOG.md. No new plugins, no supporting files.

**Tech Stack:** SKILL.md (markdown), plugin.json (JSON), CHANGELOG.md (markdown), session-start (bash script), CLAUDE.md (markdown)

---

## File Structure

```
plugins/skill-factory-toolkit/
├── skills/
│   └── sft-build-plugin/          # NEW — the orchestrator skill
│       └── SKILL.md
├── .claude-plugin/
│   └── plugin.json                # MODIFY — version 2.1.0 → 3.0.0
├── hooks/
│   ├── hooks.json                 # NO CHANGE
│   ├── run-hook.cmd               # NO CHANGE
│   └── session-start              # MODIFY — new hook message
└── CHANGELOG.md                   # MODIFY — new entry for 3.0.0

CLAUDE.md                          # MODIFY — replace Workflow + Auto-Loading sections
```

---

### Task 1: Write Baseline Test (RED)

**Files:**
- Create: `plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md` (placeholder with baseline test)

- [ ] **Step 1: Create placeholder SKILL.md with baseline test**

Write this to `plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md`:

```markdown
---
name: sft-build-plugin
description: Use when building a new skill or plugin in the skill-factory repo — orchestrates the full brainstorm → plan → implement → package workflow with automatic sft skill loading at each phase
---

<!-- BASELINE TEST (remove after GREEN)
Test: Invoke this skill to build a new plugin.
FAIL criteria (placeholder skill fails if Claude does ANY of these without being told):
1. Pre-loads sft-skill-comparison-matrix and sft-skill-anatomy before brainstorming
2. Invokes superpowers:brainstorming after pre-loading sft context
3. Intercepts brainstorming's terminal state and returns to its own checklist
4. Pre-loads sft-cross-cutting-patterns and sft-scaffold-plugin before planning
5. Invokes superpowers:writing-plans after pre-loading sft context
6. Pre-loads sft-versioning-guide before bumping version
7. Follows a 9-step checklist with sft pre-loads at phase boundaries
-->

# Build Plugin

Placeholder — skill content will be authored in subsequent tasks.
```

- [ ] **Step 2: Verify baseline test reasoning**

The placeholder says nothing about pre-loading sft skills, intercepting brainstorming's exit, or a 9-step checklist. A Claude instance loading this skill would not perform the 7 behaviors listed above. The test FAILS as expected.

- [ ] **Step 3: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md
git commit -m "test(sft-build-plugin): add baseline test for orchestrator checklist"
```

---

### Task 2: Write the Skill (GREEN)

**Files:**
- Modify: `plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md`

- [ ] **Step 1: Replace SKILL.md with the full skill content**

Write this to `plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md`:

````markdown
---
name: sft-build-plugin
description: Use when building a new skill or plugin in the skill-factory repo — orchestrates the full brainstorm → plan → implement → package workflow with automatic sft skill loading at each phase
---

<!-- BASELINE TEST (remove after GREEN)
Test: Invoke this skill to build a new plugin.
FAIL criteria (placeholder skill fails if Claude does ANY of these without being told):
1. Pre-loads sft-skill-comparison-matrix and sft-skill-anatomy before brainstorming
2. Invokes superpowers:brainstorming after pre-loading sft context
3. Intercepts brainstorming's terminal state and returns to its own checklist
4. Pre-loads sft-cross-cutting-patterns and sft-scaffold-plugin before planning
5. Invokes superpowers:writing-plans after pre-loading sft context
6. Pre-loads sft-versioning-guide before bumping version
7. Follows a 9-step checklist with sft pre-loads at phase boundaries
-->

# Build Plugin

Single entry point for building skills and plugins in the skill-factory repo. Pre-loads the right sft skills at each phase boundary, then hands off to the superpowers workflow chain.

## When to Use

- Building a new skill or plugin from scratch in the skill-factory repo
- Adding a new skill to an existing plugin in the skill-factory repo

## When NOT to Use

- Editing an existing skill (just load `sft-cross-cutting-patterns` directly)
- Working outside the skill-factory repo
- Brainstorming without intent to build (use `superpowers:brainstorming` directly)

## Phase Map

| Phase | Pre-load these sft skills | Then invoke |
|-------|--------------------------|-------------|
| **Brainstorm** | `sft-skill-comparison-matrix`, `sft-skill-anatomy` | `superpowers:brainstorming` |
| **Plan** | `sft-cross-cutting-patterns`, `sft-scaffold-plugin` | `superpowers:writing-plans` |
| **Implement** | _(sft context carried in the plan)_ | `superpowers:subagent-driven-development` or `superpowers:executing-plans` |
| **Version** | `sft-versioning-guide` | _(manual — bump version and changelog)_ |

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Pre-load brainstorm context** — invoke `skill-factory-toolkit:sft-skill-comparison-matrix` and `skill-factory-toolkit:sft-skill-anatomy`
2. **Brainstorm** — invoke `superpowers:brainstorming` (spec saved to `docs/specs/`)
3. **Pre-load plan context** — invoke `skill-factory-toolkit:sft-cross-cutting-patterns` and `skill-factory-toolkit:sft-scaffold-plugin`
4. **Plan** — invoke `superpowers:writing-plans` (plan saved to `docs/plans/`)
5. **User selects execution path** — subagent-driven (recommended) or inline
6. **Implement** — invoke the selected execution skill
7. **Pre-load version context** — invoke `skill-factory-toolkit:sft-versioning-guide`
8. **Version** — bump plugin version and update CHANGELOG.md
9. **Finish** — invoke `superpowers:finishing-a-development-branch`

### Flow Control

**Steps 1-2 are atomic.** Do not invoke brainstorming without the sft context loaded first. Load both sft skills, then invoke brainstorming in the same turn.

**Steps 3-4 intercept brainstorming's exit.** When brainstorming completes, it will say "invoke writing-plans." STOP. Return to THIS checklist. Load the sft skills in step 3 BEFORE invoking writing-plans in step 4. This checklist is the authority — it overrides brainstorming's hardcoded exit.

**Steps 5-6 pass through.** Writing-plans will offer the execution choice (subagent vs inline). Follow its lead.

**Steps 7-8 fire after implementation.** When all implementation tasks are done, return to this checklist. Load sft-versioning-guide before touching the version or changelog.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Invoking brainstorming before loading sft skills | Steps 1-2 are atomic. Load first, invoke second. |
| Following brainstorming's "invoke writing-plans" exit directly | Return to this checklist. Step 3 must happen before step 4. |
| Skipping the version phase | Step 7 exists because this was missed in the first build. Don't skip it. |
| Loading all sft skills at once | Only load the subset for the current phase. The table tells you which. |
````

- [ ] **Step 2: Verify the skill covers all spec requirements**

Check against the spec:
- Skill identity (name, plugin, type, trigger) — frontmatter ✓
- Phase map (4 phases with pre-loads) — table ✓
- Checklist (9 steps) — numbered list ✓
- Flow control rules (4 rules) — section ✓
- Technique skill with When to Use / When NOT to Use — sections ✓
- No iron law, no rationalization table — correct per spec ✓
- Common Mistakes table — present ✓

- [ ] **Step 3: Validate the plugin**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: All checks pass.

- [ ] **Step 4: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md
git commit -m "feat(sft-build-plugin): write orchestrator skill with phase map and checklist"
```

---

### Task 3: Remove Baseline Test Comment

**Files:**
- Modify: `plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md`

- [ ] **Step 1: Remove the baseline test HTML comment**

Remove the entire `<!-- BASELINE TEST ... -->` comment block from SKILL.md. Ensure exactly one blank line between the frontmatter closing `---` and the `# Build Plugin` heading.

- [ ] **Step 2: Validate the plugin**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: All checks pass.

- [ ] **Step 3: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md
git commit -m "chore(sft-build-plugin): remove baseline test comment after GREEN"
```

---

### Task 4: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Add sft-build-plugin to the Directory Structure**

In the Directory Structure code block, add `sft-build-plugin/` to the skills list under skill-factory-toolkit. The updated block should show:

```
│   │   ├── skills/
│   │   │   ├── sft-build-plugin/
│   │   │   ├── sft-scaffold-plugin/
│   │   │   ├── sft-cross-cutting-patterns/
│   │   │   ├── sft-skill-anatomy/
│   │   │   ├── sft-skill-comparison-matrix/
│   │   │   └── sft-versioning-guide/
```

- [ ] **Step 2: Add sft-build-plugin to the Factory Toolkit table**

Add a row to the Factory Toolkit table:

```
| `sft-build-plugin` | Use when building a new skill or plugin |
```

- [ ] **Step 3: Replace the Workflow section**

Replace everything from `## Workflow` through the 4-step numbered list (lines 94-101 currently) with:

```markdown
## Workflow

When creating new skills or plugins, invoke `skill-factory-toolkit:sft-build-plugin`. It orchestrates the full brainstorm → plan → implement → package workflow and automatically loads the right sft skills at each phase boundary.
```

- [ ] **Step 4: Replace the Skill Auto-Loading section**

Replace everything from `## Skill Auto-Loading` through the "Do not load all 5 at once" paragraph (lines 103-115 currently) with:

```markdown
## Skill Authoring Safety Net

When invoking `superpowers:writing-plans` in this repo, ALWAYS pre-load `sft-cross-cutting-patterns` and `sft-scaffold-plugin` first. This applies even if you arrived here from `superpowers:brainstorming` directly — the sft context is required for plan quality.

When bumping plugin versions or writing changelog entries in this repo, ALWAYS pre-load `sft-versioning-guide` first.
```

- [ ] **Step 5: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: replace workflow and auto-loading sections with sft-build-plugin directive"
```

---

### Task 5: Update SessionStart Hook

**Files:**
- Modify: `plugins/skill-factory-toolkit/hooks/session-start`

- [ ] **Step 1: Update the REMINDER variable**

In `plugins/skill-factory-toolkit/hooks/session-start`, replace the existing `REMINDER=` line (line 7) with:

```bash
REMINDER="[skill-factory-toolkit] To build a new skill or plugin, invoke sft-build-plugin.\nIt handles sft skill loading at each phase automatically."
```

This replaces the 5-row activity-to-skill mapping table with a single actionable directive.

- [ ] **Step 2: Commit**

```bash
git add plugins/skill-factory-toolkit/hooks/session-start
git commit -m "feat(skill-factory-toolkit): update SessionStart hook to point at sft-build-plugin"
```

---

### Task 6: Version Bump and Changelog

**Files:**
- Modify: `plugins/skill-factory-toolkit/.claude-plugin/plugin.json`
- Modify: `plugins/skill-factory-toolkit/CHANGELOG.md`

- [ ] **Step 1: Bump plugin.json version**

Change `plugins/skill-factory-toolkit/.claude-plugin/plugin.json` to:

```json
{
  "name": "skill-factory-toolkit",
  "description": "Authoring skills, pattern references, and governance guides for the skill factory",
  "version": "3.0.0"
}
```

- [ ] **Step 2: Add changelog entry**

Add the following entry at the top of the changelog (after the `# Changelog` heading, before the existing `## [2.1.0]` entry) in `plugins/skill-factory-toolkit/CHANGELOG.md`:

```markdown
## [3.0.0] - 2026-03-30

### Added

- sft-build-plugin skill — single entry point for building skills/plugins with automatic sft pre-loading at each phase

### Changed

- **BREAKING:** CLAUDE.md Workflow section and Skill Auto-Loading table replaced with sft-build-plugin directive
- **BREAKING:** SessionStart hook message simplified to point at sft-build-plugin
```

- [ ] **Step 3: Validate the plugin**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: All checks pass.

- [ ] **Step 4: Commit**

```bash
git add plugins/skill-factory-toolkit/.claude-plugin/plugin.json plugins/skill-factory-toolkit/CHANGELOG.md
git commit -m "chore(skill-factory-toolkit): bump version to 3.0.0 with sft-build-plugin changelog"
```

---

### Task 7: Final Review

- [ ] **Step 1: Read the complete sft-build-plugin SKILL.md**

Verify:
- No placeholder text, TODOs, or incomplete sections
- Frontmatter `name` matches directory name (`sft-build-plugin`)
- Frontmatter `description` starts with "Use when"
- Phase map table has 4 rows
- Checklist has 9 items
- Flow control section addresses the brainstorming exit interception
- When to Use / When NOT to Use sections present
- Common Mistakes table present

- [ ] **Step 2: Read CLAUDE.md and verify changes**

Verify:
- Workflow section points to `sft-build-plugin`
- Skill Auto-Loading table is removed
- Safety net directives are present (writing-plans and versioning)
- Factory Toolkit table includes `sft-build-plugin` row
- Directory Structure includes `sft-build-plugin/`

- [ ] **Step 3: Run final plugin validation**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: All checks pass.

- [ ] **Step 4: Verify git status is clean**

Run: `git status`

Expected: Nothing unstaged or untracked.
