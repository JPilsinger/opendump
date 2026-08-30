@AGENTS.md
@HARNESS_BOOTSTRAP.md
@opendump.config.md

## Claude Code bridge

This file is a **derived template-seed adapter**. `AGENTS.md` is authoritative for normal task workflow behavior; `HARNESS_BOOTSTRAP.md` is authoritative for initialization.

Because GitHub template creation copies this file verbatim, do **not** infer repository role from this file alone.

### Repository identity

- Determine the actual current repository identity during cold start.
- If it is exactly `JPilsinger/opendump`, this checkout is the public upstream workflow/template reference and user task writes are forbidden.
- If it is a different repository copied from the template, treat the inherited seed/config as `UNINITIALIZED`; do not misclassify it as the public upstream.

### Cold start

When the user sends `review https://github.com/JPilsinger/opendump and start the coldstart procedure.` (or equivalent), execute the normative state machine in `HARNESS_BOOTSTRAP.md`.

Cold start is successful only at `READY`: environment identified → capabilities established → exactly one store selected → concrete store bound → native adapter actually installed → adapter/store read back and verified → live tasks loaded.

If this seed is being used inside a user-owned opendump instance, successful cold start MUST replace/update the host's persistent opendump instructions with exactly one concrete `store` + `location`. Preserve unrelated Claude project instructions.

Generating instructions that still require manual installation is `BLOCKED`, not successful initialization.
