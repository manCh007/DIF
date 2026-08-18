---
name: requirement
description: Intake a requirement (typed or pasted from an external ticket), classify its track, write requirement.md, and stop for human approval. Recommended model tier — fast/cheap–mid.
---

# requirement

## Purpose

Turn free-form requirement text — typed directly in chat, or pasted from an external ticket/story — into a normalized `requirement.md`, classify how much ceremony the change deserves (its **track**), and stop for explicit human approval before anything else happens. This skill never writes application source code.

## Inputs read

1. `.claude/dif/active/state.json` (if present — see §Resuming below)
2. `.claude/dif/docs/architecture.md` (if present, for context on what's being asked)
3. `skills/requirement/rules.md` for project-specific overrides (e.g. custom track thresholds)
4. The requirement text supplied as the command argument (dynamic — load last)

## Outputs written

- `.claude/dif/active/requirement.md`
- `.claude/dif/active/state.json` — `phase`, `track`, `requirement_source`, `approved.requirement` (see schema)

## Resuming / avoiding accidental overwrite

If `state.json` exists and its `phase` is anything other than `"empty"` or `"explored"`, an active cycle is already in flight. Do not silently overwrite it. Tell the user what's in progress (phase, track, a one-line summary of the existing `requirement.md` if present) and ask whether to resume it, discard it and start fresh, or archive it first. Only proceed once the user has explicitly chosen.

## Process

1. Read `state.json` (create it with `phase: "empty"` if entirely absent — this can happen if `/dif:requirement` is the very first command run in a project, skipping `/dif:explore`).
2. Determine `requirement_source`: `"direct"` if the text reads as something typed inline describing a change; `"pasted"` if it carries the shape of an external ticket (structured sections, a ticket ID, acceptance-criteria headers, etc).
3. Classify **track** using the table below, unless the user's argument text includes an explicit override (e.g. `--track=standard`):

   | Track | Criteria |
   |---|---|
   | `trivial` | Single file, no new dependencies, no new flow |
   | `standard` | Few files, one existing flow/module |
   | `complex` | Cross-module, new flow, or structural change |

4. Write `requirement.md`: restate the goal in your own words, scope (files/modules you expect are involved, at a high level — exact files are the `plan` skill's job for standard/complex), acceptance criteria, explicit assumptions, and any open questions you need the human to resolve before this can be approved.
5. Update `state.json`: `phase: "requirement_drafted"`, `track: <classified>`, `requirement_source: <...>`, `approved.requirement: false`.
6. Present the HIL gate (see below). Explicitly surface the track and note the user can override it right now.

## HIL gate — stop here, no exceptions

> Stop here. Present the requirement understanding clearly, along with the classified track and a one-line note that the user can override the track. Do not call any tool, write any file beyond `requirement.md` and `state.json` already written in step 4–5, or proceed to the next phase in this turn — regardless of confidence — until the user replies with an explicit approval. On approval, set `approved.requirement` to `true` in `state.json` before ending the turn.

This gate is a **separate turn**, not a separate command that chains automatically. Concretely:

- If the user's next message (plain chat, not a slash command) requests changes: revise `requirement.md` (and `track` if asked), re-present the gate, leave `approved.requirement: false`. Repeat until approved.
- If the user's next message is an explicit approval ("approved", "yes", "looks good", etc.): set `approved.requirement: true` in `state.json`, confirm briefly, and **stop** — do not automatically invoke `plan` or `execute`. The user must run the next `/dif:*` command themselves.
- **Trivial track only**: this is the pipeline's single combined gate. Approval here means the user is approving both the requirement *and* going straight to execution with no separate plan step — say so explicitly when presenting the gate on this track, so the approval is informed. On approval, still only set `approved.requirement: true` (there is no `plan` phase to approve on this track); `/dif:execute` is gated on `phase == "requirement_approved"` directly for `trivial`.
- Also on approval, advance `phase` to `"requirement_approved"`.

## Model tiering

Recommended tier: **fast/cheap–mid**. This is extraction and classification — real judgment on track boundaries, but not the deep reasoning that `plan` or `execute` require.

## Caching discipline

- Load `architecture.md` and `rules.md` first (stable); load the requirement text argument last (dynamic, changes every invocation).
- Don't re-read `requirement.md` immediately after writing it in the same turn just to "double check" — you already have its content.

## Overrides

Read `skills/requirement/rules.md` in this same directory before acting.
