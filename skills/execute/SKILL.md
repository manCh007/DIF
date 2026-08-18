---
name: execute
description: Execute the approved task plan (or, on the trivial track, the approved requirement) and write per-task outputs. Sequential by default. Recommended model tier — strong.
---

# execute

## Purpose

Do the actual implementation work. This is the **only** skill in this plugin permitted to write or edit application source code. No HIL gate ends this skill — approval already happened at the requirement/plan gate(s); this skill's job is to carry that approved intent into the codebase faithfully.

## Inputs read

1. `.claude/dif/active/state.json`
2. `.claude/dif/active/task-plan.md` (`standard`/`complex` tracks) **or** `.claude/dif/active/requirement.md` (`trivial` track — there is no task plan)
3. `.claude/dif/docs/architecture.md` and any relevant flow/module docs
4. `skills/execute/rules.md` for project-specific overrides
5. Live source files being changed — dynamic, load per-task, only when that task starts

## Outputs written

- Application source code changes (the actual point of this skill)
- `.claude/dif/active/task-outputs/<task-id>.md` — one file per task: what changed, files touched, notes, any deviations from the plan and why
- `.claude/dif/active/state.json` — `phase`, `files_touched` (append, don't replace)

## Preconditions

- `trivial` track: `state.json.phase` must be `"requirement_approved"`.
- `standard`/`complex` tracks: `state.json.phase` must be `"plan_approved"`.
- If neither holds, stop and tell the user the actual current phase/track and what command would get them to a valid precondition. Do not guess forward.

## Process

1. Set `phase: "executing"` in `state.json` immediately on start.
2. **Trivial track**: treat the approved requirement itself as the single implicit task. Implement it, write one `task-outputs/task-1.md`.
3. **Standard/complex tracks**: iterate `task-plan.md`'s tasks **sequentially by default**, in dependency order — a task only starts once everything in its `depends_on` has completed and that output is available to read. See rules.md and §Execution model in this file for the parallel opt-in.
4. For each task: implement it, write its `task-outputs/<task-id>.md`, append any newly-touched files to `state.json.files_touched` (dedupe; don't overwrite the running list).
5. If a task turns out to need something the plan didn't anticipate (a file the plan missed, a dependency that wasn't captured), do the correct thing for the codebase and record the deviation clearly in that task's output file — don't silently diverge without a trace.
6. When all tasks are done: set `phase: "executed"`.
7. Check `state.json.run_mode`. If `false` (or absent), stop here and report what was done — the user runs `/dif:consolidate` (or `/dif:complete` on `trivial`) themselves. If `true`, immediately continue in this same turn: `standard`/`complex` tracks proceed to `skills/consolidate/SKILL.md`; `trivial` (which has no consolidate step) proceeds directly to `skills/complete/SKILL.md`.

## Execution model — sequential by default (do not deviate silently)

Parallel sub-agents each carry independent context and an independent cache; parallelizing multiplies both token cost and the blast radius of a bad plan. **Default to sequential.** Only run tasks in parallel when:

- The user explicitly opted in for *this run* (e.g. passed `--parallel` to `/dif:execute`, or said so in chat this turn), **and**
- The tasks being parallelized are genuinely independent and well-scoped (no shared files, no `depends_on` edge between them, either direction).

If both conditions aren't clearly met, run sequentially even if the user passed `--parallel` — say why.

## No HIL gate

`execute` does not stop for approval — the requirement (and, on `standard`/`complex`, the plan) were already approved. It runs to completion (or to a genuine blocker) and reports what it did. The next gate, if any, is at `/dif:consolidate`. Outside `run_mode`, that also means execute always ends its own turn afterward — the user must invoke the next command themselves, same as every other phase boundary.

## Model tiering

Recommended tier: **strong**. This is actual code correctness — the highest-stakes skill in the pipeline.

## Caching discipline

- Load `task-plan.md`/`requirement.md`, `architecture.md`, and `rules.md` first (stable for this run); load each task's live source files only when that task starts, not all up front.
- Batch each task's reads into as few tool calls as possible.
- Don't reformat `task-plan.md` or prior tasks' output files before using them in the same turn.

## Overrides

Read `skills/execute/rules.md` in this same directory before acting — it's also where the parallel-execution opt-in default lives.
