# Changelog

## [1.0.0] - 2026-03-30

### Changed

- Rewrote analyze-overfetch as a discipline skill with 4-phase pipeline (Inventory → Trace → Audit → Fix)
- Expanded scope to include GraphQL fragments as first-class analysis targets
- Added fragment dependency graph, multi-consumer tracking, and fragment splitting recommendations
- Added severity ranking (High/Medium/Low) in new Audit phase
- Strengthened enforcement with Iron Law, 13 red flags, 11-row rationalizations table, 8-item verification checklist, and When Stuck table

### Added

- tracing-patterns.md supporting file for language-specific field-access patterns (JS/TS, Python, Go)

## [0.1.0] - 2026-03-30

### Added

- analyze-overfetch skill for discovering and fixing GraphQL query over-fetching
