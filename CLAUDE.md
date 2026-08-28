@AGENTS.md
@HARNESS_BOOTSTRAP.md
@opendump.config.md

## Claude Code bridge

This file is a derived Claude Code adapter for the **public upstream template**. `AGENTS.md` is authoritative for task workflow behavior and `HARNESS_BOOTSTRAP.md` is authoritative for cold-start translation. Do not fork the workflow here.

### Not a personal store

- role: `public-upstream-template` (`JPilsinger/opendump`)
- **task-writes: forbidden** — never commit/push user tasks, progress, or completions to this repository
- On cold start or first capture: create or connect a **user-owned** repo from this template (or lock `local-files` / `artifact`), then put mode + that location in the **host** project instructions
- Review this repo for workflow docs only

When the user sends `review https://github.com/JPilsinger/opendump and start the coldstart procedure.` (or equivalent), execute `HARNESS_BOOTSTRAP.md`, lock a single **user-owned** store, and rewrite host instructions so only that mode remains. Preserve unrelated Claude project instructions if this bridge is merged into another project's `CLAUDE.md`.
