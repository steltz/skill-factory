# Skill Factory

A workspace for learning Claude Code skill authoring and producing independent skill plugins. Patterns and standards are modeled after [superpowers](https://github.com/obra/superpowers).

## Directory Structure

```
skill-factory/
├── CLAUDE.md
├── plugins/                           # Each subdirectory is an independent plugin
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json            # Required: name, description, version
│       ├── skills/                    # One or more skills (flat namespace)
│       │   └── <skill-name>/
│       │       └── SKILL.md           # Required: YAML frontmatter + content
│       └── hooks/                     # Optional
│           └── hooks.json
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
- `hooks/` is optional — include `hooks.json` only if the plugin needs session hooks
- Install with: `claude plugin add /path/to/skill-factory/plugins/<plugin-name>`

See `references/superpowers/plugin-structure/` for examples.

## Workflow

When creating new skills:

1. **Brainstorm** — Use superpowers:brainstorming to explore the idea and write a design spec to `docs/specs/`
2. **Plan** — Use superpowers:writing-plans to break the spec into bite-sized tasks, saved to `docs/plans/`
3. **Implement** — Use superpowers:test-driven-development to author the skill (baseline -> write -> close loopholes)
4. **Package** — Structure as a plugin with `.claude-plugin/plugin.json`

## Reference Files

`references/superpowers/` contains the complete skills directory from superpowers v5.0.6 — all 15 skills with supporting files. Read these to understand the patterns. See `references/superpowers/README.md` for contents and provenance.
