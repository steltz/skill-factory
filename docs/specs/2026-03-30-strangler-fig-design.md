# Strangler Fig Plugin Design

## Overview

Technology-agnostic skills guiding a solo developer + Claude through incremental legacy system modernization using the Strangler Fig pattern. Five skills, one per phase, with a shared migration tracking manifest as connective tissue.

## Plugin Structure

```
plugins/strangler-fig/
├── .claude-plugin/
│   └── plugin.json          # name: "strangler-fig", version: "1.0.0"
├── skills/
│   ├── assess/
│   │   └── SKILL.md
│   ├── seed/
│   │   └── SKILL.md
│   ├── growth/
│   │   └── SKILL.md
│   ├── strangulate/
│   │   └── SKILL.md
│   └── decommission/
│       └── SKILL.md
└── CHANGELOG.md
```

No hooks directory. No supporting files initially — inline everything unless TDD reveals content that exceeds 100 lines or needs to be reusable.

## Skills

### assess (flexible)

**Trigger:** Use when evaluating whether a legacy system is a candidate for incremental migration

**Responsible for:**
- Inventorying the system's external surface area (endpoints, interfaces, protocols)
- Mapping internal dependencies (what calls what)
- Evaluating Strangler Fig fit vs. alternatives (rewrite if small enough, wrap if read-only, leave if stable)

**Output:** A migration candidate assessment with recommended approach and rough surface area count. Seeds the migration tracking manifest with the initial inventory.

**Not responsible for:** Setting up any infrastructure.

### seed (rigid)

**Trigger:** Use when setting up the initial proxy layer in front of a legacy system

**Responsible for:**
- Standing up a proxy/gateway in front of the legacy system
- Configuring passthrough routing (all traffic flows through proxy to legacy, unchanged)
- Adding health checks against the legacy backend
- Verifying end-to-end: requests through the proxy produce identical responses to direct legacy access

**Output:** A working proxy that is transparent to consumers.

**Not responsible for:** Migrating any functionality.

**Cross-reference:** If you haven't assessed the legacy system yet, see `strangler-fig:assess`.

### growth (rigid)

**Trigger:** Use when migrating a specific endpoint or module from a legacy system to its modern replacement

**Responsible for:**
- Analyzing the legacy implementation being migrated (inputs, outputs, side effects)
- Building the modern replacement with tests proving parity
- Updating proxy routing: this endpoint routes to new implementation, everything else routes to legacy (unchanged)
- Verifying: new endpoint produces identical results, legacy routes still work
- Updating migration tracking manifest (what's migrated, what remains)

**Output:** One migrated endpoint/module with verified parity, updated routing, and updated tracking.

**Not responsible for:** Deciding what to migrate next (that's the developer's call). Removing legacy code (that's `strangulate`).

**Cross-reference:** If no proxy layer exists, see `strangler-fig:seed`.

### strangulate (rigid)

**Trigger:** Use when evaluating whether a legacy endpoint or module can be safely removed

**Responsible for:**
- Checking migration tracking: is this endpoint fully replaced?
- Analyzing traffic: is anything still hitting the legacy route?
- Checking dependencies: does any other legacy code call this internally?
- If safe: removing the legacy route, updating proxy config, updating tracking
- If not safe: documenting what's blocking removal

**Output:** Either a safely removed legacy route or a documented blocker.

**Not responsible for:** Building replacements (that's `growth`).

**Cross-reference:** If the endpoint hasn't been migrated yet, see `strangler-fig:growth`.

### decommission (rigid)

**Trigger:** Use when the legacy system handles no remaining traffic and is ready for shutdown

**Responsible for:**
- Verifying migration tracking shows 100% coverage
- Verifying zero traffic to legacy system over a defined observation period
- Removing proxy fallback configuration (proxy now routes everything to modern system)
- Archiving legacy source code
- Tearing down legacy infrastructure

**Output:** Legacy system fully decommissioned, modern stack standing independently.

**Not responsible for:** Anything where legacy routes are still active.

**Cross-reference:** If legacy routes are still active, see `strangler-fig:strangulate`.

## Cross-Skill Coordination

### Migration tracking manifest

A simple tracking file listing migrated vs. remaining endpoints. This is the connective tissue between skills:

- `assess` produces the initial inventory that seeds it
- `growth` updates it each time an endpoint is migrated
- `strangulate` reads it to verify an endpoint is replaced before removal
- `decommission` reads it to verify 100% coverage

Format is defined inline in the skill that creates it. Other skills reference the same format.

### Cross-references, not dependencies

Skills hint at each other but don't hard-gate. A developer who already has a proxy set up shouldn't be blocked from using `growth` just because they didn't use `seed`.

## Testing Strategy

Each skill gets the TDD cycle (RED/GREEN/REFACTOR). Baseline scenarios (run without the skill):

- **assess:** "I have a legacy system I want to modernize" — watch for jumping to implementation without inventory, not considering alternatives, not mapping surface area.
- **seed:** "Set up a proxy in front of my legacy app" — watch for rewriting logic instead of proxying, skipping health checks, not verifying passthrough parity.
- **growth:** "Migrate this endpoint from the legacy system to the new one" — watch for not analyzing legacy first, no parity tests, forgetting to update routing, not tracking migration.
- **strangulate:** "Can we remove this old endpoint?" — watch for removing without checking traffic, not checking internal dependencies, no rollback plan.
- **decommission:** "The legacy system isn't used anymore, shut it down" — watch for not verifying zero traffic over time, not archiving source, wrong teardown order.

## Decisions

- **Technology-agnostic:** Skills teach the pattern, not specific framework recipes.
- **Solo dev + Claude:** No team coordination workflows.
- **Four rigid, one flexible:** Migration mistakes are expensive. Only `assess` is flexible because every legacy system is different.
- **No hooks:** Skills guide decisions, they don't need session-level automation.
