# Contributing to opendump

opendump is meant to be forked, copied, and adapted.

## Personal tinkering

Change your own instance however you like — task layout, capture rules, adapters, naming, or archives. You do not need permission to experiment in a private copy.

Do not open PRs that contain real personal/business tasks or only encode personal habits. Never push user tasks into `JPilsinger/opendump`.

## Improvements to the shared template

Useful contributions include:

- clearer cold-start/setup behavior;
- simpler host-neutral adapter-generation rules;
- store-mode edge cases;
- capability detection and verification improvements;
- ambiguity reduction;
- documentation/examples free of personal data.

Prefer a focused issue or PR that explains the general problem and proposed default behavior.

## Ground rules

- Keep `private.md`, `business.md`, and completed archives empty of real tasks in the public repository.
- `AGENTS.md` is canonical for runtime task semantics.
- `HARNESS_BOOTSTRAP.md` is canonical for initialization, recovery, store selection, adapter generation, and verification.
- Keep the public template host-neutral: no product-specific adapters, rule directories, skills, bridges, project-instruction payloads, or environment-specific binding manifests.
- Generated host adapters belong only in initialized host environments.
- The installed host adapter is the sole persistent record of that host environment's store mode and concrete location.
- A GitHub store may be user-owned or otherwise explicitly authorized by the user; use **user-controlled** for both cases.
- The canonical task-state schema is `private.md`, `business.md`, `private-completed.md`, and `business-completed.md` (or equivalent logical collections in artifact mode).
- `private.md` and `business.md` must preserve `Backlog` and `In progress` sections.
- Source-ingestion mechanisms such as watched directories, upload queues, staging folders, acknowledgement queues, and processed-source archives are integrations, not canonical store state.
- Generated adapters must bind exactly one store mode and concrete location — no runtime “GitHub or local” fallback branching.
