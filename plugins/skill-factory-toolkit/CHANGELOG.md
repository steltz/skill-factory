# Changelog

## [3.2.0] - 2026-04-11

### Added

- `sft-coordination-patterns` reference skill describing the five multi-agent coordination patterns (generator-verifier, orchestrator-subagent, agent teams, message bus, shared state), five design principles, pattern-to-structure cheat sheet, and a worked example showing how the skill-factory-toolkit itself exemplifies layered orchestrator-subagent + shared state + generator-verifier patterns
- "Coordination Pattern" column in `sft-skill-comparison-matrix` primary comparison table
- `sft-coordination-patterns` pre-loaded during the Brainstorm phase of `sft-build-plugin`

## [3.1.0] - 2026-03-31

### Added

- Semantic validation checks in `bin/validate-plugin`: description length, workflow summary detection, discipline skill section enforcement, supporting file reference validation, forbidden @ links and path references
- Skill Type Decision Tree flowchart in sft-skill-comparison-matrix
- baseline-test-guide.md supporting file in sft-cross-cutting-patterns
- Quality Gate checklist (step 9) in sft-build-plugin
- Refinement Workflow section in sft-build-plugin
- Skill Evolution Patterns section in sft-versioning-guide
- Self-contained subsystem branch (item 6) in Supporting File Decision Tree

### Changed

- sft-skill-anatomy SKILL.md split for token efficiency: detailed analyses extracted to skill-analyses.md, SKILL.md now loads lean (~400 tokens)
- SessionStart hook is now context-aware: detects SKILL.md in cwd, matching plans in docs/plans/, with fallback to default message
- sft-build-plugin "When NOT to Use" now references Refinement Workflow instead of direct sft-cross-cutting-patterns loading

## [3.0.0] - 2026-03-30

### Added

- sft-build-plugin skill — single entry point for building skills/plugins with automatic sft pre-loading at each phase

### Changed

- **BREAKING:** CLAUDE.md Workflow section and Skill Auto-Loading table replaced with sft-build-plugin directive
- **BREAKING:** SessionStart hook message simplified to point at sft-build-plugin

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
