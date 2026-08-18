---
name: plan
description: Turn an approved requirement into a task plan with a dependency graph, and stop for human approval. Recommended model tier — strong.
---

# plan

## Purpose

Read the approved requirement and produce `task-plan.md`: a concrete, ordered set of tasks with a dependency graph, scoped against the real codebase (not just the requirement's stated intent). Not used on the `trivial` track — that track's single combined gate at `/dif:requirement` covers this. This skill never writes application source code.

## Inputs read

1. `.claude/dif/active/state.json`
2. `.claude/dif/active/requirement.md`
3. `.claude/dif/docs/architecture.md`
4. Relevant entries in `.claude/dif/docs/flows/_index.md` and `.claude/dif/docs/functional/_index.md`, and the specific flow/module docs the requirement touches
5. `skills/plan/rules.md` for project-specific overrides
6. Live source files for the modules/flows in scope — dynamic, load last, and only what's needed to scope tasks accurately

## Outputs written

- `.claude/dif/active/task-plan.md`
- `.claude/dif/active/state.json` — `phase: "plan_drafted"`, `approved.plan: false`
- Possibly a newly-generated flow or module doc under `.claude/dif/docs/` (see step 3) and an updated `.claude/dif/docs/_meta.json`

## Preconditions

- `state.json.phase` must be `"requirement_approved"`. If it isn't, stop and tell the user what phase the project is actually in and what command would get it there.
- `state.json.track` must not be `"trivial"`. If it is, tell the user this track skips planning — approval already happened at `/dif:requirement`, and the next command is `/dif:execute`.

## Process

1. Read the approved requirement and `architecture.md`.
2. Identify which flows/modules are actually in scope by reading the relevant source, not just guessing from the requirement's prose.
3. **Lazy doc generation trigger**: for any in-scope flow or module that has no doc yet in `flows/` or `functional/`, generate one now (this is the "first time a requirement touches this flow's code" moment described in the docs system). Add it to the relevant `_index.md` and record its `source_files` in `_meta.json`.
4. Break the requirement into concrete tasks. For each task, capture: id, one-line description, files expected to change, and `depends_on` (other task ids, or empty). Keep the dependency graph as flat as the work honestly allows — don't invent sequencing that isn't real.
5. Write `task-plan.md` as the ordered task list plus the dependency graph.
6. Update `state.json`: `phase: "plan_drafted"`, `approved.plan: false`.
7. Present the HIL gate.

## HIL gate — stop here, no exceptions

> Stop here. Present the task plan clearly, including the dependency graph. Do not call any tool, write any file beyond `task-plan.md` and `state.json` already written above, or proceed to the next phase in this turn — regardless of confidence — until the user replies with an explicit approval. On approval, set `approved.plan` to `true` in `state.json` before ending the turn.

- Revision requested: update `task-plan.md`, re-present, leave `approved.plan: false`.
- Approved: set `approved.plan: true`, advance `phase` to `"plan_approved"`, confirm briefly, and stop — do not auto-invoke `execute`.

## Execution model note (bake into scoping, not just document)

Tasks in `task-plan.md` are executed sequentially by default, one at a time, respecting `depends_on` order, regardless of how many appear independent — the `execute` skill only parallelizes when the user explicitly opts in for that run. Keep this in mind when writing task descriptions: each task's description should be understandable in isolation, since a later task may start with only its dependencies' outputs available, not the full plan's context.

## Model tiering

Recommended tier: **strong**. Scoping and dependency reasoning carry real risk if wrong — an incorrect dependency graph or missed file can silently produce broken or incomplete execution.

## Caching discipline

- Load `requirement.md`, `architecture.md`, and `rules.md` first (stable this cycle); load live source files last.
- Batch source reads into as few tool calls as possible; don't read files outside the scope you've already determined.
- Don't reformat `requirement.md` or `architecture.md` before using them in the same turn.

## Overrides

Read `skills/plan/rules.md` in this same directory before acting.
