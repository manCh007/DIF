---
name: consolidate
description: Merge task outputs into a single consolidated diff summary and stop for human validation. Recommended model tier — fast/cheap–mid.
---

# consolidate

## Purpose

Merge the per-task outputs from `execute` into one coherent, human-reviewable picture of the whole change, and stop for validation before docs get updated and the cycle closes. Not used on the `trivial` track. This skill does not write application source code — it summarizes what `execute` already wrote.

## Inputs read

1. `.claude/dif/active/state.json`
2. `.claude/dif/active/task-plan.md`
3. `.claude/dif/active/task-outputs/*.md` (all of them)
4. `skills/consolidate/rules.md` for project-specific overrides

## Outputs written

- `.claude/dif/active/consolidated-diff.md`
- `.claude/dif/active/decision-log.md` — append any deviations-from-plan or notable judgment calls surfaced in task outputs
- `.claude/dif/active/state.json` — `phase: "consolidated"`, `approved.consolidated: false`

## Preconditions

- `state.json.phase` must be `"executed"`. If not, stop and tell the user the actual phase and the command that would get them there.
- `state.json.track` must not be `"trivial"` — that track goes straight from `executed` to `/dif:complete`. If it is, say so.

## Process

1. Read every file in `task-outputs/`.
2. Produce `consolidated-diff.md`: a single narrative of what changed and why, organized by task or by area (whichever reads more clearly for this change), cross-referencing files touched. This is a summary of real work already done — do not re-derive or re-verify the diff from scratch by re-reading all application source; trust the task outputs as the record, but flag anything in them that looks internally inconsistent.
3. Pull any "deviation from plan" or notable judgment-call notes out of the task outputs into `decision-log.md`, so they survive independent of the raw task-output files.
4. Update `state.json`: `phase: "consolidated"`, `approved.consolidated: false`.
5. Present the HIL gate.

## HIL gate — stop here, no exceptions

> Stop here. Present the consolidated result clearly. Do not call any tool, write any file beyond `consolidated-diff.md`, `decision-log.md`, and `state.json` already written above, or proceed to the next phase in this turn — regardless of confidence — until the user replies with an explicit approval. On approval, set `approved.consolidated` to `true` in `state.json` before ending the turn.

- Revision requested: this generally means going back to `execute` (or even `plan`) rather than editing the summary — say which, based on what the user is objecting to. If it's genuinely just a summarization error (the diff is right but was described wrong), fix `consolidated-diff.md` directly and re-present.
- Approved: set `approved.consolidated: true`, confirm briefly, and stop — do not auto-invoke `complete`.

## Model tiering

Recommended tier: **fast/cheap–mid**. This is mostly summarizing diffs already produced, not generating new judgment.

## Caching discipline

- Load `task-plan.md` first (stable), then all `task-outputs/*.md` in one batch.
- Don't reformat task-output files before summarizing them in the same turn.

## Overrides

Read `skills/consolidate/rules.md` in this same directory before acting.
