---
name: sft-skill-anatomy
description: Use when studying how superpowers skills are structured to understand why each section exists
---

# Skill Anatomy

Structural breakdown of three representative superpowers skills. Maps every section to the writing-skills authoring guide and explains the design rationale behind each structural choice.

## How to Use This Reference

Look up the skill type closest to what you are building, then consult the summary table below. For full section maps, removal-impact analysis, and supporting file breakdowns, read skill-analyses.md.

## Skill Summary

| Skill | Type | Word Count | Supporting Files | Key Insight |
|-------|------|------------|-----------------|-------------|
| brainstorming | Technique | ~1600w | 7 (visual companion, scripts, reviewer prompt) | Layered defense: HARD-GATE + anti-pattern + checklist + flow + user gate |
| test-driven-development | Discipline | ~1100w | 1 (testing-anti-patterns.md) | Anti-rationalization architecture: Iron Law + verify steps + comparisons + table + flags |
| writing-skills | Meta | ~1800w | 6 (best practices, persuasion, testing, graphviz, renderer, example) | Self-exemplification: follows every convention it teaches |

## Cross-Skill Comparison

| Dimension | brainstorming | test-driven-development | writing-skills |
|-----------|---------------|------------------------|----------------|
| Type | Technique | Discipline-Enforcing | Meta |
| Enforcement | Flexible within steps, rigid step order | Rigid everywhere | Rigid for rules, flexible for application |
| Has Iron Law | No (uses HARD-GATE instead) | Yes | Yes |
| Has rationalization table | No (single anti-pattern block) | Yes (11 rows) | Yes (8 rows) |
| Has Red Flags list | No | Yes (13 items) | No (uses STOP section) |
| Has flowchart | Yes (process flow) | Yes (cycle) | Yes (decision) |
| Has verification checklist | No (uses user review gate) | Yes (8 items) | Yes (full TDD checklist) |
| Defensive layers | 5 | 5 | 5 |
| Primary risk | Premature implementation | Rationalization to skip TDD | Undiscoverable or untested skills |

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

## Supporting Files

| File | Purpose |
|------|---------|
| skill-analyses.md | Full section maps, removal-impact analysis, and supporting file breakdowns for all three skills |
