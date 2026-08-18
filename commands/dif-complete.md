---
description: Incrementally update project docs, archive this cycle to history/, and close out the workflow state.
argument-hint: ""
---

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/complete/SKILL.md` exactly, including its instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/complete/rules.md` for project-specific overrides.

Arguments passed to this command: $ARGUMENTS

This command runs the `complete` skill: it incrementally regenerates only the flow/functional docs affected by this cycle's changes, archives `.claude/dif/active/` into `.claude/dif/history/`, clears `active/`, and resets state to `empty` for the next cycle.
