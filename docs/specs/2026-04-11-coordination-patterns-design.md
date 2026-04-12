# Coordination Patterns for Skill Factory Toolkit — Design Spec

**Date:** 2026-04-11
**Status:** Draft
**Target plugin:** `plugins/skill-factory-toolkit` (v3.1.0 → v3.2.0)
**Source material:** [Multi-Agent Coordination Patterns (Anthropic)](https://claude.com/blog/multi-agent-coordination-patterns)

## Problem

The `skill-factory-toolkit` plugin has no vocabulary or reference material for multi-agent coordination. When an author builds a new skill, they have no systematic way to decide how that skill should coordinate work: whether it orchestrates subagents, runs a generator-verifier loop, dispatches parallel workers, coordinates via shared state, or routes events on a bus. The toolkit itself is already a multi-agent coordination system (`sft-build-plugin` orchestrates phase-specific sft skills; specs and plans are shared state; review loops are generator-verifier) but this is invisible to authors using the toolkit.

## Goals

1. **Teach:** Authors can identify which coordination pattern fits their new skill and structure it accordingly.
2. **Default to simple:** The toolkit nudges authors toward orchestrator-subagent as the default, matching the article's "start simple" principle.
3. **Self-demonstrate:** The toolkit documents itself as a worked example of the patterns, so authors can see a concrete case in the same repo they are working in.
4. **Lightweight enforcement:** Integration is a soft nudge, not a hard gate. No new validation rules.

## Non-Goals

- Structural refactor of `sft-build-plugin` or any existing skill
- New hooks or validation logic in `bin/validate-plugin`
- Pattern-validation enforcement (checking that a declared pattern matches structural markers)
- Changes to any superpowers skill outside `skill-factory-toolkit`
- A new plugin — this work stays inside `skill-factory-toolkit`

## Architecture

Three changes, each at its own scale:

### Change A — New `sft-coordination-patterns` reference skill

Location: `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md`

Type: **reference** (tables primary, lookup-optimized, minimal prose, no enforcement language). Modeled after `sft-skill-comparison-matrix` and `sft-cross-cutting-patterns`.

Target word count: 1100-1400 words (sits between the two existing reference skills in the toolkit).

**Frontmatter:**
```yaml
---
name: sft-coordination-patterns
description: Use when deciding how a new skill or plugin should coordinate work across agents — choosing between orchestrator, generator-verifier, agent teams, message bus, or shared state patterns.
---
```

**Section outline:**

| # | Section | Purpose |
|---|---------|---------|
| 1 | Overview | 1-2 sentences: match pattern to information flow, default to orchestrator-subagent |
| 2 | Pattern Reference Table | 5 rows × (Pattern, Purpose, When to Use, Key Elements, Anti-Patterns, Exemplar in Ecosystem) |
| 3 | Pattern Details | One subsection per pattern expanding the table row with skill-authoring implications |
| 4 | Design Principles | Five principles from the article, each as a one-liner |
| 5 | Worked Example: Skill Factory Toolkit | Map the toolkit's own architecture onto the patterns |
| 6 | Pattern-to-Structure Cheat Sheet | Table: pattern → structural markers a SKILL.md should likely have |
| 7 | Common Mistakes | Table of authoring mistakes around pattern selection |

**Pattern-to-exemplar mapping** (used in section 2 and section 5):

| Pattern | Exemplar in superpowers/sft ecosystem |
|---------|---------------------------------------|
| Generator-Verifier | `superpowers:requesting-code-review`; spec and plan review loops in `brainstorming` and `writing-plans` |
| Orchestrator-Subagent | `superpowers:subagent-driven-development`; `sft-build-plugin` |
| Agent Teams | `superpowers:dispatching-parallel-agents` |
| Message Bus | No clean exemplar — noted as "pattern applies to event-driven systems, not commonly seen in current superpowers corpus" |
| Shared State | `docs/specs/`, `docs/plans/`, memory files, TodoWrite state |

**Worked example content** (section 5):

The worked example describes how `skill-factory-toolkit` itself exhibits three patterns:

- **Orchestrator-Subagent:** `sft-build-plugin` orchestrates the build workflow by delegating to phase-specific sft skills (comparison matrix and anatomy during brainstorm, cross-cutting patterns and scaffold-plugin during plan, versioning-guide during version bump). Each sub-skill operates in a bounded context and returns results to the orchestrator's checklist.
- **Shared State:** `docs/specs/` and `docs/plans/` are persistent state shared across build phases and potentially across sessions. The spec file written during brainstorm is read during planning; the plan file written during planning is read during implementation.
- **Generator-Verifier:** The brainstorming skill's spec review loop (spec self-review → user review gate) and the writing-plans skill's plan review loop are generator-verifier patterns with the user as the verifier.

**Content rules** (all sections):

- No `@` link references
- No file path references (`../`, `/plugins/`, `/skills/`, `/references/`)
- All content inline — no supporting files
- Tables for structured content; prose only for section leads (1-2 sentences each)
- Use skill names as cross-references (`superpowers:subagent-driven-development`, not paths)

### Change B — Add "Coordination Pattern" column to `sft-skill-comparison-matrix`

Location: `plugins/skill-factory-toolkit/skills/sft-skill-comparison-matrix/SKILL.md`

Add a single column to the "Primary Comparison Table" covering all 14 tracked skills. Values drawn from the pattern-to-exemplar mapping above (with some skills labeled "none" if they do not embody a specific pattern).

**Proposed column values:**

| Skill | Coordination Pattern |
|-------|----------------------|
| brainstorming | generator-verifier (with user) |
| dispatching-parallel-agents | agent teams |
| executing-plans | orchestrator-subagent |
| finishing-a-development-branch | none (utility) |
| receiving-code-review | generator-verifier (as generator side) |
| requesting-code-review | generator-verifier (as verifier side) |
| subagent-driven-development | orchestrator-subagent |
| systematic-debugging | none (discipline) |
| test-driven-development | generator-verifier (test/code) |
| using-git-worktrees | none (utility) |
| using-superpowers | none (meta-dispatcher) |
| verification-before-completion | none (discipline) |
| writing-plans | generator-verifier (with user) |
| writing-skills | none (meta) |

Add one short subsection after the table titled "Coordination Pattern" with one sentence pointing readers at `sft-coordination-patterns` for definitions and design principles.

Update the "Column Definitions" table to include the new column.

Update the Overview paragraph if it references "14 skills by X characteristics" to include coordination pattern as an additional characteristic.

### Change C — Soft nudge in `sft-build-plugin`

Location: `plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md`

Two modifications to the brainstorm phase — no new checklist items, no hard gates.

**Modification 1:** Update the Phase Map table row for Brainstorm:

Before:
```
| **Brainstorm** | `sft-skill-comparison-matrix`, `sft-skill-anatomy` | `superpowers:brainstorming` |
```

After:
```
| **Brainstorm** | `sft-skill-comparison-matrix`, `sft-skill-anatomy`, `sft-coordination-patterns` | `superpowers:brainstorming` |
```

**Modification 2:** Update step 1 of the checklist:

Before:
```
1. **Pre-load brainstorm context** — invoke `skill-factory-toolkit:sft-skill-comparison-matrix` and `skill-factory-toolkit:sft-skill-anatomy`
```

After:
```
1. **Pre-load brainstorm context** — invoke `skill-factory-toolkit:sft-skill-comparison-matrix`, `skill-factory-toolkit:sft-skill-anatomy`, and `skill-factory-toolkit:sft-coordination-patterns`
```

Nothing else in `sft-build-plugin` changes. The Quality Gate stays as-is. The Common Mistakes table stays as-is. The Refinement Workflow stays as-is.

## File Inventory

| File | Operation | Scope |
|------|-----------|-------|
| `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md` | create | ~1100-1400 words, reference type, 7 sections |
| `plugins/skill-factory-toolkit/skills/sft-skill-comparison-matrix/SKILL.md` | edit | Add "Coordination Pattern" column (14 cells), add short subsection, update column definitions |
| `plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md` | edit | Phase Map row update, checklist step 1 update |
| `plugins/skill-factory-toolkit/CHANGELOG.md` | edit | New `[3.2.0]` entry, Keep a Changelog format |
| `plugins/skill-factory-toolkit/.claude-plugin/plugin.json` | edit | Version bump `3.1.0` → `3.2.0` |
| `CLAUDE.md` (repo-level) | edit | Add `sft-coordination-patterns` row to Factory Toolkit table |
| `docs/specs/2026-04-11-coordination-patterns-design.md` | create | This document |

## Versioning

Per `sft-versioning-guide`:

- New skill added to an existing plugin → **minor bump**
- Existing skills receive additive non-breaking changes only (new column, new pre-load entry)
- No renames, no removals, no trigger changes
- Final version: **3.1.0 → 3.2.0**

## Validation & Testing

### Baseline test (RED phase)

Since `sft-coordination-patterns` is a reference skill, the baseline test follows the reference-type pattern: give Claude a prompt about building a new skill and see whether it can correctly identify the coordination pattern without the skill loaded.

**Test scenario:**

> "I want to build a skill that takes a user's PR, runs several quality checks in parallel against it, and synthesizes a single review. Which coordination pattern applies, and what structural markers should the SKILL.md have?"

**Expected RED (without `sft-coordination-patterns`):**
- Claude may not name a specific coordination pattern at all
- Claude may pick an arbitrary pattern without justifying it against the information flow
- Claude may not connect the pattern to concrete structural markers (prompt templates, review loops, synthesis steps)

**Expected GREEN (with `sft-coordination-patterns`):**
- Claude names **orchestrator-subagent** (possibly layered with generator-verifier for the synthesis step)
- Claude cites "parallel bounded subtasks" as the when-to-use criterion
- Claude points at `superpowers:subagent-driven-development` as the exemplar
- Claude names specific structural markers: prompt templates as supporting files, synthesis step in the orchestrator, review loop as optional layer

The baseline test result will be captured in a short note during implementation, following the format in `sft-cross-cutting-patterns/baseline-test-guide.md`.

### Validation gate

Run `bin/validate-plugin plugins/skill-factory-toolkit` after each file change. Must pass with zero errors. Checks include:

- `sft-coordination-patterns` frontmatter starts with "Use when" (triggering conditions only)
- No `@` links or file path references in any edited SKILL.md
- All `.md` files referenced in SKILL.md body exist in the skill directory (this skill has no supporting files)
- `plugin.json` version matches the latest changelog entry
- Semver validity of the bump

### Self-consistency checks

Hand-check the following beyond the validator:

- The "Coordination Pattern" column values in `sft-skill-comparison-matrix` match the pattern-to-exemplar mapping in `sft-coordination-patterns`
- The worked example in `sft-coordination-patterns` correctly describes `sft-build-plugin`'s actual structure (not aspirational)
- The brainstorm-phase changes in `sft-build-plugin` do not contradict the existing Flow Control section
- Word count for `sft-coordination-patterns` falls in the 1100-1400 range

### Quality gate (from `sft-build-plugin`)

Before marking the work complete:
- [ ] Description starts with "Use when" — triggering conditions only, no workflow summary
- [ ] Skill has all reference-type sections (tables primary, minimal prose)
- [ ] No `@` link references in any edited SKILL.md
- [ ] No file path references (`../`, `/plugins/`, `/skills/`, `/references/`) in any edited SKILL.md
- [ ] Word count appropriate for reference skill type
- [ ] `bin/validate-plugin plugins/skill-factory-toolkit` passes with zero errors
- [ ] Baseline test result documented (RED failure captured, GREEN verified)

## Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| The "message bus" pattern has no clean exemplar in the ecosystem, leaving a gap in the reference table | Explicitly note the gap rather than invent an exemplar. Reference skills are allowed to mark a row as "not yet exemplified." |
| Adding a column to `sft-skill-comparison-matrix` grows an already-dense table | Keep values to one short phrase per cell. Acceptable trade-off for discoverability. |
| Soft nudge is too soft — authors skip the pattern question entirely | Accepted. The article explicitly argues against pattern-for-pattern's-sake; a soft nudge matches "start simple." Escalation to hard enforcement is a future change, not this change. |
| Worked example in the new skill becomes stale if `sft-build-plugin` evolves | Keep the worked example short and link only to concepts (orchestration phases, shared state files), not to specific checklist step numbers. |

## Open Questions

None. All major decisions resolved during brainstorming.
