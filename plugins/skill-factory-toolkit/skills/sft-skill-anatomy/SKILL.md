---
name: sft-skill-anatomy
description: Use when studying how superpowers skills are structured to understand why each section exists
---

# Skill Anatomy

Structural breakdown of three representative superpowers skills. Maps every section to the writing-skills authoring guide and explains the design rationale behind each structural choice.

## How to Use This Reference

Look up the skill type closest to what you are building, then use the section map and removal-impact analysis to decide which sections your skill needs.

## Skill 1: brainstorming (Complex Flexible Skill)

**Type:** Technique
**Files:** SKILL.md, visual-companion.md, spec-document-reviewer-prompt.md, scripts/ (5 files)
**Word count:** ~1600w (SKILL.md alone)

### Section Map

| Section | Writing-Skills Mapping | Purpose | Removal Impact |
|---------|----------------------|---------|----------------|
| YAML frontmatter | Frontmatter (YAML) | CSO trigger with aggressive MUST language: forces loading before any creative work | Claude skips skill for design tasks, proceeds straight to implementation |
| `# Brainstorming Ideas Into Designs` | Overview | Core principle in 2 sentences: understand context, ask questions, present design | Agent has no mental model of the process boundary |
| `<HARD-GATE>` | Red Flags / Iron Law equivalent | Prevents ANY implementation before design approval | Agent implements prematurely -- the single highest-risk failure mode |
| Anti-Pattern: "Too Simple" | Common Rationalizations (inline) | Blocks the most frequent escape hatch | Agent skips brainstorming for "simple" projects, which are where unexamined assumptions cause the most waste |
| Checklist (9 items) | Process / Checklist | Ordered task list with explicit TodoWrite tracking | Agent freestyles the process, skips steps, or reorders in ways that lose rigor (e.g., writing spec before getting approval) |
| Process Flow (dot) | Flowchart Usage | State machine showing decisions, loops, and terminal state | Agent misses revision loops or exits to wrong skill |
| The Process (detailed) | Implementation | Expanded guidance per phase: understanding, approaches, presenting, isolation, codebases | Agent lacks the judgment heuristics for each phase (e.g., when to decompose scope) |
| After the Design | Implementation (post) | Documentation path, spec self-review checklist, user review gate | Specs ship with TODOs, contradictions, or without user sign-off |
| Key Principles | Quick Reference | 6 scannable bullets for fast recall | No quick-scan option; agent must re-read full skill each time |
| Visual Companion | Separate supporting section | Browser-based mockup tool with consent gate and per-question decision rules | Agent either never uses visuals or uses them for everything, wasting tokens |

### Why This Structure Works

Brainstorming is high-risk for scope creep and premature action. The structure uses a layered defense:

1. **HARD-GATE** -- absolute prohibition on implementation
2. **Anti-Pattern block** -- closes the "too simple" escape hatch
3. **Numbered checklist** -- forces ordered progression with task tracking
4. **Process flow diagram** -- shows loops and terminal state, preventing wrong-exit
5. **User review gate** -- ensures human sign-off before transition

Flexible conversation adapts within each step, but the step sequence is rigid.

### Supporting Files and Roles

| File | Role | Writing-Skills Rule |
|------|------|-------------------|
| visual-companion.md | Detailed guide for browser-based mockups (~400 lines) | Heavy reference (100+ lines) goes in separate file |
| spec-document-reviewer-prompt.md | Subagent dispatch template for spec review | Reusable template goes in separate file |
| scripts/server.cjs | Local HTTP server for visual companion | Reusable tool goes in separate file |
| scripts/start-server.sh | Server lifecycle management | Reusable tool |
| scripts/stop-server.sh | Server lifecycle management | Reusable tool |
| scripts/frame-template.html | HTML template for visual frames | Reusable tool |
| scripts/helper.js | Client-side utilities for companion | Reusable tool |

**Design rationale:** The visual companion is a full subsystem (server + templates + scripts). Inlining it would bloat SKILL.md beyond usability. Separation keeps SKILL.md focused on process while companion details load only when needed.

## Skill 2: test-driven-development (Rigid Discipline Skill)

**Type:** Discipline-Enforcing
**Files:** SKILL.md, testing-anti-patterns.md
**Word count:** ~1100w (SKILL.md alone)

### Section Map

| Section | Writing-Skills Mapping | Purpose | Removal Impact |
|---------|----------------------|---------|----------------|
| YAML frontmatter | Frontmatter (YAML) | Concise trigger: "before writing implementation code" -- no workflow summary | With workflow in description, Claude follows description shortcut and skips the discipline checks |
| Overview (3 lines) | Overview | Core principle + "Violating the letter...is violating the spirit" | Agents use "spirit vs letter" rationalization to bypass rules |
| When to Use | When to Use | Always/Exceptions split with rationalization callout | Agent debates applicability instead of defaulting to TDD |
| The Iron Law | Iron Law pattern | `NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST` in code block | The core constraint disappears; entire skill loses its anchor |
| Red-Green-Refactor (dot) | Flowchart Usage | State machine with verify-fail loops at RED and GREEN | Agent skips verification steps or does not loop back on wrong failure |
| RED section | Implementation | Write one minimal test + requirements (one behavior, clear name, real code) | Agent writes vague or multi-behavior tests |
| Verify RED | Implementation | MANDATORY verification with 3-point confirm checklist | Agent never watches test fail, so never knows if test tests the right thing |
| GREEN section | Implementation | Minimal code with Good/Bad comparison | Agent over-engineers beyond what test requires |
| Verify GREEN | Implementation | MANDATORY verification + "fix code not test" | Agent tweaks test to match wrong behavior |
| REFACTOR section | Implementation | Constrained to cleanup only -- no behavior additions | Agent adds features during refactor phase |
| Good Tests table | Quick Reference | 3-row quality checklist: Minimal, Clear, Shows intent | Agent writes poor tests that pass for wrong reasons |
| Why Order Matters | Common Rationalizations (extended) | 5 detailed rebuttals to "tests after" arguments | Each removed rebuttal reopens a rationalization vector |
| Common Rationalizations table | Common Rationalizations | 11-row Excuse/Reality table | Agent rationalizes skipping TDD with uncountered excuses |
| Red Flags list | Red Flags | 13 specific thoughts that mean "STOP and start over" | Agent lacks self-check mechanism for rationalization |
| Bug Fix example | Real-World Example | Complete RED-verify-GREEN-verify-REFACTOR walkthrough | Agent has no concrete model of the cycle applied to a real scenario |
| Verification Checklist | Verification Checklist | 8 checkbox items before marking work complete | Agent declares completion without verification |
| When Stuck table | Common Mistakes | 4-row Problem/Solution for testing difficulties | Agent abandons TDD when encountering friction |
| Testing Anti-Patterns reference | Cross-reference to supporting file | Pointer to testing-anti-patterns.md | Agent repeats common mock/test-utility mistakes |

### Why This Structure Works

TDD is a discipline that engineers resist. The skill's entire architecture is anti-rationalization:

1. **Iron Law** -- non-negotiable anchor; everything traces back to this
2. **Verify steps marked MANDATORY** -- prevents the most common shortcut (skipping verification)
3. **Good/Bad comparisons** -- shows exactly what minimal means (prevents gold-plating)
4. **Rationalization table** -- pre-empts every known excuse with a direct rebuttal
5. **Red Flags list** -- makes self-monitoring concrete: "if you are thinking X, stop"

Minimal word count is intentional. Discipline skills must load fast and hit hard. No theory, no history, no justification beyond what is needed to counter specific rationalizations.

### Supporting Files and Roles

| File | Role | Writing-Skills Rule |
|------|------|-------------------|
| testing-anti-patterns.md | Common mock/test-utility pitfalls (~250 lines) | Heavy reference (100+ lines) goes in separate file |

**Design rationale:** Anti-patterns are reference material consulted when adding mocks or test utilities. Inlining them would dilute the main skill's focus on the TDD cycle itself.

## Skill 3: writing-skills (Meta/Self-Referential Skill)

**Type:** Meta
**Files:** SKILL.md, anthropic-best-practices.md, persuasion-principles.md, testing-skills-with-subagents.md, render-graphs.js, graphviz-conventions.dot, examples/CLAUDE_MD_TESTING.md
**Word count:** ~1800w (SKILL.md alone)

### Section Map

| Section | Writing-Skills Mapping | Purpose | Removal Impact |
|---------|----------------------|---------|----------------|
| YAML frontmatter | Frontmatter (YAML) -- self-exemplifying | Demonstrates its own CSO rules: "Use when creating new skills, editing existing skills, or verifying skills work" | The meta-example of correct CSO disappears |
| Overview | Overview | "Writing skills IS TDD applied to process documentation" + core principle | Agent misunderstands the fundamental approach: treats skill writing as creative writing instead of TDD |
| TDD Mapping table | Core Pattern | 10-row mapping from TDD concepts to skill-creation equivalents | Agent cannot translate TDD discipline to documentation |
| When to Create | When to Use | Create-when / Don't-create-for split | Agent creates skills for one-off solutions or project-specific conventions |
| Skill Types | Quick Reference | Technique / Pattern / Reference taxonomy with examples | Agent conflates skill types and applies wrong structural patterns |
| Directory Structure | Implementation | Flat namespace rule + file separation criteria | Agent creates nested hierarchies or wrong file organization |
| SKILL.md Structure | Implementation | Template with all standard sections | Agent invents ad-hoc formats |
| CSO section | Implementation (extended) | 4 subsections: Rich Description, Keyword Coverage, Descriptive Naming, Token Efficiency | Agent writes undiscoverable skills -- correct content but Claude never loads them |
| CSO Description deep-dive | Common Mistakes (embedded) | "CRITICAL: Description = When to Use, NOT What the Skill Does" with failure evidence | Agent summarizes workflow in description, causing Claude to shortcut the skill body |
| Token Efficiency | Implementation | Word count targets + compression techniques + verification command | Skills waste context budget on every load |
| Cross-Referencing | Implementation | 4-pattern table: required sub-skill, required background, called-by, inline mention | Agent uses `@` links (force-loads files) or unclear references |
| Flowchart Usage | Implementation | Use-for / Never-use-for decision + dot flowchart | Agent uses flowcharts for linear instructions or reference material |
| Code Examples | Implementation | "One excellent example beats many mediocre ones" | Agent creates multi-language dilution |
| File Organization | Implementation | 3 archetypes: self-contained, with-tool, with-heavy-reference | Agent either inlines everything or splits everything |
| The Iron Law | Iron Law pattern | "NO SKILL WITHOUT A FAILING TEST FIRST" -- same as TDD | Agent deploys untested skills |
| Testing All Skill Types | Implementation (extended) | 4 type-specific test strategies: discipline, technique, pattern, reference | Agent uses wrong test approach for skill type |
| Common Rationalizations table | Common Rationalizations | 8-row table for skipping testing | Agent skips testing with uncountered excuses |
| Bulletproofing section | Implementation (extended) | 4 techniques: close loopholes explicitly, address spirit-vs-letter, build rationalization table, create red flags list | Agent writes discipline skills that agents can rationalize around |
| RED-GREEN-REFACTOR for Skills | Core Pattern | Complete TDD cycle adapted for documentation | Agent writes skills without baseline testing |
| Anti-Patterns | Anti-patterns | 4 named anti-patterns with why-bad explanation | Agent repeats known failure modes |
| STOP: Before Moving | Red Flags / Hard Gate | "After writing ANY skill, you MUST STOP and complete deployment" | Agent batch-creates skills without testing each |
| Skill Creation Checklist | Verification Checklist | RED/GREEN/REFACTOR phases + Quality Checks + Deployment | Agent has no systematic completion criteria |
| Discovery Workflow | Quick Reference | 5-step flow showing how future Claude finds skills | Agent does not optimize for discoverability |

### Why This Structure Works

Meta-skills must teach by example. Every convention this skill describes, it follows:

1. **Frontmatter** -- demonstrates correct CSO with "Use when..." trigger
2. **Iron Law** -- the same pattern it teaches discipline skills to use
3. **Rationalization table** -- present because the skill says discipline skills need one
4. **TDD cycle** -- the skill uses the exact RED-GREEN-REFACTOR framing it requires of all skills
5. **Token-efficient structure** -- tables over prose, matching its own token efficiency guidance

The skill is large (~1800w) because it must cover both the authoring process and the testing methodology. It mitigates this by front-loading critical content (CSO, structure template) and pushing heavy reference to supporting files.

### Supporting Files and Roles

| File | Role | Writing-Skills Rule |
|------|------|-------------------|
| anthropic-best-practices.md | Official Anthropic skill authoring guidance (~1500 lines) | Heavy reference goes in separate file |
| persuasion-principles.md | Research foundation for bulletproofing techniques | Heavy reference |
| testing-skills-with-subagents.md | Complete testing methodology for skill verification | Heavy reference |
| graphviz-conventions.dot | Style rules for dot flowcharts | Reusable tool/reference |
| render-graphs.js | Renders skill flowcharts to SVG | Reusable tool |
| examples/CLAUDE_MD_TESTING.md | Testing example for CLAUDE.md-style skills | Reusable tool |

**Design rationale:** The supporting files total ~3000+ lines. Inlining even one of them would double the skill's token cost on every load. Each file serves a distinct purpose consulted at specific points in the authoring workflow.

## Cross-Skill Comparison

| Dimension | brainstorming | test-driven-development | writing-skills |
|-----------|---------------|------------------------|----------------|
| Type | Technique | Discipline-Enforcing | Meta |
| Enforcement | Flexible within steps, rigid step order | Rigid everywhere | Rigid for rules, flexible for application |
| Word count | ~1600w | ~1100w | ~1800w |
| Supporting files | 7 | 1 | 6 |
| Has Iron Law | No (uses HARD-GATE instead) | Yes | Yes |
| Has rationalization table | No (single anti-pattern block) | Yes (11 rows) | Yes (8 rows) |
| Has Red Flags list | No | Yes (13 items) | No (uses STOP section) |
| Has flowchart | Yes (process flow) | Yes (cycle) | Yes (decision) |
| Has verification checklist | No (uses user review gate) | Yes (8 items) | Yes (full TDD checklist) |
| Defensive layers | 5 (gate, anti-pattern, checklist, flow, user gate) | 5 (law, verify steps, comparisons, table, flags) | 5 (example, law, table, TDD cycle, stop gate) |
| Primary risk | Premature implementation | Rationalization to skip TDD | Writing undiscoverable or untested skills |

## Structural Principles

Patterns that hold across all three skills:

| Principle | Evidence |
|-----------|----------|
| Defense in depth | Every skill has 5+ independent enforcement mechanisms |
| Front-load critical content | Hard gates and iron laws appear in the first 20% of each skill |
| Close specific loopholes, not general ones | Rationalization tables address verbatim excuses from baseline testing |
| Supporting files for heavy reference only | SKILL.md stays focused; large content loads on demand |
| Flowcharts only for non-obvious decisions | All three use flowcharts for state machines or decision trees, never for linear steps |
| Self-exemplification in meta skills | writing-skills follows every convention it teaches |
| Token budget awareness | Discipline skills stay minimal; technique skills scale sections to complexity |
