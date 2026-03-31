# Changelog

## [1.1.0] - 2026-03-30

### Added

- Route-scoped analysis mode — opt-in scope selection to analyze only the queries used by a single page or route
- Route Resolution Gate (Step 1.2) — framework detection, component set resolution, and user confirmation before query discovery
- Boundary fragment concept — fragments with consumers outside the scope boundary are flagged and handled conservatively
- Info severity level for boundary fragment fields that require full-project analysis to confirm
- route-resolution-patterns.md supporting file for framework-specific route-to-component mapping (Next.js, React Router, Vue Router, Angular Router)
- 5 red flags, 4 rationalizations, 4 verification checklist items, and 3 When Stuck rows for route-scoped analysis

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
