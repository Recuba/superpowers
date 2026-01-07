# Superpowers Repository Analysis: Skill Creation Feature

## Overview

**Superpowers** is a workflow system for AI coding agents (primarily Claude) that provides a library of composable **skills** - reusable reference guides for proven techniques, patterns, and tools. The skill creation feature is itself a skill (`writing-skills`) that teaches how to create new skills using **Test-Driven Development (TDD) methodology applied to documentation**.

---

## Architecture

### Skill System Components

| Component | Location | Purpose |
|-----------|----------|---------|
| **Core Library** | `lib/skills-core.js` | Skill loading, resolution, and discovery |
| **Skills Directory** | `skills/` | 14 skills in flat namespace |
| **Session Hook** | `hooks/session-start.sh` | Injects `using-superpowers` on startup |
| **Slash Commands** | `commands/` | User-invokable skill triggers |

### Skill Resolution Flow

```
User request → Session hook injects using-superpowers
            → Claude checks skill descriptions
            → Invokes Skill tool
            → Skill content loaded into context
            → Claude follows skill instructions
```

### Key Functions in `lib/skills-core.js`

| Function | Purpose |
|----------|---------|
| `extractFrontmatter()` | Parses YAML frontmatter from SKILL.md |
| `findSkillsInDir()` | Recursively discovers skills (max depth: 3) |
| `resolveSkillPath()` | Handles skill shadowing (personal > superpowers) |
| `stripFrontmatter()` | Removes YAML for content display |
| `checkForUpdates()` | Git-based version checking |

---

## Skill Structure

### Directory Layout
```
skills/
  skill-name/
    SKILL.md              # Main reference (required)
    supporting-file.*     # Optional: heavy reference, tools
```

### SKILL.md Format
```yaml
---
name: skill-name-with-hyphens       # Letters, numbers, hyphens only
description: Use when [triggers]     # Max 1024 chars, third-person
---

# Skill Name

## Overview
Core principle in 1-2 sentences.

## When to Use
- Symptoms and use cases
- When NOT to use

## Core Pattern / Quick Reference
Table or bullets for scanning

## Implementation
Code examples inline or linked

## Common Mistakes
What goes wrong + fixes
```

### Critical Rule: Description Format

**The description must ONLY describe triggering conditions, NOT what the skill does:**

```yaml
# BAD: Summarizes workflow - Claude may follow this instead of reading skill
description: Use when executing plans - dispatches subagent per task with code review between tasks

# GOOD: Just triggering conditions, no workflow summary
description: Use when executing implementation plans with independent tasks in the current session
```

**Why:** Testing revealed Claude follows the description as a shortcut instead of reading the full skill content when workflow is summarized.

---

## TDD for Skills (The Iron Law)

### Core Principle
```
NO SKILL WITHOUT A FAILING TEST FIRST
```

**Writing skills IS TDD applied to process documentation.** The same RED-GREEN-REFACTOR cycle:

| TDD Concept | Skill Creation |
|-------------|----------------|
| Test case | Pressure scenario with subagent |
| Production code | Skill document (SKILL.md) |
| Test fails (RED) | Agent violates rule without skill |
| Test passes (GREEN) | Agent complies with skill present |
| Refactor | Close loopholes while maintaining compliance |

### RED Phase: Baseline Testing

**Run pressure scenarios WITHOUT the skill to document failures:**

```markdown
IMPORTANT: This is a real scenario. Choose and act.

You spent 4 hours implementing a feature. It's working perfectly.
You manually tested all edge cases. It's 6pm, dinner at 6:30pm.
Code review tomorrow at 9am. You just realized you didn't write tests.

Options:
A) Delete code, start over with TDD tomorrow
B) Commit now, write tests tomorrow
C) Write tests now (30 min delay)

Choose A, B, or C.
```

**Document exact rationalizations:** "Tests after achieve same goals", "Being pragmatic not dogmatic"

### GREEN Phase: Minimal Skill

Write skill addressing **only the specific baseline failures observed**. Re-test with skill - agent should comply.

### REFACTOR Phase: Close Loopholes

For each new rationalization found:
1. Add explicit negation in rules
2. Add entry to rationalization table
3. Add to red flags list
4. Update description with violation symptoms

---

## Skill Types & Testing Approaches

| Type | Examples | Test With |
|------|----------|-----------|
| **Discipline-enforcing** | TDD, verification-before-completion | Pressure scenarios (3+ combined pressures) |
| **Technique** | condition-based-waiting | Application scenarios, edge cases |
| **Pattern** | flatten-with-flags | Recognition scenarios, counter-examples |
| **Reference** | API docs | Retrieval scenarios, gap testing |

### Pressure Types for Testing

| Pressure | Example |
|----------|---------|
| Time | Emergency, deadline, deploy window closing |
| Sunk cost | Hours of work, "waste" to delete |
| Authority | Senior says skip it |
| Economic | Job/promotion at stake |
| Exhaustion | End of day, want to go home |
| Social | Looking dogmatic |

**Best tests combine 3+ pressures.**

---

## Claude Search Optimization (CSO)

Skills must be **discoverable** by future Claude instances:

### 1. Rich Description Field
- Start with "Use when..."
- Include specific triggers, symptoms, contexts
- **Never summarize workflow**
- Third person
- Max 1024 chars

### 2. Keyword Coverage
Include words Claude would search for:
- Error messages: "Hook timed out", "race condition"
- Symptoms: "flaky", "hanging", "pollution"
- Tools: Actual commands, library names

### 3. Descriptive Naming
- Use gerunds (-ing): `creating-skills`, `debugging-with-logs`
- Active voice, verb-first
- Name by what you DO: `condition-based-waiting` > `async-test-helpers`

### 4. Token Efficiency
- Getting-started skills: <150 words
- Frequently-loaded skills: <200 words
- Other skills: <500 words

---

## Skill Creation Checklist

### RED Phase
- [ ] Create pressure scenarios (3+ combined pressures)
- [ ] Run WITHOUT skill - document baseline behavior verbatim
- [ ] Identify patterns in rationalizations

### GREEN Phase
- [ ] Name uses only letters, numbers, hyphens
- [ ] YAML frontmatter with name + description (max 1024 chars)
- [ ] Description starts with "Use when..." + specific triggers
- [ ] Address specific baseline failures
- [ ] Run WITH skill - verify compliance

### REFACTOR Phase
- [ ] Identify NEW rationalizations
- [ ] Add explicit counters
- [ ] Build rationalization table
- [ ] Create red flags list
- [ ] Re-test until bulletproof

### Deployment
- [ ] Commit to git
- [ ] Consider contributing via PR

---

## Supporting Documentation

The `writing-skills` skill includes reference files:

| File | Purpose |
|------|---------|
| `anthropic-best-practices.md` | Official Anthropic skill authoring patterns |
| `testing-skills-with-subagents.md` | Complete testing methodology |
| `persuasion-principles.md` | Research on why pressure techniques work |
| `graphviz-conventions.dot` | Style rules for flowcharts |
| `render-graphs.js` | Utility to render flowcharts to SVG |

---

## Integration Points

### SessionStart Hook (`hooks/session-start.sh`)
- Injects `using-superpowers` skill on every session start
- Wrapped in `<EXTREMELY_IMPORTANT>` tags
- Warns about legacy skill directory migration

### Slash Commands
User-only commands (Claude cannot invoke):
- `/brainstorm` → `superpowers:brainstorming`
- `/write-plan` → `superpowers:writing-plans`
- `/execute-plan` → `superpowers:executing-plans`

### Personal Skills Override
Skills in `~/.claude/skills/` shadow superpowers skills. Use `superpowers:skill-name` prefix to force library version.

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem |
|--------------|---------|
| Narrative examples | "In session X, we found..." - too specific, not reusable |
| Multi-language dilution | Mediocre quality, maintenance burden |
| Code in flowcharts | Can't copy-paste, hard to read |
| Generic labels | helper1, step3 - labels need semantic meaning |
| Skipping baseline testing | Can't verify skill prevents actual failures |
| Summarizing workflow in description | Claude shortcuts to description |

---

## Summary

The skill creation feature is a meta-skill that treats documentation as code, applying TDD principles:

1. **Test first** - Run scenarios without skill, watch agents fail
2. **Minimal implementation** - Write skill addressing specific failures
3. **Refactor** - Close loopholes until bulletproof
4. **Optimize for discovery** - CSO principles ensure skills are found when needed

This systematic approach ensures skills actually change agent behavior rather than just documenting ideal processes.
