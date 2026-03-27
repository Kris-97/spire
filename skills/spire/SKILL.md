---
name: spire
description: "Use when the user wants to build something from scratch or add a major feature — apps, analysis pipelines, CLI tools, APIs, workflows. Triggers on: 'build me', 'create a', 'I want to make', 'let's build', '/spire'. NOT for small edits, bug fixes, or questions."
---

# Spire — Full-Lifecycle Project Builder

Spire takes an idea and builds it into working software through 5 phases with 4 checkpoint gates. It combines its own build pipeline with smart delegation to existing skills.

## Core Principles

1. **Checkpoint-driven**: Never proceed past a gate without user approval
2. **Fresh context per agent**: Every builder/reviewer gets a clean context window with just what it needs
3. **Wave-based parallelism**: Independent tasks execute simultaneously, dependent tasks wait
4. **Artifacts as source of truth**: All state lives in `.spire/` as human-readable markdown
5. **Delegate when possible**: Use existing skills (brainstorming, writing-plans, frontend-design) instead of reinventing

## Phase State Machine

```
SCOPE ──GATE 1──▶ DESIGN ──GATE 2──▶ PLAN ──GATE 3──▶ BUILD ──GATE 4──▶ VERIFY & SHIP
```

---

## Phase 0: SCOPE

**Goal**: Understand what to build, decide execution mode.

**Steps**:
1. If working in an existing codebase, dispatch `spire-analyst` agent to explore:
   - Read CLAUDE.md, package.json/pyproject.toml, directory structure
   - Identify existing patterns, tech stack, conventions
   - Report constraints and integration points

2. If the user's request is genuinely unclear or exploratory, delegate to `superpowers:brainstorming` skill. Ingest the resulting spec as input for Phase 1.

3. Otherwise, ask 2-4 clarifying questions (one at a time, using AskUserQuestion):
   - What is the core functionality?
   - What's the tech stack preference? (or detect from existing codebase)
   - Any constraints (performance, compatibility, dependencies)?
   - Who/what consumes the output?

4. Determine execution mode:
   - **Solo mode** (default): 1-3 independent work streams, no inter-agent coordination needed
   - **Team mode**: 4+ parallel streams OR agents need to coordinate on shared interfaces

5. Write `.spire/PROJECT.md`:
   ```markdown
   # Project: {name}

   ## Description
   {what we're building, 2-3 sentences}

   ## Requirements
   - REQ-1: {requirement}
   - REQ-2: {requirement}
   ...

   ## Constraints
   - {tech stack, performance, compatibility}

   ## Execution Mode
   {solo | team} — {reasoning}

   ## Existing Codebase Context
   {if applicable: patterns found, integration points, conventions to follow}
   ```

6. Present **GATE 1**:
   ```
   ╔══════════════════════════════════════════╗
   ║  SPIRE GATE 1: Scope Confirmation       ║
   ╠══════════════════════════════════════════╣
   ║  Project: {name}                        ║
   ║  Type: {new project | feature}          ║
   ║  Mode: {Solo | Team}                    ║
   ║                                         ║
   ║  Requirements: {count} items            ║
   ║  Key decisions:                         ║
   ║  • {decision 1}                         ║
   ║  • {decision 2}                         ║
   ╚══════════════════════════════════════════╝
   ```
   Options: `[Approve] [Adjust scope] [Abort]`

---

## Phase 1: DESIGN

**Goal**: Define how to build it.

**Steps**:
1. Dispatch `spire-architect` agent with PROJECT.md content. The architect must:
   - Propose 2-3 approaches with tradeoffs
   - Recommend one approach with reasoning
   - Include: component breakdown, data flow, file structure, tech decisions

2. Present approaches to user. User picks one.

3. For UI-heavy projects, delegate the interface design portion to `frontend-design:frontend-design` skill.

4. Write `.spire/DESIGN.md`:
   ```markdown
   # Design: {project name}

   ## Chosen Approach
   {approach name and summary}

   ## Architecture
   {component diagram or description}

   ## Components
   ### {Component 1}
   - Purpose: {what it does}
   - Files: {file paths}
   - Dependencies: {what it depends on}

   ### {Component 2}
   ...

   ## Data Flow
   {how data moves through the system}

   ## File Structure
   ```
   {planned directory tree}
   ```

   ## Tech Stack
   - {language/framework}: {why}
   ```

5. Present **GATE 2**:
   ```
   ╔══════════════════════════════════════════╗
   ║  SPIRE GATE 2: Design Approval          ║
   ╠══════════════════════════════════════════╣
   ║  Approach: {chosen approach}            ║
   ║  Components: {count}                    ║
   ║  Files: {count} planned                 ║
   ║  Tech: {stack summary}                  ║
   ║                                         ║
   ║  Design saved to: .spire/DESIGN.md      ║
   ╚══════════════════════════════════════════╝
   ```
   Options: `[Approve] [Revise design] [Back to scope] [Abort]`

---

## Phase 2: PLAN

**Goal**: Create executable task plan with wave assignments.

**Steps**:
1. Read DESIGN.md. Decompose into concrete tasks. Each task must have:
   - Clear title and description
   - Exact file paths to create/modify
   - Code snippets or pseudocode for the implementation
   - Acceptance criteria (grep-verifiable, not subjective)
   - Dependencies (which other tasks must complete first)
   - Complexity estimate: simple | moderate | complex

2. Dependency analysis and wave grouping:
   - Wave 1: tasks with zero dependencies (all execute in parallel)
   - Wave 2: tasks depending only on Wave 1 outputs
   - Wave N: tasks depending on Waves 1 through N-1
   - Detect and reject circular dependencies

3. For single-wave projects (all tasks independent), consider delegating directly to `superpowers:writing-plans` for the plan format.

4. Plan validation (max 3 iterations):
   - Check: every file in DESIGN.md is covered by at least one task
   - Check: no task references files created by a same-wave task (dependency error)
   - Check: acceptance criteria are concrete and testable
   - Fix issues and re-validate

5. Write `.spire/PLAN.md`:
   ```markdown
   # Build Plan: {project name}

   ## Summary
   {count} tasks in {N} waves

   ## Tasks

   ### TASK-001: {title}
   - **Wave**: 1
   - **Complexity**: simple
   - **Dependencies**: none
   - **Files**: `src/config.ts` (create)
   - **Description**: {what to do}
   - **Acceptance**: {grep-verifiable criteria}

   ### TASK-002: {title}
   - **Wave**: 1
   - **Complexity**: moderate
   - **Dependencies**: none
   ...

   ### TASK-003: {title}
   - **Wave**: 2
   - **Dependencies**: TASK-001, TASK-002
   ...
   ```

6. Write `.spire/WAVES.md`:
   ```markdown
   # Wave Breakdown

   ## Wave 1 (parallel, no dependencies)
   - TASK-001: {title} [simple]
   - TASK-002: {title} [moderate]

   ## Wave 2 (depends on Wave 1)
   - TASK-003: {title} [complex]
   - TASK-004: {title} [simple]

   ## Dependency Graph
   TASK-003 ← TASK-001, TASK-002
   TASK-004 ← TASK-001
   ```

7. Present **GATE 3**:
   ```
   ╔══════════════════════════════════════════╗
   ║  SPIRE GATE 3: Build Plan Approval      ║
   ╠══════════════════════════════════════════╣
   ║  Tasks: {count} in {N} waves            ║
   ║                                         ║
   ║  Wave 1: {count} tasks (parallel)       ║
   ║  Wave 2: {count} tasks (parallel)       ║
   ║  ...                                    ║
   ║                                         ║
   ║  Plan: .spire/PLAN.md                   ║
   ║  Waves: .spire/WAVES.md                 ║
   ╚══════════════════════════════════════════╝
   ```
   Options: `[Build] [Revise plan] [Back to design] [Abort]`

---

## Phase 3: BUILD

**Goal**: Execute the plan wave by wave with parallel agents.

**Before starting**: Initialize `.spire/STATE.md`:
```markdown
# Spire State

## Project
name: {name}
mode: {solo | team}
phase: build
started: {ISO timestamp}

## Progress
current_wave: 1
total_waves: {N}

## Waves
{initialized from WAVES.md with all tasks set to pending}
```

### Wave Execution Loop

For each wave (1 through N):

#### Solo Mode Execution:
1. For each task in the current wave, dispatch a `spire-builder` agent in parallel:
   ```
   Agent(
     subagent_type: "general-purpose",
     prompt: "{task spec from PLAN.md}\n\nContext:\n{relevant sections of DESIGN.md}\n\nFiles to read first: {list}\n\nAcceptance criteria: {criteria}",
     run_in_background: true,
     name: "builder-{TASK-ID}"
   )
   ```
2. Wait for all builders to complete.

#### Team Mode Execution:
1. Create team: `TeamCreate(team_name: "spire-wave-{N}")`
2. Create tasks: `TaskCreate(subject: "TASK-{ID}: {title}")` for each task
3. Dispatch builders: `Agent(team_name: "spire-wave-{N}", name: "builder-{ID}", ...)`
4. Builders can coordinate via `SendMessage` if needed (e.g., API contracts)

#### After Each Wave:
1. Dispatch `spire-reviewer` agent to check:
   - Does the code match the task spec in PLAN.md?
   - Are acceptance criteria met?
   - Code quality: no obvious bugs, security issues, or style violations
2. If issues found → dispatch fix agents (max 2 correction rounds per task)
3. Dispatch `spire-integrator` agent to:
   - Check for conflicting edits between parallel tasks
   - Verify import/export consistency
   - Run tests if test infrastructure exists
4. Update `.spire/STATE.md`: mark wave tasks as complete/failed
5. If team mode: `TeamDelete(team_name: "spire-wave-{N}")`

#### Error Handling:
- **Builder reports BLOCKED**: Assess. If context issue, re-dispatch with more info. If plan issue, escalate to user.
- **Reviewer finds MAJOR_ISSUES**: Dispatch fix, max 2 rounds. If still broken after 2 rounds, pause and present emergency gate to user.
- **Integrator finds conflicts**: Attempt auto-resolution. If ambiguous, ask user.

After all waves complete → proceed to Phase 4.

---

## Phase 4: VERIFY & SHIP

**Goal**: Validate the build meets requirements, deliver.

**Steps**:
1. Dispatch `spire-verifier` agent:
   - Run all tests (`npm test`, `pytest`, etc.)
   - Run linting and type checks if configured
   - Cross-check: for each requirement in PROJECT.md, verify it's implemented
   - Report: requirements met, requirements missed, test results

2. If gaps found:
   - Create mini gap-closure plan (same format as Phase 2 tasks)
   - Execute gap-closure tasks (single wave, same execution model)
   - Re-verify

3. Write `.spire/COMPLETE.md`:
   ```markdown
   # Completion Report

   ## Requirements Status
   - REQ-1: PASS ✓
   - REQ-2: PASS ✓
   - REQ-3: FIXED (gap-closure round 1) ✓

   ## Test Results
   {test output summary}

   ## Files Created/Modified
   {list}

   ## Known Limitations
   {any caveats}
   ```

4. Present **GATE 4**:
   ```
   ╔══════════════════════════════════════════╗
   ║  SPIRE GATE 4: Delivery                 ║
   ╠══════════════════════════════════════════╣
   ║  Requirements: {met}/{total} met        ║
   ║  Tests: {pass}/{total} passing          ║
   ║  Files: {count} created/modified        ║
   ║                                         ║
   ║  Report: .spire/COMPLETE.md             ║
   ╚══════════════════════════════════════════╝
   ```
   Options:
   - `[Commit & PR]` — delegates to `commit-commands:commit-push-pr`
   - `[Keep working]` — identify remaining tasks, loop back to Plan
   - `[Save & pause]` — STATE.md is already saved, user can `/spire resume` later
   - `[Done]` — mark project complete

---

## Delegation Decision Table

Before doing something yourself, check if an existing skill handles it better:

| Situation | Delegate to | How |
|-----------|-------------|-----|
| Requirements unclear, need exploration | `superpowers:brainstorming` | Invoke skill, ingest resulting spec |
| Single-wave project, sequential tasks | `superpowers:writing-plans` → `superpowers:executing-plans` | Let existing pipeline handle it |
| UI/frontend design needed | `frontend-design:frontend-design` | Invoke for the design portion only |
| Complex build, 4+ parallel streams | TeamCreate pattern from `launch-team` | Switch to team mode |
| Need to explore existing codebase | `feature-dev:code-explorer` agent type | Use for Phase 0 analysis |
| Final commit and PR | `commit-commands:commit-push-pr` | Invoke at Gate 4 |
| Git worktree isolation needed | `superpowers:using-git-worktrees` | Use before Phase 3 |

**Spire's unique contributions** (not delegated):
- Wave dependency analysis and grouping
- Multi-wave execution with inter-wave integration
- The `.spire/` artifact system
- Mode selection (solo vs team)
- The four-gate checkpoint protocol
- Gap-closure verification loop

---

## Resume Protocol

When user invokes `/spire resume`:

1. Read `.spire/STATE.md`
2. Determine current phase and last completed action
3. Present status summary to user
4. Resume from the next incomplete step:
   - If mid-wave: re-dispatch incomplete tasks
   - If between waves: start next wave
   - If at a gate: re-present the gate
   - If in Phase 0-2: re-present the last question or gate

---

## Abort Protocol

When user invokes `/spire abort`:

1. Confirm: "This will archive the current Spire project. Are you sure?"
2. If confirmed:
   - Rename `.spire/` to `.spire-archived-{YYYY-MM-DD-HHmmss}/`
   - If team mode was active, clean up any active teams
3. Report: "Project archived. Start a new build with `/spire {description}`."
