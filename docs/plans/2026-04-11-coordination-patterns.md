# Coordination Patterns Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `sft-coordination-patterns` reference skill to `skill-factory-toolkit` (v3.1.0 → v3.2.0), with matrix column + brainstorm pre-load integration, teaching the five multi-agent coordination patterns from the Anthropic article.

**Architecture:** One new reference-type SKILL.md (inline content, no supporting files), one additive column in the comparison matrix, two-line edit to sft-build-plugin's Phase Map and checklist, plus standard version bump machinery (plugin.json, CHANGELOG.md, CLAUDE.md Factory Toolkit table).

**Tech Stack:** Markdown, YAML frontmatter, the existing `bin/validate-plugin` bash script, git.

**Spec:** `docs/specs/2026-04-11-coordination-patterns-design.md`

**Note on baseline tests (Tasks 0 and 6):** The RED and GREEN phases are most rigorously executed by a human running a fresh Claude Code session with and without the skill loaded. A subagent executing these tasks inline can capture the expected RED failures from the template and verify GREEN by content review — checking that each documented RED failure has a corresponding skill section that addresses it. Both approaches are acceptable; pick one and be explicit in the result file.

---

## Task 0: Baseline Test (RED Phase)

**Files:**
- Create: `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/baseline-test-result.md` (temporary scratch file, deleted at end of Task 6)

**Why:** Per sft-build-plugin quality gate, no skill ships without a documented baseline test failure first. This captures the RED state — Claude failing to answer a coordination-pattern question without the new skill — so we can verify GREEN in Task 6.

- [ ] **Step 1: Create the baseline test prompt**

Create `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/baseline-test-result.md` with this content:

```markdown
# Baseline Test Result — sft-coordination-patterns

## Test Prompt

"I want to build a skill that takes a user's PR, runs several quality checks in parallel against it, and synthesizes a single review. Which multi-agent coordination pattern applies, and what structural markers should the SKILL.md have?"

## Expected RED (without sft-coordination-patterns)

- Claude may not name a specific coordination pattern at all
- Claude may pick an arbitrary pattern without justifying it against the information flow
- Claude may not connect the pattern to concrete structural markers (prompt templates, review loops, synthesis steps)
- Claude may not reference the "start simple / orchestrator-subagent default" principle

## RED Result (to be captured before GREEN phase)

<run the baseline test in a fresh Claude Code session without loading sft-coordination-patterns>
<paste response below>

...

## Failures Observed (mapped to skill sections)

| # | Failure | Skill Section That Fixes It |
|---|---------|----------------------------|
| 1 | ... | ... |

## Expected GREEN (with sft-coordination-patterns loaded)

- Claude names **orchestrator-subagent** (possibly layered with generator-verifier for synthesis)
- Claude cites "parallel bounded subtasks" as the when-to-use criterion
- Claude points at `superpowers:subagent-driven-development` as the exemplar
- Claude names specific structural markers: prompt templates as supporting files, synthesis step in orchestrator, review loop as optional layer
- Claude references the "start simple" design principle

## GREEN Result (to be captured after skill content complete)

...
```

- [ ] **Step 2: Run the baseline test**

Open a fresh Claude Code session (no skill loaded). Paste the test prompt. Record Claude's response in the RED Result section of `baseline-test-result.md`. Fill in the Failures Observed table by mapping each observed weakness to which skill section will address it.

Expected: Claude gives a generic answer without naming a specific pattern from the article, or names one without citing when-to-use criteria. Record whatever actually happens — this is the literal RED state.

- [ ] **Step 3: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-coordination-patterns/baseline-test-result.md
git commit -m "test(sft-coordination-patterns): baseline test RED result"
```

---

## Task 1: Scaffold `sft-coordination-patterns` Directory

**Files:**
- Create: `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md` (stub only)

**Why:** Create the directory and a minimal valid SKILL.md that the validator will accept, so Task 2's content additions are drop-in edits. Keeps scaffold and content in separate commits per `sft-scaffold-plugin` guidance.

- [ ] **Step 1: Create the stub SKILL.md**

Create `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md`:

```markdown
---
name: sft-coordination-patterns
description: Use when deciding how a new skill or plugin should coordinate work across agents — choosing between orchestrator, generator-verifier, agent teams, message bus, or shared state patterns.
---

# Coordination Patterns

Machine-readable reference for multi-agent coordination patterns. Use to pick the right pattern for a new skill.

## Overview

Stub — content added in next task.
```

- [ ] **Step 2: Validate the stub passes structural checks**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: Validator passes with zero errors. The stub has valid frontmatter, starts with "Use when", and has an Overview section.

If validation fails, fix the stub before continuing. Do NOT proceed to Task 2 until the validator is green.

- [ ] **Step 3: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md
git commit -m "feat(sft-coordination-patterns): scaffold skill directory"
```

---

## Task 2: Write `sft-coordination-patterns` Content

**Files:**
- Modify: `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md` (replace stub with full content)

**Why:** This is the primary deliverable — the reference skill itself. Content is inline (no supporting files), tables-primary, follows the reference-skill pattern modeled after `sft-skill-comparison-matrix` and `sft-cross-cutting-patterns`.

- [ ] **Step 1: Replace the stub with full reference content**

Replace the entire contents of `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md` with:

````markdown
---
name: sft-coordination-patterns
description: Use when deciding how a new skill or plugin should coordinate work across agents — choosing between orchestrator, generator-verifier, agent teams, message bus, or shared state patterns.
---

# Coordination Patterns

Machine-readable reference for the five multi-agent coordination patterns. Use to pick the right pattern when authoring a new skill, and to recognize which pattern an existing skill embodies. Default to orchestrator-subagent when in doubt.

## Pattern Reference Table

| Pattern | Purpose | When to Use | Key Elements | Anti-Patterns | Exemplar |
|---------|---------|-------------|--------------|---------------|----------|
| Generator-Verifier | Ensure quality via iterative refinement | Quality-critical output with explicit evaluation criteria | Generator produces; verifier evaluates; feedback loop revises until accepted or max iterations | Vague verification rubber-stamps results; evaluation treated as separable when it's as hard as generation; oscillation without fallback | `superpowers:requesting-code-review`; spec/plan review loops in `brainstorming` and `writing-plans` |
| Orchestrator-Subagent | Hierarchical task decomposition with centralized coordination | Clear decomposition, bounded subtasks, minimal interdependence | Lead agent plans, delegates to isolated specialized subagents, synthesizes results | Orchestrator bottlenecks information flow; sequential execution limits parallelism; critical details lost in handoffs | `superpowers:subagent-driven-development`; `skill-factory-toolkit:sft-build-plugin` |
| Agent Teams | Parallel persistent autonomous workers on independent workstreams | Parallel workload, independent long-running subtasks, minimal cross-worker coordination | Workers claim tasks from a queue, retain state across assignments, minimal inter-agent communication | Undetected conflicts on shared resources; difficulty detecting completion; inadequate task partitioning | `superpowers:dispatching-parallel-agents` |
| Message Bus | Event-driven coordination with extensible agent discovery | Event-driven pipelines, growing agent ecosystems | Agents publish/subscribe to topics; central router matches events to capable agents; loose coupling | Silent failures from misclassified events; hard to debug cascading chains; router as single point of failure | Not commonly seen in current superpowers corpus |
| Shared State | Decentralized coordination through persistent collaborative stores | Collaborative research, agents share discoveries, no single point of failure required | Agents read/write directly to persistent store; no central coordinator; explicit termination conditions | Unreduced duplicate work; reactive loops consuming tokens without convergence; absent termination causing indefinite cycling | `docs/specs/`, `docs/plans/`, memory files, TodoWrite state |

## Pattern Details

### Generator-Verifier

The generator produces an artifact; the verifier evaluates it against explicit criteria; the pair iterates until acceptance or a max-iteration fallback. The verifier may be another agent, a test suite, or a human reviewer. When authoring a skill that embodies this pattern, include an explicit review loop with stopping conditions and a fallback path for unresolved oscillation.

**Skill-authoring implications:** Surface the verification criteria in the skill body, not just the prompt. Separate generation and verification into distinct steps so the agent can't conflate them.

### Orchestrator-Subagent

A lead agent decomposes work, delegates bounded subtasks to specialized subagents running in isolated contexts, and synthesizes their results. This is the default pattern and the one the Anthropic article recommends starting with.

**Skill-authoring implications:** Include prompt templates for subagents as supporting files so the orchestrator can dispatch them reliably. Name the synthesis step explicitly — it's where hierarchical decomposition delivers value or fails.

### Agent Teams

Multiple persistent workers claim tasks from a queue and operate independently across long-running workstreams. Each worker retains state across assignments. Use when work partitions cleanly and inter-worker coordination is minimal.

**Skill-authoring implications:** Design task partitioning rules upfront — agent teams fail when the partition leaks shared state. Document completion detection; variable timelines make "are we done" a non-trivial question.

### Message Bus

Agents publish events to topics; a router matches events to capable subscriber agents. Loose coupling allows independent deployment and a growing agent ecosystem. Well-suited to security operations, alert routing, or any domain with evolving categories of work.

**Skill-authoring implications:** Define topic schemas explicitly. Silent misclassification is the most common failure — pair the skill with monitoring or a dead-letter topic. Not commonly needed for single-skill authoring; more applicable at the plugin-coordination level.

### Shared State

Agents read and write directly to a persistent store (files, databases, memory). No central coordinator mediates. Convergence is managed through explicit termination conditions like time budgets or quality thresholds.

**Skill-authoring implications:** State the termination condition in the skill body. Shared-state skills that loop without termination burn tokens indefinitely. Document which files or paths are the shared state so other skills can read them.

## Design Principles

| # | Principle | Meaning |
|---|-----------|---------|
| 1 | Start simple | Begin with orchestrator-subagent; evolve to other patterns only when specific constraints force it |
| 2 | Context-centric decomposition | Divide work by context requirements, not by task types or team boundaries |
| 3 | Explicit criteria | Verification, routing, and completion conditions must be stated upfront, not emerge from behavior |
| 4 | Match pattern to information flow | Choose based on workflow predictability, agent interdependence, and whether discoveries must propagate in real time |
| 5 | Plan termination | Shared-state and message-bus systems need explicit stopping conditions to prevent resource waste |

## Worked Example: Skill Factory Toolkit

The `skill-factory-toolkit` plugin itself is an instance of three layered patterns:

- **Orchestrator-Subagent:** `sft-build-plugin` orchestrates the build workflow by delegating to phase-specific sft skills. During the brainstorm phase it pre-loads `sft-skill-comparison-matrix`, `sft-skill-anatomy`, and `sft-coordination-patterns`. During the plan phase it pre-loads `sft-cross-cutting-patterns` and `sft-scaffold-plugin`. During the version phase it pre-loads `sft-versioning-guide`. Each pre-loaded skill operates in a bounded context and returns results to the orchestrator's checklist.

- **Shared State:** `docs/specs/` and `docs/plans/` are persistent state shared across build phases and potentially across sessions. The spec file written during brainstorm is read during planning; the plan file written during planning is read during implementation. No central coordinator mediates — the files themselves are the coordination medium.

- **Generator-Verifier:** The `superpowers:brainstorming` skill's spec review loop (spec self-review followed by user review gate) and the `superpowers:writing-plans` skill's plan review loop are generator-verifier patterns. The user is the verifier; the spec or plan is the artifact; the feedback loop iterates until approval.

Recognizing these patterns in the toolkit helps distinguish a structural change (touches the orchestrator) from a content change (touches a single specialized sub-skill) from a coordination change (touches shared state or review loops).

## Pattern-to-Structure Cheat Sheet

When a new skill embodies a pattern, its SKILL.md likely needs these structural markers:

| Pattern | Likely Structural Markers |
|---------|---------------------------|
| Generator-Verifier | Review loop section, stopping conditions, fallback for unresolved oscillation, explicit criteria table |
| Orchestrator-Subagent | Phase map table, subagent prompt templates as supporting files, synthesis step, dispatch checklist |
| Agent Teams | Task queue definition, partition rules, completion detection, worker state documentation |
| Message Bus | Topic schema, subscriber registration, dead-letter handling, event-routing diagram |
| Shared State | Shared file or path documentation, termination condition, read/write contract, convergence check |

Use these markers as a starting point, not a rigid requirement. A skill may exhibit a pattern without every marker.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Picking a pattern before understanding the information flow | Map inputs, outputs, and agent interdependence first; then choose |
| Treating the pattern label as decorative | Patterns imply structural markers; use the cheat sheet to enforce them |
| Defaulting to agent teams for anything parallel | Agent teams require independent workstreams; most "parallel" work is actually orchestrator-subagent with bounded subtasks |
| Using shared state without a termination condition | Loops without stopping criteria are the most common shared-state failure |
| Forcing message bus for a small workflow | Message bus pays off with many agents and evolving topics; overkill for a single skill |
| Inventing a pattern not in the reference | If your coordination need doesn't match any of the five, you probably need an orchestrator-subagent with a custom synthesis step, not a new pattern |
````

- [ ] **Step 2: Check word count**

Run: `wc -w plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md`

Expected: Between 1100 and 1400 words. If above 1400, tighten prose in Pattern Details; if below 1100, that's fine for a reference skill (brevity over padding).

- [ ] **Step 3: Validate**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: Passes with zero errors. Specifically:
- Frontmatter valid (starts with "Use when")
- No `@` links
- No file path references (`../`, `/plugins/`, `/skills/`, `/references/`)
- All referenced files exist (should be none — all content inline)

If the validator flags file path references, check for stray paths in the Worked Example section. The allowed path-like references are `docs/specs/` and `docs/plans/` which are content references, not skill-asset references — verify the validator's detection rules against existing skills that mention these same paths (e.g., `sft-build-plugin`'s own body text) before concluding it's a real violation.

- [ ] **Step 4: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md
git commit -m "feat(sft-coordination-patterns): add full reference content"
```

---

## Task 3: Add "Coordination Pattern" Column to `sft-skill-comparison-matrix`

**Files:**
- Modify: `plugins/skill-factory-toolkit/skills/sft-skill-comparison-matrix/SKILL.md`

**Why:** Make the new reference skill discoverable from the primary comparison table. Readers looking up skills by structural dimension will now see which coordination pattern each skill embodies, and a pointer to `sft-coordination-patterns` for definitions.

- [ ] **Step 1: Update the Primary Comparison Table header**

Find the existing table header (around line 13):

```markdown
| Skill | Type | Flow | Enforcement | Supporting Files | Word Count | Dispatches To | CSO Strategy |
|-------|------|------|-------------|------------------|------------|---------------|--------------|
```

Replace with:

```markdown
| Skill | Type | Flow | Enforcement | Supporting Files | Word Count | Dispatches To | CSO Strategy | Coordination Pattern |
|-------|------|------|-------------|------------------|------------|---------------|--------------|----------------------|
```

- [ ] **Step 2: Update each data row to include the new column**

Add the coordination pattern to each of the 14 rows. The column values are:

| Skill | Column Value |
|-------|--------------|
| brainstorming | generator-verifier (with user) |
| dispatching-parallel-agents | agent teams |
| executing-plans | orchestrator-subagent |
| finishing-a-development-branch | none (utility) |
| receiving-code-review | generator-verifier (as generator) |
| requesting-code-review | generator-verifier (as verifier) |
| subagent-driven-development | orchestrator-subagent |
| systematic-debugging | none (discipline) |
| test-driven-development | generator-verifier (test/code) |
| using-git-worktrees | none (utility) |
| using-superpowers | none (meta) |
| verification-before-completion | none (discipline) |
| writing-plans | generator-verifier (with user) |
| writing-skills | none (meta) |

Edit each row in the table to append the new column value. For example:

Before:
```markdown
| brainstorming | technique | checklist | flexible | 7 (visual companion, scripts, reviewer prompt) | ~1550 | writing-plans | symptoms |
```

After:
```markdown
| brainstorming | technique | checklist | flexible | 7 (visual companion, scripts, reviewer prompt) | ~1550 | writing-plans | symptoms | generator-verifier (with user) |
```

- [ ] **Step 3: Update the Column Definitions table**

Find the "Column Definitions" table and append this row at the end:

```markdown
| Coordination Pattern | generator-verifier / orchestrator-subagent / agent teams / message bus / shared state / none | Multi-agent coordination pattern the skill embodies (see `sft-coordination-patterns`) |
```

- [ ] **Step 4: Add a short cross-reference subsection**

After the Column Definitions section (and before the "Decision Guide: Choosing a Structural Template" section), add:

```markdown
## Coordination Pattern

See `sft-coordination-patterns` for pattern definitions, design principles, and the worked example of how this toolkit itself embodies layered patterns.
```

- [ ] **Step 5: Update the overview sentence if needed**

Find the opening sentence (around line 8): "Machine-readable reference comparing all 14 superpowers skills (v5.0.6) by structure, enforcement, and design choices."

No change needed — "structure, enforcement, and design choices" already covers coordination pattern implicitly.

- [ ] **Step 6: Validate**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: Passes with zero errors.

- [ ] **Step 7: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-skill-comparison-matrix/SKILL.md
git commit -m "feat(sft-skill-comparison-matrix): add coordination pattern column"
```

---

## Task 4: Add Soft Nudge to `sft-build-plugin`

**Files:**
- Modify: `plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md`

**Why:** Ensure `sft-coordination-patterns` is pre-loaded during the brainstorm phase of any new skill build, so authors encounter the reference naturally when shaping a new skill's structure.

- [ ] **Step 1: Update the Phase Map table**

Find the Phase Map table row for Brainstorm (around line 25):

Before:
```markdown
| **Brainstorm** | `sft-skill-comparison-matrix`, `sft-skill-anatomy` | `superpowers:brainstorming` |
```

After:
```markdown
| **Brainstorm** | `sft-skill-comparison-matrix`, `sft-skill-anatomy`, `sft-coordination-patterns` | `superpowers:brainstorming` |
```

- [ ] **Step 2: Update checklist step 1**

Find checklist step 1 (around line 34):

Before:
```markdown
1. **Pre-load brainstorm context** — invoke `skill-factory-toolkit:sft-skill-comparison-matrix` and `skill-factory-toolkit:sft-skill-anatomy`
```

After:
```markdown
1. **Pre-load brainstorm context** — invoke `skill-factory-toolkit:sft-skill-comparison-matrix`, `skill-factory-toolkit:sft-skill-anatomy`, and `skill-factory-toolkit:sft-coordination-patterns`
```

- [ ] **Step 3: Verify no other changes needed**

Scan the rest of `sft-build-plugin/SKILL.md` for any references to the brainstorm pre-load list. Expected: only the two locations above. Quality Gate, Common Mistakes, and Refinement Workflow sections should be untouched.

- [ ] **Step 4: Validate**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: Passes with zero errors.

- [ ] **Step 5: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-build-plugin/SKILL.md
git commit -m "feat(sft-build-plugin): pre-load sft-coordination-patterns during brainstorm"
```

---

## Task 5: Version Bump + CHANGELOG + Root CLAUDE.md

**Files:**
- Modify: `plugins/skill-factory-toolkit/.claude-plugin/plugin.json`
- Modify: `plugins/skill-factory-toolkit/CHANGELOG.md`
- Modify: `CLAUDE.md` (repo root)

**Why:** Per `sft-versioning-guide`, adding a new skill to an existing plugin is a minor bump. All three files must be kept in sync: plugin.json holds the canonical version, CHANGELOG.md documents the change, CLAUDE.md's Factory Toolkit table lists user-facing skills.

- [ ] **Step 1: Bump plugin.json version**

Edit `plugins/skill-factory-toolkit/.claude-plugin/plugin.json`:

Before:
```json
{
  "name": "skill-factory-toolkit",
  "description": "Authoring skills, pattern references, and governance guides for the skill factory",
  "version": "3.1.0"
}
```

After:
```json
{
  "name": "skill-factory-toolkit",
  "description": "Authoring skills, pattern references, and governance guides for the skill factory",
  "version": "3.2.0"
}
```

- [ ] **Step 2: Add CHANGELOG entry**

Edit `plugins/skill-factory-toolkit/CHANGELOG.md`. Insert this entry directly below the `# Changelog` header and above the existing `## [3.1.0]` entry:

```markdown
## [3.2.0] - 2026-04-11

### Added

- `sft-coordination-patterns` reference skill describing the five multi-agent coordination patterns (generator-verifier, orchestrator-subagent, agent teams, message bus, shared state), five design principles, pattern-to-structure cheat sheet, and a worked example showing how the skill-factory-toolkit itself exemplifies layered orchestrator-subagent + shared state + generator-verifier patterns
- "Coordination Pattern" column in `sft-skill-comparison-matrix` primary comparison table
- `sft-coordination-patterns` pre-loaded during the Brainstorm phase of `sft-build-plugin`
```

- [ ] **Step 3: Update repo-root CLAUDE.md Factory Toolkit table**

Edit `CLAUDE.md` at the repo root. Find the Factory Toolkit table under the "## Factory Toolkit" section. Append a new row:

Before (end of table):
```markdown
| `sft-versioning-guide` | Use when bumping versions or creating changelog entries |
```

After (end of table):
```markdown
| `sft-versioning-guide` | Use when bumping versions or creating changelog entries |
| `sft-coordination-patterns` | Use when deciding a skill's multi-agent coordination pattern |
```

- [ ] **Step 4: Verify version consistency**

Run: `grep -E '(version|\[3\.)' plugins/skill-factory-toolkit/.claude-plugin/plugin.json plugins/skill-factory-toolkit/CHANGELOG.md | head`

Expected: `plugin.json` shows `"version": "3.2.0"`, CHANGELOG.md shows `## [3.2.0]` as the top entry. These must match.

- [ ] **Step 5: Validate**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: Passes with zero errors. The validator checks that plugin.json's version matches the latest CHANGELOG.md entry.

- [ ] **Step 6: Commit**

```bash
git add plugins/skill-factory-toolkit/.claude-plugin/plugin.json plugins/skill-factory-toolkit/CHANGELOG.md CLAUDE.md
git commit -m "chore(skill-factory-toolkit): bump version to 3.2.0"
```

---

## Task 6: Baseline Test (GREEN Phase) + Cleanup

**Files:**
- Modify: `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/baseline-test-result.md` (fill in GREEN result)
- Delete (optional): `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/baseline-test-result.md` if you don't want it shipped

**Why:** Close the TDD loop by verifying that with the skill loaded, Claude now answers the baseline test correctly. This is the quality gate item: "Baseline test is documented (failures recorded, mapped to skill sections)."

- [ ] **Step 1: Run the GREEN baseline test**

Open a fresh Claude Code session. Load the new `sft-coordination-patterns` skill (or paste its contents into the context). Ask the same baseline test prompt:

> "I want to build a skill that takes a user's PR, runs several quality checks in parallel against it, and synthesizes a single review. Which multi-agent coordination pattern applies, and what structural markers should the SKILL.md have?"

Record Claude's response in the "GREEN Result" section of `baseline-test-result.md`.

- [ ] **Step 2: Verify GREEN expectations**

Check that Claude's response includes all of:
- Names **orchestrator-subagent** (possibly layered with generator-verifier for synthesis)
- Cites "parallel bounded subtasks" (or equivalent) as the when-to-use criterion
- Points at `superpowers:subagent-driven-development` as an exemplar
- Names structural markers: prompt templates, synthesis step, optional review loop
- References the "start simple" principle

If any expectation fails, return to Task 2 and strengthen the skill content in the relevant section before re-running the GREEN test. Do NOT proceed until GREEN passes.

- [ ] **Step 3: Decide on baseline-test-result.md disposition**

Either:
- **Keep:** Leave `baseline-test-result.md` in the skill directory as historical documentation. Valid — other skills in the repo don't ship their baseline tests, but it's not forbidden.
- **Delete:** Remove the file — the changelog entry documents that the RED/GREEN cycle happened.

Recommended: Delete, to match existing skill directories which don't carry baseline test artifacts. If deleted, also check that no SKILL.md body references the file (none should).

- [ ] **Step 4: Final validation**

Run: `bin/validate-plugin plugins/skill-factory-toolkit`

Expected: Passes with zero errors.

Run: `wc -w plugins/skill-factory-toolkit/skills/sft-coordination-patterns/SKILL.md`

Expected: 1100-1400 words.

- [ ] **Step 5: Quality gate checklist**

Before committing the final state, verify each item from `sft-build-plugin`'s quality gate:

- [ ] Description starts with "Use when" and contains triggering conditions only (no workflow summary)
- [ ] Skill has all reference-type sections (tables primary, minimal prose, no enforcement language)
- [ ] All `.md` files referenced in SKILL.md body exist in the skill directory (none referenced — all content inline)
- [ ] No `@` link references in SKILL.md
- [ ] No file path references (`../`, `/plugins/`, `/skills/`, `/references/`) in SKILL.md body (except the allowed mentions of `docs/specs/` and `docs/plans/` in the Worked Example — these are content references, same as used in `sft-build-plugin`'s own body)
- [ ] Word count falls in 1100-1400 range
- [ ] `bin/validate-plugin plugins/skill-factory-toolkit` passes with zero errors
- [ ] Baseline test RED and GREEN documented

If any item fails, fix it before committing.

- [ ] **Step 6: Commit**

```bash
git add plugins/skill-factory-toolkit/skills/sft-coordination-patterns/
git commit -m "test(sft-coordination-patterns): verify GREEN baseline + cleanup"
```

---

## Final State

After Task 6, the repo should have:

- A new skill directory `plugins/skill-factory-toolkit/skills/sft-coordination-patterns/` containing only `SKILL.md` (baseline test file deleted)
- An updated `sft-skill-comparison-matrix` with a new column
- An updated `sft-build-plugin` pre-loading the new skill during brainstorm
- `plugin.json` at version `3.2.0`
- A new `[3.2.0]` CHANGELOG entry
- An updated root `CLAUDE.md` Factory Toolkit table
- Six commits, each semantic and reversible

Run a final sanity check:

```bash
git log --oneline -10
bin/validate-plugin plugins/skill-factory-toolkit
```

Expected: Six recent commits related to coordination patterns; validator passes.
