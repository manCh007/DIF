# SETUP

Standalone how-to for installing, running, and customizing DIF (Do It Fast). See [README.md](README.md) for the conceptual overview.

## 1. Prerequisites

- Claude Code installed (any recent version that supports the plugin system: `.claude-plugin/plugin.json`, `commands/`, `skills/`).
- A git repository or plain directory you want DIF to work in — DIF creates its runtime state inside whatever project you run its commands from, not inside the plugin itself.

## 2. Installation

Via a marketplace (once this repo is published):

```
/plugin marketplace add <owner>/dif
/plugin install dif@dif-marketplace
```

Local development / trying it before publishing anywhere:

```
claude --plugin-dir ./dif
```

(run from wherever you cloned this plugin; point `--plugin-dir` at this repo's root, the directory containing `.claude-plugin/`).

## 3. First run

The first time you run **any** `/dif:*` command inside a project, DIF creates `.claude/dif/` in that project (not in the plugin's own directory). The cleanest way to start is:

```
/dif:explore
```

This generates `.claude/dif/docs/architecture.md` — a high-level description of the project's topology, tech stack, module boundaries, and entry points — plus empty `flows/_index.md` and `functional/_index.md` tables (per-flow and per-module docs are generated lazily, only once a requirement actually touches that code, not upfront). You can skip straight to `/dif:requirement` too; it'll create the minimal state it needs on its own.

## 4. Walkthrough — one worked example per track

### Trivial: fixing a typo'd log level

```
/dif:requirement "Fix the log level in src/worker.py:42 — it's logging retries at ERROR, should be WARN"
```
DIF drafts `requirement.md`, classifies it `trivial`, and stops:

> **Requirement understanding:** ... **Track: trivial** (single file, no new deps, no new flow) — reply to override, e.g. "make this standard".
> This is the pipeline's single combined gate for the trivial track: approving here also approves going straight to execution, with no separate plan step.
> *Waiting for your approval.*

You reply `approved`. DIF sets `approved.requirement: true` and stops — it does **not** auto-continue.

```
/dif:execute
```
DIF makes the one-line fix, writes `task-outputs/task-1.md`.

```
/dif:complete
```
DIF checks whether `src/worker.py` maps to a documented flow/module; if so, and if the fix is doc-relevant, updates it — usually a no-op for a change this small — archives the cycle to `history/`, and resets state.

### Standard: adding a rate limiter to one endpoint

```
/dif:explore
/dif:requirement "Add a rate limiter to the /api/upload endpoint, 10 req/min per API key."
```
Track comes back `standard`. Gate, you reply `approved`.

```
/dif:plan
```
DIF reads the relevant flow (generating `flows/upload.md` now, lazily, since this is the first time it's touched), produces `task-plan.md` with tasks like `middleware`, `config`, `tests` and their dependencies. Gate:

> **Task plan:** 3 tasks — 1) add rate-limit middleware, 2) wire config for the per-key threshold, 3) add tests for both under- and over-limit. `tests` depends on `middleware` and `config`.
> *Waiting for your approval.*

You reply `approved`.

```
/dif:execute
```
Tasks run sequentially (`middleware` → `config` → `tests`, respecting the dependency), each writes its own `task-outputs/<task-id>.md`.

```
/dif:consolidate
```
DIF merges the three task outputs into `consolidated-diff.md`. Gate, you review, reply `approved`.

```
/dif:complete
```
DIF updates `flows/upload.md` (and only that doc — nothing else was touched), archives the cycle, resets state.

### Complex: introducing a new background job queue

Same shape as `standard`, classified `complex` because it's a new flow/structural change. The only difference lands at `/dif:complete`: DIF additionally re-evaluates whether `architecture.md` itself needs updating (new service/module/external dependency) — it updates it if warranted, and says so either way. `trivial`/`standard` cycles never touch `architecture.md` from `complete`.

### Same standard-track example, auto-chained with `/dif:run`

```
/dif:run "Add a rate limiter to the /api/upload endpoint, 10 req/min per API key."
```
DIF sets `run_mode: true`, drafts `requirement.md`, classifies `standard`, and stops at the requirement gate exactly as before. You reply `approved` — and instead of stopping there, DIF immediately drafts `task-plan.md` in the same reply and presents the plan gate. You reply `approved` again — DIF runs all three tasks, merges them into `consolidated-diff.md`, and presents the consolidate gate in that same reply. You reply `approved` a third time — DIF runs `/dif:complete`'s logic and reports the cycle is closed. Three approvals total, no other commands typed.

At any of those three gates, replying with something like "approved, but pause after this" instead of a bare "approved" drops back to step-by-step mode: that gate's approval still goes through, but DIF stops and waits for you to type the next `/dif:*` command yourself from there on.

## 5. Customization

- **Per-skill tuning**: each skill ships a near-empty `skills/<skill>/rules.md` with commented-out examples (track thresholds, doc-exclusion patterns, task-granularity preferences, the history-archive toggle, etc). Edit it directly in your copy of the plugin.
- **Project-level override without forking**: create a same-named file under your project's own `.claude/skills/<skill>/rules.md` (or `.claude/commands/dif-<name>.md` to override a command entirely) — project-level files take precedence over the plugin's shipped defaults, so you never need to fork this repo just to tweak one project's behavior.

## 6. Uninstall / reset

**Remove the plugin:**
```
/plugin uninstall dif@dif-marketplace
```
(or stop passing `--plugin-dir` if you were running it locally).

**Wipe a project's DIF state for a clean slate** (this deletes the active cycle, all generated docs, and local history for that project — it does not touch application source code):
```
rm -rf .claude/dif/
```
Run this from the project's root, not the plugin's. The next `/dif:*` command run afterward recreates `.claude/dif/` from scratch.

**Clear only the active cycle, keeping docs and history:**
```
rm -rf .claude/dif/active/
```

## 7. Troubleshooting

**A HIL gate seems stuck / DIF won't move past a gate.** This is by design — gates only advance on an explicit reply from you (e.g. "approved") to the message where DIF presented the gate; a vague or partial reply, or a new `/dif:*` command typed before approving, won't advance it. Reply directly with approval, or with the specific changes you want, in plain chat.

**Recovering from an interrupted run.** DIF's full state lives in one file: `.claude/dif/active/state.json`. Open it and check:

```json
{
  "phase": "plan_drafted",
  "track": "standard",
  "requirement_source": "direct",
  "approved": { "requirement": true, "plan": false, "consolidated": false },
  "files_touched": [],
  "run_mode": false
}
```

- `phase` tells you exactly where the cycle stopped.
- `approved.*` tells you what's actually been signed off — if `plan` is `false` here, no plan has been approved yet, regardless of what got printed in chat.
- `run_mode` tells you whether this cycle is auto-chaining (started via `/dif:run`, or resumed into it) or step-by-step. If it's `true` and you expected step-by-step, your next approval will auto-continue — say "pause" in that reply, or hand-edit it to `false`, if you don't want that.
- Run `/dif:status` to get this printed back to you in readable form instead of reading the JSON directly.
- If `phase` and the actual files in `active/` disagree (e.g. `phase: "executed"` but `task-outputs/` is empty), something was interrupted mid-write — the safest fix is usually to re-run the command for that phase; skills are idempotent about re-reading their inputs and re-writing their outputs.
- As a last resort, you can hand-edit `state.json` to move `phase` backward (never forward past an `approved` flag that's still `false` — that's the one thing to never fake), then re-run the corresponding command.
