# Contributing to opendump

opendump is meant to be forked, copied, and bent to fit how you work.

## Personal tinkering

Change the workflow in your own **template instance** however you like — store layout, capture rules, adapters, naming, archives. That is expected. You do not need permission to experiment in your private copy.

Do **not** open PRs that only encode your personal task habits or contain real personal/business tasks. Do **not** push personal tasks into `JPilsinger/opendump`.

## Improvements to the shared template

Ideas and patches that make the **public** template better for everyone are welcome, for example:

- clearer cold-start / setup wording
- better harness adapters
- store-mode edge cases
- docs and examples that stay free of personal data

Prefer a short issue or PR that explains the general problem and the proposed default behavior.

## Ground rules

- Keep `private.md`, `business.md`, and completed archives empty of real tasks in this public repo.
- Treat `AGENTS.md` and `HARNESS_BOOTSTRAP.md` as the canonical sources; refresh derived adapters after semantic changes.
- Project instructions generated on cold start must lock a **single** store mode — no ambivalent “GitHub or local” branching.
