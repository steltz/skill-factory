# Skill Factory

A production environment for authoring, validating, and publishing independent [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill plugins. Patterns modeled after [superpowers](https://github.com/obra/superpowers) v5.0.6.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- [superpowers](https://github.com/obra/superpowers) plugin installed (used for brainstorming, planning, TDD, and code review workflows)

## Quick Start

```bash
# Clone the repo
git clone https://github.com/steltz/skill-factory.git
cd skill-factory

# Install the factory toolkit (gives Claude Code authoring skills)
claude plugin add ./plugins/skill-factory-toolkit

# Validate the toolkit is working
bin/validate-plugin plugins/skill-factory-toolkit
```

## Creating a New Plugin

The factory workflow uses superpowers skills to guide each phase:

### 1. Brainstorm

Start a Claude Code session and describe what you want to build. The `superpowers:brainstorming` skill will guide you through requirements and design, producing a spec in `docs/specs/`.

### 2. Plan

The `superpowers:writing-plans` skill breaks your spec into bite-sized implementation tasks, saved to `docs/plans/`.

### 3. Scaffold

When ready to create files, the `skill-factory-toolkit:scaffold-plugin` skill generates the correct directory structure:

```
plugins/my-plugin/
├── .claude-plugin/
│   └── plugin.json        # name, description, version
├── skills/
│   └── my-skill/
│       └── SKILL.md       # YAML frontmatter + skill content
├── CHANGELOG.md
```

### 4. Implement

Author each skill using `superpowers:test-driven-development`:

1. **RED** — Run a baseline test (subagent without the skill) to see what fails
2. **GREEN** — Write the SKILL.md content
3. **REFACTOR** — Close loopholes found during testing

### 5. Validate and Ship

```bash
# Check your plugin passes all quality gates
bin/validate-plugin plugins/my-plugin
```

## Plugin Structure

Every plugin under `plugins/` is self-contained and independently installable:

| File | Required | Purpose |
|------|----------|---------|
| `.claude-plugin/plugin.json` | Yes | Plugin metadata: `name`, `description`, `version` |
| `skills/<name>/SKILL.md` | Yes | Skill content with YAML frontmatter |
| `CHANGELOG.md` | Yes | Version history in [Keep a Changelog](https://keepachangelog.com) format |
| `hooks/hooks.json` | No | Claude Code session hooks |

Install any plugin with:

```bash
claude plugin add /path/to/skill-factory/plugins/<plugin-name>
```

## SKILL.md Format

Every skill file requires YAML frontmatter:

```yaml
---
name: my-skill-name
description: Use when <triggering conditions only>
---
```

Rules:
- `name` must be kebab-case and match the directory name
- `description` must start with "Use when" and describe only **when** to trigger the skill, never the workflow itself
- See `references/superpowers/skills/writing-skills/SKILL.md` for the full authoring guide

## Validation

Quality gates run automatically and on-demand:

- **Pre-commit hook** (`hooks/hooks.json`) — Runs `bin/validate-plugin` against all plugins before each commit
- **Manual validation** — `bin/validate-plugin plugins/<name>` checks:
  - `plugin.json` has required fields and valid semver
  - Directory name matches `plugin.json` name
  - `CHANGELOG.md` exists with an entry for the current version
  - Every skill has a `SKILL.md` with correct frontmatter
  - Skill names are kebab-case
  - Descriptions start with "Use when"

## Versioning

Plugins follow semantic versioning:

| Change | Bump |
|--------|------|
| Typo fix, clarification | Patch (x.y.**Z**) |
| New skill, new supporting file | Minor (x.**Y**.0) |
| Skill rename, removal, breaking trigger change | Major (**X**.0.0) |

## Factory Toolkit Skills

The `skill-factory-toolkit` plugin provides authoring guidance to Claude Code:

| Skill | Purpose |
|-------|---------|
| `scaffold-plugin` | Generates correct plugin directory structure and templates |
| `cross-cutting-patterns` | Patterns distilled from all 14 superpowers skills |
| `skill-anatomy` | Structural breakdown of 3 representative skill types |
| `skill-comparison-matrix` | Comparison table across all skills with decision guide |
| `versioning-guide` | Semver rules, changelog format, compatibility conventions |

## Reference Materials

`references/superpowers/` contains the complete skills directory from superpowers v5.0.6 (14 skills with all supporting files). These serve as the canonical examples for skill structure and conventions.

## Project Layout

```
skill-factory/
├── plugins/                    # Independent, installable plugins
│   └── skill-factory-toolkit/  # Built-in authoring toolkit
├── hooks/hooks.json            # Pre-commit validation hook
├── bin/validate-plugin         # Plugin validation script
├── references/superpowers/     # superpowers v5.0.6 reference files
├── docs/specs/                 # Design specs from brainstorming
├── docs/plans/                 # Implementation plans
└── CLAUDE.md                   # Project conventions (read by Claude Code)
```
