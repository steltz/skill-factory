---
name: sft-build-plugin
description: Use when building a new skill or plugin in the skill-factory repo — orchestrates the full brainstorm → plan → implement → package workflow with automatic sft skill loading at each phase
---

# Build Plugin

Single entry point for building skills and plugins in the skill-factory repo. Pre-loads the right sft skills at each phase boundary, then hands off to the superpowers workflow chain.

## When to Use

- Building a new skill or plugin from scratch in the skill-factory repo
- Adding a new skill to an existing plugin in the skill-factory repo

## When NOT to Use

- Editing an existing skill (use the Refinement Workflow below instead)
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
9. **Quality gate** — run the quality gate checklist below
10. **Finish** — invoke `superpowers:finishing-a-development-branch`

### Flow Control

**Steps 1-2 are atomic.** Do not invoke brainstorming without the sft context loaded first. Load both sft skills, then invoke brainstorming in the same turn.

**Steps 3-4 intercept brainstorming's exit.** When brainstorming completes, it will say "invoke writing-plans." STOP. Return to THIS checklist. Load the sft skills in step 3 BEFORE invoking writing-plans in step 4. This checklist is the authority — it overrides brainstorming's hardcoded exit.

**Steps 5-6 pass through.** Writing-plans will offer the execution choice (subagent vs inline). Follow its lead.

**Steps 7-8 fire after implementation.** When all implementation tasks are done, return to this checklist. Load sft-versioning-guide before touching the version or changelog.

## Quality Gate

Before finishing (step 10), verify ALL of these pass:

- [ ] Description starts with "Use when" and contains triggering conditions only (no workflow summary)
- [ ] Skill has all type-required sections (discipline: Iron Law, Red Flags, Rationalizations, Verification Checklist)
- [ ] All `.md` files referenced in SKILL.md body exist in the skill directory
- [ ] No `@` link references in SKILL.md
- [ ] No file path references (`../`, `/plugins/`, `/skills/`, `/references/`) in SKILL.md
- [ ] Word count is appropriate for skill type (see sft-cross-cutting-patterns Word Count Targets)
- [ ] `bin/validate-plugin plugins/<plugin-name>` passes with zero errors
- [ ] Baseline test is documented (failures recorded, mapped to skill sections)

If any item fails, fix it before proceeding to step 10.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Invoking brainstorming before loading sft skills | Steps 1-2 are atomic. Load first, invoke second. |
| Following brainstorming's "invoke writing-plans" exit directly | Return to this checklist. Step 3 must happen before step 4. |
| Skipping the version phase | Step 7 exists because this was missed in the first build. Don't skip it. |
| Loading all sft skills at once | Only load the subset for the current phase. The table tells you which. |

## Refinement Workflow

For iterating on existing skills (not building new ones):

1. Load `skill-factory-toolkit:sft-cross-cutting-patterns`
2. Read the target SKILL.md
3. Identify what needs to change (content, structure, enforcement, supporting files)
4. If adding supporting files or sections, check the comparison matrix: `skill-factory-toolkit:sft-skill-comparison-matrix`
5. Make the changes
6. Load `skill-factory-toolkit:sft-versioning-guide`
7. Determine bump type (patch for content fixes, minor for new sections/files)
8. Update version and CHANGELOG.md
9. Run `bin/validate-plugin plugins/<plugin-name>` to verify
