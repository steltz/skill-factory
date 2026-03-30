# Superpowers Reference Files

Complete skills directory from [superpowers](https://github.com/obra/superpowers) v5.0.6 for studying Claude Code skill authoring patterns.

## Contents

### skills/
All 15 superpowers skills with their supporting files, copied in full:

| Skill | Purpose |
|-------|---------|
| `brainstorming/` | Turn ideas into validated designs through Socratic dialogue |
| `dispatching-parallel-agents/` | Run independent tasks concurrently with subagents |
| `executing-plans/` | Batch execution of plans with review checkpoints |
| `finishing-a-development-branch/` | Guide completion of dev work (merge, PR, cleanup) |
| `receiving-code-review/` | Handle code review feedback with technical rigor |
| `requesting-code-review/` | Pre-submission quality review template |
| `subagent-driven-development/` | Per-task subagent dispatch with two-stage review |
| `systematic-debugging/` | 4-phase root cause investigation |
| `test-driven-development/` | RED-GREEN-REFACTOR cycle enforcement |
| `using-git-worktrees/` | Isolated workspace setup for feature work |
| `using-superpowers/` | Meta-skill for skill invocation and discovery |
| `verification-before-completion/` | Post-fix validation before claiming done |
| `writing-plans/` | Break specs into bite-sized implementation tasks |
| `writing-skills/` | Skill authoring guide (TDD applied to documentation) |

### plugin-structure/
- `plugin.json` — Example plugin metadata file (from superpowers itself)
- `hooks.json` — Example hooks configuration

## Usage

Read and study these files to understand the patterns that make superpowers skills effective. The most important reference for skill authoring is `skills/writing-skills/SKILL.md`.

Do not modify these files. If superpowers releases a new version, replace them with updated copies and note the new version here.

## Provenance

- **Source:** https://github.com/obra/superpowers
- **Version:** 5.0.6
- **Copied:** 2026-03-30
- **License:** MIT
