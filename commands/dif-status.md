---
description: Print the current DIF phase, track, and approval flags for this project. Read-only, no side effects.
argument-hint: ""
---

Read `.claude/dif/active/state.json` in the current project (if it does not exist, the project has no active DIF cycle — say so and stop).

Print, in a short human-readable form:

- `phase` — the current pipeline phase
- `track` — `trivial` / `standard` / `complex`, and how it was set (auto-classified or user-overridden, if recorded)
- `approved` — the `requirement` / `plan` / `consolidated` flags
- `files_touched` — count and list, if any
- Which `/dif:*` command is the valid next step given the current phase and track (see the phase table in `${CLAUDE_PLUGIN_ROOT}/skills/requirement/rules.md` and the pipeline description in `${CLAUDE_PLUGIN_ROOT}/README.md` if you need to double check phase ordering)

This command does not invoke any skill, write any file, or change state. It only reads and reports.
