---
name: opendump
description: >-
  opendump template seed and task workflow. Use on cold-start one-liners and
  normal task capture/status after a user-owned store has been verified.
---

# opendump

This skill is derived. `AGENTS.md` defines normal task semantics; `HARNESS_BOOTSTRAP.md` defines initialization.

## Cold start

On `review https://github.com/JPilsinger/opendump and start the coldstart procedure.` or equivalent:

1. Determine the actual repository identity. Only exact `JPilsinger/opendump` is the public upstream; a different repo with copied seed metadata is an uninitialized template instance.
2. Execute the formal bootstrap state machine: environment → capabilities → store selection → store binding → adapter installation → verification → task load.
3. Use the deterministic store selection in `HARNESS_BOOTSTRAP.md`; do not ask the user to choose merely because several technical modes exist.
4. `github` must bind a concrete user-owned writable repo and must never bind `JPilsinger/opendump`.
5. A generated/manual adapter payload is not successful installation. If the persistent instruction surface cannot be written, return `BLOCKED` until the user installs it and it can be read back.
6. Claim success only at `READY`, after adapter/store read-back verification and live task loading.

## Runtime after READY

- New trackable work → deduplicate/classify → Backlog in the locked store → persist same turn.
- Corrections/progress/completion mutate only the locked store.
- Status prompts open with all open Private + Business tasks.
- Do not load completed archives during routine startup/status.
- Never commit raw `inbox/` / `processed/` captures.
- If the locked store becomes inaccessible, report that fact and do not silently switch mode or fall back to the public template.
