---
name: spire
description: "Use when the user wants to build something end-to-end — software, research papers, analysis projects, documentation, workflows, or any multi-step creation. Triggers on: 'build me', 'create a', 'I want to make', 'let's build', 'write a thesis', 'create an analysis', '/spire'. NOT for small edits, bug fixes, or quick questions."
---

# Spire — Full-Lifecycle Project Builder

Spire takes an idea and builds it into a finished deliverable through 5 phases with 4 checkpoint gates. It works for any kind of project — software, research, analysis, documentation, or creative work. It combines its own build pipeline with smart delegation to existing skills.

## Project Type Detection

At the start of every project, determine the project type. This shapes how every phase behaves:

| Type | Examples | Build = | Verify = |
|------|----------|---------|----------|
| **software** | Apps, APIs, CLIs, scripts, pipelines | Write code, run tests | Tests pass, code runs |
| **research** | Thesis, analysis paper, literature review | Write sections, run analyses, create figures | Arguments sound, sources cited, data verified |
| **document** | Reports, memos, guides, playbooks | Write content, create diagrams/tables | Complete, accurate, well-structured |
| **workflow** | Automation, process design, pipelines | Create scripts/configs, write procedures | Process runs end-to-end, steps are clear |
| **hybrid** | Data analysis with code + report | Both code and written output | Both code works and narrative is coherent |

The type is recorded in `.spire/PROJECT.md` and referenced by all agents.

## Core Principles

1. **Checkpoint-driven**: Never proceed past a gate without user approval
2. **Fresh context per agent**: Every builder/reviewer gets a clean context window with just what it needs
3. **Wave-based parallelism**: Independent tasks execute simultaneously, dependent tasks wait
4. **Artifacts as source of truth**: All state lives in `.spire/` as human-readable markdown
5. **Delegate when possible**: Use existing skills (brainstorming, writing-plans, frontend-design) instead of reinventing
6. **Domain-agnostic**: Same phases work for code, research, analysis, or any creative work

## Phase State Machine

```
SCOPE ──GATE 1──▶ DESIGN ──GATE 2──▶ PLAN ──GATE 3──▶ BUILD ──GATE 4──▶ VERIFY & SHIP
```

---

## Phase 0: SCOPE

**Goal**: Understand what to create, decide execution mode.

**Steps**:
1. **Detect project type** from the user's description (software / research / document / workflow / hybrid).

2. If working with existing material, dispatch `spire-analyst` agent to explore:
   - **Software**: Read CLAUDE.md, package.json/pyproject.toml, directory structure, conventions
   - **Research/Document**: Read existing drafts, data sources, reference materials, style guides
   - **Workflow**: Read existing processes, tools, configurations

3. If the user's request is genuinely unclear or exploratory, delegate to `superpowers:brainstorming` skill. Ingest the resulting spec as input for Phase 1.

4. Otherwise, ask 2-4 clarifying questions (one at a time, using AskUserQuestion). Adapt questions to project type:
   - **All types**: What is the core goal? Who is the audience? Any constraints?
   - **Software**: Tech stack preference? What consumes the output?
   - **Research**: What's the thesis/question? What data sources? What methodology?
   - **Document**: What format? What structure (chapters, sections)? Any templates?
   - **Workflow**: What triggers it? What are the inputs/outputs? What tools are involved?

5. Determine execution mode:
   - **Solo mode** (default): 1-3 independent work streams, no inter-agent coordination needed
   - **Team mode**: 4+ parallel streams OR agents need to coordinate on shared deliverables

6. Write `.spire/PROJECT.md`:
   ```markdown
   # Project: {name}

   ## Type
   {software | research | document | workflow | hybrid}

   ## Description
   {what we're creating, 2-3 sentences}

   ## Requirements
   - REQ-1: {requirement}
   - REQ-2: {requirement}
   ...

   ## Constraints
   - {format, tools, audience, tech stack, deadlines, etc.}

   ## Execution Mode
   {solo | team} — {reasoning}

   ## Existing Context
   {if applicable: existing work, data sources, codebase, references}
   ```

7. Present **GATE 1**:
   ```
   ╔══════════════════════════════════════════════╗
   ║  SPIRE GATE 1: Scope Confirmation           ║
   ╠══════════════════════════════════════════════╣
   ║  Project: {name}                            ║
   ║  Type: {project type}                       ║
   ║  Mode: {Solo | Team}                        ║
   ║                                             ║
   ║  Requirements: {count} items                ║
   ║  Key decisions:                             ║
   ║  • {decision 1}                             ║
   ║  • {decision 2}                             ║
   ╚══════════════════════════════════════════════╝
   ```
   Options: `[Approve] [Adjust scope] [Abort]`

---

## Phase 1: DESIGN

**Goal**: Define the structure and approach.

**Steps**:
1. Dispatch `spire-architect` agent with PROJECT.md content. The architect must:
   - Propose 2-3 approaches with tradeoffs
   - Recommend one approach with reasoning
   - Adapt the design to the project type:
     - **Software**: Component breakdown, data flow, file structure, tech decisions
     - **Research**: Thesis structure, chapter outline, methodology framework, analysis plan
     - **Document**: Document structure, sections, visual/data elements, narrative arc
     - **Workflow**: Process steps, decision points, tooling, automation opportunities

2. Present approaches to user. User picks one.

3. For projects with visual/UI elements, delegate to `frontend-design:frontend-design` skill.

4. Write `.spire/DESIGN.md`:
   ```markdown
   # Design: {project name}

   ## Project Type
   {type}

   ## Chosen Approach
   {approach name and summary}

   ## Structure
   {adapted to project type — architecture diagram, document outline, or process flow}

   ## Components / Sections
   ### {Component/Section 1}
   - Purpose: {what it covers}
   - Deliverables: {files, sections, or outputs}
   - Dependencies: {what it depends on}

   ### {Component/Section 2}
   ...

   ## Flow
   {how data/content/logic flows through the project}

   ## Deliverable Structure
   ```
   {planned directory tree or document outline}
   ```

   ## Tools & Stack
   - {language/framework/tool}: {why}
   ```

5. Present **GATE 2**:
   ```
   ╔══════════════════════════════════════════════╗
   ║  SPIRE GATE 2: Design Approval              ║
   ╠══════════════════════════════════════════════╣
   ║  Approach: {chosen approach}                ║
   ║  Components/Sections: {count}               ║
   ║  Deliverables: {count} planned              ║
   ║                                             ║
   ║  Design saved to: .spire/DESIGN.md          ║
   ╚══════════════════════════════════════════════╝
   ```
   Options: `[Approve] [Revise design] [Back to scope] [Abort]`

---

## Phase 2: PLAN

**Goal**: Create executable task plan with wave assignments.

**Steps**:
1. Read DESIGN.md. Decompose into concrete tasks. Each task must have:
   - Clear title and description
   - Exact deliverables (file paths, sections, outputs)
   - Content guidance or implementation spec (what to write/build/create)
   - Acceptance criteria (verifiable, not subjective)
   - Dependencies (which other tasks must complete first)
   - Complexity estimate: simple | moderate | complex

2. Dependency analysis and wave grouping:
   - Wave 1: tasks with zero dependencies (all execute in parallel)
   - Wave 2: tasks depending only on Wave 1 outputs
   - Wave N: tasks depending on Waves 1 through N-1
   - Detect and reject circular dependencies

3. For single-wave projects (all tasks independent), consider delegating directly to `superpowers:writing-plans` for the plan format.

4. Plan validation (max 3 iterations):
   - Check: every deliverable in DESIGN.md is covered by at least one task
   - Check: no task references outputs from a same-wave task (dependency error)
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
   - **Deliverables**: {files/sections to create or modify}
   - **Description**: {what to do}
   - **Acceptance**: {verifiable criteria}

   ### TASK-002: {title}
   - **Wave**: 1
   - **Complexity**: moderate
   - **Dependencies**: none
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
   ╔══════════════════════════════════════════════╗
   ║  SPIRE GATE 3: Build Plan Approval          ║
   ╠══════════════════════════════════════════════╣
   ║  Tasks: {count} in {N} waves                ║
   ║                                             ║
   ║  Wave 1: {count} tasks (parallel)           ║
   ║  Wave 2: {count} tasks (parallel)           ║
   ║  ...                                        ║
   ║                                             ║
   ║  Plan: .spire/PLAN.md                       ║
   ║  Waves: .spire/WAVES.md                     ║
   ╚══════════════════════════════════════════════╝
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
type: {project type}
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
     prompt: "{task spec from PLAN.md}\n\nProject type: {type}\n\nContext:\n{relevant sections of DESIGN.md}\n\nExisting material to read first: {list}\n\nAcceptance criteria: {criteria}",
     run_in_background: true,
     name: "builder-{TASK-ID}"
   )
   ```
2. Wait for all builders to complete.

#### Team Mode Execution:
1. Create team: `TeamCreate(team_name: "spire-wave-{N}")`
2. Create tasks: `TaskCreate(subject: "TASK-{ID}: {title}")` for each task
3. Dispatch builders: `Agent(team_name: "spire-wave-{N}", name: "builder-{ID}", ...)`
4. Builders can coordinate via `SendMessage` if needed

#### After Each Wave:
1. Dispatch `spire-reviewer` agent to check:
   - Does the output match the task spec in PLAN.md?
   - Are acceptance criteria met?
   - Quality check adapted to project type:
     - **Software**: Bugs, security issues, style violations
     - **Research**: Argument soundness, citation accuracy, logical flow
     - **Document**: Completeness, clarity, consistency, formatting
     - **Workflow**: Process completeness, edge cases, error handling
2. If issues found → dispatch fix agents (max 2 correction rounds per task)
3. Dispatch `spire-integrator` agent to:
   - Check for conflicts between parallel task outputs
   - Verify cross-references and dependencies between deliverables
   - Run tests (software) or check consistency (research/documents)
4. Update `.spire/STATE.md`: mark wave tasks as complete/failed
5. If team mode: `TeamDelete(team_name: "spire-wave-{N}")`

#### Error Handling:
- **Builder reports BLOCKED**: Assess. If context issue, re-dispatch with more info. If plan issue, escalate to user.
- **Reviewer finds MAJOR_ISSUES**: Dispatch fix, max 2 rounds. If still broken after 2 rounds, pause and present emergency gate to user.
- **Integrator finds conflicts**: Attempt auto-resolution. If ambiguous, ask user.

After all waves complete → proceed to Phase 4.

---

## Phase 4: VERIFY & SHIP

**Goal**: Validate the project meets requirements, deliver.

**Steps**:
1. Dispatch `spire-verifier` agent, adapted to project type:
   - **Software**: Run tests, linting, type checks. Smoke test key functionality.
   - **Research**: Verify citations, check data accuracy, validate arguments, check methodology.
   - **Document**: Check completeness, cross-references, formatting, consistency.
   - **Workflow**: Run the process end-to-end, verify each step works.
   - **All types**: Cross-check each requirement in PROJECT.md against deliverables.

2. If gaps found:
   - Create mini gap-closure plan (same format as Phase 2 tasks)
   - Execute gap-closure tasks (single wave, same execution model)
   - Re-verify

3. Write `.spire/COMPLETE.md`:
   ```markdown
   # Completion Report

   ## Requirements Status
   - REQ-1: PASS
   - REQ-2: PASS
   - REQ-3: FIXED (gap-closure round 1)

   ## Quality Check
   {adapted to project type — test results, review findings, consistency check}

   ## Deliverables Created/Modified
   {list of all files and outputs}

   ## Known Limitations
   {any caveats or future work}
   ```

4. Present **GATE 4**:
   ```
   ╔══════════════════════════════════════════════╗
   ║  SPIRE GATE 4: Delivery                     ║
   ╠══════════════════════════════════════════════╣
   ║  Requirements: {met}/{total} met            ║
   ║  Quality: {summary}                         ║
   ║  Deliverables: {count} created/modified     ║
   ║                                             ║
   ║  Report: .spire/COMPLETE.md                 ║
   ╚══════════════════════════════════════════════╝
   ```
   Options:
   - `[Commit & PR]` — delegates to `commit-commands:commit-push-pr` (if git project)
   - `[Export / Deliver]` — package deliverables for sharing
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
| Presentation/deck needed | `office-pptx` or `presentation-designer` | Invoke for slide creation |
| Document creation (DOCX/PDF) | `office-docx` or `office-pdf` | Invoke for document formatting |
| Financial analysis/modeling | `financial-analysis:*` skills | Delegate specific financial tasks |
| Final commit and PR | `commit-commands:commit-push-pr` | Invoke at Gate 4 |
| Git worktree isolation needed | `superpowers:using-git-worktrees` | Use before Phase 3 |

**Spire's unique contributions** (not delegated):
- Wave dependency analysis and grouping
- Multi-wave execution with inter-wave integration
- The `.spire/` artifact system
- Mode selection (solo vs team)
- The four-gate checkpoint protocol
- Gap-closure verification loop
- Project type detection and adaptive behavior

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
