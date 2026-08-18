---
description: Generate or refresh the living architecture doc for this project and report documentation coverage.
argument-hint: "[--refresh]"
---

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/explore/SKILL.md` exactly, including its instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/explore/rules.md` for project-specific overrides.

Arguments passed to this command: $ARGUMENTS

This command runs the `explore` skill: it creates `.claude/dif/` in this project on first run, generates or refreshes `architecture.md`, and reports doc coverage. It does not require human approval and does not modify any application source code.
