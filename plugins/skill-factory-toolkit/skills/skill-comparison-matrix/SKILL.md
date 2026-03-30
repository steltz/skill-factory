---
name: skill-comparison-matrix
description: Use when deciding how to structure a new skill — whether to use checklists, supporting files, rigid enforcement, etc.
---

# Skill Comparison Matrix

Machine-readable reference comparing all 14 superpowers skills (v5.0.6) by structure, enforcement, and design choices. Use this to pick the best structural template for a new skill.

## Primary Comparison Table

| Skill | Type | Flow | Enforcement | Supporting Files | Word Count | Dispatches To | CSO Strategy |
|-------|------|------|-------------|------------------|------------|---------------|--------------|
| brainstorming | technique | checklist | flexible | 7 (visual companion, scripts, reviewer prompt) | ~1550 | writing-plans | symptoms |
| dispatching-parallel-agents | technique | freeform | flexible | 0 | ~920 | none | symptoms |
| executing-plans | technique | checklist | flexible | 0 | ~360 | finishing-a-development-branch, using-git-worktrees, subagent-driven-development | tools |
| finishing-a-development-branch | technique | checklist | rigid | 0 | ~680 | none (called by others) | symptoms |
| receiving-code-review | discipline | checklist | rigid | 0 | ~930 | none | symptoms |
| requesting-code-review | technique | checklist | flexible | 1 (code-reviewer.md template) | ~400 | code-reviewer subagent | tools |
| subagent-driven-development | technique | hybrid | rigid | 3 (implementer, spec-reviewer, code-quality-reviewer prompts) | ~1530 | finishing-a-development-branch, requesting-code-review, test-driven-development, using-git-worktrees | mixed |
| systematic-debugging | discipline | checklist | rigid | 10 (techniques, tests, scripts) | ~1500 | test-driven-development, verification-before-completion | symptoms |
| test-driven-development | discipline | checklist | rigid | 1 (testing-anti-patterns.md) | ~1500 | none | symptoms |
| using-git-worktrees | technique | checklist | flexible | 0 | ~780 | finishing-a-development-branch | tools |
| using-superpowers | meta | hybrid | rigid | 2 (codex-tools.md, gemini-tools.md) | ~760 | any skill (dispatcher) | mixed |
| verification-before-completion | discipline | checklist | rigid | 0 | ~670 | none | symptoms |
| writing-plans | technique | checklist | flexible | 1 (plan-document-reviewer-prompt.md) | ~910 | subagent-driven-development, executing-plans | tools |
| writing-skills | meta | hybrid | rigid | 6 (testing guide, best practices, persuasion, graphviz, examples, renderer) | ~3210 | test-driven-development | mixed |

## Column Definitions

| Column | Values | Meaning |
|--------|--------|---------|
| Type | discipline / technique / reference / meta | What category of skill |
| Flow | checklist / freeform / hybrid | How steps are organized |
| Enforcement | rigid / flexible | Must follow exactly vs. adapt to context |
| Supporting Files | count (purpose) | Files beyond SKILL.md |
| Word Count | approximate | Size of SKILL.md only |
| Dispatches To | skill names or "none" | Skills explicitly invoked during execution |
| CSO Strategy | symptoms / tools / error messages / mixed | What keywords the description targets |

## Decision Guide: Choosing a Structural Template

Given characteristics of a new skill, pick the closest existing template:

| If your skill... | Model after | Why |
|------------------|-------------|-----|
| Enforces a process discipline (rules that must not be broken) | `test-driven-development` | Iron Law + rationalizations table + red flags + verification checklist |
| Guides a creative or exploratory process | `brainstorming` | Checklist with flexibility, visual companion pattern, user-gated progression |
| Is a reference that Claude looks up (tables, lookup data) | `cross-cutting-patterns` | Tables as primary format, minimal prose, scan-optimized |
| Dispatches subagents to do work | `subagent-driven-development` | Prompt templates as supporting files, status handling, review loops |
| Is a meta-skill about skills or tool usage | `writing-skills` | Self-referential structure, TDD mapping, skill type taxonomy |
| Coordinates a multi-step workflow | `executing-plans` | Lightweight checklist, integration section, calls other skills |
| Provides a behavioral protocol (how to respond) | `receiving-code-review` | Response pattern pseudocode, forbidden responses, source-specific handling |
| Offers a diagnostic methodology | `systematic-debugging` | Phased investigation, supporting technique files, escalation rules |
| Is a lightweight utility process | `using-git-worktrees` | Priority-ordered decision, safety checks, quick reference table |

## Structural Patterns by Type

### Discipline Skills

**Always have:**
- Iron Law (`NO X WITHOUT Y` in code block)
- Common Rationalizations table (`| Excuse | Reality |`)
- Red Flags list (bullet list of "STOP" thoughts)
- Verification Checklist (checkboxes)
- Anti-rationalization phrase: "Violating the letter of the rules is violating the spirit of the rules."

**Usually have:**
- Phase gates (must complete N before N+1)
- When Stuck table
- Real examples showing good vs. bad

**Skills:** test-driven-development, systematic-debugging, verification-before-completion, receiving-code-review

### Technique Skills

**Always have:**
- Step-by-step process (numbered or checkbox)
- When to Use / When NOT to Use
- Common Mistakes section

**Usually have:**
- Decision flowchart (graphviz dot)
- Integration section (Called by / Pairs with)
- Quick Reference table
- Real-World Example

**Skills:** brainstorming, dispatching-parallel-agents, executing-plans, finishing-a-development-branch, requesting-code-review, subagent-driven-development, using-git-worktrees, writing-plans

### Reference Skills

**Always have:**
- Tables as primary content
- Lookup-optimized structure (scan, not read)
- Minimal prose

**Usually have:**
- Column/field definitions
- Decision guide or lookup key
- No enforcement language

**Skills:** cross-cutting-patterns, skill-comparison-matrix (this skill)

### Meta Skills

**Always have:**
- Priority/precedence rules
- Red Flags rationalization table
- Cross-references to many other skills

**Usually have:**
- Flowcharts showing invocation logic
- Skill type taxonomy
- Self-referential examples

**Skills:** using-superpowers, writing-skills

## Supporting File Decision Tree

```
Does your skill need supporting files?

1. All content < 100 lines?
   YES -> Inline everything. No supporting files. (9 of 14 skills do this)

2. Reusable prompt templates for subagents?
   YES -> Separate .md files (see subagent-driven-development: 3 prompt files)

3. Heavy reference material (500+ lines)?
   YES -> Separate reference files (see writing-skills: 6 files)

4. Interactive tools (scripts, HTML)?
   YES -> Separate scripts/ directory (see brainstorming: scripts/)

5. Test scenarios for skill validation?
   YES -> Separate test-*.md files (see systematic-debugging: 3 test files)
```

## Size Calibration

| Complexity | Word Count | Example |
|------------|------------|---------|
| Minimal (lightweight utility) | 300-400 | executing-plans |
| Standard (single technique) | 600-950 | finishing-a-development-branch, dispatching-parallel-agents, writing-plans |
| Complex (discipline with checklists) | 1400-1600 | test-driven-development, systematic-debugging, brainstorming |
| Comprehensive (meta with taxonomy) | 3000+ | writing-skills |

**Rule of thumb:** Discipline skills run 2-3x longer than technique skills due to rationalization tables and enforcement language.

## Dispatch Topology

Skills that dispatch others (orchestrators):
- `brainstorming` -> `writing-plans`
- `writing-plans` -> `subagent-driven-development` or `executing-plans`
- `executing-plans` -> `finishing-a-development-branch`
- `subagent-driven-development` -> `finishing-a-development-branch`, `requesting-code-review`, `test-driven-development`
- `using-superpowers` -> any skill (meta-dispatcher)

Skills that are dispatched (leaves):
- `test-driven-development` (called by subagents)
- `verification-before-completion` (called during finishing)
- `using-git-worktrees` (called before execution starts)
- `finishing-a-development-branch` (called after execution completes)

Standalone skills (neither dispatch nor are dispatched as part of a chain):
- `dispatching-parallel-agents`
- `receiving-code-review`
