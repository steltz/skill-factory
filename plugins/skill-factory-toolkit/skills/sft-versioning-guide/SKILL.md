---
name: sft-versioning-guide
description: Use when bumping plugin versions, creating changelog entries, or declaring compatibility requirements
---

# Versioning Guide

Semantic versioning rules and changelog conventions for skill-factory plugins.

## When to Use

- Preparing a commit that changes a plugin's version
- Writing CHANGELOG.md entries
- Deciding whether a change is patch, minor, or major
- Adding compatibility declarations to plugin.json
- NOT for skill content decisions — use sft-cross-cutting-patterns for that

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

````markdown
# Changelog

## [Unreleased]

### Added
- New feature or skill

## [1.1.0] - 2026-03-30

### Added
- new-skill skill for doing specific things

### Fixed
- Corrected typo in existing-skill description
````

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

````json
{
  "name": "my-plugin",
  "description": "Plugin description",
  "version": "1.0.0",
  "compatibility": {
    "claude-code": ">=1.0.0"
  }
}
````

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
