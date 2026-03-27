# Spire Wave Engine

## Concept

Waves are groups of tasks that can execute in parallel because they have no dependencies on each other. Tasks in Wave N depend only on tasks from Waves 1 through N-1.

This works for any project type — parallel code modules, independent research sections, concurrent document chapters, or simultaneous workflow components.

## Wave Planning Algorithm

```
Input: List of tasks, each with dependencies[]
Output: Wave assignments

1. Build dependency graph: task → [tasks it depends on]
2. Find all tasks with dependencies = [] → these are Wave 1
3. Remove Wave 1 tasks from the graph
4. Find all tasks whose remaining dependencies are now [] → these are Wave 2
5. Repeat until all tasks are assigned
6. If tasks remain unassigned → circular dependency detected → ERROR
```

## Dependency Rules

Valid dependencies (adapt to project type):
- **Software**: Task B needs a file that Task A creates; Task B imports from Task A's module
- **Research**: Task B's analysis depends on data Task A produces; conclusion depends on all analysis sections
- **Document**: Summary section depends on detail sections; introduction depends on body being written
- **Workflow**: Step B takes output from Step A; testing depends on implementation
- **General**: Task B references, builds on, or requires output from Task A

Invalid dependencies (flag as errors during planning):
- Two tasks in the same wave depend on each other (circular)
- A task depends on itself
- A task depends on something not in the plan

## Wave Execution Protocol

### Pre-Wave
1. Read `.spire/STATE.md` to confirm previous wave completed
2. Read `.spire/PLAN.md` to get task specs for current wave
3. Read `.spire/DESIGN.md` for project context

### Dispatch (Solo Mode)
```
For each task in wave:
  Agent(
    prompt: [task spec + design context + material to read],
    run_in_background: true,
    name: "builder-TASK-{ID}"
  )
```

All agents launch simultaneously. Wait for all to complete.

### Dispatch (Team Mode)
```
TeamCreate(team_name: "spire-wave-{N}")

For each task in wave:
  TaskCreate(subject: "TASK-{ID}: {title}")
  Agent(
    team_name: "spire-wave-{N}",
    name: "builder-{ID}",
    prompt: [task spec + design context],
    run_in_background: true
  )
```

### Post-Wave
1. Collect all builder results
2. Dispatch `spire-reviewer` with all task specs and the wave number
3. If reviewer returns ISSUES_FOUND:
   - For each issue, dispatch a fix agent (builder with issue context)
   - Max 2 fix rounds per task
   - If still broken after 2 rounds → emergency gate
4. Dispatch `spire-integrator` to merge and verify consistency
5. Update STATE.md: mark wave complete, record any issues
6. If team mode: `TeamDelete(team_name: "spire-wave-{N}")`

### Emergency Gate
Presented when a wave cannot be completed after correction rounds:

```
╔══════════════════════════════════════════════╗
║  SPIRE: Wave {N} Issue                      ║
╠══════════════════════════════════════════════╣
║  {count} task(s) could not be completed:    ║
║  • TASK-{ID}: {issue summary}               ║
║                                             ║
║  Options:                                   ║
║  1. Skip and continue to next wave          ║
║  2. Revise the plan for these tasks         ║
║  3. Let me handle it manually               ║
║  4. Abort the build                         ║
╚══════════════════════════════════════════════╝
```

## Context Provided to Each Builder

Each builder agent receives exactly this context (nothing more, nothing less):

```
## Your Task
{full task spec from PLAN.md including title, description, deliverables, acceptance criteria}

## Project Type
{software | research | document | workflow | hybrid}

## Design Context
{relevant section from DESIGN.md — only the parts this task touches}

## Existing Material to Read First
{list of files/resources the task depends on}

## Conventions
{if applicable: coding style, writing style, formatting, methodology conventions}

## Important
- Execute ONLY this task
- Report status: DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
- Follow acceptance criteria exactly
```

## Model Selection for Builders

Based on task complexity estimate from PLAN.md:

| Complexity | Model | Examples |
|------------|-------|---------|
| simple | haiku or sonnet | Config files, boilerplate, data formatting, simple sections |
| moderate | sonnet | Multi-file changes, analytical writing, methodology implementation |
| complex | opus | Architectural decisions, complex analysis, synthesis across sources, integration logic |

Use `model` parameter on the Agent tool call to set this.
