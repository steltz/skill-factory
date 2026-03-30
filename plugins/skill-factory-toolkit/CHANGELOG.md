# Changelog

## [2.1.0] - 2026-03-30

### Added

- SessionStart hook that prints sft skill auto-loading mapping at conversation start
- CLAUDE.md directive mapping skill-authoring activities to contextual sft skill subsets

## [2.0.0] - 2026-03-30

### Changed

- **BREAKING:** Renamed all skills with `sft-` prefix for autocomplete discoverability
  - scaffold-plugin → sft-scaffold-plugin
  - cross-cutting-patterns → sft-cross-cutting-patterns
  - skill-anatomy → sft-skill-anatomy
  - skill-comparison-matrix → sft-skill-comparison-matrix
  - versioning-guide → sft-versioning-guide

## [1.0.0] - 2026-03-30

### Added

- scaffold-plugin skill for generating new plugin directory structures
- cross-cutting-patterns skill distilling authoring patterns across superpowers v5.0.6
- skill-anatomy skill breaking down representative skill structures
- skill-comparison-matrix skill comparing structural dimensions across all skills
- versioning-guide skill for semver rules, changelog format, and compatibility conventions
