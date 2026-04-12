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
