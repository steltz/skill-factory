---
name: sft-cross-cutting-patterns
description: Use when authoring a SKILL.md to ensure it follows established patterns across all superpowers skills
---

# Cross-Cutting Patterns

Machine-readable reference for SKILL.md authoring. Patterns extracted from 14 superpowers skills (v5.0.6).

## Frontmatter Rules

| Field | Rule |
|-------|------|
| `name` | kebab-case, letters/numbers/hyphens only, must match directory name |
| `description` | Starts with "Use when", max 1024 chars, triggering conditions ONLY |

**Description traps:**
- NEVER summarize workflow (Claude follows description, skips skill body)
- NEVER use first person
- NEVER describe what the skill does, only WHEN to use it

## Skill Types and Structural Signatures

### Discipline-Enforcing

**Examples:** test-driven-development, verification-before-completion, systematic-debugging

**Structural markers:**
- Iron Law (a `NO X WITHOUT Y` declaration in code block)
- Hard gates (`<HARD-GATE>` tags or equivalent)
- Phase gates (must complete phase N before phase N+1)
- Common Rationalizations table (`| Excuse | Reality |`)
- Red Flags list (thoughts that mean STOP)
- Anti-rationalization phrase: "Violating the letter of the rules is violating the spirit of the rules."
- Verification Checklist (checkboxes at end)

**Enforcement style:** Rigid. No adaptation. Follow exactly.

### Technique

**Examples:** brainstorming, dispatching-parallel-agents, requesting-code-review

**Structural markers:**
- Step-by-step phases (numbered process)
- Decision trees (dot flowcharts for "when to use")
- When to Use / When NOT to Use sections
- Real-World Example with concrete scenario
- Flexible adaptation allowed

**Enforcement style:** Flexible. Adapt principles to context.

### Reference

**Examples:** sft-cross-cutting-patterns (this skill), skill comparison matrices

**Structural markers:**
- Tables as primary content format
- Lookup-optimized structure (scan, don't read linearly)
- Minimal prose, maximal structure
- No enforcement language

**Enforcement style:** Informational. Consult as needed.

### Meta

**Examples:** using-superpowers, writing-skills

**Structural markers:**
- Self-referential (teaches by example)
- Priority/precedence rules
- Flowcharts showing skill invocation logic
- Red Flags rationalization table
- Cross-references to many other skills

**Enforcement style:** Rigid for invocation rules, flexible for application.

## Section Inventory

Sections observed across all 14 skills, ordered by frequency:

| Section | Frequency | Purpose |
|---------|-----------|---------|
| Overview | 14/14 | 1-2 sentences, core principle |
| Common Mistakes / Rationalizations | 12/14 | Table format: `Excuse \| Reality` or `Mistake \| Fix` |
| Red Flags | 10/14 | Bullet list of thoughts that mean STOP |
| Integration / Cross-references | 10/14 | Called by, Pairs with, Required sub-skills |
| When to Use / When NOT to Use | 9/14 | Bullet lists or dot flowchart |
| Process / Checklist | 9/14 | Numbered steps or checkbox items |
| Quick Reference | 7/14 | Table for scanning |
| Process Flow | 6/14 | Graphviz dot notation |
| Real-World Example | 6/14 | Concrete scenario with outcome |
| Anti-patterns | 4/14 | What NOT to do, with why |

**Required sections (all skills):** Frontmatter, Overview.

**Strongly recommended:** Common Mistakes, Red Flags, Integration.

## CSO (Claude Search Optimization)

| Rule | Rationale |
|------|-----------|
| Description = triggering conditions only | Workflow summaries cause Claude to shortcut the skill body |
| Embed symptom keywords in body | Error messages, tool names, command names, common synonyms |
| Front-load critical content | Claude may not read to end of long skills |
| Use "Use when..." description format | Matches Claude's skill-selection heuristic |
| Name with active voice, verb-first | `condition-based-waiting` not `async-test-helpers` |
| Gerunds (-ing) for processes | `writing-plans`, `requesting-code-review` |

## Word Count Targets

| Category | Target | Examples |
|----------|--------|----------|
| Frequently-loaded (every session) | <200 words | using-superpowers (~350w observed, aim lower) |
| Standard skills | <500 words | executing-plans (~250w), finishing-a-development-branch (~400w) |
| Complex with checklists/examples | Size as needed | writing-skills (~1800w), systematic-debugging (~900w) |

**Rule:** Front-load critical content regardless of total length.

## Cross-Skill References

| Pattern | Example | Use When |
|---------|---------|----------|
| Required sub-skill | `**REQUIRED SUB-SKILL:** Use superpowers:<name>` | Skill MUST be invoked |
| Required background | `**REQUIRED BACKGROUND:** You MUST understand superpowers:<name>` | Prerequisite knowledge |
| Called by / Pairs with | `**Called by:** superpowers:<name>` | Integration documentation |
| Inline mention | `Use the superpowers:<name> skill` | Soft reference |

**NEVER use `@` links** -- they force-load files, burning 200k+ context.

**NEVER use file paths** -- reference by skill name only.

## Supporting File Conventions

| Condition | Action |
|-----------|--------|
| All content <100 lines | Inline everything in SKILL.md |
| Reusable templates or prompts | Separate `.md` file (e.g., `code-reviewer.md`, `implementer-prompt.md`) |
| Large code examples or scripts | Separate file with appropriate extension |
| Heavy reference (500+ lines) | Separate file, referenced from SKILL.md |

**Observed supporting file types:**
- Subagent prompt templates (subagent-driven-development: 3 prompt files)
- Visual companion guides (brainstorming: visual-companion.md + scripts/)
- Testing scenarios (systematic-debugging: test-pressure-*.md)
- Anti-pattern references (test-driven-development: testing-anti-patterns.md)
- Tool references (using-superpowers: references/codex-tools.md)

## Flowchart Usage

Use graphviz `dot` notation ONLY for:
- Non-obvious decision points (when to use skill A vs B)
- Process loops where agent might stop too early
- Multi-branch workflows

NEVER use flowcharts for:
- Linear instructions (use numbered lists)
- Reference material (use tables)
- Code examples (use markdown blocks)

## Common Authoring Mistakes

| Mistake | Fix |
|---------|-----|
| Description summarizes workflow | Rewrite to triggering conditions only |
| Narrative storytelling | Convert to tables, checklists, structured data |
| No rationalization counters in discipline skill | Add `Common Rationalizations` table from baseline testing |
| Missing Red Flags section | Add bullet list of "thoughts that mean STOP" |
| Using `@` links to other skills | Replace with `**REQUIRED SUB-SKILL:** Use superpowers:<name>` |
| Multi-language code examples | One excellent example in most relevant language |
| Generic flowchart labels (step1, helper2) | Use semantic labels describing the action |
| Skipping "When NOT to Use" | Add explicit exclusion criteria |
| Deploying without baseline test | Run RED phase first -- no skill without failing test |

## Supporting Files

| File | Purpose |
|------|---------|
| baseline-test-guide.md | How to design and run baseline tests per skill type, with result template and real example |
