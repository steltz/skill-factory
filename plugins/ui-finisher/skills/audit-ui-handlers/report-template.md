# Report Template

Exact output format for `audit-ui-handlers`. SKILL.md references this file by name.

## Audit Report Format

After Phase 4 completes (but BEFORE any write), emit exactly this shape:

```
UI-Finisher Audit: <target-file-path>

[Tier 1 — will propose diffs]
  <N>. <element-description or handler name>  (line <line>)
     Issue: <one-line description of what is unfinished>
     Proposal: <one-line summary of the planned patch>
     Confidence: high

[Tier 2 — will propose diffs]
  <N>. <element-description or handler name>  (line <line>)
     Issue: <one-line description>
     Proposal: <one-line summary>
     Confidence: medium
     Rationale: <why this qualifies as Tier 2>

[Tier 3 — flagged, no proposal]
  <N>. <element-description or handler name>  (line <line>)
     Status: <why the skill is not acting>
     Note: <optional "could look wider" text if downgraded>

[Clean — not touching]
  - <element description> (line <line>)
  - <element description> (line <line>)
```

Omit any section that has no entries. Numbering restarts at 1 within each tier. Every proposal MUST also have a corresponding unified diff held in memory — the one-line summary is just the report view.

## Approval Prompt Format

Immediately after the report:

```
Proposals: <count> (<comma-separated proposal numbers>). Apply which?
  a) all
  b) pick (e.g., "1,3")
  c) none
```

Wait for a response. The default — and the response to anything ambiguous — is "none".

## QA Checklist Format

AFTER successful Edit calls, emit:

```
Manual QA Checklist for <target-file-path>:

[ ] <human action> — verify <observable outcome>
[ ] <human action> — verify:
    - <sub-check 1>
    - <sub-check 2>
    - <sub-check 3>
```

Rules:
- One checkbox per patched handler (Tier 1 + Tier 2 that the user accepted)
- Each checkbox begins with a concrete human action ("Click X", "Fill Y and submit", "Navigate to Z")
- Each checkbox ends with an observable outcome the human can verify
- Multi-step handlers get sub-check bullets (loading state, success toast, error toast, etc.)
- Do not include checklist items for Tier 3 or Clean findings

## Error Output

If any Edit fails during Phase 5:

```
UI-Finisher: Edit failed on proposal <N>. Stopping.
  Reason: <error message>
  Remaining proposals not applied: <list of numbers>
```

Do not auto-recover. Do not continue applying subsequent proposals after a failure.
