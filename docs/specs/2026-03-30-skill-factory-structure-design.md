# Skill Factory Structure Updates — Design Spec

**Date:** 2026-03-30
**Status:** Draft

## Purpose

Update the skill-factory repo structure to make it a production-ready environment for authoring, validating, and governing Claude Code skill plugins. Three focus areas:

1. **Production readiness** — Scaffold tooling and authoring skills so Claude Code never starts from a blank page
2. **Learning optimization** — Machine-readable pattern references that Claude Code consults during skill creation (not human study guides)
3. **Governance & quality gates** — Versioning, changelogs, compatibility declarations, and mechanical validation

## Approach

**Hybrid:** Skills for brains, scripts for guardrails.

- A self-hosted plugin (`plugins/skill-factory-toolkit/`) with skills Claude Code invokes during authoring
- Repo-level hooks and scripts for mechanical validation that runs regardless of plugin install state

## Section 1: Plugin — `skill-factory-toolkit`

Located at `plugins/skill-factory-toolkit/`. Structure:

```
plugins/skill-factory-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── scaffold-plugin/
│   │   └── SKILL.md
│   ├── cross-cutting-patterns/
│   │   └── SKILL.md
│   ├── skill-anatomy/
│   │   └── SKILL.md
│   ├── skill-comparison-matrix/
│   │   └── SKILL.md
│   └── versioning-guide/
│       └── SKILL.md
├── CHANGELOG.md
```

### Skills

**`scaffold-plugin`**
- Trigger: "Use when creating a new plugin — generates the full directory structure, plugin.json, SKILL.md template with correct frontmatter, and optional hooks/"
- Generates a complete plugin scaffold so Claude Code doesn't create files from memory
- Produces: `.claude-plugin/plugin.json`, `skills/<name>/SKILL.md` (with YAML frontmatter template), `CHANGELOG.md`, optional `hooks/hooks.json`

**`cross-cutting-patterns`**
- Trigger: "Use when authoring a SKILL.md to ensure it follows established patterns across all superpowers skills"
- Machine-readable reference distilling patterns across all 14 superpowers skills
- Covers: frontmatter conventions, Claude Search Optimization (CSO) techniques, checklist patterns, supporting file usage, rigid vs. flexible skill types, word count targets, description format rules

**`skill-anatomy`**
- Trigger: "Use when studying how superpowers skills are structured to understand why each section exists"
- Breaks apart 2-3 representative skills piece by piece:
  - `brainstorming` — complex flexible skill with visual companion, multi-step checklist, supporting files
  - `test-driven-development` — rigid checklist skill, minimal supporting files, strict enforcement
  - `writing-skills` — meta/self-referential skill that teaches skill authoring using its own patterns
- For each: maps sections to the writing-skills authoring guide, explains why each section exists

**`skill-comparison-matrix`**
- Trigger: "Use when deciding how to structure a new skill — whether to use checklists, supporting files, rigid enforcement, etc."
- Structured comparison table covering all 14 skills across key dimensions:
  - Checklist vs. freeform flow
  - Rigid vs. flexible
  - Supporting files vs. self-contained
  - Approximate word count
  - CSO keyword strategy
  - Whether it dispatches to other skills

**`versioning-guide`**
- Trigger: "Use when bumping plugin versions, creating changelog entries, or declaring compatibility requirements"
- Semver rules, CHANGELOG.md format, compatibility conventions (see Section 3)

### plugin.json

```json
{
  "name": "skill-factory-toolkit",
  "description": "Authoring skills, pattern references, and governance guides for the skill factory",
  "version": "1.0.0"
}
```

## Section 2: Repo-Level Validation

### hooks/hooks.json

Claude Code session hooks at the repo root. Uses a `PreCommit` hook that triggers `bin/validate-plugin` against any staged plugin directories before a commit is finalized. This is a Claude Code hook (not a git hook) — it runs within Claude Code's hook system via `hooks.json`.

Files matched:
- `plugins/*/skills/*/SKILL.md`
- `plugins/*/.claude-plugin/plugin.json`

Checks enforced on commit:
- SKILL.md has YAML frontmatter with `name` and `description` fields
- `description` starts with "Use when"
- `name` is kebab-case (letters, numbers, hyphens only)
- `plugin.json` has required fields: `name`, `description`, `version`
- `version` is valid semver

### bin/validate-plugin

Standalone shell script for on-demand full validation of a plugin directory. Runs the same checks as the hook plus:
- Every skill directory under `skills/` contains a `SKILL.md`
- No orphan directories (directories without required files)
- `CHANGELOG.md` exists and has an entry for the current version
- Plugin directory name matches the `name` field in `plugin.json`

Usage: `bin/validate-plugin plugins/<plugin-name>`

## Section 3: Governance Conventions

### Plugin Versioning (Semver)

- **Patch** (x.y.Z) — Skill content fixes, typo corrections, clarifications
- **Minor** (x.Y.0) — New skills added, supporting files added, non-breaking changes to existing skill triggers
- **Major** (X.0.0) — Breaking changes to skill trigger descriptions, skill renames, skill removals

### CHANGELOG.md

Required in each plugin root. Format follows Keep a Changelog:

```markdown
# Changelog

## [1.0.0] - 2026-03-30
### Added
- Initial release with scaffold-plugin, cross-cutting-patterns, skill-anatomy, skill-comparison-matrix, versioning-guide skills
```

Sections: Added, Changed, Fixed, Removed. Each entry tied to a version and date.

### Compatibility Declarations

Optional `"compatibility"` field in `plugin.json` for declaring minimum Claude Code version or plugin dependencies. Not enforced mechanically — documented as a convention for future use.

```json
{
  "name": "example-plugin",
  "description": "Example",
  "version": "1.0.0",
  "compatibility": {
    "claude-code": ">=1.0.0"
  }
}
```

## Section 4: CLAUDE.md Updates

### Modified Sections

- **Directory Structure** — Update ASCII tree to include `hooks/`, `bin/`, and show `skill-factory-toolkit` as a concrete plugin example
- **Plugin Packaging** — Add CHANGELOG.md as a required file

### New Sections

- **Governance** — Semver rules for plugins, CHANGELOG.md requirement, compatibility convention. Points to `versioning-guide` skill for full details.
- **Validation** — Documents repo-level hooks and `bin/validate-plugin`. Explains how to install and run.
- **Factory Toolkit** — Notes `plugins/skill-factory-toolkit/` as the self-hosted authoring toolkit. Lists skills and their triggers so Claude Code knows when to invoke them.

### Unchanged Sections

- Skill Authoring Standards
- Workflow
- Reference Files

## Deliverables

1. `plugins/skill-factory-toolkit/` — Complete plugin with 5 skills and plugin.json
2. `hooks/hooks.json` — Pre-commit validation hooks
3. `bin/validate-plugin` — Standalone validation script
4. Updated `CLAUDE.md` — New sections for governance, validation, factory toolkit
5. Each skill's SKILL.md content authored using the superpowers writing-skills TDD process
