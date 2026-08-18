---
name: explore
description: Generate or refresh the living architecture doc for the project and report documentation coverage. No HIL gate. Recommended model tier — fast/cheap.
---

# explore

## Purpose

Maintain `.claude/dif/docs/` as a living, incrementally-updated description of the codebase: one top-level `architecture.md`, plus lazily-generated per-flow and per-module docs. This skill never touches application source code — it only reads the codebase and writes into `.claude/dif/docs/`.

This is the only skill that may run before any requirement exists, and the only entry point that creates `.claude/dif/` in a project for the first time.

## Inputs read (load stable content first, dynamic content last — see §Caching)

1. `.claude/dif/docs/_meta.json` (if present)
2. `.claude/dif/docs/architecture.md` (if present)
3. `.claude/dif/active/state.json` (if present)
4. `skills/explore/rules.md` (this directory) for project-specific overrides
5. The project source tree — read only what's needed to describe topology, tech stack, module boundaries, and entry points; do not exhaustively read every file

## Outputs written

- `.claude/dif/docs/architecture.md`
- `.claude/dif/docs/flows/_index.md` (table: flow name, one-line summary, entry point file, doc file)
- `.claude/dif/docs/functional/_index.md` (same shape, for modules/domains)
- `.claude/dif/docs/_meta.json`
- `.claude/dif/active/state.json` (only the `phase` field, only under the condition in step 4 below)

## Process

1. **First run in this project** (`.claude/dif/` does not exist, or `_meta.json` is absent): create the full folder structure from the runtime layout below, generate `architecture.md` by analyzing the codebase's topology, tech stack, module boundaries, and entry points, and write `_meta.json` with `architecture.md`'s hash. Create `flows/_index.md` and `functional/_index.md` as empty tables — do **not** generate per-flow or per-module docs yet; those are lazy (see rule 1 in rules.md and §7 of the parent spec).

   ```
   .claude/dif/
   ├── active/
   ├── docs/
   │   ├── architecture.md
   │   ├── flows/_index.md
   │   ├── functional/_index.md
   │   └── _meta.json
   └── history/
   ```

2. **Subsequent run, no `--refresh`**: if `_meta.json` exists and `architecture.md` exists, do **not** regenerate it — per the parent spec, `architecture.md` regenerates only on a landed `complex`-track change (handled by the `complete` skill) or an explicit `--refresh` here. Report "up to date" plus a doc-coverage summary: how many discovered flows/modules have a doc in `flows/`/`functional/` vs. how many are referenced in code but undocumented.

3. **`--refresh` passed**: force full regeneration of `architecture.md` regardless of state, update its hash in `_meta.json`, and report what changed. Still do not regenerate lazy flow/functional docs — those refresh independently, on demand, when a requirement touches them (see the `plan` skill).

4. **State transition**: if `.claude/dif/active/state.json` does not exist, create it with `phase: "empty"` and the rest of the schema in §State Schema, then immediately set `phase: "explored"`. If `state.json` already exists with any phase other than `"empty"`, leave it untouched — exploring mid-cycle is a supplementary refresh, not a pipeline step, and must never move an in-progress requirement's phase backward or forward.

## State schema (for reference — see `requirement` skill for the authoritative description)

```json
{
  "phase": "empty",
  "track": null,
  "requirement_source": null,
  "approved": { "requirement": false, "plan": false, "consolidated": false },
  "files_touched": [],
  "run_mode": false
}
```

`run_mode: true` means this cycle was started via `/dif:run` and gated skills auto-continue to the next phase immediately after each approval, instead of waiting for the user to type the next `/dif:*` command. See `skills/run/SKILL.md`.

## No HIL gate

`explore` never modifies application source code and never advances the requirement pipeline, so it does not stop for approval. Run it, report results, end the turn normally.

## Model tiering

Recommended tier: **fast/cheap**. This is pattern extraction over a codebase (topology, entry points, tech stack) — low judgment, high volume. A strong-tier model is unnecessary overhead here.

## Caching discipline

- Read `_meta.json` and any existing `architecture.md` first (stable); read live source files last.
- Batch source reads into as few tool calls as possible. Don't open files you haven't already decided are relevant to topology/entry-points.
- Never rewrite `architecture.md` or `_meta.json` immediately before reading them back in the same turn — that busts the prefix cache for no benefit.

## Overrides

Read `skills/explore/rules.md` in this same directory before acting — it holds project-specific overrides to the defaults above (e.g. directories to exclude from topology scans).
