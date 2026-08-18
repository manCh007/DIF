# requirement — project-specific overrides

This file is read by the `requirement` skill on every run, after the defaults in `SKILL.md`. Override it at the project level via `.claude/skills/requirement/rules.md` if you'd rather not fork this plugin (see SETUP.md §5).

## Reference: full phase order

`empty` → `explored` (optional) → `requirement_drafted` → `requirement_approved` → `plan_drafted` → `plan_approved` → `executing` → `executed` → `consolidated` → `complete`

`trivial` track skips `plan_drafted` / `plan_approved` entirely (`requirement_approved` → `executing` directly) and skips `consolidated` (`executed` → `complete` directly, via `/dif:complete`).

## Reference: track pipelines

| Track | Pipeline |
|---|---|
| `trivial` | requirement → 1 combined HIL gate → execute → complete |
| `standard` | requirement → HIL → plan → HIL → execute → consolidate → HIL → complete |
| `complex` | full pipeline, plus forces an `architecture.md` review at completion |

<!--
## Custom track thresholds
(Override the default trivial/standard/complex criteria for this project —
e.g. "any change touching /packages/shared is always at least standard"
regardless of file count.)
- 

## Requirement-source detection hints
(Patterns specific to your ticket system that should always be classified as "pasted")
- 
-->
