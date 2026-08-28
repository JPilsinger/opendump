@AGENTS.md
@HARNESS_BOOTSTRAP.md

## Claude Code bridge

This file is a derived Claude Code adapter. `AGENTS.md` is authoritative for task workflow behavior and `HARNESS_BOOTSTRAP.md` is authoritative for cold-start translation. Do not fork the workflow here.

When the user explicitly requests a **cold start** and this repository is accessible, execute the harness bootstrap protocol before normal opendump operation. Preserve unrelated Claude project instructions if this bridge is merged into another project's `CLAUDE.md`.
