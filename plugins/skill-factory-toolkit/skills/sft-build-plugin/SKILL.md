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
