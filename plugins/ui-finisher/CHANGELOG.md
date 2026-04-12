# Changelog

All notable changes to the `ui-finisher` plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this plugin adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-04-11

### Added

- `audit-ui-handlers` skill: six-phase audit pipeline (Resolve → Harvest → Trace → Classify → Propose → Approve & Patch) for finding unfinished interactive elements on a single page
- Phase 0 (Resolve) accepts a page URL or route as the primary input and maps it to a target file on disk via filesystem conventions (Next.js app/pages router, SvelteKit, Astro, Remix) with a bounded React Router fallback; direct file paths are still accepted for users who want to skip resolution
- Static-analysis discipline: the skill never fetches the URL, runs the app, or launches a headless browser
- `tiering-rules.md` supporting file with detailed tier classification heuristics
- `report-template.md` supporting file with exact report and QA checklist format
- Two rigid gates: no writes before user approval, no repo-wide grep beyond direct file imports during Phases 1–5 (Phase 0 has one narrow Grep allowance for route-registration files)
