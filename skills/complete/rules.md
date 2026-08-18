# complete — project-specific overrides

This file is read by the `complete` skill on every run, after the defaults in `SKILL.md`. Override it at the project level via `.claude/skills/complete/rules.md` if you'd rather not fork this plugin (see SETUP.md §5).

## History archive

Default: **on**. Every completed cycle is archived to `.claude/dif/history/<date>-<slug>/` before `active/` is cleared.

```
history_archive: on   # set to "off" below to disable local history entirely
```

<!--
To disable, uncomment:
history_archive: off
-->

<!--
## Doc regeneration exceptions
(Files/patterns that should never trigger flow/functional doc regeneration,
e.g. generated files, lockfiles)
- 

## Slug format
(Default: kebab-case of requirement.md's first line, truncated. Override here
if you want a different convention, e.g. prefixed with a ticket ID.)
- 
-->
