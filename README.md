# DIF — Do It Fast

DIF is a Claude Code plugin that provides a **developer-only** agentic workflow, scoped strictly to implementation rather than planning ceremony. It's inspired by BMAD-style multi-agent orchestrators, but deliberately narrower: one role (developer), no story/ticket ID system, and hard human-in-the-loop (HIL) gates that never proceed without your explicit approval in that turn. It's size-adaptive — a one-line fix and a multi-file feature don't pay the same ceremony cost — and it keeps a living architecture/flow/functional doc set inside your project, generated once and updated incrementally as you go.

DIF is **not** a general-purpose agent orchestration framework, not a replacement for planning tools/ticket systems/CI, and doesn't attempt cross-repo orchestration or default to parallel execution.

New to this repo? Start with [SETUP.md](SETUP.md).

## Pipeline

Every requirement is classified into a **track**, and the track determines how much of the pipeline runs:

| Track | Criteria | Pipeline |
|---|---|---|
| `trivial` | Single file, no new deps, no new flow | requirement → **1 combined HIL gate** → execute → complete |
| `standard` | Few files, one existing flow/module | requirement → HIL → plan → HIL → execute → consolidate → HIL → complete |
| `complex` | Cross-module, new flow, or structural change | full pipeline, plus a forced `architecture.md` review at completion |

```
                      ┌────────────┐
                      │  explore   │  (optional, any time — maps the codebase)
                      └─────┬──────┘
                            │
                      ┌─────▼──────┐
                      │ requirement│  writes requirement.md, classifies track
                      └─────┬──────┘
                            │  ●── HIL gate ──●  (combined gate on trivial track)
                      ┌─────▼──────┐
             ┌────────┤    plan    │  writes task-plan.md   (skipped on trivial)
             │        └─────┬──────┘
             │              │  ●── HIL gate ──●
             │        ┌─────▼──────┐
             └───────►│  execute   │  writes code, per-task outputs
                      └─────┬──────┘
                            │
                      ┌─────▼──────┐
                      │ consolidate│  merges task outputs   (skipped on trivial)
                      └─────┬──────┘
                            │  ●── HIL gate ──●
                      ┌─────▼──────┐
                      │  complete  │  updates docs, archives to history/, resets state
                      └────────────┘
```

Each phase is its own slash command, and — outside of `/dif:run` (below) — never chained inside one prompt, so there's no code path that lets DIF "continue" past a gate without you invoking the next command yourself.

## Commands

| Command | Behavior |
|---|---|
| `/dif:explore` | Generate/refresh `architecture.md`; report doc coverage. `--refresh` forces full regen. |
| `/dif:requirement "<text>"` | Intake a requirement, classify its track, write `requirement.md`, stop for approval. |
| `/dif:plan` | Produce `task-plan.md` with a dependency graph, stop for approval. |
| `/dif:execute` | Execute the approved plan (or, on `trivial`, the approved requirement), sequential by default. |
| `/dif:consolidate` | Merge task outputs into `consolidated-diff.md`, stop for validation. |
| `/dif:complete` | Incrementally update docs, archive to `history/`, clear `active/`, close state. |
| `/dif:status` | Print current phase, track, approval flags, and `run_mode`. Read-only. |
| `/dif:run "<text>"` | Run the whole pipeline, auto-continuing to the next phase immediately after each gate is approved — no gate is skipped, you just don't retype the next command yourself. Say "pause" in an approval reply to drop back to step-by-step at any point. |

`/dif:run` is a convenience wrapper, not a separate pipeline: it sets `run_mode: true` on the cycle and then hands off to the exact same `requirement`/`plan`/`execute`/`consolidate`/`complete` skills the individual commands use. See [`skills/run/SKILL.md`](skills/run/SKILL.md) for exactly what it does and doesn't change about the HIL guarantee.

## Quick example — `standard` track

```
/dif:explore
/dif:requirement "Add a rate limiter to the /api/upload endpoint, 10 req/min per API key."
   → DIF drafts requirement.md, classifies it "standard", presents it — you review and reply "approved"
/dif:plan
   → DIF drafts task-plan.md (e.g. 3 tasks: middleware, config, tests) — you review and reply "approved"
/dif:execute
   → DIF implements each task in order, writes task-outputs/
/dif:consolidate
   → DIF merges the outputs into consolidated-diff.md — you review and reply "approved"
/dif:complete
   → DIF updates the affected flow doc, archives the cycle, resets state
```

The same cycle, auto-chained so you only type `/dif:run` once and then just reply "approved" at each gate:

```
/dif:run "Add a rate limiter to the /api/upload endpoint, 10 req/min per API key."
   → requirement gate → approved → plan gate → approved → execute → consolidate gate → approved → complete
```

## Where things live

- **This plugin**: `.claude-plugin/`, `commands/`, `skills/` — shipped, not project-specific.
- **Per-project runtime state**: `.claude/dif/` inside whatever project you run DIF in — created on first use, holds the active cycle, generated docs, and history. See [SETUP.md](SETUP.md) for the full layout and how to customize or reset it.

## License

MIT — see [LICENSE](LICENSE).
