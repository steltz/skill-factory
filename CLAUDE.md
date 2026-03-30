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
│   │   │   ├── scaffold-plugin/
│   │   │   ├── cross-cutting-patterns/
│   │   │   ├── skill-anatomy/
│   │   │   ├── skill-comparison-matrix/
│   │   │   └── versioning-guide/
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
- Install with: `claude plugin add /path/to/skill-factory/plugins/<plugin-name>`

See `references/superpowers/plugin-structure/` for examples.

## Governance

Plugins follow semantic versioning:

- **Patch** (x.y.Z) — Content fixes, typos, clarifications
- **Minor** (x.Y.0) — New skills, supporting files, non-breaking trigger changes
- **Major** (X.0.0) — Skill renames, removals, breaking trigger changes

See the `skill-factory-toolkit:versioning-guide` skill for full rules, CHANGELOG format, and compatibility conventions.

## Validation

Repo-level hooks and scripts enforce plugin quality:

- **`hooks/hooks.json`** — Pre-commit hook that runs `bin/validate-plugin` against all staged plugins. Checks frontmatter format, naming conventions, plugin.json schema, and semver validity.
- **`bin/validate-plugin <plugin-dir>`** — On-demand full validation. Run manually to check a plugin before committing: `bin/validate-plugin plugins/<plugin-name>`

## Factory Toolkit

`plugins/skill-factory-toolkit/` is the self-hosted authoring toolkit. Install it to access these skills:

| Skill | Trigger |
|-------|---------|
| `scaffold-plugin` | Use when creating a new plugin |
| `cross-cutting-patterns` | Use when authoring a SKILL.md |
| `skill-anatomy` | Use when studying how superpowers skills are structured |
| `skill-comparison-matrix` | Use when deciding how to structure a new skill |
| `versioning-guide` | Use when bumping versions or creating changelog entries |

Install: `claude plugin add /path/to/skill-factory/plugins/skill-factory-toolkit`

## Workflow

When creating new skills:

1. **Brainstorm** — Use superpowers:brainstorming to explore the idea and write a design spec to `docs/specs/`
2. **Plan** — Use superpowers:writing-plans to break the spec into bite-sized tasks, saved to `docs/plans/`
3. **Implement** — Use superpowers:test-driven-development to author the skill (baseline -> write -> close loopholes)
4. **Package** — Structure as a plugin with `.claude-plugin/plugin.json`

## Reference Files

`references/superpowers/` contains the complete skills directory from superpowers v5.0.6 — all 15 skills with supporting files. Read these to understand the patterns. See `references/superpowers/README.md` for contents and provenance.
