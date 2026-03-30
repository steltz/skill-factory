# Implicit sft Skill Activation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ensure sft skills auto-load as context when superpowers workflows run in this repo.

**Architecture:** Two-layer approach — CLAUDE.md directive (highest priority instructions) maps skill-authoring activities to contextual subsets of sft skills, and a SessionStart hook in the plugin prints the mapping as a system reminder at conversation start.

**Tech Stack:** Bash (hook script), JSON (hooks.json), Markdown (CLAUDE.md, CHANGELOG.md)

---

### Task 1: Add Skill Auto-Loading section to CLAUDE.md

**Files:**
- Modify: `CLAUDE.md:101` (insert after `## Workflow` section, before `## Reference Files`)

- [ ] **Step 1: Add the `## Skill Auto-Loading` section to CLAUDE.md**

Insert the following after line 101 (after the Workflow section's step 4) and before `## Reference Files`:

```markdown

## Skill Auto-Loading

When doing skill-authoring work in this repo, you MUST invoke the relevant sft skills before proceeding. This is not optional — these skills contain the patterns and standards that ensure quality.

| When you are... | Invoke these sft skills first |
|---|---|
| Brainstorming a new skill or plugin idea | `sft-skill-comparison-matrix`, `sft-skill-anatomy` |
| Planning skill implementation | `sft-cross-cutting-patterns`, `sft-scaffold-plugin` |
| Writing or editing a SKILL.md | `sft-cross-cutting-patterns`, `sft-skill-comparison-matrix` |
| Creating a new plugin | `sft-scaffold-plugin` |
| Bumping versions or writing changelog | `sft-versioning-guide` |

Load the mapped skills at the start of the relevant workflow stage. Do not load all 5 at once — only the subset for your current activity.
```

- [ ] **Step 2: Verify the edit**

Read `CLAUDE.md` and confirm:
- The new section appears between `## Workflow` and `## Reference Files`
- The table has 5 rows with the correct skill mappings
- No existing content was displaced

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: add skill auto-loading directives to CLAUDE.md"
```

---

### Task 2: Create SessionStart hook for skill-factory-toolkit

**Files:**
- Create: `plugins/skill-factory-toolkit/hooks/hooks.json`
- Create: `plugins/skill-factory-toolkit/hooks/run-hook.cmd`
- Create: `plugins/skill-factory-toolkit/hooks/session-start`

- [ ] **Step 1: Create the hooks directory**

```bash
mkdir -p plugins/skill-factory-toolkit/hooks
```

- [ ] **Step 2: Create `hooks.json`**

Write `plugins/skill-factory-toolkit/hooks/hooks.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 3: Create `run-hook.cmd`**

Write `plugins/skill-factory-toolkit/hooks/run-hook.cmd` — this is the cross-platform polyglot wrapper (same pattern as superpowers):

```bash
: << 'CMDBLOCK'
@echo off
if "%~1"=="" (
    echo run-hook.cmd: missing script name >&2
    exit /b 1
)
set "HOOK_DIR=%~dp0"
if exist "C:\Program Files\Git\bin\bash.exe" (
    "C:\Program Files\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)
if exist "C:\Program Files (x86)\Git\bin\bash.exe" (
    "C:\Program Files (x86)\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)
where bash >nul 2>nul
if %ERRORLEVEL% equ 0 (
    bash "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)
exit /b 0
CMDBLOCK

# Unix: run the named script directly
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

Make it executable:

```bash
chmod +x plugins/skill-factory-toolkit/hooks/run-hook.cmd
```

- [ ] **Step 4: Create `session-start` hook script**

Write `plugins/skill-factory-toolkit/hooks/session-start`:

```bash
#!/usr/bin/env bash
# SessionStart hook for skill-factory-toolkit plugin
# Prints sft skill auto-loading mapping as a system reminder

set -euo pipefail

REMINDER="[skill-factory-toolkit] When doing skill-authoring work, auto-load the relevant sft skills:\n\n  Brainstorming a skill/plugin  -> sft-skill-comparison-matrix, sft-skill-anatomy\n  Planning skill implementation -> sft-cross-cutting-patterns, sft-scaffold-plugin\n  Writing/editing a SKILL.md    -> sft-cross-cutting-patterns, sft-skill-comparison-matrix\n  Creating a new plugin         -> sft-scaffold-plugin\n  Bumping versions/changelog    -> sft-versioning-guide\n\nInvoke the mapped skills at the start of the relevant workflow stage. Do not load all 5 at once."

escape_for_json() {
    local s="$1"
    s="${s//\\/\\\\}"
    s="${s//\"/\\\"}"
    s="${s//$'\n'/\\n}"
    s="${s//$'\r'/\\r}"
    s="${s//$'\t'/\\t}"
    printf '%s' "$s"
}

escaped=$(escape_for_json "$REMINDER")

if [ -n "${CURSOR_PLUGIN_ROOT:-}" ]; then
  printf '{\n  "additional_context": "%s"\n}\n' "$escaped"
elif [ -n "${CLAUDE_PLUGIN_ROOT:-}" ]; then
  printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$escaped"
else
  printf '{\n  "additional_context": "%s"\n}\n' "$escaped"
fi

exit 0
```

Make it executable:

```bash
chmod +x plugins/skill-factory-toolkit/hooks/session-start
```

- [ ] **Step 5: Test the hook locally**

```bash
CLAUDE_PLUGIN_ROOT=plugins/skill-factory-toolkit bash plugins/skill-factory-toolkit/hooks/session-start
```

Expected: Valid JSON output with `hookSpecificOutput.additionalContext` containing the reminder text. Verify with:

```bash
CLAUDE_PLUGIN_ROOT=plugins/skill-factory-toolkit bash plugins/skill-factory-toolkit/hooks/session-start | python3 -m json.tool
```

Expected: Pretty-printed JSON, no parse errors.

- [ ] **Step 6: Commit**

```bash
git add plugins/skill-factory-toolkit/hooks/hooks.json plugins/skill-factory-toolkit/hooks/run-hook.cmd plugins/skill-factory-toolkit/hooks/session-start
git commit -m "feat(skill-factory-toolkit): add SessionStart hook for sft auto-loading reminders"
```

---

### Task 3: Bump version and update changelog

**Files:**
- Modify: `plugins/skill-factory-toolkit/.claude-plugin/plugin.json:4` (version field)
- Modify: `plugins/skill-factory-toolkit/CHANGELOG.md:3` (insert new version entry)

- [ ] **Step 1: Update plugin.json version**

In `plugins/skill-factory-toolkit/.claude-plugin/plugin.json`, change:

```json
"version": "2.0.0"
```

to:

```json
"version": "2.1.0"
```

- [ ] **Step 2: Add CHANGELOG entry**

In `plugins/skill-factory-toolkit/CHANGELOG.md`, insert after line 2 (`# Changelog`):

```markdown

## [2.1.0] - 2026-03-30

### Added

- SessionStart hook that prints sft skill auto-loading mapping at conversation start
- CLAUDE.md directive mapping skill-authoring activities to contextual sft skill subsets
```

- [ ] **Step 3: Run plugin validation**

```bash
bin/validate-plugin plugins/skill-factory-toolkit/
```

Expected: Passes with no errors. Validates that version `2.1.0` appears in CHANGELOG and plugin.json fields are valid.

- [ ] **Step 4: Commit**

```bash
git add plugins/skill-factory-toolkit/.claude-plugin/plugin.json plugins/skill-factory-toolkit/CHANGELOG.md
git commit -m "chore(skill-factory-toolkit): bump version to 2.1.0"
```
