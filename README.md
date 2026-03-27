# Spire

Full-lifecycle project builder plugin for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Takes an idea and builds it into working software through checkpoint-driven autonomy, wave-based parallel execution, and smart delegation to existing skills.

Inspired by [karpathy/autoresearch](https://github.com/karpathy/autoresearch), [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done), and Claude Code's TeamCreate system.

## How It Works

Spire runs a 5-phase pipeline with 4 checkpoint gates where you approve before continuing:

```
SCOPE ──GATE 1──> DESIGN ──GATE 2──> PLAN ──GATE 3──> BUILD ──GATE 4──> VERIFY & SHIP
```

1. **Scope** — Understand what to build, explore existing codebase, decide execution mode
2. **Design** — Architect proposes 2-3 approaches with tradeoffs, you pick one
3. **Plan** — Decompose into tasks, group into dependency waves for parallel execution
4. **Build** — Wave-based parallel execution with review and integration after each wave
5. **Verify & Ship** — Run tests, cross-check requirements, deliver

## Key Features

- **Wave-based parallel execution** — Independent tasks run simultaneously, dependent tasks wait. Inspired by get-shit-done's wave engine.
- **Fresh context per agent** — Every builder gets a clean context window with just its task. No context rot.
- **Checkpoint gates** — Nothing happens without your approval. You stay in control.
- **Smart delegation** — Uses existing skills (brainstorming, writing-plans, frontend-design) when they fit, own agents when they don't.
- **Two execution modes** — Solo mode (subagents) for focused builds, Team mode (TeamCreate) for complex multi-stream projects.
- **Resumable state** — All state lives in `.spire/` as markdown. Close Claude Code, come back tomorrow, `/spire resume`.

## 6 Specialist Agents

| Agent | Role |
|-------|------|
| `spire-analyst` | Explores codebase, identifies constraints and patterns |
| `spire-architect` | Designs architecture, proposes approaches with tradeoffs |
| `spire-builder` | Implements individual tasks with fresh context |
| `spire-reviewer` | Checks spec compliance and code quality after each wave |
| `spire-verifier` | Runs tests, validates requirements are met |
| `spire-integrator` | Merges parallel wave outputs, resolves conflicts |

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
├── PROJECT.md      # What we're building
├── DESIGN.md       # How we're building it
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

After each wave: review, fix issues (max 2 rounds), integrate, then next wave.

## Delegation

Spire delegates to existing Claude Code skills when they handle the job better:

| Situation | Delegates to |
|-----------|-------------|
| Unclear requirements | `superpowers:brainstorming` |
| Single-wave project | `superpowers:writing-plans` + `executing-plans` |
| UI/frontend design | `frontend-design:frontend-design` |
| Complex parallel builds | TeamCreate (`launch-team` pattern) |
| Commit and PR | `commit-commands:commit-push-pr` |

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or desktop app
- For delegation features: relevant Claude Code plugins installed (superpowers, frontend-design, etc.)

## License

MIT
