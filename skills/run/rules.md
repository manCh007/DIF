# run — project-specific overrides

This file is read by the `run` skill on every invocation, after the defaults in `SKILL.md`. Override it at the project level via `.claude/skills/run/rules.md` if you'd rather not fork this plugin (see SETUP.md §5).

## Why this skill exists, and what it deliberately doesn't do

The original DIF spec (§12) made every phase a separate slash command specifically so nothing could proceed past a human approval gate without the user invoking the next command themselves — "there is no code path that lets the agent continue without the user invoking the next command themselves." `/dif:run` loosens exactly one part of that: once a gate is approved, DIF auto-continues to the next phase instead of waiting for a fresh command. It does not loosen the part that matters most — every gate still ends the turn and requires an explicit human reply before anything past it happens.

There is intentionally no setting in this file (or anywhere else in this plugin) to make `/dif:run` skip approvals outright. If a future maintainer of a fork wants that, it's a bigger step than this skill should take quietly via a config flag — it changes the plugin's core safety property, not just its ergonomics, and deserves its own explicit, visible decision in the skill itself rather than a toggle buried in a rules file.

<!--
## Auto-run explore first
(Default: run skill does not force an explore step before requirement intake —
requirement already reads architecture.md if present, and explore is cheap to
run separately. Uncomment to have /dif:run always call explore first on a
fresh cycle.)
- 

## Default pause phrase recognition
(Words/phrases beyond "pause"/"stop"/"slow down" that should be treated as a
request to drop back to run_mode: false when included in an approval reply)
- 
-->
