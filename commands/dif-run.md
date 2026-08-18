---
description: Run the whole DIF pipeline, auto-continuing past each phase boundary once its gate is approved. Every gate still stops and waits for explicit approval — this only removes the need to retype the next command yourself.
argument-hint: "<requirement text> (omit to resume an in-flight cycle in auto-chain mode)"
---

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/run/SKILL.md` exactly, including its instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/run/rules.md` for project-specific overrides.

Arguments passed to this command: $ARGUMENTS

This command sets `run_mode: true` in `.claude/dif/active/state.json` and then dispatches to the same `requirement` → `plan` → `execute` → `consolidate` → `complete` skills the individual `/dif:*` commands use. It does **not** skip any HIL gate — each one still stops the turn and waits for your explicit approval. It only removes the need to type the next `/dif:*` command yourself after approving. Say "pause" (or similar) in any approval reply to drop back to step-by-step mode at that gate.
