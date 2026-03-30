# Skill Factory Repo Setup Design

## Purpose

This repo is a **skill factory** — a workspace for learning how Claude Code skills work (by studying superpowers) and for producing independent Claude Code skill plugins.

Each plugin produced here is a self-contained, installable unit that can contain one or more related skills.

## Directory Structure

```
skill-factory/
├── CLAUDE.md                          # Project instructions
├── plugins/                           # Each subdirectory is an independent plugin
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json            # Plugin metadata (required)
│       ├── skills/                    # One or more skills
│       │   ├── <skill-a>/
│       │   │   └── SKILL.md
│       │   └── <skill-b>/
│       │       └── SKILL.md
│       └── hooks/                     # Optional hooks
│           └── hooks.json
├── references/superpowers/            # Curated superpowers files for study
│   ├── README.md                      # Version, contents, usage notes
│   ├── writing-skills/
│   │   ├── SKILL.md
│   │   ├── anthropic-best-practices.md
│   │   └── persuasion-principles.md
│   ├── brainstorming/
│   │   └── SKILL.md
│   ├── writing-plans/
│   │   └── SKILL.md
│   └── plugin-structure/
│       ├── plugin.json
│       └── hooks.json
├── docs/
│   ├── specs/                         # Brainstorming design docs
│   └── plans/                         # Implementation plans
```

## CLAUDE.md Content

### Section 1: Project Overview

Declares the repo as a skill factory for producing Claude Code plugins. Dual purpose: learning from superpowers + producing skills.

### Section 2: Directory Structure

Documents the layout above so Claude always knows where things go.

### Section 3: Skill Authoring Standards

A short, directive list of non-negotiable rules — not a reproduction of the writing-skills guide. The CLAUDE.md states the rules and points to `references/superpowers/writing-skills/SKILL.md` as the authoritative, detailed reference:

- Every SKILL.md needs YAML frontmatter with `name` and `description`
- Description starts with "Use when..." — triggering conditions only, never workflow summary
- Flat namespace within each plugin's `skills/` directory
- Follow TDD cycle for skill creation (baseline test -> write skill -> close loopholes)

### Section 4: Plugin Packaging Convention

How each plugin directory must be structured to be installable:
- `.claude-plugin/plugin.json` with required fields (`name`, `description`, `version`)
- Optional `hooks/` directory with `hooks.json`
- Each plugin is installable via `claude plugin add /path/to/skill-factory/plugins/<plugin-name>`

### Section 5: Reference Files

What's in `references/superpowers/`, version (5.0.6), and usage guidance: read and study to understand patterns, don't copy verbatim.

Reference files included:
- `writing-skills/` — The authoritative guide on skill authoring (SKILL.md + supporting docs)
- `brainstorming/` — The design process reference
- `writing-plans/` — The plan authoring reference
- `plugin-structure/` — Example plugin.json and hooks.json

### Section 6: Workflow

Expected flow when creating a new skill:
1. Brainstorm (use superpowers:brainstorming) -> write spec to `docs/specs/`
2. Write plan (use superpowers:writing-plans) -> write plan to `docs/plans/`
3. Implement skill using TDD cycle (use superpowers:test-driven-development)
4. Package as plugin with `.claude-plugin/plugin.json`

## Deliverables

1. `CLAUDE.md` at repo root
2. Directory structure scaffolded (empty dirs with .gitkeep where needed)
3. Reference files copied from superpowers v5.0.6
4. `references/superpowers/README.md` documenting provenance
