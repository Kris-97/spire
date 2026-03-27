# Spire Phase Definitions

## Phase Transitions

Each phase has:
- **Entry condition**: what must be true to enter this phase
- **Exit condition**: what must be true to leave (always a gate approval)
- **Rollback**: what happens if the user wants to go back

## Phase 0: SCOPE

| Property | Value |
|----------|-------|
| Entry | User invokes `/spire {description}` |
| Exit | Gate 1 approved |
| Rollback | N/A (this is the start) |
| Agents | `spire-analyst` (if existing codebase) |
| Artifacts | `.spire/PROJECT.md` |
| Delegation | `superpowers:brainstorming` if exploratory |

**Key decisions made here**:
- What are we building?
- Solo mode or team mode?
- New project or feature in existing codebase?

## Phase 1: DESIGN

| Property | Value |
|----------|-------|
| Entry | Gate 1 approved, PROJECT.md exists |
| Exit | Gate 2 approved |
| Rollback | User can go "Back to scope" at Gate 2 |
| Agents | `spire-architect` |
| Artifacts | `.spire/DESIGN.md` |
| Delegation | `frontend-design` for UI components |

**Key decisions made here**:
- Which architectural approach?
- Component breakdown and responsibilities
- Tech stack and file structure

## Phase 2: PLAN

| Property | Value |
|----------|-------|
| Entry | Gate 2 approved, DESIGN.md exists |
| Exit | Gate 3 approved |
| Rollback | User can go "Back to design" at Gate 3 |
| Agents | None (orchestrator handles planning) |
| Artifacts | `.spire/PLAN.md`, `.spire/WAVES.md` |
| Delegation | `superpowers:writing-plans` for single-wave projects |

**Key decisions made here**:
- Task decomposition and ordering
- Wave assignments
- Complexity estimates (affects model selection for builders)

## Phase 3: BUILD

| Property | Value |
|----------|-------|
| Entry | Gate 3 approved, PLAN.md and WAVES.md exist |
| Exit | All waves complete |
| Rollback | Can pause and resume via `/spire resume` |
| Agents | `spire-builder`, `spire-reviewer`, `spire-integrator` |
| Artifacts | `.spire/STATE.md` (continuously updated), project source files |
| Delegation | TeamCreate for team mode |

**Special behaviors**:
- This is the only phase where project source files are created/modified
- STATE.md is the checkpoint — enables resume after interruption
- Emergency gate if 2 correction rounds fail for any task

## Phase 4: VERIFY & SHIP

| Property | Value |
|----------|-------|
| Entry | All waves complete |
| Exit | Gate 4 — user chooses delivery option |
| Rollback | "Keep working" loops back to a mini Plan phase |
| Agents | `spire-verifier` |
| Artifacts | `.spire/COMPLETE.md` |
| Delegation | `commit-commands:commit-push-pr` for delivery |

**Special behaviors**:
- Gap-closure loop: if requirements aren't met, create mini-plan and execute
- Max 2 gap-closure rounds before escalating to user
