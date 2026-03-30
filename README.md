# Skill Factory

Domain expertise and guardrails for authoring Claude Code skill plugins.

[Superpowers](https://github.com/obra/superpowers) drives the process — brainstorming, planning, TDD, code review. Skill Factory provides what superpowers doesn't: reference material on what good skills look like, scaffolding to get the structure right, and automated checks to catch mistakes before you commit.

## What's in the Box

**Authoring toolkit** (`plugins/skill-factory-toolkit/`) — Five skills that give Claude Code the domain knowledge to write effective skills:

| Skill | What it provides |
|-------|-----------------|
| `sft-scaffold-plugin` | Generates correct plugin structure so Claude doesn't guess from memory |
| `sft-cross-cutting-patterns` | Patterns distilled from studying all 14 superpowers skills — section frequency, word count targets, when to inline vs. use supporting files |
| `sft-skill-anatomy` | Structural breakdowns of representative skills showing why they're built the way they are |
| `sft-skill-comparison-matrix` | Decision guide for choosing between rigid and flexible skill structures |
| `sft-versioning-guide` | Semver rules specific to plugins (skill renames = major bumps because they break `plugin:skill` invocations) |

**Validation infrastructure** — Automated quality gates that catch structural errors:

- `bin/validate-plugin` checks frontmatter format, naming conventions, plugin.json schema, semver validity, and changelog coverage
- `hooks/hooks.json` runs validation on every commit so broken plugins never land

**Reference library** (`references/superpowers/`) — The complete superpowers v5.0.6 skills directory (14 skills with all supporting files) as canonical examples of structure and conventions.

## Quick Start

```bash
git clone https://github.com/steltz/skill-factory.git
cd skill-factory

# Register the skill-factory as a local marketplace
claude plugin marketplace add /path/to/skill-factory

# Install the authoring toolkit
claude plugin install skill-factory-toolkit

# Verify everything works
bin/validate-plugin plugins/skill-factory-toolkit
```

Requires [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [superpowers](https://github.com/obra/superpowers).

## How Superpowers and Skill Factory Work Together

Superpowers provides the development methodology. Skill Factory provides the skill-specific knowledge that methodology operates on.

| Phase | Superpowers provides | Skill Factory provides |
|-------|---------------------|----------------------|
| Design | `brainstorming` — structured exploration of intent and requirements | — |
| Plan | `writing-plans` — breaks spec into tasks | — |
| Scaffold | — | `sft-scaffold-plugin` — correct directory structure, plugin.json, CHANGELOG, SKILL.md templates |
| Author | — | `sft-cross-cutting-patterns`, `sft-skill-anatomy`, `sft-skill-comparison-matrix` — what patterns work, what to avoid, how to structure each skill |
| Test | `test-driven-development` — RED/GREEN/REFACTOR cycle | Reference library — 14 real skills to compare against |
| Validate | — | `bin/validate-plugin` + pre-commit hooks — automated quality gates |
| Version | — | `sft-versioning-guide` — semver rules for plugins |

## Plugin Structure

Every plugin under `plugins/` is self-contained and independently installable:

```
plugins/my-plugin/
├── .claude-plugin/
│   └── plugin.json        # name, description, version (required)
├── skills/
│   └── my-skill/
│       └── SKILL.md       # YAML frontmatter + skill content (required)
├── CHANGELOG.md           # Keep a Changelog format (required)
└── hooks/                 # Session hooks (optional)
    └── hooks.json
```

Install any plugin:

```bash
claude plugin add /path/to/skill-factory/plugins/<plugin-name>
```

## SKILL.md Format

```yaml
---
name: my-skill-name
description: Use when <triggering conditions only>
---
```

- `name` must be kebab-case and match the directory name
- `description` must start with "Use when" and describe only **when** to trigger, never the workflow
- This matters for Claude Search Optimization (CSO) — when descriptions summarize workflow, Claude follows the summary and skips the skill body

## Versioning

| Change | Bump |
|--------|------|
| Typo fix, clarification | Patch (x.y.**Z**) |
| New skill, new supporting file | Minor (x.**Y**.0) |
| Skill rename, removal, breaking trigger change | Major (**X**.0.0) |
