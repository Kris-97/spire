# Spire

Full-lifecycle project builder plugin for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Takes an idea and builds it into a finished deliverable — software, research, analysis, documentation, workflows, or anything else — through checkpoint-driven autonomy, wave-based parallel execution, and smart delegation to existing skills.

## Why Spire Exists

AI coding assistants are powerful, but they have a fundamental problem: they degrade. The longer a conversation runs, the more context accumulates, the more the model forgets its original instructions, and the worse the output gets. This is **context rot** — and it's why ambitious projects built in a single Claude session tend to start strong and end sloppy.

Three projects attacked this problem from different angles, and Spire takes the best ideas from each:

**[Andrej Karpathy's autoresearch](https://github.com/karpathy/autoresearch)** showed that AI agents can run autonomous optimization loops overnight — modify code, evaluate, keep what works, discard what doesn't. The human's job shifts from writing code to writing *strategy*. The insight: **stop micromanaging agents and start directing them with clear objectives and evaluation criteria.**

**[Get Shit Done (GSD)](https://github.com/gsd-build/get-shit-done)** solved context rot head-on by giving each agent a fresh 200K-token context window with only the information it needs. No accumulated conversation history, no degradation. It also introduced **wave-based parallel execution** — grouping tasks by dependency level so independent work runs simultaneously while dependent work waits. And it proved that project state belongs in human-readable files on disk, not trapped in a conversation that will eventually be forgotten.

**Claude Code's TeamCreate system** added something neither of the above had: **inter-agent communication**. Agents in a team can message each other, coordinate on shared interfaces, and work on the same project without stepping on each other's toes.

### What's actually new here

Spire isn't just a remix. The combination creates capabilities none of the sources have individually:

- **Domain-agnostic building.** Autoresearch optimizes ML models. GSD builds software. Spire builds *anything* — the same pipeline adapts to code, research, analysis, documents, or workflows. The project type is detected at the start and every agent adjusts its behavior accordingly.

- **Checkpoint-driven autonomy.** Autoresearch runs fully autonomous (no human approval loops). GSD has checkpoints but they're software-focused. Spire's four gates are a deliberate design choice: the human stays in the loop at every phase boundary, but the agents run free within each phase. This is the practical middle ground between "let the AI do everything" and "approve every line."

- **Smart delegation to existing skills.** Neither autoresearch nor GSD knew about the user's existing toolchain. Spire checks a delegation table before each step — if an existing skill (brainstorming, writing-plans, frontend-design, office-docx) handles the task better, Spire delegates instead of reinventing. This means Spire gets better as your skill library grows, without Spire itself needing to change.

### Honest limitations

Spire is an orchestration layer, not magic. Some things to be realistic about:

- **Quality depends on the underlying model.** Spire can structure the work perfectly, but if the builder agent writes bad code or weak analysis, the reviewer can only catch so much. Wave-based execution with fresh context *helps* — it reduces the context rot problem that causes late-session quality drops — but it doesn't eliminate the model's inherent limitations.

- **Checkpoint gates add friction.** Four approval gates mean four interruptions. For a quick script, that's overkill. Spire is designed for projects substantial enough to benefit from the structure — if you can describe the whole thing in one sentence and build it in five minutes, you probably don't need Spire.

- **The .spire/ artifact system is only as good as what's written to it.** If Phase 0 captures the wrong requirements, every downstream phase builds on a flawed foundation. The gates are designed to catch this (you review PROJECT.md before proceeding), but they require you to actually read the artifacts, not just click approve.

- **Parallel execution has coordination costs.** Running 4 builders simultaneously is faster than running them sequentially, but the integration step afterwards can surface conflicts. The spire-integrator handles most of these automatically, but complex cross-cutting concerns may require your input.

The thesis behind Spire is simple: **the hard part of building things isn't writing — it's organizing the work so that writing happens in the right order, with the right context, and gets verified against the right criteria.** That's what Spire does.

## How It Works

Spire runs a 5-phase pipeline with 4 checkpoint gates where you approve before continuing:

```
SCOPE ──GATE 1──> DESIGN ──GATE 2──> PLAN ──GATE 3──> BUILD ──GATE 4──> VERIFY & SHIP
```

1. **Scope** — Understand what to create, detect project type, decide execution mode
2. **Design** — Architect proposes 2-3 approaches with tradeoffs, you pick one
3. **Plan** — Decompose into tasks, group into dependency waves for parallel execution
4. **Build** — Wave-based parallel execution with review and integration after each wave
5. **Verify & Ship** — Validate requirements are met, deliver

## What Can You Build?

| Project Type | Examples |
|-------------|----------|
| **Software** | Apps, APIs, CLIs, scripts, data pipelines |
| **Research** | Thesis, analysis papers, literature reviews, due diligence |
| **Documents** | Reports, memos, guides, playbooks, presentations |
| **Workflows** | Automation, process design, operational procedures |
| **Hybrid** | Data analysis with code + written report |

Spire detects the project type and adapts every phase — how it designs, what it builds, and how it verifies.

## Key Features

- **Domain-agnostic** — Same 5-phase pipeline works for code, research, writing, or anything else. Agents adapt their behavior to the project type.
- **Wave-based parallel execution** — Independent tasks run simultaneously, dependent tasks wait.
- **Fresh context per agent** — Every builder gets a clean context window with just its task. No context rot.
- **Checkpoint gates** — Nothing happens without your approval. You stay in control.
- **Smart delegation** — Uses existing skills (brainstorming, writing-plans, frontend-design, office-docx) when they fit, own agents when they don't.
- **Two execution modes** — Solo mode (subagents) for focused builds, Team mode (TeamCreate) for complex multi-stream projects.
- **Resumable state** — All state lives in `.spire/` as markdown. Close Claude Code, come back tomorrow, `/spire resume`.

## 6 Specialist Agents

| Agent | Role |
|-------|------|
| `spire-analyst` | Explores existing material — code, data, documents, references |
| `spire-architect` | Designs project structure, proposes approaches with tradeoffs |
| `spire-builder` | Executes individual tasks — writes code, analysis, content, whatever the task requires |
| `spire-reviewer` | Checks spec compliance and output quality after each wave |
| `spire-verifier` | Validates requirements are met — runs tests, checks accuracy, verifies completeness |
| `spire-integrator` | Merges parallel wave outputs, resolves conflicts, ensures coherence |

## Installation

Clone into your Claude Code plugins directory:

```bash
# macOS / Linux
git clone https://github.com/Kris-97/spire.git ~/.claude/plugins/spire

# Windows
git clone https://github.com/Kris-97/spire.git %USERPROFILE%\.claude\plugins\spire
```

Restart Claude Code to load the plugin.

## Usage

```
/spire build a REST API for managing book reviews
/spire write an analysis of Nordic PE market trends
/spire create a due diligence workflow for co-investments
/spire build a data pipeline that processes quarterly reports
```

Other commands:

```
/spire resume     # Resume a paused project
/spire status     # Show current project state
/spire abort      # Archive and start fresh
```

## Project State (`.spire/`)

Spire creates a `.spire/` directory in your project root with all state as human-readable markdown:

```
.spire/
├── PROJECT.md      # What we're creating (includes project type)
├── DESIGN.md       # How we're creating it
├── PLAN.md         # Task-level plan with acceptance criteria
├── WAVES.md        # Wave breakdown with dependency graph
├── STATE.md        # Execution state, updated per wave
├── COMPLETE.md     # Completion report
└── history/        # Gate decisions and wave logs
```

## Wave Execution

Tasks are grouped into waves by dependency analysis:

- **Wave 1**: Tasks with no dependencies — all run in parallel
- **Wave 2**: Tasks that depend on Wave 1 — run after Wave 1 completes
- **Wave N**: Tasks depending on Waves 1 through N-1

This works for any project type — parallel code modules, independent research sections, concurrent document chapters, or simultaneous workflow steps.

After each wave: review, fix issues (max 2 rounds), integrate, then next wave.

## Delegation

Spire delegates to existing Claude Code skills when they handle the job better:

| Situation | Delegates to |
|-----------|-------------|
| Unclear requirements | `superpowers:brainstorming` |
| Single-wave project | `superpowers:writing-plans` + `executing-plans` |
| UI/frontend design | `frontend-design:frontend-design` |
| Complex parallel builds | TeamCreate (`launch-team` pattern) |
| Document creation | `office-docx`, `office-pdf` |
| Presentations | `office-pptx`, `presentation-designer` |
| Spreadsheets / models | `office-xlsx`, `excel-pipeline` |
| Commit and PR | `commit-commands:commit-push-pr` |

## What Happens When You Run `/spire`

Here's the full flow you can expect, shown with two examples — a software project and a research project.

---

### Example 1: `/spire build a CLI tool that converts CSV to JSON`

**Phase 0 — Scope** (you answer 2-3 questions)
```
Spire: What's the core functionality?
You:   Read CSV, output JSON. Support stdin and file input.

Spire: Any constraints?
You:   Python, no external dependencies.

╔══════════════════════════════════════════════╗
║  SPIRE GATE 1: Scope Confirmation           ║
╠══════════════════════════════════════════════╣
║  Project: csv-to-json                       ║
║  Type: software                             ║
║  Mode: Solo                                 ║
║  Requirements: 4 items                      ║
╚══════════════════════════════════════════════╝
→ You approve
```

**Phase 1 — Design** (architect proposes approaches)
```
Spire Architect proposes:
  A) Single-file script — simple, no structure
  B) CLI with argparse + converter module — clean separation
  C) Package with setup.py — installable via pip

Recommends B. You pick B.

╔══════════════════════════════════════════════╗
║  SPIRE GATE 2: Design Approval              ║
║  Approach: CLI + converter module            ║
║  Components: 3 (cli.py, converter.py, tests) ║
╚══════════════════════════════════════════════╝
→ You approve
```

**Phase 2 — Plan** (tasks grouped into waves)
```
╔══════════════════════════════════════════════╗
║  SPIRE GATE 3: Build Plan Approval          ║
║  Tasks: 4 in 2 waves                        ║
║                                             ║
║  Wave 1: converter module + CLI parser      ║
║  Wave 2: integration + tests                ║
╚══════════════════════════════════════════════╝
→ You approve
```

**Phase 3 — Build** (agents work in parallel)
```
Wave 1: Two builders launch simultaneously
  → builder-TASK-001: writes converter.py    ✓ DONE
  → builder-TASK-002: writes cli.py          ✓ DONE
  → reviewer checks both                     ✓ APPROVED
  → integrator verifies imports work         ✓ CLEAN

Wave 2: Tests and integration
  → builder-TASK-003: writes tests           ✓ DONE
  → builder-TASK-004: adds error handling    ✓ DONE
  → reviewer + integrator                    ✓ APPROVED
```

**Phase 4 — Verify & Ship**
```
Verifier runs: pytest → 8/8 passing
Requirements: 4/4 met

╔══════════════════════════════════════════════╗
║  SPIRE GATE 4: Delivery                     ║
║  Requirements: 4/4 met                      ║
║  Tests: 8/8 passing                         ║
║  Files: 5 created                           ║
╚══════════════════════════════════════════════╝
→ You choose: Commit & PR
```

---

### Example 2: `/spire write an analysis of Nordic infrastructure fund performance`

**Phase 0 — Scope** (adapted questions for research)
```
Spire: What's the research question?
You:   How have Nordic infra funds performed vs. European benchmarks?

Spire: What data sources are available?
You:   Preqin data exports, annual reports from EIF and Argentum.

Spire: Who's the audience?
You:   Internal IC, so professional but not academic.

╔══════════════════════════════════════════════╗
║  SPIRE GATE 1: Scope Confirmation           ║
║  Project: Nordic infra performance analysis  ║
║  Type: research                             ║
║  Mode: Solo                                 ║
║  Requirements: 5 items                      ║
╚══════════════════════════════════════════════╝
→ You approve
```

**Phase 1 — Design** (architect proposes thesis structures)
```
Spire Architect proposes:
  A) Benchmark comparison — straight performance vs. index
  B) Factor decomposition — break returns into vintage, strategy, geography
  C) Case study approach — deep dive on 3-4 funds

Recommends B. You pick B.

╔══════════════════════════════════════════════╗
║  SPIRE GATE 2: Design Approval              ║
║  Approach: Factor decomposition              ║
║  Sections: 6 (intro, data, methodology,      ║
║    vintage analysis, strategy analysis,       ║
║    conclusions)                               ║
╚══════════════════════════════════════════════╝
→ You approve
```

**Phase 2 — Plan** (research tasks in waves)
```
╔══════════════════════════════════════════════╗
║  SPIRE GATE 3: Build Plan Approval          ║
║  Tasks: 7 in 3 waves                        ║
║                                             ║
║  Wave 1: Data processing + lit review       ║
║  Wave 2: Vintage analysis + strategy analysis║
║  Wave 3: Synthesis + conclusions + intro     ║
╚══════════════════════════════════════════════╝
→ You approve
```

**Phase 3 — Build** (research tasks execute)
```
Wave 1: Foundation work (parallel)
  → builder-TASK-001: processes Preqin data   ✓ DONE
  → builder-TASK-002: writes lit review       ✓ DONE
  → builder-TASK-003: creates methodology     ✓ DONE
  → reviewer: checks data handling, sources   ✓ APPROVED

Wave 2: Analysis sections (parallel, depend on Wave 1 data)
  → builder-TASK-004: vintage year analysis   ✓ DONE
  → builder-TASK-005: strategy breakdown      ✓ DONE
  → reviewer: checks argument soundness       ✓ ISSUES_FOUND
  → fix agent: adds missing benchmark data    ✓ DONE (round 1)
  → integrator: checks cross-references       ✓ RESOLVED

Wave 3: Synthesis (depends on Waves 1-2)
  → builder-TASK-006: conclusions             ✓ DONE
  → builder-TASK-007: introduction + exec sum ✓ DONE
  → integrator: checks narrative coherence    ✓ CLEAN
```

**Phase 4 — Verify & Ship**
```
Verifier checks:
  - All 5 requirements addressed              ✓
  - Data citations accurate                   ✓
  - Methodology applied consistently          ✓
  - No unsupported claims                     ✓
  - Sections complete (no stubs)              ✓

╔══════════════════════════════════════════════╗
║  SPIRE GATE 4: Delivery                     ║
║  Requirements: 5/5 met                      ║
║  Quality: all checks passed                 ║
║  Deliverables: 8 files created              ║
╚══════════════════════════════════════════════╝
→ You choose: Done
```

---

### Key Things to Know

- **You're always in control.** Nothing advances without your approval at each gate.
- **You can go back.** Every gate offers "Back to previous phase" if something isn't right.
- **You can pause anytime.** State is saved to `.spire/`. Come back later with `/spire resume`.
- **Agents work in parallel.** Independent tasks in each wave run simultaneously — you don't wait for sequential execution.
- **Issues get fixed automatically.** If a reviewer finds problems, fix agents are dispatched (max 2 rounds) before escalating to you.
- **It adapts to what you're building.** The same pipeline works whether you're writing code, research, documents, or workflows.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or desktop app
- For delegation features: relevant Claude Code plugins installed (superpowers, frontend-design, office skills, etc.)

## License

MIT
