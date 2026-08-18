---
description: Turn an approved requirement into a task plan with a dependency graph, and present it for approval.
argument-hint: ""
---

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/plan/SKILL.md` exactly, including its instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/plan/rules.md` for project-specific overrides.

Arguments passed to this command: $ARGUMENTS

This command runs the `plan` skill: it writes `.claude/dif/active/task-plan.md` and then **stops at a human-in-the-loop gate**. Do not proceed past that gate in this turn under any circumstance. Not used on the `trivial` track — that track approves requirement and plan together at `/dif:requirement`.
