---
name: multi-role-workflow
description: Use when tackling software engineering tasks — features, refactors, migrations, bug fixes, or greenfield builds — that benefit from separated Tech Lead analysis, focused implementation, adversarial code review, and knowledge capture. Especially when the task has security implications, architectural decisions, high regression risk, or reusable patterns to extract.
---

# Multi-Role Workflow for Software Engineering

## Overview

Decompose software engineering tasks across four cognitively separated roles, each running as a **separate subagent** with a distinct persona. Each role maps to a real software engineering function.

Core principle: **Cognitive separation prevents blind spots.** The architect must not review their own design. The developer must not grade their own code.

## Role Map

| Role | SE Equivalent | This Agent... |
|------|--------------|----------------|
| **Advisor** | Tech Lead / Architect | Analyzes strategy, designs architecture, defines acceptance criteria. Does NOT write code. |
| **Executor** | Developer | Implements to spec, writes tests, handles all boundary conditions. Does NOT design architecture or self-review. |
| **Grader** | Code Reviewer / QA | Adversarially reviews for correctness, completeness, quality, robustness, test quality, performance, security, regression. Does NOT fix anything. |
| **Dreamer** | Knowledge Manager | Extracts reusable design patterns, prompt fragments, checklists, knowledge base entries, and process improvements. |

## When to Use

```
Task received (software engineering)
    │
    ├─ New feature or greenfield build? ──→ Use multi-role
    ├─ Refactor with regression risk? ────→ Use multi-role
    ├─ Bug fix in complex subsystem? ─────→ Use multi-role
    ├─ API / schema / contract change? ───→ Use multi-role
    ├─ Security or data integrity? ───────→ Use multi-role
    ├─ Architecture decision needed? ─────→ Use multi-role
    ├─ New library/framework adoption? ───→ Use multi-role
    │
    └─ Typo / formatting / simple config? → Skip
```

**Skip when:** typo fix, formatting, adding a single log line, updating a constant, tasks with zero design choices.

## Workflow

```
Task → Advisor → Executor → Grader → Executor(fix) → Dreamer → Done
         │           │           │            │            │
         ▼           ▼           ▼            ▼            ▼
    Architecture  Production  Code Review   Revisions   Knowledge
    + Criteria      Code       Findings      + Tests     Capture
```

Each arrow = spawn a subagent. Each box = concrete artifact passed to the next. No phase is skippable.

### Phase 1: Advisor (Tech Lead)

Spawn with: [prompts/advisor.md](prompts/advisor.md)

Produces: task classification, architecture recommendation, module decomposition, API contract, data model, tech stack decisions, risk assessment table with mitigation, specific testable acceptance criteria, non-functional requirements, exhaustive boundary conditions.

**This is the contract.** Executor follows it. Grader verifies against it.

### Phase 2: Executor (Developer)

Spawn with: [prompts/executor.md](prompts/executor.md) — "Initial Implementation" section

Produces: complete implementation matching existing code conventions, tests covering every acceptance criterion and boundary condition, passing test output.

Rules: follow the plan, flag deviations, no placeholders, proper types, explicit error handling, runnable tests.

### Phase 3: Grader (Code Reviewer)

Spawn with: [prompts/grader.md](prompts/grader.md)

Reviews across 7 dimensions: correctness, completeness, code quality, robustness, test quality, performance/security, regression risk.

Produces: findings with severity (Critical/Major/Minor) and category, location, detail, fix direction. Final VERDICT: PASS / PASS WITH MINOR / NEEDS FIXES / FAIL.

### Phase 4: Executor — Fixes

Spawn with: [prompts/executor.md](prompts/executor.md) — "Fix Phase" section

Fixes all Critical and Major findings. Shows before/after. Re-runs tests. Responds to false positives with evidence.

### Phase 5: Dreamer (Knowledge Manager)

Spawn with: [prompts/dreamer.md](prompts/dreamer.md)

Produces: design patterns catalog, reusable prompt fragments, checklists, knowledge base entries (`[[entry-name]]` format), process retrospective, skill improvement suggestions.

**This phase compounds.** Each run makes the NEXT run faster and higher quality.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Merging Advisor + Executor into one agent | Separate. The Advisor's output is the Executor's constraint. |
| Developer self-reviews own code | Never. Self-review is structurally blind. Always use Grader. |
| Skipping Dreamer ("I'll document later") | You won't. Run Dreamer while process context is fresh. |
| Grader is too gentle (validates instead of attacks) | Prompt is explicitly adversarial. Grader TRIES to find problems. |
| Advisor produces vague criteria | Demand falsifiable criteria. "Code works" is not a criterion. |
| Executor diverges from the architecture plan | If plan is wrong, loop back to Advisor. Don't let Executor redesign. |
| Skipping phases because "it's a simple change" | Simple changes cause production incidents. Full workflow. No shortcuts. |

## Red Flags — Stop and Follow the Workflow

- "This is simple, I'll just write it" → Hidden complexity. Run Advisor first.
- "I can review my own PR" → You cannot. Grader is independent for a reason.
- "I'll capture patterns after I ship" → Context decays exponentially. Dreamer runs NOW.
- "Five subagents is overkill" → Cost of separation < cost of missed bugs + lost knowledge.
- "Let me combine Advisor and Executor to save time" → Merging roles CREATES the blind spots this workflow prevents.

**All of these mean: Follow the full workflow. No shortcuts.**

## Quick Reference

```
Phase 1: Advisor    → prompts/advisor.md    → Architecture + Acceptance Criteria
Phase 2: Executor   → prompts/executor.md   → Implementation + Tests
Phase 3: Grader     → prompts/grader.md     → Findings + Verdict
Phase 4: Executor   → prompts/executor.md   → Fixes + Re-verification
Phase 5: Dreamer    → prompts/dreamer.md    → Reusable Knowledge + Process Improvements
```

Each phase is a separate subagent spawn. Each consumes the previous phase's full output.
