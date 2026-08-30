# Contributing to opendump

opendump is meant to be forked, copied, and bent to fit how you work.

## Personal tinkering

Change the workflow in your own template instance however you like — store layout, capture rules, generated adapters, naming, archives. That is expected. You do not need permission to experiment in your private copy.

Do not open PRs that only encode your personal task habits or contain real personal/business tasks. Do not push personal tasks into `JPilsinger/opendump`.

## Improvements to the shared template

Ideas and patches that make the public template better for everyone are welcome, for example:

- clearer cold-start / setup wording
- better host-independent adapter-generation rules
- store-mode edge cases
- capability-detection and verification improvements
- docs and examples that stay free of personal data

Prefer a short issue or PR that explains the general problem and the proposed default behavior.

## Ground rules

- Keep `private.md`, `business.md`, and completed archives empty of real tasks in this public repo.
- Treat `AGENTS.md` and `HARNESS_BOOTSTRAP.md` as the canonical sources.
- Keep the public template free of host-specific adapter files, rule directories, skills, bridges, project-instruction payloads, and environment-specific binding manifests.
- Host-specific adapters belong only in initialized user environments and must be generated from the canonical protocol.
- The installed host adapter is the sole persistent record of that host environment's active store mode and concrete location; do not duplicate the binding into the canonical store.
- Source-ingestion mechanisms such as watched directories, upload queues, staging folders, acknowledgement queues, and processed-source archives are integrations, not part of the canonical opendump store schema.
- The canonical task-state schema is `private.md`, `business.md`, `private-completed.md`, and `business-completed.md` (or equivalent logical sections in artifact mode).
- Generated project instructions must lock a single store mode and concrete location — no ambivalent “GitHub or local” branching.
