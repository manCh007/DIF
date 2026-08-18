# execute — project-specific overrides

This file is read by the `execute` skill on every run, after the defaults in `SKILL.md`. Override it at the project level via `.claude/skills/execute/rules.md` if you'd rather not fork this plugin (see SETUP.md §5).

## Parallel execution default

Sequential by default (see SKILL.md §Execution model). This project's default:

```
parallel_execution: opt-in-per-run   # do not change to "always" without understanding the cost/blast-radius tradeoff in SKILL.md
```

<!--
## Commands to run after each task (tests, linters, type checks)
- 

## Files/directories execute must never touch
(e.g. generated code, vendored dependencies, migration history already applied)
- 

## Commit conventions
(If execute should commit per-task rather than leaving changes uncommitted for
consolidate to summarize — off by default)
- 
-->
