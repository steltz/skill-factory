# Skill Factory

A workspace for learning Claude Code skill authoring and producing independent skill plugins. Patterns and standards are modeled after [superpowers](https://github.com/obra/superpowers).

## Directory Structure

```
skill-factory/
├── CLAUDE.md
├── plugins/                           # Each subdirectory is an independent plugin
│   ├── skill-factory-toolkit/         # Self-hosted authoring toolkit
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── skills/
│   │   │   ├── sft-build-plugin/
│   │   │   ├── sft-scaffold-plugin/
│   │   │   ├── sft-cross-cutting-patterns/
│   │   │   ├── sft-skill-anatomy/
│   │   │   ├── sft-skill-comparison-matrix/
│   │   │   └── sft-versioning-guide/
│   │   └── CHANGELOG.md
│   └── <plugin-name>/                 # Your plugins follow this structure
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── skills/
│       │   └── <skill-name>/
│       │       └── SKILL.md
│       ├── CHANGELOG.md               # Required
│       └── hooks/                     # Optional
│           └── hooks.json
├── hooks/
│   └── hooks.json                     # Pre-commit validation
├── bin/
│   └── validate-plugin                # On-demand plugin validation
├── references/superpowers/            # Study material from superpowers v5.0.6
├── docs/
│   ├── specs/                         # Design specs from brainstorming
│   └── plans/                         # Implementation plans
```

## Skill Authoring Standards

The authoritative guide is `references/superpowers/skills/writing-skills/SKILL.md`. These rules are non-negotiable:

- Every SKILL.md has YAML frontmatter with `name` and `description`
- `description` starts with "Use when..." — triggering conditions only, never a workflow summary
- Skills use a flat namespace within each plugin's `skills/` directory
- Skill names use kebab-case with letters, numbers, and hyphens only
- Follow the TDD cycle: baseline test (RED) -> write skill (GREEN) -> close loopholes (REFACTOR)
- No skill ships without a failing baseline test first

## Plugin Packaging

Each plugin under `plugins/` is a self-contained, installable unit:

- `.claude-plugin/plugin.json` is required with fields: `name`, `description`, `version`
- `skills/` contains one or more skill directories, each with a `SKILL.md`
- `CHANGELOG.md` is required — follows Keep a Changelog format with an entry for every version
- `hooks/` is optional — include `hooks.json` only if the plugin needs session hooks
- Install: register the repo as a marketplace (`claude plugin marketplace add /path/to/skill-factory`), then `claude plugin install <plugin-name>`

See `references/superpowers/plugin-structure/` for examples.

## Governance

Plugins follow semantic versioning:

- **Patch** (x.y.Z) — Content fixes, typos, clarifications
- **Minor** (x.Y.0) — New skills, supporting files, non-breaking trigger changes
- **Major** (X.0.0) — Skill renames, removals, breaking trigger changes

See the `skill-factory-toolkit:sft-versioning-guide` skill for full rules, CHANGELOG format, and compatibility conventions.

## Validation

Repo-level hooks and scripts enforce plugin quality:

- **`hooks/hooks.json`** — Pre-commit hook that runs `bin/validate-plugin` against all staged plugins. Checks frontmatter format, naming conventions, plugin.json schema, and semver validity.
- **`bin/validate-plugin <plugin-dir>`** — On-demand full validation. Run manually to check a plugin before committing: `bin/validate-plugin plugins/<plugin-name>`

## Factory Toolkit

`plugins/skill-factory-toolkit/` is the self-hosted authoring toolkit. Install it to access these skills:

| Skill | Trigger |
|-------|---------|
| `sft-build-plugin` | Use when building a new skill or plugin |
| `sft-scaffold-plugin` | Use when creating a new plugin |
| `sft-cross-cutting-patterns` | Use when authoring a SKILL.md |
| `sft-skill-anatomy` | Use when studying how superpowers skills are structured |
| `sft-skill-comparison-matrix` | Use when deciding how to structure a new skill |
| `sft-versioning-guide` | Use when bumping versions or creating changelog entries |

Install: `claude plugin marketplace add /path/to/skill-factory && claude plugin install skill-factory-toolkit`

## Workflow

When creating new skills or plugins, invoke `skill-factory-toolkit:sft-build-plugin`. It orchestrates the full brainstorm → plan → implement → package workflow and automatically loads the right sft skills at each phase boundary.

## Skill Authoring Safety Net

When invoking `superpowers:writing-plans` in this repo, ALWAYS pre-load `sft-cross-cutting-patterns` and `sft-scaffold-plugin` first. This applies even if you arrived here from `superpowers:brainstorming` directly — the sft context is required for plan quality.

When bumping plugin versions or writing changelog entries in this repo, ALWAYS pre-load `sft-versioning-guide` first.

## Reference Files

`references/superpowers/` contains the complete skills directory from superpowers v5.0.6 — all 15 skills with supporting files. Read these to understand the patterns. See `references/superpowers/README.md` for contents and provenance.
