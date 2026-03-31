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

## Skill Evolution Patterns

Typical version progression for a skill plugin, with the graphql-overfetch-analyzer as a real example:

| Stage | Version | What Happens | Example (graphql-overfetch-analyzer) |
|-------|---------|--------------|--------------------------------------|
| MVP | 0.1.0 | Core skill ships with baseline test passing. Minimal sections, no supporting files. | N/A (shipped at 1.0.0) |
| Production | 1.0.0 | Skill is complete: all type-required sections present, supporting files extracted, validation passes. | Core analyze-overfetch skill with Iron Law, rationalizations, red flags, verification checklist |
| Feature | 1.x.0 | New supporting files, expanded sections, additional modes. No breaking changes. | 1.1.0: added route-resolution-patterns.md; 1.2.0: added tracing-patterns.md; 1.3.0: route-scoped analysis mode |
| Breaking | 2.0.0 | Skill rename, trigger change, or removal. Existing invocations may break. | Not yet reached |

**When to go straight to 1.0.0:** If the skill was built using the full sft-build-plugin workflow (brainstorm → plan → implement → test), it ships production-ready. Use 0.x.0 only for experimental skills that need user feedback before stabilizing.

## Common Mistakes

- Forgetting to bump version after changes → Validation catches missing CHANGELOG entry
- Treating skill renames as minor → Renames break existing invocations, always major
- Separate commits for content and version → Keep them together so version always matches content
