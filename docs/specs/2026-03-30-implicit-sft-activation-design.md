# Implicit sft Skill Activation

## Problem

The skill-factory-toolkit (sft) skills exist to help superpowers produce better skills when authoring in this repo. But they only activate when manually invoked — users must remember to load them at the right workflow stage. This creates a gap where the toolkit's value is invisible during the workflows that need it most.

## Solution

Two-layer implicit activation, scoped to this repo.

### Layer 1: CLAUDE.md Directive (Primary)

Add a `## Skill Auto-Loading` section to `CLAUDE.md` after the `## Workflow` section. It maps specific skill-authoring activities to contextual subsets of sft skills using mandatory language.

**Trigger → Skills mapping:**

| When you are... | Invoke these sft skills first |
|---|---|
| Brainstorming a new skill or plugin idea | `sft-skill-comparison-matrix`, `sft-skill-anatomy` |
| Planning skill implementation | `sft-cross-cutting-patterns`, `sft-scaffold-plugin` |
| Writing or editing a SKILL.md | `sft-cross-cutting-patterns`, `sft-skill-comparison-matrix` |
| Creating a new plugin | `sft-scaffold-plugin` |
| Bumping versions or writing changelog | `sft-versioning-guide` |

This is the primary mechanism. CLAUDE.md instructions have highest priority — they override superpowers skill defaults and system prompt behavior.

### Layer 2: SessionStart Hook (Safety Net)

Add `hooks/hooks.json` to `plugins/skill-factory-toolkit/` with a `SessionStart` hook that prints the trigger-to-skill mapping at conversation start. This fires in all repos where the plugin is installed, reinforcing the CLAUDE.md directive.

**Hook type:** `command` (echo)
**Async:** false
**Output:** The trigger-to-skill mapping table as plain text

### Token Budget

Max 2 skills loaded per stage (~2-3k tokens). Never all 5 at once. This is acceptable overhead given Opus's 200k context window.

## Changes

### Modified files

1. **`CLAUDE.md`** — Add `## Skill Auto-Loading` section after `## Workflow`

### New files

2. **`plugins/skill-factory-toolkit/hooks/hooks.json`** — SessionStart hook

### Version bump

3. **`plugins/skill-factory-toolkit/.claude-plugin/plugin.json`** — `2.0.0` → `2.1.0` (new capability, non-breaking)
4. **`plugins/skill-factory-toolkit/CHANGELOG.md`** — Add `[2.1.0]` entry under Added

## What We're NOT Changing

- No modifications to superpowers skills (pinned reference copies)
- No changes to sft skill content
- No changes to `bin/validate-plugin` or the repo-level pre-commit hook
- No changes to skill descriptions or frontmatter
