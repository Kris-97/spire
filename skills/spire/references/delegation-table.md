# Spire Delegation Table

## Principle

Spire delegates to existing skills when they handle the task better. Spire's value is in orchestration — deciding what to do and when — not reinventing individual steps.

## Decision Process

Before starting any phase step, check:
1. Is there an existing skill that handles this exact scenario?
2. Would the existing skill produce better results than doing it ourselves?
3. Can we cleanly integrate the skill's output into our pipeline?

If yes to all three → delegate. Otherwise → use Spire's own agents.

## Delegation Map

### Phase 0: SCOPE

| Condition | Action |
|-----------|--------|
| Requirements unclear, user needs exploration | Invoke `superpowers:brainstorming`. Ingest resulting spec as PROJECT.md input |
| Existing codebase needs deep analysis | Use `feature-dev:code-explorer` agent type for thorough exploration |
| Simple, clear request | Handle directly — ask 2-4 questions, write PROJECT.md |

### Phase 1: DESIGN

| Condition | Action |
|-----------|--------|
| UI/frontend design needed | Invoke `frontend-design:frontend-design` for the interface portion. Integrate into DESIGN.md |
| API design needed | Use `spire-architect` agent (no existing skill covers this well) |
| Standard architecture | Use `spire-architect` agent |

### Phase 2: PLAN

| Condition | Action |
|-----------|--------|
| Single-wave, sequential project | Delegate to `superpowers:writing-plans` — it handles simple plans well |
| Multi-wave project | Handle directly — wave grouping is Spire's unique contribution |

### Phase 3: BUILD

| Condition | Action |
|-----------|--------|
| Single-wave, sequential execution | Can delegate to `superpowers:executing-plans` or `superpowers:subagent-driven-development` |
| Multi-wave parallel execution | Handle directly — wave engine is Spire's core |
| 4+ parallel streams, inter-agent coordination | Use TeamCreate pattern from `launch-team` |
| Git isolation needed | Invoke `superpowers:using-git-worktrees` before starting build |

### Phase 4: VERIFY & SHIP

| Condition | Action |
|-----------|--------|
| Commit and PR requested | Delegate to `commit-commands:commit-push-pr` |
| Code review before shipping | Invoke `superpowers:requesting-code-review` |
| Final verification | Use `spire-verifier` agent (custom cross-check against requirements) |

## How to Invoke Existing Skills

```
Skill(skill: "superpowers:brainstorming", args: "{context}")
Skill(skill: "superpowers:writing-plans")
Skill(skill: "frontend-design:frontend-design")
Skill(skill: "commit-commands:commit-push-pr")
```

## What Spire Never Delegates

These are Spire's unique contributions:

1. **Wave dependency analysis** — topological sort, grouping, validation
2. **Multi-wave execution** — dispatching waves, inter-wave integration
3. **The .spire/ artifact system** — creating, updating, reading state files
4. **Mode selection** — deciding solo vs. team based on project analysis
5. **Four-gate checkpoint protocol** — presenting gates, recording decisions
6. **Gap-closure loop** — identifying unmet requirements and creating fix plans
7. **Resume protocol** — reading STATE.md and resuming from last checkpoint
