---
name: multi-role-workflow
description: Use when facing complex tasks that need separated strategic analysis, implementation, independent review, and knowledge capture — especially tasks with security implications, architectural decisions, high regression risk, or when reusable patterns should be extracted and preserved.
---

# Multi-Role Workflow

## Overview

Decompose complex tasks across four cognitively separated roles, each running as a **separate subagent** with a distinct persona. No single agent wears two hats.

Core principle: **Cognitive separation prevents blind spots.** The agent that designs must not review. The agent that implements must not judge its own work.

## When to Use

```
Task received
    │
    ├─ Security / data integrity? ──→ Use multi-role
    ├─ Architectural decisions? ────→ Use multi-role
    ├─ High regression risk? ───────→ Use multi-role
    ├─ New abstraction or pattern? ──→ Use multi-role
    ├─ Multiple files/subsystems? ───→ Use multi-role
    │
    └─ Simple mechanical edit? ─────→ Skip (direct OK)
```

**Skip:** typos, formatting, simple config, pure information retrieval, <5 trivial steps.

## Workflow

```
Task → Advisor → Executor → Grader → Executor(fix) → Dreamer → Done
         │           │           │            │            │
         ▼           ▼           ▼            ▼            ▼
      Analysis   Implementation Findings    Fixes      Knowledge
```

Each arrow = spawn a subagent. Each box = concrete artifact passed to the next.

### Phase 1: Advisor — Strategic Analysis

Spawn with prompt template: [prompts/advisor.md](prompts/advisor.md)

**Output required:** Task classification, risk assessment table, specific testable acceptance criteria, recommended approach, boundary conditions list.

**Why separate:** Prevents jumping to implementation before understanding risks. The Advisor's analysis becomes the contract for all later phases.

### Phase 2: Executor — Implementation

Spawn with prompt template: [prompts/executor.md](prompts/executor.md) (use "Initial Implementation" section)

**Output required:** Complete implementation with tests covering every acceptance criterion.

**Why separate:** The Executor follows orders. It does not question architecture. This prevents scope creep and design-by-implementation.

### Phase 3: Grader — Independent Review

Spawn with prompt template: [prompts/grader.md](prompts/grader.md)

**Output required:** Findings list with severity (Critical/Major/Minor), category, description, location, fix hint.

**Why separate:** Self-review is structurally blind. The Grader is explicitly adversarial — it TRIES to find problems. This catches what the Executor couldn't see.

### Phase 4: Executor — Fixes

Spawn with prompt template: [prompts/executor.md](prompts/executor.md) (use "Fix Phase" section)

**Output required:** Fixed implementation + before/after for each fix + responses to false positives.

### Phase 5: Dreamer — Knowledge Capture

Spawn with prompt template: [prompts/dreamer.md](prompts/dreamer.md)

**Output required:** Reusable patterns, prompt templates, checklists, knowledge base entries (`[[entry-name]]` format), process retrospective.

**Why separate:** Context decays fast. Extract patterns while everything is fresh. This phase pays for itself on the next similar task.

## Role Reference

| Role | Does | Does NOT | Trait |
|------|------|----------|-------|
| Advisor | Strategy, risks, criteria, approach | Implement, review, decide specifics | Strategic |
| Executor | Implement, write, build, fix | Design architecture, self-review | Precise |
| Grader | Find bugs, omissions, gaps | Fix anything, suggest architecture | Skeptical |
| Dreamer | Extract patterns, save knowledge | Judge quality, redo work | Reflective |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Merging Advisor + Executor into one agent | Separate them. Advisor's output constrains Executor. |
| Executor self-reviews | Never. Self-review is blind. Always use Grader. |
| Skipping Dreamer ("I'll remember") | You won't. Run Dreamer while context is fresh. |
| Grader is too gentle | Prompt Grader to be adversarial. "Find what's wrong" not "validate." |
| Advisor produces vague criteria | Demand specific, testable criteria. "Works correctly" is not a criterion. |
| Executor diverges from plan | If plan is wrong, go back to Advisor. Don't let Executor redesign. |

## Red Flags

- "This is simple enough to skip the Advisor" → Hidden complexity exists. Run Advisor.
- "I can grade my own work" → You cannot. Independence is the point.
- "I'll capture patterns later" → Context decays. Dreamer runs NOW.
- "The roles are overkill for this" → Cost of separation < cost of missed bugs or lost knowledge.
- "Let me combine two phases" → Each phase prevents a specific failure mode.

**All of these mean: Follow the full workflow. No shortcuts.**
