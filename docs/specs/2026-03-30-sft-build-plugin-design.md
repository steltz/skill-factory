# sft-build-plugin — Design Spec

## Overview

A thin orchestrator skill that becomes the single entry point for building skills and plugins in the skill-factory repo. It wraps the superpowers brainstorm → plan → implement → package workflow, automatically pre-loading the right sft skills at each phase boundary.

## Problem

The current auto-loading mechanism uses CLAUDE.md directives and a SessionStart hook to remind Claude to load sft skills at each workflow phase. This is passive — it relies on Claude independently remembering to check the table at each phase transition. In practice, Claude loaded the correct sft skills at 2 of 5 checkpoints during the graphql-overfetch-analyzer build, missing `sft-skill-comparison-matrix`, `sft-skill-anatomy`, and `sft-versioning-guide`.

## Solution

Replace the passive reminder system with an active orchestrator skill. `sft-build-plugin` owns the workflow checklist and explicitly loads the sft skills before handing off to each superpowers skill. The superpowers skills run unmodified — the gateway operates only at the seams between them.

## Skill Identity

- **Skill name:** `sft-build-plugin`
- **Plugin:** `skill-factory-toolkit` (alongside the existing 5 sft skills)
- **Type:** Technique (orchestrator pattern, modeled after `executing-plans`)
- **Trigger:** "Use when building a new skill or plugin in the skill-factory repo — orchestrates the full brainstorm → plan → implement → package workflow with automatic sft skill loading at each phase"

## Phase Map

| Phase | Pre-load these sft skills | Then invoke |
|-------|--------------------------|-------------|
| **Brainstorm** | `sft-skill-comparison-matrix`, `sft-skill-anatomy` | `superpowers:brainstorming` |
| **Plan** | `sft-cross-cutting-patterns`, `sft-scaffold-plugin` | `superpowers:writing-plans` |
| **Implement** | _(sft context carried in the plan document)_ | `superpowers:subagent-driven-development` or `superpowers:executing-plans` |
| **Version** | `sft-versioning-guide` | _(manual — bump version and changelog after implementation)_ |

### Design Decisions

- **Implement phase has no pre-load.** The sft patterns inform the plan, which implementer subagents receive as context. Loading sft skills inside subagent dispatches would bloat their context and fight subagent-driven-development's own flow.
- **Version phase is manual.** It happens after all tasks complete. The checklist reminds Claude to load `sft-versioning-guide` before bumping the version.
- **Pre-loading is sequential.** Load sft skills first (Claude reads and absorbs the reference material), then invoke the superpowers skill which takes over the flow.

## Checklist

The skill uses a 9-step numbered checklist. Claude creates tasks from it and works through them in order.

1. Pre-load `sft-skill-comparison-matrix` and `sft-skill-anatomy`
2. Invoke `superpowers:brainstorming` (design spec produced, saved to `docs/specs/`)
3. Pre-load `sft-cross-cutting-patterns` and `sft-scaffold-plugin`
4. Invoke `superpowers:writing-plans` (implementation plan produced, saved to `docs/plans/`)
5. User selects execution path (subagent-driven or inline)
6. Invoke the selected execution skill
7. After implementation completes, pre-load `sft-versioning-guide`
8. Bump version and changelog
9. Invoke `superpowers:finishing-a-development-branch`

### Flow Control Rules

- Steps 1-2 are atomic: pre-load THEN invoke. Do not invoke brainstorming without the sft context.
- Steps 3-4 intercept brainstorming's terminal state. Brainstorming hardcodes "invoke writing-plans" as its exit. The checklist overrides this: "After brainstorming completes, return to this checklist. Load sft skills in step 3 BEFORE invoking writing-plans in step 4."
- Steps 5-6 mirror writing-plans' existing user choice (subagent vs inline). The gateway passes through without modification.
- Steps 7-8 fire after all implementation tasks are done.
- The checklist is the authority. When brainstorming says "invoke writing-plans" and this checklist says "come back here first," the checklist wins because the user invoked `sft-build-plugin` as the controlling skill.

## CLAUDE.md Changes

### Replace Workflow Section and Auto-Loading Table

The existing Workflow section (4-step numbered list) and Skill Auto-Loading table (5 rows with mapping) get replaced with:

```markdown
## Workflow

When creating new skills or plugins, invoke `skill-factory-toolkit:sft-build-plugin`. It orchestrates
the full brainstorm → plan → implement → package workflow and automatically loads the right sft skills
at each phase boundary.
```

### Add Safety Net Directive

New section after the Workflow section:

```markdown
## Skill Authoring Safety Net

When invoking `superpowers:writing-plans` in this repo, ALWAYS pre-load `sft-cross-cutting-patterns`
and `sft-scaffold-plugin` first. This applies even if you arrived here from `superpowers:brainstorming`
directly — the sft context is required for plan quality.
```

This is the belt-and-suspenders backstop for the brainstorming → writing-plans transition.

Similarly, add a versioning safety net:

```markdown
When bumping plugin versions or writing changelog entries in this repo, ALWAYS pre-load
`sft-versioning-guide` first.
```

This catches the versioning phase even if Claude reaches it without going through the sft-build-plugin checklist.

### What Gets Removed

- The Workflow section (4-step numbered list starting with "When creating new skills:")
- The Skill Auto-Loading table (5 rows mapping activities to sft skills)
- The paragraph about loading mapped skills at the start of each stage

### What Stays

- The Factory Toolkit table (documents what each sft skill does — still useful as reference)

## SessionStart Hook Update

Replace the current hook message (5-row activity-to-skill mapping table) with:

```
[skill-factory-toolkit] To build a new skill or plugin, invoke sft-build-plugin.
It handles sft skill loading at each phase automatically.
```

Shorter, more actionable — one thing to remember instead of five.

## Plugin Structure Change

No new plugin. Add the skill to the existing skill-factory-toolkit:

```
plugins/skill-factory-toolkit/
├── skills/
│   ├── sft-build-plugin/          # NEW
│   │   └── SKILL.md
│   ├── sft-scaffold-plugin/
│   ├── sft-cross-cutting-patterns/
│   ├── sft-skill-anatomy/
│   ├── sft-skill-comparison-matrix/
│   └── sft-versioning-guide/
├── .claude-plugin/
│   └── plugin.json                # version bump to 3.0.0
├── hooks/
│   └── hooks.json                 # updated hook message
└── CHANGELOG.md                   # new entry
```

### Version Bump

This is a **major version bump** (2.1.0 → 3.0.0) because:
- The CLAUDE.md Workflow section and Auto-Loading table are removed (breaking change for any session that relied on them)
- The SessionStart hook message changes
- A new skill is added that changes the expected workflow entry point

## Skill Enforcement

- **Technique skill** — flexible within steps, rigid step order
- **Checklist-driven** — 9 steps, each a task
- **No iron law** — the skill orchestrates, it doesn't enforce discipline
- **No rationalization table** — the superpowers skills it delegates to have their own enforcement
- **Quick Reference** — phase map table for scanning
- **When to Use / When NOT to Use** — scopes the skill to skill-factory repo only
