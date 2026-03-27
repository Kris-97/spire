# Spire Gate Protocol

## Purpose

Gates are mandatory checkpoints between phases. No phase transition happens without explicit user approval. This prevents misaligned builds and gives the user control without requiring constant supervision.

## Gate Presentation Format

All gates use this visual format:

```
╔══════════════════════════════════════════════╗
║  SPIRE GATE {N}: {title}                    ║
╠══════════════════════════════════════════════╣
║  {key information, 3-5 lines}               ║
║                                             ║
║  {artifact references}                      ║
╚══════════════════════════════════════════════╝
```

Followed by options presented via `AskUserQuestion` with clear choices.

## Gate Definitions

### Gate 1: Scope Confirmation

**When**: End of Phase 0
**Approves**: What we're building and how (solo vs team mode)
**Artifact to review**: `.spire/PROJECT.md`

Options:
- **Approve** → proceed to Phase 1 (Design)
- **Adjust scope** → re-enter Phase 0, modify requirements
- **Abort** → cancel the build

### Gate 2: Design Approval

**When**: End of Phase 1
**Approves**: Architecture, component breakdown, tech stack
**Artifact to review**: `.spire/DESIGN.md`

Options:
- **Approve** → proceed to Phase 2 (Plan)
- **Revise design** → re-enter Phase 1, modify architecture
- **Back to scope** → return to Phase 0
- **Abort** → cancel the build

### Gate 3: Build Plan Approval

**When**: End of Phase 2
**Approves**: Task breakdown, wave assignments, execution order
**Artifacts to review**: `.spire/PLAN.md`, `.spire/WAVES.md`

Options:
- **Build** → proceed to Phase 3 (Build)
- **Revise plan** → re-enter Phase 2, modify tasks
- **Back to design** → return to Phase 1
- **Abort** → cancel the build

### Gate 4: Delivery

**When**: End of Phase 4
**Approves**: Final output, delivery method
**Artifact to review**: `.spire/COMPLETE.md`

Options:
- **Commit & PR** → create commit and pull request
- **Keep working** → identify gaps, loop back to planning
- **Save & pause** → preserve state for `/spire resume`
- **Done** → mark project complete, no further action

## Rules

1. **Never skip a gate** — even if the answer seems obvious
2. **Always present the artifact reference** — user should know where to look for details
3. **Use AskUserQuestion** — structured options, not open-ended questions
4. **Record gate decisions** — write to `.spire/history/gate-{N}.md` after each approval
5. **Allow backward movement** — users can always go back to a previous phase
6. **Abort is always an option** — never trap the user in a build they don't want

## Gate Decision Record Format

Written to `.spire/history/gate-{N}.md` after approval:

```markdown
# Gate {N}: {title}

- Decision: {option chosen}
- Timestamp: {ISO timestamp}
- Notes: {any user notes or modifications requested}
- Phase entering: {next phase}
```
