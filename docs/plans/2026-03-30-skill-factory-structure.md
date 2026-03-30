# Skill Factory Structure Updates — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add production tooling, machine-readable pattern references, and governance conventions to the skill-factory repo via a self-hosted plugin and repo-level validation.

**Architecture:** Hybrid approach — a `plugins/skill-factory-toolkit/` plugin containing 5 skills that Claude Code invokes during authoring, plus repo-level `hooks/hooks.json` and `bin/validate-plugin` for mechanical validation. CLAUDE.md updated with new governance, validation, and toolkit sections.

**Tech Stack:** Claude Code skills (SKILL.md markdown), shell scripting (bash for `bin/validate-plugin`), Claude Code hooks (hooks.json)

**Spec:** `docs/specs/2026-03-30-skill-factory-structure-design.md`

---

## File Structure

```
skill-factory/
├── CLAUDE.md                                          # MODIFY — add governance, validation, toolkit sections; update directory tree
├── plugins/
│   └── skill-factory-toolkit/
│       ├── .claude-plugin/
│       │   └── plugin.json                            # CREATE — plugin metadata
│       ├── skills/
│       │   ├── scaffold-plugin/
│       │   │   └── SKILL.md                           # CREATE — plugin scaffolding skill
│       │   ├── cross-cutting-patterns/
│       │   │   └── SKILL.md                           # CREATE — pattern reference for authoring
│       │   ├── skill-anatomy/
│       │   │   └── SKILL.md                           # CREATE — structural breakdown of representative skills
│       │   ├── skill-comparison-matrix/
│       │   │   └── SKILL.md                           # CREATE — comparison table across all skills
│       │   └── versioning-guide/
│       │       └── SKILL.md                           # CREATE — semver and changelog governance
│       └── CHANGELOG.md                               # CREATE — initial changelog
├── hooks/
│   └── hooks.json                                     # CREATE — pre-commit validation hook
├── bin/
│   └── validate-plugin                                # CREATE — standalone validation script
```

---

### Task 1: Scaffold the plugin directory structure and plugin.json

**Files:**
- Create: `plugins/skill-factory-toolkit/.claude-plugin/plugin.json`
- Create: `plugins/skill-factory-toolkit/CHANGELOG.md`
- Create: `plugins/skill-factory-toolkit/skills/.gitkeep` (temporary, removed as skills are added)

- [ ] **Step 1: Create the plugin.json**

Create `plugins/skill-factory-toolkit/.claude-plugin/plugin.json`:

```json
{
  "name": "skill-factory-toolkit",
  "description": "Authoring skills, pattern references, and governance guides for the skill factory",
  "version": "1.0.0"
}
```

- [ ] **Step 2: Create the CHANGELOG.md**

Create `plugins/skill-factory-toolkit/CHANGELOG.md`:

```markdown
# Changelog

## [1.0.0] - 2026-03-30

### Added

- scaffold-plugin skill for generating new plugin directory structures
- cross-cutting-patterns skill distilling authoring patterns across superpowers v5.0.6
- skill-anatomy skill breaking down representative skill structures
- skill-comparison-matrix skill comparing structural dimensions across all skills
- versioning-guide skill for semver rules, changelog format, and compatibility conventions
```

- [ ] **Step 3: Create temporary .gitkeep for skills directory**

Create `plugins/skill-factory-toolkit/skills/.gitkeep` as an empty file. This will be removed once the first skill is added.

- [ ] **Step 4: Commit**

```bash
git add plugins/skill-factory-toolkit/.claude-plugin/plugin.json plugins/skill-factory-toolkit/CHANGELOG.md plugins/skill-factory-toolkit/skills/.gitkeep
git commit -m "chore: scaffold skill-factory-toolkit plugin structure"
```

---

### Task 2: Create the `bin/validate-plugin` script

This must exist before the hooks (Task 3) since the hook delegates to this script. It must also exist before skills (Tasks 4-8) so we can validate them as they're created.

**Files:**
- Create: `bin/validate-plugin`

- [ ] **Step 1: Write the validation script**

Create `bin/validate-plugin`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Validate a skill-factory plugin directory.
# Usage: bin/validate-plugin plugins/<plugin-name>

PLUGIN_DIR="${1:?Usage: bin/validate-plugin <plugin-dir>}"

if [[ ! -d "$PLUGIN_DIR" ]]; then
  echo "FAIL: Directory does not exist: $PLUGIN_DIR"
  exit 1
fi

ERRORS=0

fail() {
  echo "FAIL: $1"
  ERRORS=$((ERRORS + 1))
}

# --- plugin.json checks ---

PLUGIN_JSON="$PLUGIN_DIR/.claude-plugin/plugin.json"

if [[ ! -f "$PLUGIN_JSON" ]]; then
  fail "Missing $PLUGIN_JSON"
else
  # Required fields: name, description, version
  for field in name description version; do
    val=$(python3 -c "import json,sys; d=json.load(open('$PLUGIN_JSON')); print(d.get('$field',''))" 2>/dev/null || true)
    if [[ -z "$val" ]]; then
      fail "plugin.json missing required field: $field"
    fi
  done

  # Version must be valid semver (basic check: X.Y.Z where X,Y,Z are numbers)
  version=$(python3 -c "import json; print(json.load(open('$PLUGIN_JSON')).get('version',''))" 2>/dev/null || true)
  if [[ -n "$version" ]] && ! echo "$version" | grep -qE '^[0-9]+\.[0-9]+\.[0-9]+$'; then
    fail "plugin.json version '$version' is not valid semver (expected X.Y.Z)"
  fi

  # Plugin directory name should match plugin.json name
  dir_name=$(basename "$PLUGIN_DIR")
  json_name=$(python3 -c "import json; print(json.load(open('$PLUGIN_JSON')).get('name',''))" 2>/dev/null || true)
  if [[ -n "$json_name" && "$dir_name" != "$json_name" ]]; then
    fail "Directory name '$dir_name' does not match plugin.json name '$json_name'"
  fi
fi

# --- CHANGELOG.md checks ---

CHANGELOG="$PLUGIN_DIR/CHANGELOG.md"

if [[ ! -f "$CHANGELOG" ]]; then
  fail "Missing $CHANGELOG"
else
  # Check that current version appears in CHANGELOG
  if [[ -n "${version:-}" ]] && ! grep -qF "[$version]" "$CHANGELOG"; then
    fail "CHANGELOG.md has no entry for current version $version"
  fi
fi

# --- Skills checks ---

SKILLS_DIR="$PLUGIN_DIR/skills"

if [[ -d "$SKILLS_DIR" ]]; then
  for skill_dir in "$SKILLS_DIR"/*/; do
    [[ -d "$skill_dir" ]] || continue
    skill_name=$(basename "$skill_dir")

    # Skip .gitkeep-only directories
    if [[ "$skill_name" == ".gitkeep" ]]; then
      continue
    fi

    skill_md="$skill_dir/SKILL.md"

    if [[ ! -f "$skill_md" ]]; then
      fail "Skill '$skill_name' missing SKILL.md"
      continue
    fi

    # Check YAML frontmatter exists (starts with ---)
    first_line=$(head -1 "$skill_md")
    if [[ "$first_line" != "---" ]]; then
      fail "Skill '$skill_name' SKILL.md missing YAML frontmatter (must start with ---)"
      continue
    fi

    # Extract frontmatter (between first and second ---)
    frontmatter=$(sed -n '1,/^---$/{ /^---$/d; p; }' "$skill_md" | tail -n +1)

    # Check name field exists and is kebab-case
    fm_name=$(echo "$frontmatter" | grep -E '^name:' | sed 's/^name:[[:space:]]*//' | tr -d '"' | tr -d "'" || true)
    if [[ -z "$fm_name" ]]; then
      fail "Skill '$skill_name' SKILL.md frontmatter missing 'name' field"
    elif ! echo "$fm_name" | grep -qE '^[a-z0-9]+(-[a-z0-9]+)*$'; then
      fail "Skill '$skill_name' name '$fm_name' is not kebab-case (letters, numbers, hyphens only)"
    fi

    # Check description field exists and starts with "Use when"
    fm_desc=$(echo "$frontmatter" | grep -E '^description:' | sed 's/^description:[[:space:]]*//' | sed 's/^["'"'"']//' | sed 's/["'"'"']$//' || true)
    if [[ -z "$fm_desc" ]]; then
      fail "Skill '$skill_name' SKILL.md frontmatter missing 'description' field"
    elif ! echo "$fm_desc" | grep -qiE '^Use when'; then
      fail "Skill '$skill_name' description must start with 'Use when' (got: '${fm_desc:0:40}...')"
    fi
  done
fi

# --- Orphan directory check ---

if [[ -d "$SKILLS_DIR" ]]; then
  for entry in "$SKILLS_DIR"/*/; do
    [[ -d "$entry" ]] || continue
    entry_name=$(basename "$entry")
    [[ "$entry_name" == ".gitkeep" ]] && continue
    if [[ ! -f "$entry/SKILL.md" ]]; then
      fail "Orphan directory: $entry (no SKILL.md)"
    fi
  done
fi

# --- Summary ---

if [[ $ERRORS -gt 0 ]]; then
  echo ""
  echo "Validation FAILED with $ERRORS error(s)"
  exit 1
else
  echo "Validation PASSED: $PLUGIN_DIR"
  exit 0
fi
```

- [ ] **Step 2: Make the script executable**

Run: `chmod +x bin/validate-plugin`

- [ ] **Step 3: Test the script against the toolkit plugin**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected output: `Validation PASSED: plugins/skill-factory-toolkit`

(The plugin has no skill directories yet beyond `.gitkeep`, so only plugin.json and CHANGELOG checks run.)

- [ ] **Step 4: Test the script catches errors**

Create a temporary bad plugin to verify failure detection:

```bash
mkdir -p /tmp/bad-plugin/.claude-plugin
echo '{"name": "wrong-name"}' > /tmp/bad-plugin/.claude-plugin/plugin.json
bin/validate-plugin /tmp/bad-plugin
```

Expected: Multiple FAIL lines (missing description, missing version, no CHANGELOG, name mismatch). Exit code 1.

Clean up: `rm -rf /tmp/bad-plugin`

- [ ] **Step 5: Commit**

```bash
git add bin/validate-plugin
git commit -m "feat: add bin/validate-plugin for plugin validation"
```

---

### Task 3: Create `hooks/hooks.json`

**Files:**
- Create: `hooks/hooks.json`

- [ ] **Step 1: Write the hooks file**

Create `hooks/hooks.json`:

```json
{
  "hooks": {
    "PreCommit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "for dir in plugins/*/; do [ -f \"$dir/.claude-plugin/plugin.json\" ] && bin/validate-plugin \"$dir\"; done",
            "async": false
          }
        ]
      }
    ]
  }
}
```

This runs `bin/validate-plugin` against every plugin directory that has a `plugin.json` before any commit is finalized.

- [ ] **Step 2: Commit**

```bash
git add hooks/hooks.json
git commit -m "feat: add pre-commit validation hook for plugin quality gates"
```

---

### Task 4: Write the `scaffold-plugin` skill

**Files:**
- Create: `plugins/skill-factory-toolkit/skills/scaffold-plugin/SKILL.md`
- Remove: `plugins/skill-factory-toolkit/skills/.gitkeep` (no longer needed)

This skill is authored using the TDD process from `writing-skills/SKILL.md`: baseline test (without the skill) → write skill → close loopholes.

- [ ] **Step 1: Run baseline test (RED)**

Dispatch a subagent with the following prompt and NO access to the skill:

> "You are setting up a new Claude Code plugin called `example-weather` in a `plugins/` directory. Create the full directory structure, plugin.json, and a SKILL.md template for a skill called `fetch-forecast`. Follow superpowers conventions."

Evaluate the output. Without the scaffold-plugin skill, the subagent will likely:
- Miss CHANGELOG.md
- Use incorrect frontmatter format (e.g., description that summarizes workflow instead of "Use when...")
- Omit required plugin.json fields or use wrong structure
- May not follow kebab-case naming conventions consistently

Document which failures occurred.

- [ ] **Step 2: Write the skill (GREEN)**

Create `plugins/skill-factory-toolkit/skills/scaffold-plugin/SKILL.md`:

```markdown
---
name: scaffold-plugin
description: Use when creating a new plugin — generates the full directory structure, plugin.json, SKILL.md template with correct frontmatter, and optional hooks/
---

# Scaffold Plugin

Generate a complete, valid plugin directory structure so you never create plugin files from memory.

## When to Use

- Starting a new plugin from scratch
- Adding a new skill to an existing plugin (use the SKILL.md template section only)
- NOT for modifying existing skills — edit them directly

## Plugin Directory Template

When asked to create a new plugin, generate this exact structure:

​```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── <skill-name>/
│       └── SKILL.md
├── CHANGELOG.md
​```

### plugin.json

​```json
{
  "name": "<plugin-name>",
  "description": "<one-line description of the plugin>",
  "version": "0.1.0"
}
​```

Required fields: `name`, `description`, `version`. Version starts at `0.1.0` for new plugins. The `name` field MUST match the plugin directory name.

### SKILL.md Template

​```markdown
---
name: <skill-name>
description: Use when <triggering conditions only — never summarize the workflow>
---

# <Skill Title>

<Core principle in 1-2 sentences.>

## When to Use

- <Symptom or triggering condition>
- NOT for <explicit exclusion>

## <Core Section>

<Skill content — checklists, patterns, reference material, code examples as appropriate>

## Common Mistakes

- <What goes wrong> → <Fix>
​```

### CHANGELOG.md

​```markdown
# Changelog

## [0.1.0] - YYYY-MM-DD

### Added

- <skill-name> skill for <brief purpose>
​```

### Optional: hooks/hooks.json

Only create if the plugin needs session hooks. Structure:

​```json
{
  "hooks": {
    "<HookType>": [
      {
        "matcher": "<pattern>",
        "hooks": [
          {
            "type": "command",
            "command": "<command>",
            "async": false
          }
        ]
      }
    ]
  }
}
​```

## Naming Rules

- Plugin directory name: kebab-case, letters/numbers/hyphens only
- Skill directory name: kebab-case, letters/numbers/hyphens only
- Plugin name in plugin.json MUST match the directory name
- Skill name in SKILL.md frontmatter MUST match the skill directory name

## After Scaffolding

1. Run `bin/validate-plugin plugins/<plugin-name>` to confirm the structure is valid
2. Author the skill content using superpowers:test-driven-development
3. Commit the scaffold before writing skill content — keep scaffolding and content in separate commits

## Quick Reference

| Field | Location | Rule |
|-------|----------|------|
| plugin name | plugin.json `name` | Must match directory name |
| plugin version | plugin.json `version` | Start at `0.1.0`, semver |
| skill name | SKILL.md frontmatter `name` | Kebab-case, match directory |
| skill description | SKILL.md frontmatter `description` | Starts with "Use when", triggering conditions only |
| changelog | CHANGELOG.md | Keep a Changelog format, entry for every version |
```

- [ ] **Step 3: Run baseline test again (GREEN)**

Dispatch the same subagent prompt from Step 1, this time with access to the scaffold-plugin skill. Verify the output now:
- Includes CHANGELOG.md with correct format
- Has plugin.json with all required fields
- Has SKILL.md with correct frontmatter (description starts with "Use when...")
- Uses kebab-case consistently
- Suggests running `bin/validate-plugin` after scaffolding

- [ ] **Step 4: Close loopholes (REFACTOR)**

Review the skill for gaps:
- Does it handle the case of adding a skill to an existing plugin (not just new plugins)?
- Are the naming rules unambiguous?
- Is the SKILL.md template complete enough that Claude Code won't fill in wrong patterns?

Fix any issues found.

- [ ] **Step 5: Remove .gitkeep and commit**

```bash
rm plugins/skill-factory-toolkit/skills/.gitkeep
git add plugins/skill-factory-toolkit/skills/scaffold-plugin/SKILL.md
git rm plugins/skill-factory-toolkit/skills/.gitkeep
git commit -m "feat: add scaffold-plugin skill for generating plugin structures"
```

- [ ] **Step 6: Validate**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: `Validation PASSED: plugins/skill-factory-toolkit`

---

### Task 5: Write the `cross-cutting-patterns` skill

**Files:**
- Create: `plugins/skill-factory-toolkit/skills/cross-cutting-patterns/SKILL.md`

**Context needed:** Read all 14 SKILL.md files in `references/superpowers/skills/` to extract patterns. This is a research-heavy task.

- [ ] **Step 1: Run baseline test (RED)**

Dispatch a subagent with this prompt and NO access to the skill:

> "You are authoring a new SKILL.md for a skill called `migrate-database`. The skill helps with database migration tasks. Write the complete SKILL.md following superpowers conventions. Pay attention to frontmatter format, description rules, structure, and Claude Search Optimization."

Evaluate: Without the cross-cutting-patterns skill, the subagent will likely produce a reasonable SKILL.md but miss subtle patterns (e.g., CSO keyword placement, when to use checklists vs. freeform, correct word count targets, how to structure supporting files).

Document which patterns were missed.

- [ ] **Step 2: Research — extract patterns from all 14 skills**

Read every SKILL.md in `references/superpowers/skills/`. For each, note:
- Frontmatter format and description phrasing
- Section structure (which sections appear, in what order)
- Whether it uses checklists, flowcharts, tables, or freeform prose
- How it references other skills
- Word count (approximate)
- Whether it has supporting files and what they contain
- CSO keywords used (error messages, tool names, symptoms)
- Rigid vs. flexible enforcement style

- [ ] **Step 3: Write the skill (GREEN)**

Create `plugins/skill-factory-toolkit/skills/cross-cutting-patterns/SKILL.md`. The skill must cover:

**Frontmatter conventions:**
- `name`: kebab-case, must match directory name
- `description`: starts with "Use when", triggering conditions only, never workflow summary, max 1024 chars

**CSO (Claude Search Optimization):**
- Description triggers skill loading — if it summarizes workflow, Claude follows the summary and skips the skill content
- Embed symptom keywords throughout skill body: error messages, tool names, command names, common synonyms
- Front-load the most important content — Claude may not read to the end of long skills

**Structural patterns observed across all 14 skills:**
- Overview (1-2 sentences, core principle)
- When to Use / When NOT to Use
- Checklist (if discipline-enforcing) or Core Pattern (if technique/reference)
- Process Flow (graphviz dot notation for complex flows)
- Quick Reference (table or bullets for scanning)
- Common Mistakes (what goes wrong → fix)
- Anti-patterns section (optional, for discipline skills)

**Skill types and their structural signatures:**
- Discipline-enforcing (TDD, verification): Hard gates, checklists, red-flag tables, anti-rationalization lists
- Technique (debugging, brainstorming): Step-by-step phases, decision trees, flexible adaptation
- Reference (patterns, anatomy): Tables, structured data, lookup-optimized format
- Meta (using-superpowers, writing-skills): Self-referential, teach-by-example

**Supporting file conventions:**
- Inline everything unless content exceeds ~100 lines of reference material
- Supporting files are for: reusable templates, prompt text for subagents, large code examples, scripts
- Reference with relative paths from the skill directory

**Word count targets:**
- Frequently-loaded skills: <200 words
- Standard skills: <500 words
- Complex skills with checklists and examples: size as needed, but front-load critical content

**Cross-skill references:**
- Use `**REQUIRED SUB-SKILL:** Use superpowers:<skill-name>` format
- Never use `@` links (they force-load files)
- Reference by skill name, not file path

- [ ] **Step 4: Run baseline test again (GREEN)**

Re-dispatch the same subagent prompt from Step 1, this time with access to the cross-cutting-patterns skill. Verify the output now demonstrates improved pattern adherence.

- [ ] **Step 5: Close loopholes (REFACTOR)**

Review: Is there any pattern present in the superpowers skills that this reference misses? Is any guidance ambiguous enough that Claude Code could misinterpret it?

- [ ] **Step 6: Commit and validate**

```bash
git add plugins/skill-factory-toolkit/skills/cross-cutting-patterns/SKILL.md
git commit -m "feat: add cross-cutting-patterns skill for authoring guidance"
bin/validate-plugin plugins/skill-factory-toolkit
```

---

### Task 6: Write the `skill-anatomy` skill

**Files:**
- Create: `plugins/skill-factory-toolkit/skills/skill-anatomy/SKILL.md`

**Context needed:** Deep reading of `brainstorming/SKILL.md`, `test-driven-development/SKILL.md`, and `writing-skills/SKILL.md` from the references.

- [ ] **Step 1: Run baseline test (RED)**

Dispatch a subagent with this prompt and NO access to the skill:

> "Explain the structural differences between the `brainstorming` and `test-driven-development` skills in superpowers. Why does brainstorming have a visual companion and supporting files while TDD is mostly self-contained? How does each skill's structure serve its purpose?"

Evaluate: Without the skill-anatomy skill, the subagent will give generic answers and miss the structural reasoning (why checklist order matters, why brainstorming has a hard gate, how supporting files relate to the main SKILL.md).

- [ ] **Step 2: Write the skill (GREEN)**

Create `plugins/skill-factory-toolkit/skills/skill-anatomy/SKILL.md`. Break down three representative skills:

**Skill 1: `brainstorming`** (complex flexible skill)
- Frontmatter: aggressive CSO with "MUST" language in description
- Hard gate: prevents premature implementation
- Checklist: 9 ordered steps with explicit task tracking
- Process flow: graphviz dot diagram showing state machine
- Visual companion: separate supporting file for browser-based mockups
- Supporting files: spec reviewer prompt, visual companion guide, server scripts
- Why this structure: brainstorming is high-risk for scope creep and premature action; the checklist and hard gate enforce discipline while the flexible conversation within each step adapts to context

**Skill 2: `test-driven-development`** (rigid checklist skill)
- Frontmatter: concise trigger, no workflow summary
- Core pattern: RED → GREEN → REFACTOR with exact steps
- Anti-rationalization table: "Too simple to test" → rebuttal format
- Single supporting file: `testing-anti-patterns.md`
- Minimal word count: discipline skills must be fast to load
- Why this structure: TDD is a discipline that engineers resist; the skill's job is to prevent rationalization, not teach testing theory

**Skill 3: `writing-skills`** (meta/self-referential skill)
- Frontmatter: demonstrates its own CSO rules
- TDD applied to documentation: baseline test → write → close loopholes
- Multiple supporting files: best practices, persuasion principles, examples, testing guide
- Why this structure: meta-skills must teach by example; the skill itself follows every convention it describes

For each skill: map every section to the writing-skills authoring guide, explain why the section exists, and note what would break if it were removed.

- [ ] **Step 3: Run baseline test again (GREEN)**

Re-dispatch the same prompt with access to the skill. Verify the response now gives structurally precise answers grounded in the anatomy breakdown.

- [ ] **Step 4: Close loopholes (REFACTOR)**

Review: Does the anatomy breakdown help Claude Code decide which structural pattern to follow for a new skill? Are the "why" explanations concrete enough?

- [ ] **Step 5: Commit and validate**

```bash
git add plugins/skill-factory-toolkit/skills/skill-anatomy/SKILL.md
git commit -m "feat: add skill-anatomy skill with structural breakdowns"
bin/validate-plugin plugins/skill-factory-toolkit
```

---

### Task 7: Write the `skill-comparison-matrix` skill

**Files:**
- Create: `plugins/skill-factory-toolkit/skills/skill-comparison-matrix/SKILL.md`

**Context needed:** All 14 SKILL.md files from references, plus the patterns extracted in Task 5.

- [ ] **Step 1: Run baseline test (RED)**

Dispatch a subagent with this prompt and NO access to the skill:

> "I'm creating a new skill that helps developers set up CI/CD pipelines. Should this skill use a checklist or freeform flow? Should it be rigid or flexible? Should it have supporting files? How long should it be?"

Evaluate: Without the comparison matrix, the subagent will guess based on general principles rather than referencing how existing skills with similar characteristics are structured.

- [ ] **Step 2: Research — catalog all 14 skills**

For each of the 14 superpowers skills, record:
- Skill name
- Type: discipline / technique / reference / meta
- Flow: checklist / freeform / hybrid
- Enforcement: rigid / flexible
- Supporting files: count and purpose
- Approximate word count of SKILL.md
- Dispatches to other skills: yes/no (which ones)
- CSO keyword strategy: symptoms / tools / error messages / mixed

- [ ] **Step 3: Write the skill (GREEN)**

Create `plugins/skill-factory-toolkit/skills/skill-comparison-matrix/SKILL.md`. Include:

**Primary comparison table** with columns:
| Skill | Type | Flow | Enforcement | Supporting Files | Word Count | Dispatches To | CSO Strategy |

All 14 rows filled with data from Step 2.

**Decision guide** — given characteristics of a new skill, which existing skill is the best structural template:
- "If your skill enforces a process discipline → model after `test-driven-development`"
- "If your skill guides a creative/exploratory process → model after `brainstorming`"
- "If your skill is a reference that Claude Code looks up → model after `cross-cutting-patterns`"
- "If your skill dispatches subagents → model after `subagent-driven-development`"
- "If your skill is a meta-skill about skills → model after `writing-skills`"

**Structural patterns by type** — what sections discipline skills always have vs. technique skills vs. reference skills.

- [ ] **Step 4: Run baseline test again (GREEN)**

Re-dispatch the same prompt with access to the skill. Verify the response now references specific comparable skills and makes structurally grounded recommendations.

- [ ] **Step 5: Close loopholes (REFACTOR)**

Review: Is the table complete? Does the decision guide cover all reasonable new skill types? Are there edge cases (e.g., a skill that's both discipline and technique)?

- [ ] **Step 6: Commit and validate**

```bash
git add plugins/skill-factory-toolkit/skills/skill-comparison-matrix/SKILL.md
git commit -m "feat: add skill-comparison-matrix with structural decision guide"
bin/validate-plugin plugins/skill-factory-toolkit
```

---

### Task 8: Write the `versioning-guide` skill

**Files:**
- Create: `plugins/skill-factory-toolkit/skills/versioning-guide/SKILL.md`

- [ ] **Step 1: Run baseline test (RED)**

Dispatch a subagent with this prompt and NO access to the skill:

> "I just added a new skill to my plugin and fixed a typo in an existing skill's description. I also renamed another skill from `db-migrate` to `database-migration`. What version should my plugin be now (it was 1.2.0), and what should the CHANGELOG entry look like?"

Evaluate: Without the versioning-guide skill, the subagent will likely give reasonable but inconsistent advice (might miss that renaming a skill is a breaking change, might not use Keep a Changelog format).

- [ ] **Step 2: Write the skill (GREEN)**

Create `plugins/skill-factory-toolkit/skills/versioning-guide/SKILL.md`:

```markdown
---
name: versioning-guide
description: Use when bumping plugin versions, creating changelog entries, or declaring compatibility requirements
---

# Versioning Guide

Semantic versioning rules and changelog conventions for skill-factory plugins.

## When to Use

- Preparing a commit that changes a plugin's version
- Writing CHANGELOG.md entries
- Deciding whether a change is patch, minor, or major
- Adding compatibility declarations to plugin.json
- NOT for skill content decisions — use cross-cutting-patterns for that

## Semver Rules for Plugins

| Change Type | Bump | Examples |
|-------------|------|----------|
| Content fix | Patch (x.y.Z) | Typo in SKILL.md, clarification, rewording |
| New skill added | Minor (x.Y.0) | Adding a new skill directory with SKILL.md |
| New supporting file | Minor (x.Y.0) | Adding examples, templates, reference docs |
| Non-breaking trigger change | Minor (x.Y.0) | Expanding when a skill fires (wider match) |
| Skill renamed | Major (X.0.0) | Changing a skill directory name |
| Skill removed | Major (X.0.0) | Deleting a skill directory |
| Breaking trigger change | Major (X.0.0) | Narrowing or changing when a skill fires |

**Key principle:** If existing users invoking `plugin-name:skill-name` would break or get different behavior than expected, it's a major version bump.

## CHANGELOG.md Format

Every plugin MUST have a `CHANGELOG.md` in its root directory. Format follows [Keep a Changelog](https://keepachangelog.com):

​```markdown
# Changelog

## [Unreleased]

### Added
- New feature or skill

## [1.1.0] - 2026-03-30

### Added
- new-skill skill for doing specific things

### Fixed
- Corrected typo in existing-skill description
​```

**Sections** (use only those that apply):
- **Added** — New skills, supporting files, features
- **Changed** — Modifications to existing skills
- **Fixed** — Bug fixes, typo corrections
- **Removed** — Removed skills or files

**Rules:**
- Every version in plugin.json MUST have a matching `## [X.Y.Z]` entry in CHANGELOG.md
- Use `## [Unreleased]` for changes not yet versioned
- Most recent version appears first
- Date format: YYYY-MM-DD

## Compatibility Declarations

Optional field in plugin.json for future use:

​```json
{
  "name": "my-plugin",
  "description": "Plugin description",
  "version": "1.0.0",
  "compatibility": {
    "claude-code": ">=1.0.0"
  }
}
​```

Not mechanically enforced. Document it when you know a plugin requires specific Claude Code features.

## Version Bump Workflow

1. Make your changes to skill content
2. Determine bump type using the semver table above
3. Update `version` in plugin.json
4. Add CHANGELOG.md entry under the new version heading
5. Run `bin/validate-plugin plugins/<plugin-name>` to confirm
6. Commit all changes together (content + version bump + changelog)

## Common Mistakes

- Forgetting to bump version after changes → Validation catches missing CHANGELOG entry
- Treating skill renames as minor → Renames break existing invocations, always major
- Separate commits for content and version → Keep them together so version always matches content
```

- [ ] **Step 3: Run baseline test again (GREEN)**

Re-dispatch the same prompt from Step 1 with access to the skill. Verify the response now:
- Correctly identifies the rename as a major bump (1.2.0 → 2.0.0)
- Uses Keep a Changelog format
- Lists all three changes in the correct sections (Added, Fixed, Changed/Removed)

- [ ] **Step 4: Close loopholes (REFACTOR)**

Review: Are there edge cases not covered? (e.g., What about changing a skill's description field — is that patch or minor? Answer: it depends on whether the trigger conditions narrow, which the semver table covers.)

- [ ] **Step 5: Commit and validate**

```bash
git add plugins/skill-factory-toolkit/skills/versioning-guide/SKILL.md
git commit -m "feat: add versioning-guide skill for semver and changelog governance"
bin/validate-plugin plugins/skill-factory-toolkit
```

---

### Task 9: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Update the Directory Structure section**

Replace the existing ASCII tree with:

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

- [ ] **Step 2: Update the Plugin Packaging section**

Add CHANGELOG.md as a required file. The section should read:

```markdown
## Plugin Packaging

Each plugin under `plugins/` is a self-contained, installable unit:

- `.claude-plugin/plugin.json` is required with fields: `name`, `description`, `version`
- `skills/` contains one or more skill directories, each with a `SKILL.md`
- `CHANGELOG.md` is required — follows Keep a Changelog format with an entry for every version
- `hooks/` is optional — include `hooks.json` only if the plugin needs session hooks
- Install with: `claude plugin add /path/to/skill-factory/plugins/<plugin-name>`

See `references/superpowers/plugin-structure/` for examples.
```

- [ ] **Step 3: Add the Governance section**

Add after Plugin Packaging:

```markdown
## Governance

Plugins follow semantic versioning:

- **Patch** (x.y.Z) — Content fixes, typos, clarifications
- **Minor** (x.Y.0) — New skills, supporting files, non-breaking trigger changes
- **Major** (X.0.0) — Skill renames, removals, breaking trigger changes

See the `skill-factory-toolkit:versioning-guide` skill for full rules, CHANGELOG format, and compatibility conventions.
```

- [ ] **Step 4: Add the Validation section**

Add after Governance:

```markdown
## Validation

Repo-level hooks and scripts enforce plugin quality:

- **`hooks/hooks.json`** — Pre-commit hook that runs `bin/validate-plugin` against all staged plugins. Checks frontmatter format, naming conventions, plugin.json schema, and semver validity.
- **`bin/validate-plugin <plugin-dir>`** — On-demand full validation. Run manually to check a plugin before committing: `bin/validate-plugin plugins/<plugin-name>`
```

- [ ] **Step 5: Add the Factory Toolkit section**

Add after Validation:

```markdown
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
```

- [ ] **Step 6: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md with governance, validation, and toolkit sections"
```

---

### Task 10: Final validation and cleanup

**Files:**
- Remove: `plugins/.gitkeep` (the toolkit plugin now provides directory content)

- [ ] **Step 1: Run full validation**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: `Validation PASSED: plugins/skill-factory-toolkit`

- [ ] **Step 2: Remove plugins/.gitkeep**

```bash
git rm plugins/.gitkeep
```

- [ ] **Step 3: Verify all skills are accessible**

List all skill directories:

```bash
ls -1 plugins/skill-factory-toolkit/skills/
```

Expected output:
```
cross-cutting-patterns
scaffold-plugin
skill-anatomy
skill-comparison-matrix
versioning-guide
```

- [ ] **Step 4: Verify CLAUDE.md is complete**

Read CLAUDE.md and confirm it has all sections:
1. Directory Structure (updated)
2. Skill Authoring Standards (unchanged)
3. Plugin Packaging (updated)
4. Governance (new)
5. Validation (new)
6. Factory Toolkit (new)
7. Workflow (unchanged)
8. Reference Files (unchanged)

- [ ] **Step 5: Commit cleanup**

```bash
git add -A
git commit -m "chore: remove plugins/.gitkeep, finalize structure"
```
