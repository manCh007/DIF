---
description: Execute the approved task plan (or, on the trivial track, the approved requirement) and write per-task outputs.
argument-hint: "[--parallel]"
---

Read and follow `${CLAUDE_PLUGIN_ROOT}/skills/execute/SKILL.md` exactly, including its instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/execute/rules.md` for project-specific overrides.

Arguments passed to this command: $ARGUMENTS

This command runs the `execute` skill. It is the only phase that writes application source code. It executes tasks sequentially by default, respecting `depends_on`; pass `--parallel` only if you explicitly want independent tasks run concurrently for this run.
