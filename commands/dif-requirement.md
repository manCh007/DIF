---
description: Intake a requirement (typed or pasted from an external ticket), classify its track, and present it for approval.
argument-hint: "<requirement text>"
---

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/requirement/SKILL.md` exactly, including its instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/requirement/rules.md` for project-specific overrides.

Requirement text passed to this command: $ARGUMENTS

This command runs the `requirement` skill: it writes `.claude/dif/active/requirement.md`, classifies the change as `trivial`, `standard`, or `complex`, and then **stops at a human-in-the-loop gate**. Do not proceed past that gate in this turn under any circumstance.
