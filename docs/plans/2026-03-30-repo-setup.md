# Skill Factory Repo Setup Implementation Plan

> **For agentic workers:** This is a scaffolding plan — no production code, so TDD does not apply. Execute tasks sequentially and commit after each.

**Goal:** Set up the skill-factory repo with directory structure, CLAUDE.md, and curated superpowers reference files.

**Architecture:** Flat repo with `plugins/` for independent plugin projects, `references/superpowers/` for study material, and `docs/` for specs and plans.

**Tech Stack:** Markdown, JSON, git

---

### Task 1: Scaffold directory structure

**Files:**
- Create: `plugins/.gitkeep`
- Create: `docs/specs/.gitkeep` (already exists from spec, but ensure it's tracked)
- Create: `docs/plans/.gitkeep` (already exists from plan, but ensure it's tracked)
- Create: `references/superpowers/writing-skills/.gitkeep`
- Create: `references/superpowers/brainstorming/.gitkeep`
- Create: `references/superpowers/writing-plans/.gitkeep`
- Create: `references/superpowers/plugin-structure/.gitkeep`

- [ ] **Step 1: Create all directories with .gitkeep files**

```bash
mkdir -p plugins
touch plugins/.gitkeep

mkdir -p references/superpowers/writing-skills
mkdir -p references/superpowers/brainstorming
mkdir -p references/superpowers/writing-plans
mkdir -p references/superpowers/plugin-structure
```

Note: `docs/specs/` and `docs/plans/` already exist from the brainstorming phase.

- [ ] **Step 2: Commit scaffold**

```bash
git add plugins/.gitkeep references/superpowers/ docs/
git commit -m "chore: scaffold repo directory structure"
```

---

### Task 2: Copy superpowers reference files

**Files:**
- Create: `references/superpowers/writing-skills/SKILL.md` (copy from superpowers v5.0.6)
- Create: `references/superpowers/writing-skills/anthropic-best-practices.md` (copy)
- Create: `references/superpowers/writing-skills/persuasion-principles.md` (copy)
- Create: `references/superpowers/brainstorming/SKILL.md` (copy)
- Create: `references/superpowers/writing-plans/SKILL.md` (copy)
- Create: `references/superpowers/plugin-structure/plugin.json` (copy)
- Create: `references/superpowers/plugin-structure/hooks.json` (copy)

Source path: `~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.6/`

- [ ] **Step 1: Copy writing-skills reference files**

```bash
cp ~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.6/skills/writing-skills/SKILL.md references/superpowers/writing-skills/SKILL.md
cp ~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.6/skills/writing-skills/anthropic-best-practices.md references/superpowers/writing-skills/anthropic-best-practices.md
cp ~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.6/skills/writing-skills/persuasion-principles.md references/superpowers/writing-skills/persuasion-principles.md
```

- [ ] **Step 2: Copy brainstorming reference**

```bash
cp ~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.6/skills/brainstorming/SKILL.md references/superpowers/brainstorming/SKILL.md
```

- [ ] **Step 3: Copy writing-plans reference**

```bash
cp ~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.6/skills/writing-plans/SKILL.md references/superpowers/writing-plans/SKILL.md
```

- [ ] **Step 4: Copy plugin structure references**

```bash
cp ~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.6/.claude-plugin/plugin.json references/superpowers/plugin-structure/plugin.json
cp ~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.6/hooks/hooks.json references/superpowers/plugin-structure/hooks.json
```

- [ ] **Step 5: Verify all files copied correctly**

```bash
ls -la references/superpowers/writing-skills/
ls -la references/superpowers/brainstorming/
ls -la references/superpowers/writing-plans/
ls -la references/superpowers/plugin-structure/
```

Expected: 3 files in writing-skills, 1 in brainstorming, 1 in writing-plans, 2 in plugin-structure.

- [ ] **Step 6: Commit reference files**

```bash
git add references/superpowers/
git commit -m "docs: add curated superpowers v5.0.6 reference files"
```

---

### Task 3: Create references/superpowers/README.md

**Files:**
- Create: `references/superpowers/README.md`

- [ ] **Step 1: Write the README**

```markdown
# Superpowers Reference Files

Curated files from [superpowers](https://github.com/obra/superpowers) v5.0.6 for studying Claude Code skill authoring patterns.

## Contents

### writing-skills/
The authoritative guide on how to author Claude Code skills. Includes:
- `SKILL.md` — Skill authoring process, structure, naming, testing, and CSO (Claude Search Optimization)
- `anthropic-best-practices.md` — Anthropic's official skill authoring guidance
- `persuasion-principles.md` — Psychology research on making skills resistant to rationalization

### brainstorming/
- `SKILL.md` — The brainstorming process for turning ideas into validated designs before implementation

### writing-plans/
- `SKILL.md` — How to write bite-sized implementation plans from a design spec

### plugin-structure/
- `plugin.json` — Example plugin metadata file (from superpowers itself)
- `hooks.json` — Example hooks configuration

## Usage

Read and study these files to understand the patterns that make superpowers skills effective. When authoring new skills in this repo, follow the conventions documented here — especially `writing-skills/SKILL.md`.

Do not modify these files. If superpowers releases a new version, replace them with updated copies and note the new version here.

## Provenance

- **Source:** https://github.com/obra/superpowers
- **Version:** 5.0.6
- **Copied:** 2026-03-30
- **License:** MIT
```

- [ ] **Step 2: Commit**

```bash
git add references/superpowers/README.md
git commit -m "docs: add superpowers reference README with provenance"
```

---

### Task 4: Write CLAUDE.md

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Write CLAUDE.md**

```markdown
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

The authoritative guide is `references/superpowers/writing-skills/SKILL.md`. These rules are non-negotiable:

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

`references/superpowers/` contains curated files from superpowers v5.0.6. Read these to understand the patterns. See `references/superpowers/README.md` for contents and provenance.
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: add CLAUDE.md with project conventions and skill authoring standards"
```

---

### Task 5: Commit the spec and plan docs

**Files:**
- Existing: `docs/specs/2026-03-30-repo-setup-design.md`
- Existing: `docs/plans/2026-03-30-repo-setup.md`

- [ ] **Step 1: Stage and commit the design documents**

```bash
git add docs/specs/2026-03-30-repo-setup-design.md docs/plans/2026-03-30-repo-setup.md
git commit -m "docs: add repo setup design spec and implementation plan"
```

---

### Task 6: Verify final state

- [ ] **Step 1: Check repo structure matches spec**

```bash
find . -not -path './.git/*' -not -path './.git' | sort
```

Expected output:
```
.
./CLAUDE.md
./docs
./docs/plans
./docs/plans/2026-03-30-repo-setup.md
./docs/specs
./docs/specs/2026-03-30-repo-setup-design.md
./plugins
./plugins/.gitkeep
./references
./references/superpowers
./references/superpowers/README.md
./references/superpowers/brainstorming
./references/superpowers/brainstorming/SKILL.md
./references/superpowers/plugin-structure
./references/superpowers/plugin-structure/hooks.json
./references/superpowers/plugin-structure/plugin.json
./references/superpowers/writing-plans
./references/superpowers/writing-plans/SKILL.md
./references/superpowers/writing-skills
./references/superpowers/writing-skills/SKILL.md
./references/superpowers/writing-skills/anthropic-best-practices.md
./references/superpowers/writing-skills/persuasion-principles.md
```

- [ ] **Step 2: Check git log for clean commit history**

```bash
git log --oneline
```

Expected: 5 commits in order:
1. `chore: scaffold repo directory structure`
2. `docs: add curated superpowers v5.0.6 reference files`
3. `docs: add superpowers reference README with provenance`
4. `docs: add CLAUDE.md with project conventions and skill authoring standards`
5. `docs: add repo setup design spec and implementation plan`
