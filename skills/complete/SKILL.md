---
name: complete
description: Incrementally update affected docs, archive the cycle to history/, clear active/, and reset state. Recommended model tier — fast/cheap.
---

# complete

## Purpose

Close out a DIF cycle: update only the docs actually affected by what changed (never a full re-crawl), archive the cycle's `active/` artifacts, and reset state so the project is ready for the next requirement. This skill does not write application source code.

## Inputs read

1. `.claude/dif/active/state.json`
2. `.claude/dif/active/consolidated-diff.md` (`standard`/`complex`) or `.claude/dif/active/task-outputs/*.md` (`trivial`, which has no consolidate step)
3. `.claude/dif/docs/_meta.json`
4. `skills/complete/rules.md` for project-specific overrides (including whether history archiving is enabled)

## Outputs written

- Updated `.claude/dif/docs/flows/<name>.md` and/or `.claude/dif/docs/functional/<name>.md` for affected areas only
- Newly-created flow/module docs for any touched file that maps to none yet
- `.claude/dif/docs/architecture.md` — **only** on `complex` track, and only if warranted (see step 5)
- Updated `.claude/dif/docs/_meta.json` hashes for everything regenerated
- `.claude/dif/history/<date>-<slug>/` — the archived `active/` contents (unless disabled, see rules.md)
- Reset `.claude/dif/active/state.json` — back to `phase: "empty"`, `approved` all `false`, `files_touched: []`, ready for the next cycle

## Preconditions

- `standard`/`complex` tracks: `state.json.phase` must be `"consolidated"` and `approved.consolidated` must be `true`.
- `trivial` track: `state.json.phase` must be `"executed"` (no consolidate step on this track).
- If neither holds, stop and tell the user the actual phase/track and what command would get them there. Do not run `complete` on unapproved work.

## Process (incremental update, not a full re-crawl)

1. Read `files_touched` from `state.json` (this is the authoritative list built up across `execute`; also cross-check against `consolidated-diff.md` if present for narrative context).
2. Cross-reference each touched file against `_meta.json`'s `source_files` entries to find which specific `flows/*.md` / `functional/*.md` docs are affected.
3. Regenerate **only** those files. Do not touch docs whose `source_files` weren't in `files_touched`.
4. If a touched file maps to no existing flow/functional doc, flag it as a new flow or module to document, and generate that doc now (same as the lazy-generation trigger in `plan`, for anything that slipped through).
5. **`complex` track only**: re-evaluate whether `architecture.md` needs updating — new service, new module, new external dependency. Do not update it for changes that don't rise to that level, even on `complex` track. `trivial`/`standard` never touch `architecture.md` here.
6. Update `_meta.json` hashes for everything just regenerated.
7. **History archive** (default-on; see rules.md to disable): derive `<slug>` from the first line of `requirement.md`, move all of `active/`'s contents into `history/<date>-<slug>/`, then clear `active/`.
8. Reset `state.json` to a fresh `phase: "empty"` state, `approved` flags all `false`, `files_touched: []`. If history archiving is disabled, still clear `active/`'s working files (requirement.md, task-plan.md, task-outputs/, consolidated-diff.md, decision-log.md) since the cycle is genuinely done — just don't copy them anywhere first.

## No HIL gate

Approval already happened at the `consolidated` gate (or the combined `trivial` gate). `complete` is finalization, not a decision point — it runs to completion and reports what docs it updated and where the cycle was archived.

## Model tiering

Recommended tier: **fast/cheap**. This is doc updates and logging against an already-known, already-approved diff — low judgment beyond the cross-referencing logic above.

## Caching discipline

- Load `state.json` and `_meta.json` first (stable, small); load `consolidated-diff.md`/`task-outputs` next; only open the specific existing doc files that step 2 determined are actually affected — don't open every doc in `flows/`/`functional/` to check.
- Don't reformat `_meta.json` or any doc file before using it in the same turn.

## Overrides

Read `skills/complete/rules.md` in this same directory before acting — it's also where history-archiving is toggled off, if a project doesn't want a local trail.
