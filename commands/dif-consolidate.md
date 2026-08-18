---
description: Merge this cycle's task outputs into a single consolidated diff and present it for validation.
argument-hint: ""
---

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/consolidate/SKILL.md` exactly, including its instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/consolidate/rules.md` for project-specific overrides.

Arguments passed to this command: $ARGUMENTS

This command runs the `consolidate` skill: it writes `.claude/dif/active/consolidated-diff.md` and then **stops at a human-in-the-loop gate**. Do not proceed past that gate in this turn under any circumstance. Not used on the `trivial` track.
