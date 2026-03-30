---
name: sft-scaffold-plugin
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

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── <skill-name>/
│       └── SKILL.md
├── CHANGELOG.md
```

### plugin.json

```json
{
  "name": "<plugin-name>",
  "description": "<one-line description of the plugin>",
  "version": "0.1.0"
}
```

Required fields: `name`, `description`, `version`. Version starts at `0.1.0` for new plugins. The `name` field MUST match the plugin directory name.

### SKILL.md Template

````markdown
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
````

### CHANGELOG.md

```markdown
# Changelog

## [0.1.0] - YYYY-MM-DD

### Added

- <skill-name> skill for <brief purpose>
```

### Optional: hooks/hooks.json

Only create if the plugin needs session hooks. Structure:

```json
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
```

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
