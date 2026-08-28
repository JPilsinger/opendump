@AGENTS.md
@HARNESS_BOOTSTRAP.md
@opendump.config.md

## Claude Code bridge

This file is a derived Claude Code adapter. `AGENTS.md` is authoritative for task workflow behavior and `HARNESS_BOOTSTRAP.md` is authoritative for cold-start translation. Do not fork the workflow here.

### Locked store (no ambivalence)

- store: `github`
- location: this repository on `main` (the checkout / remote this project is using)
- Persist: commit and push task mutations per `AGENTS.md`

When the user explicitly requests a **cold start** and wants a different mode or host project, execute `HARNESS_BOOTSTRAP.md`, re-select a single mode, and rewrite instructions so only that mode remains. Preserve unrelated Claude project instructions if this bridge is merged into another project's `CLAUDE.md`.
