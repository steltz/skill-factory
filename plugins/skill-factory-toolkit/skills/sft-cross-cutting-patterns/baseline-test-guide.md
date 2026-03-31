# Baseline Test Guide

How to design and run a baseline test for each skill type. A baseline test proves a skill is necessary by showing Claude fails without it.

## What Is a Baseline Test?

A baseline test gives Claude a task that requires the skill, **without loading the skill**, and records how Claude fails. The failure patterns become the skill's enforcement targets.

No skill ships without a failing baseline test first. This is the RED phase of the TDD cycle applied to skills.

## Designing Baseline Tests by Skill Type

| Skill Type | Test Design | What to Record |
|------------|-------------|----------------|
| **Discipline** | Ask Claude to do the constrained task (e.g., "fix this bug") without the skill loaded | Which rules Claude violates, which shortcuts it takes, which rationalizations it uses |
| **Technique** | Ask Claude to follow the process (e.g., "brainstorm a feature") without guidance | Which steps it skips, where it freestyles, what quality issues appear |
| **Reference** | Ask Claude a question the reference answers (e.g., "how should I structure this skill?") | What incorrect defaults it uses, what patterns it misses |
| **Meta** | Ask Claude to perform the meta-task (e.g., "write a new skill") without the meta-skill | Which conventions it violates, which structural patterns it misses |

## Running a Baseline Test

1. Start a fresh Claude session (no skill loaded)
2. Give Claude the task the skill is designed for
3. Let Claude complete the task fully
4. Record every failure in a result table

## Result Template

```markdown
## Baseline Test Results

| # | Failure | Skill Section That Fixes It |
|---|---------|----------------------------|
| 1 | [What Claude did wrong] | [Section name or "NEW — needs section"] |
| 2 | ... | ... |
```

## Mapping Failures to Skill Sections

| Failure Pattern | Maps To |
|-----------------|---------|
| Claude skipped a required step | Process / Checklist section |
| Claude rationalized skipping a rule | Common Rationalizations table |
| Claude had a "this is too simple" thought | Red Flags list |
| Claude used wrong defaults | Quick Reference table |
| Claude didn't verify its work | Verification Checklist |
| Claude chose wrong approach | Decision flowchart or When to Use |

## Real Example: graphql-overfetch-analyzer

**Baseline test:** Asked Claude to analyze a GraphQL codebase for overfetching without the skill.

**Key failures observed:**
- Checked only query files, missed fragments and resolvers
- No systematic tracing from UI component to schema
- Declared "no overfetching" after surface-level scan
- No verification that all routes were covered

**How failures mapped to skill sections:**
- Missed resolvers → Tracing methodology section with explicit resolver check
- Surface scan → Iron Law: `NO OVERFETCH JUDGMENT WITHOUT FULL TRACE`
- No route coverage → Route-scoped analysis mode
- Premature declaration → Verification checklist with coverage requirements

Each baseline failure became either a section, a checklist item, or a rationalization table entry.
