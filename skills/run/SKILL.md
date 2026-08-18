---
name: run
description: Orchestrate the full DIF pipeline in one command, auto-continuing past each phase boundary once its gate is approved — but never past a gate itself. Recommended model tier — none (this skill only sequences; each phase it hands off to uses its own recommended tier).
---

# run

## Purpose

Give a single entry point for "just run the whole flow," without weakening the HIL guarantee in §12 of the original spec. **Every gate is still a hard stop that ends the turn and waits for your explicit reply.** What this skill removes is the need to re-type `/dif:plan`, `/dif:execute`, `/dif:consolidate`, `/dif:complete` yourself between gates — once you approve a gate, DIF immediately drafts the next artifact and presents the next gate (or, for ungated phases like `execute`, just runs them) in that same reply.

This skill does not introduce any new capability — it's a thin orchestrator that sets `state.json.run_mode: true` and then hands off to the exact same skills (`explore`, `requirement`, `plan`, `execute`, `consolidate`, `complete`) described elsewhere in this plugin, which already know how to check `run_mode` and auto-continue (see the "run_mode" sections in each of their `SKILL.md` files).

## Inputs read

1. `.claude/dif/active/state.json` (if present)
2. `skills/run/rules.md` for project-specific overrides
3. The requirement text supplied as the command argument, if starting fresh (dynamic — load last)

## Outputs written

- `.claude/dif/active/state.json` — sets `run_mode: true` at the start of orchestration (everything else it writes is written by whichever skill it's currently handing off to)
- Whatever the phase currently running writes — see that phase's own `SKILL.md`

## Process

1. Read `state.json`.
2. **No active cycle** (`phase` is `"empty"` or `"explored"`, or `state.json` doesn't exist yet): the command argument is required — it's the requirement text. Set `run_mode: true`, then read and follow `skills/requirement/SKILL.md` from the top, exactly as `/dif:requirement` would, with that text as its input.
3. **Active cycle already in flight** (`phase` is anything past `"explored"`): don't discard it. Tell the user what phase/track it's at, set `run_mode: true` on it, and resume by reading and following whichever skill corresponds to the *current* phase (e.g. `phase: "plan_drafted"` with `approved.plan: false` → re-present the existing plan's gate via `skills/plan/SKILL.md`; `phase: "requirement_approved"` → proceed straight into the next phase per that track, same as an approval would). If a command argument was also given here, ignore it and say you're resuming the existing cycle instead of starting a new one, rather than silently overwriting `requirement.md`.
4. From there, control genuinely passes to the normal phase skills. This skill's only job was steps 1–3; it does not re-implement gate logic, phase preconditions, or doc generation — those all live in the phase skills themselves, and duplicating them here would be exactly the kind of drift the rest of this plugin is designed to avoid.

## The gate guarantee — read this before assuming `run` skips approval

`run_mode: true` changes what happens **after** an approval, never whether one is required. Concretely, nothing changes about:

- `explore` having no gate (unchanged — it never did).
- `requirement`, `plan`, and `consolidate` each ending their turn and waiting for an explicit human reply before advancing — they still do this identically under `run_mode`. The *only* difference is that once that reply is a genuine approval, the assistant continues in the same reply instead of also stopping to wait for a new slash command.
- `execute` writing code only for approved requirement/plan.

If at any point the user's approval reply also says to pause, stop, slow down, or review manually from here — honor that immediately: set `run_mode: false` in `state.json` and stop, same as if `run_mode` had never been set. `run_mode` is a standing preference for *this cycle*, not an irrevocable commitment; the user can drop back to step-by-step at any gate, and can re-invoke `/dif:run` later to resume auto-chaining if they change their mind again.

## What this skill deliberately does not do

- It does not add a "skip approval entirely" mode, and there is no config flag anywhere in this plugin to add one back. Removing the requirement to retype a command between gates is already a deliberate loosening of the original spec's §12 hard-stop design (see `skills/run/rules.md`); skipping the approval itself would remove the actual safety property, not just the ceremony around it, and is out of scope for this skill by design.
- It does not change model tiering, caching discipline, doc-generation rules, or execution parallelism — all of that is still owned entirely by the phase skill currently running.

## Model tiering

No dedicated tier — this skill only reads `state.json` and dispatches to another skill's instructions; the actual work happens under that skill's own recommended tier.

## Caching discipline

Read `state.json` and `rules.md` first (tiny, stable); don't do anything else here before handing off — all substantive caching guidance lives in the skill being dispatched to.

## Overrides

Read `skills/run/rules.md` in this same directory before acting.
