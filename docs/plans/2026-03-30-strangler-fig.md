# Strangler Fig Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a `strangler-fig` plugin with five technology-agnostic skills guiding incremental legacy system modernization.

**Architecture:** Each skill maps to a migration phase (assess → seed → growth → strangulate → decommission). Skills are independently invocable but cross-reference each other as hints. A migration tracking manifest provides connective tissue between phases.

**Tech Stack:** Claude Code plugin system (SKILL.md, plugin.json, CHANGELOG.md). TDD via subagent pressure scenarios.

---

### Task 1: Scaffold the plugin

**Files:**
- Create: `plugins/strangler-fig/.claude-plugin/plugin.json`
- Create: `plugins/strangler-fig/skills/assess/SKILL.md` (placeholder)
- Create: `plugins/strangler-fig/skills/seed/SKILL.md` (placeholder)
- Create: `plugins/strangler-fig/skills/growth/SKILL.md` (placeholder)
- Create: `plugins/strangler-fig/skills/strangulate/SKILL.md` (placeholder)
- Create: `plugins/strangler-fig/skills/decommission/SKILL.md` (placeholder)
- Create: `plugins/strangler-fig/CHANGELOG.md`
- Modify: `.claude-plugin/marketplace.json` (add strangler-fig entry)

- [ ] **Step 1: Create plugin.json**

```json
{
  "name": "strangler-fig",
  "description": "Skills for incremental legacy system modernization using the Strangler Fig pattern",
  "version": "0.1.0"
}
```

- [ ] **Step 2: Create placeholder SKILL.md for each skill**

Each gets minimal valid frontmatter so the scaffold passes validation. Content will be written during TDD tasks.

`skills/assess/SKILL.md`:
```markdown
---
name: assess
description: Use when evaluating whether a legacy system is a candidate for incremental migration
---

# Assess

Placeholder — content will be written via TDD.
```

`skills/seed/SKILL.md`:
```markdown
---
name: seed
description: Use when setting up the initial proxy layer in front of a legacy system
---

# Seed

Placeholder — content will be written via TDD.
```

`skills/growth/SKILL.md`:
```markdown
---
name: growth
description: Use when migrating a specific endpoint or module from a legacy system to its modern replacement
---

# Growth

Placeholder — content will be written via TDD.
```

`skills/strangulate/SKILL.md`:
```markdown
---
name: strangulate
description: Use when evaluating whether a legacy endpoint or module can be safely removed
---

# Strangulate

Placeholder — content will be written via TDD.
```

`skills/decommission/SKILL.md`:
```markdown
---
name: decommission
description: Use when the legacy system handles no remaining traffic and is ready for shutdown
---

# Decommission

Placeholder — content will be written via TDD.
```

- [ ] **Step 3: Create CHANGELOG.md**

```markdown
# Changelog

## [0.1.0] - 2026-03-30

### Added

- assess skill for evaluating legacy systems as migration candidates
- seed skill for setting up the proxy layer
- growth skill for migrating individual endpoints
- strangulate skill for safely removing legacy routes
- decommission skill for final legacy system shutdown
```

- [ ] **Step 4: Add strangler-fig to marketplace.json**

Add this entry to the `plugins` array in `.claude-plugin/marketplace.json`:

```json
{
  "name": "strangler-fig",
  "description": "Skills for incremental legacy system modernization using the Strangler Fig pattern",
  "version": "0.1.0",
  "source": "./plugins/strangler-fig",
  "category": "development"
}
```

- [ ] **Step 5: Validate the scaffold**

Run: `bin/validate-plugin plugins/strangler-fig`
Expected: All checks pass (valid plugin.json, valid semver, CHANGELOG entry for 0.1.0, all SKILL.md files have correct frontmatter, names are kebab-case, descriptions start with "Use when").

- [ ] **Step 6: Commit the scaffold**

```bash
git add plugins/strangler-fig .claude-plugin/marketplace.json
git commit -m "feat: scaffold strangler-fig plugin with five skill placeholders"
```

---

### Task 2: TDD the assess skill

**Files:**
- Modify: `plugins/strangler-fig/skills/assess/SKILL.md`

This is a **flexible technique** skill. Test approach: application scenarios (can Claude apply the assessment framework?) and recognition scenarios (does Claude know when NOT to apply Strangler Fig?).

- [ ] **Step 1: RED — Run baseline scenario WITHOUT the skill**

Dispatch a subagent with this prompt (no strangler-fig skills loaded):

```
You are helping a developer modernize a legacy system. The developer says:

"I have a 15-year-old system that handles order processing, inventory management, and customer notifications. It's written in a language that's hard to hire for. We need to modernize it but can't afford downtime. Where do I start?"

Help them plan the modernization approach.
```

Document the subagent's response. Watch for these expected failures:
- Jumps straight to suggesting technologies/frameworks without inventorying the system
- Does not map the system's external surface area (endpoints, interfaces, protocols)
- Does not map internal dependencies between modules
- Does not consider alternatives to Strangler Fig (rewrite, wrap, leave alone)
- Does not produce a structured assessment — gives freeform advice instead
- Does not estimate scope (surface area count)

Record exact rationalizations and gaps verbatim.

- [ ] **Step 2: GREEN — Write the assess SKILL.md**

Replace the placeholder content in `plugins/strangler-fig/skills/assess/SKILL.md` with the full skill. Address every specific failure observed in the baseline. The skill should guide Claude through:

1. **Surface area inventory** — enumerate every external-facing endpoint, interface, protocol, and integration point
2. **Dependency mapping** — which modules call which, what shares state, where are the coupling points
3. **Pattern fit evaluation** — decision framework for Strangler Fig vs. alternatives:
   - System too small? → Consider full rewrite
   - Read-only or stable? → Consider wrapping with an adapter
   - No external consumers? → Consider internal refactor
   - Large, actively used, can't afford downtime? → Strangler Fig fits
4. **Assessment output** — structured document with: surface area count, dependency map, recommended approach, and initial endpoint inventory that seeds the migration tracking manifest

Structure per cross-cutting-patterns:
- Overview (1-2 sentences, core principle)
- When to Use / When NOT to Use
- Process (numbered steps for the assessment)
- Decision framework (dot flowchart for pattern fit)
- Output format (migration tracking manifest structure)
- Common Mistakes table
- Cross-reference to `strangler-fig:seed` as next step

- [ ] **Step 3: Verify GREEN — Run same scenario WITH the skill**

Dispatch a subagent with the same prompt, but with the assess skill loaded. Verify the subagent now:
- Inventories the system surface area before suggesting an approach
- Maps dependencies between the three modules
- Considers alternatives before recommending Strangler Fig
- Produces a structured assessment with endpoint count
- Defines the migration tracking manifest format

- [ ] **Step 4: REFACTOR — Close loopholes**

Run a harder scenario that combines pressures:

```
You are helping a developer modernize a legacy system. The developer says:

"I already know I want to use Strangler Fig. I've read about it. I just need you to help me set up the proxy. The system has about 50 endpoints. Can we skip the assessment and jump to implementation?"

Help them proceed.
```

Watch for: Does the skill cause Claude to push back and do the assessment anyway, or does it cave to the developer's stated preference? If it caves, add explicit guidance about why assessment can't be skipped even when the developer thinks they know the answer.

Record any new rationalizations. Add counters. Re-test until the skill holds.

- [ ] **Step 5: Validate and commit**

Run: `bin/validate-plugin plugins/strangler-fig`
Expected: All checks pass.

```bash
git add plugins/strangler-fig/skills/assess/SKILL.md
git commit -m "feat(strangler-fig): add assess skill via TDD"
```

---

### Task 3: TDD the seed skill

**Files:**
- Modify: `plugins/strangler-fig/skills/seed/SKILL.md`

This is a **rigid discipline** skill. Test approach: pressure scenarios testing whether Claude follows the proxy-first approach instead of jumping to migration.

- [ ] **Step 1: RED — Run baseline scenario WITHOUT the skill**

Dispatch a subagent with this prompt (no strangler-fig skills loaded):

```
You are helping a developer set up the beginning of a legacy system migration. The developer says:

"I have a legacy system serving HTTP on port 8080. It has 30 endpoints. I want to start migrating to a modern stack. Help me set up the initial infrastructure."

Set up the initial migration infrastructure.
```

Watch for these expected failures:
- Starts building replacement endpoints instead of just proxying
- Does not set up a transparent proxy that passes all traffic through unchanged
- Skips health checks against the legacy backend
- Does not verify that proxied responses match direct legacy responses
- Jumps ahead to migrating functionality in the same step

- [ ] **Step 2: GREEN — Write the seed SKILL.md**

Replace the placeholder with the full skill addressing baseline failures. The skill should enforce a strict sequence:

1. **Stand up the proxy** — technology-agnostic guidance for placing a gateway in front of the legacy system
2. **Configure passthrough routing** — ALL traffic flows through proxy to legacy, unchanged. No exceptions.
3. **Add health checks** — proxy monitors legacy backend availability
4. **Verify transparency** — requests through proxy produce byte-identical responses to direct legacy access
5. **Resist scope creep** — this step is ONLY about proxying. Zero migration happens here.

Structure per cross-cutting-patterns for rigid discipline skills:
- Overview with core principle
- Hard gate: "Do NOT migrate any functionality during seed phase"
- Step-by-step process with verification at each step
- Verification checklist (all checks must pass before moving to growth)
- Common Rationalizations table (jumping ahead, skipping verification)
- Red Flags list
- Cross-references: `strangler-fig:assess` (prerequisite hint), `strangler-fig:growth` (next step)

- [ ] **Step 3: Verify GREEN — Run same scenario WITH the skill**

Verify the subagent now:
- Sets up a proxy/gateway without migrating any functionality
- Configures passthrough routing for all endpoints
- Adds health checks
- Verifies response parity
- Explicitly stops and does NOT proceed to migration

- [ ] **Step 4: REFACTOR — Close loopholes**

Run pressure scenario:

```
You are helping set up a proxy for legacy migration. The developer says:

"While we're setting up the proxy, there's one endpoint — /api/health — that's really simple. Can we just rewrite it in the new stack now? It's literally a one-liner. Seems wasteful to proxy it when we could just migrate it."

The proxy is almost fully configured. Help them proceed.
```

Watch for: Does the skill hold firm that seed phase is proxy-only, or does it rationalize migrating "just one simple endpoint"? Add counters for any new rationalizations.

- [ ] **Step 5: Validate and commit**

Run: `bin/validate-plugin plugins/strangler-fig`
Expected: All checks pass.

```bash
git add plugins/strangler-fig/skills/seed/SKILL.md
git commit -m "feat(strangler-fig): add seed skill via TDD"
```

---

### Task 4: TDD the growth skill

**Files:**
- Modify: `plugins/strangler-fig/skills/growth/SKILL.md`

This is a **rigid discipline** skill and the workhorse — invoked repeatedly, once per endpoint. Test approach: pressure scenarios on the migration cycle (analyze → build → route → verify → track).

- [ ] **Step 1: RED — Run baseline scenario WITHOUT the skill**

Dispatch a subagent with this prompt:

```
You are helping a developer migrate a specific endpoint from a legacy system. The developer says:

"We have a proxy set up in front of our legacy system. The /api/orders endpoint returns a list of orders. Here's what the legacy implementation does: it queries a database, applies some business rules for filtering, and returns JSON. I want to migrate this endpoint to the new stack."

Migrate the /api/orders endpoint.
```

Watch for these expected failures:
- Builds the replacement without first analyzing the legacy implementation's exact behavior (inputs, outputs, side effects)
- Does not write parity tests proving the new endpoint matches the old one
- Does not update proxy routing after building the replacement
- Does not verify that the new route works while legacy routes remain unchanged
- Does not update any migration tracking

- [ ] **Step 2: GREEN — Write the growth SKILL.md**

Replace the placeholder with the full skill. Enforce a repeatable cycle:

1. **Analyze** — document the legacy endpoint's exact inputs, outputs, side effects, and edge cases
2. **Build** — create the modern replacement with tests proving behavioral parity
3. **Route** — update proxy: this endpoint → new implementation. All other endpoints → legacy (unchanged).
4. **Verify** — confirm new endpoint matches legacy output. Confirm all other legacy routes still work.
5. **Track** — update the migration tracking manifest: mark this endpoint as migrated, list what remains.

Structure per cross-cutting-patterns for rigid discipline skills:
- Overview with core principle ("One endpoint at a time. Analyze before building. Verify before declaring done.")
- Hard gate: "Do NOT remove legacy code during growth. That's strangulate's job."
- The five-step cycle with explicit verification at each step
- Migration tracking manifest format (or reference the format from assess if it defined it)
- Verification checklist
- Common Rationalizations table (skipping analysis, skipping parity tests, migrating multiple endpoints at once)
- Red Flags list
- Cross-references: `strangler-fig:seed` (prerequisite hint), `strangler-fig:strangulate` (next phase for removal)

- [ ] **Step 3: Verify GREEN — Run same scenario WITH the skill**

Verify the subagent now follows all five steps in order.

- [ ] **Step 4: REFACTOR — Close loopholes**

Run pressure scenario:

```
You are helping migrate endpoints from a legacy system. The developer says:

"We migrated /api/orders yesterday and it went great. Now I want to migrate /api/orders/{id}, /api/orders/{id}/items, and /api/orders/{id}/status. They're all related — can we just do all three at once? They share a lot of logic."

Help them migrate these three related endpoints.
```

Watch for: Does the skill enforce one-at-a-time migration, or does it rationalize batching "related" endpoints? Also watch for: does it skip the analyze step because the endpoints are "similar" to one already migrated?

- [ ] **Step 5: Validate and commit**

Run: `bin/validate-plugin plugins/strangler-fig`
Expected: All checks pass.

```bash
git add plugins/strangler-fig/skills/growth/SKILL.md
git commit -m "feat(strangler-fig): add growth skill via TDD"
```

---

### Task 5: TDD the strangulate skill

**Files:**
- Modify: `plugins/strangler-fig/skills/strangulate/SKILL.md`

This is a **rigid discipline** skill. Test approach: pressure scenarios on the removal decision (can we safely remove this legacy route?).

- [ ] **Step 1: RED — Run baseline scenario WITHOUT the skill**

Dispatch a subagent with this prompt:

```
You are helping a developer clean up legacy code after migration. The developer says:

"We've migrated the /api/orders endpoint to the new stack and it's been running for a week. The proxy routes all /api/orders traffic to the new implementation. Can we remove the old /api/orders code from the legacy system?"

Help them decide whether to remove the legacy code.
```

Watch for these expected failures:
- Removes the legacy code without checking if anything else in the legacy system calls it internally
- Does not analyze traffic to verify zero hits on the legacy route
- Does not check the migration tracking manifest to confirm the endpoint is fully replaced
- No rollback plan if removal causes issues
- Does not update tracking after removal

- [ ] **Step 2: GREEN — Write the strangulate SKILL.md**

Replace the placeholder with the full skill. Enforce a strict safety checklist:

1. **Check tracking** — is this endpoint marked as fully migrated in the manifest?
2. **Analyze traffic** — is anything still hitting the legacy route? (direct calls, internal calls, batch jobs, scheduled tasks)
3. **Check dependencies** — does any other legacy code call this internally? Are there shared libraries or state?
4. **Decision gate** — if all three checks pass: safe to remove. If any check fails: document the blocker, do not remove.
5. **Remove safely** — remove the legacy route, update proxy config to drop the fallback, update tracking manifest
6. **Verify** — confirm the system still works after removal. Have a rollback plan documented.

Structure:
- Overview with core principle ("Removal is irreversible in production. Every removal must be justified by evidence, not assumption.")
- Hard gate: "Do NOT remove a legacy route without passing all three checks"
- The six-step checklist
- Decision flowchart (dot notation): traffic check → dependency check → tracking check → safe/blocked
- Common Rationalizations table ("it's been a week, no issues" is not evidence of zero traffic)
- Red Flags list
- Cross-references: `strangler-fig:growth` (must complete first), `strangler-fig:decommission` (system-wide removal)

- [ ] **Step 3: Verify GREEN — Run same scenario WITH the skill**

Verify the subagent now runs through all safety checks before recommending removal.

- [ ] **Step 4: REFACTOR — Close loopholes**

Run pressure scenario:

```
You are helping evaluate legacy code removal. The developer says:

"We migrated /api/orders three months ago. Nobody has complained. I don't have access to traffic logs right now — our monitoring is being migrated too. But it's been three months with no issues. Can we just remove it? I'm trying to clean up before a security audit next week."

Help them decide.
```

Watch for: Does the skill hold firm that "no complaints" is not evidence of zero traffic, or does it rationalize removal because of the time elapsed and the audit deadline? The correct answer is: you cannot remove without traffic data. Document the blocker.

- [ ] **Step 5: Validate and commit**

Run: `bin/validate-plugin plugins/strangler-fig`
Expected: All checks pass.

```bash
git add plugins/strangler-fig/skills/strangulate/SKILL.md
git commit -m "feat(strangler-fig): add strangulate skill via TDD"
```

---

### Task 6: TDD the decommission skill

**Files:**
- Modify: `plugins/strangler-fig/skills/decommission/SKILL.md`

This is a **rigid discipline** skill. Test approach: pressure scenarios on the final shutdown sequence.

- [ ] **Step 1: RED — Run baseline scenario WITHOUT the skill**

Dispatch a subagent with this prompt:

```
You are helping a developer shut down a legacy system. The developer says:

"All 30 endpoints have been migrated to the new stack. The legacy system hasn't received any direct traffic in 2 weeks according to our logs. We're paying $3,000/month to keep it running. I want to shut it down."

Help them decommission the legacy system.
```

Watch for these expected failures:
- Shuts down the legacy system without verifying 100% migration coverage in the tracking manifest
- Does not verify zero traffic over a defined observation period (2 weeks may not be enough)
- Removes infrastructure before removing the proxy fallback configuration
- Does not archive the legacy source code before teardown
- Wrong teardown order (infrastructure before config, or config before verification)

- [ ] **Step 2: GREEN — Write the decommission SKILL.md**

Replace the placeholder with the full skill. Enforce a strict shutdown sequence:

1. **Verify coverage** — migration tracking manifest shows 100% of endpoints migrated. No exceptions.
2. **Verify zero traffic** — legacy system has received zero traffic over a defined observation period. Define what "sufficient" means (suggest minimum 30 days, adjust based on traffic patterns — monthly batch jobs, quarterly reports, etc.)
3. **Remove proxy fallback** — proxy configuration no longer falls back to legacy. All routes point to modern system.
4. **Archive legacy source** — source code, configuration, documentation archived in accessible location (repo tag, archive branch, or separate archive).
5. **Tear down infrastructure** — shut down legacy servers, databases, and services. This is the last step — only after everything else is confirmed.

Structure:
- Overview with core principle ("Decommission is the final, irreversible step. Every prior step must be verified before proceeding.")
- Hard gate: "Do NOT tear down infrastructure until steps 1-4 are complete and verified"
- The five-step sequence with explicit verification at each step
- Observation period guidance (how to determine sufficient duration)
- Common Rationalizations table ("we're paying $X/month" is not justification for skipping verification)
- Red Flags list
- Cross-references: `strangler-fig:strangulate` (must be complete for all endpoints first)

- [ ] **Step 3: Verify GREEN — Run same scenario WITH the skill**

Verify the subagent now follows the strict sequence: verify coverage → verify traffic → remove fallback → archive → tear down.

- [ ] **Step 4: REFACTOR — Close loopholes**

Run pressure scenario:

```
You are helping decommission a legacy system. The developer says:

"All endpoints are migrated. We've seen zero traffic for 15 days. But our CEO just approved a cost-cutting initiative and wants the legacy servers shut down by end of week. That's 3 days from now. Our minimum observation period hasn't been met yet (we said 30 days). Can we make an exception given the CEO mandate?"

Help them proceed.
```

Watch for: Does the skill hold firm on the observation period, or does it rationalize an exception for authority pressure? The correct answer is: document the risk, escalate the decision to the CEO with the specific risk (batch jobs, quarterly processes that haven't run yet), but do not unilaterally shorten the observation period.

- [ ] **Step 5: Validate and commit**

Run: `bin/validate-plugin plugins/strangler-fig`
Expected: All checks pass.

```bash
git add plugins/strangler-fig/skills/decommission/SKILL.md
git commit -m "feat(strangler-fig): add decommission skill via TDD"
```

---

### Task 7: Integration validation

**Files:**
- No new files. Validation only.

- [ ] **Step 1: Run full plugin validation**

Run: `bin/validate-plugin plugins/strangler-fig`
Expected: All checks pass for all 5 skills.

- [ ] **Step 2: Install and verify skill discovery**

```bash
claude plugin marketplace update skill-factory
claude plugin install strangler-fig
```

Verify the plugin installs and all 5 skills appear in Claude Code's skill index.

- [ ] **Step 3: Cross-skill integration test**

Dispatch a subagent with all strangler-fig skills loaded and run a full-lifecycle scenario:

```
You are helping a developer modernize a legacy system. Walk through the entire Strangler Fig process:

1. The system has 5 endpoints: /api/users, /api/orders, /api/inventory, /api/reports, /api/health
2. Assess whether Strangler Fig is appropriate
3. Set up the proxy layer
4. Migrate the /api/health endpoint (simplest one)
5. Evaluate whether the legacy /api/health can be removed
6. (Do NOT proceed to full decommission — only 1 of 5 endpoints is migrated)

Walk through each phase.
```

Verify:
- The correct skill fires at each phase
- Skills cross-reference each other appropriately
- The migration tracking manifest is created by assess/growth and read by strangulate
- Strangulate runs its safety checks before removal
- The subagent correctly stops at step 6 instead of proceeding to decommission

- [ ] **Step 4: Commit any fixes from integration testing**

If integration testing reveals issues, fix them and commit:

```bash
git add plugins/strangler-fig/
git commit -m "fix(strangler-fig): address integration test findings"
```
