# Contributing to opendump

OpenDump is both a task workflow and a concrete test case for portable agentic workflows across AI hosts.

## Architectural boundary

Keep these roles distinct:

```text
AGENTS.md             canonical host-neutral runtime contract
HARNESS_BOOTSTRAP.md  host-neutral compiler / installer / verifier
host adapter          derived host-specific realization
four task collections authoritative workflow state
```

The host adapter is the host-specific manifestation of `AGENTS.md`, not merely a store-binding file. One logical adapter may span multiple host-native assets when necessary.

## Personal tinkering

Change your own instance however you like — task layout, capture rules, adapters, naming, or archives. You do not need permission to experiment in a private copy.

Do not open PRs that contain real personal/business tasks or only encode personal habits. Never push user tasks into `JPilsinger/opendump`.

## Improvements to the shared template

Useful contributions include:

- clearer host-neutral runtime semantics;
- better host/agentic-primitive discovery;
- simpler compilation and adapter-generation rules;
- linked vs compiled realization strategies;
- semantic coverage and verification improvements;
- store-mode/capability edge cases;
- conformance scenarios;
- ambiguity reduction;
- documentation/examples free of personal data.

Prefer a focused issue or PR that explains the general problem and proposed default behavior.

## Ground rules

- Keep `private.md`, `business.md`, and completed archives empty of real tasks in the public repository.
- `AGENTS.md` is the **sole canonical source of OpenDump runtime semantics**.
- Do not add host-specific rule filenames, skill formats, project settings, or adapter serialization to `AGENTS.md`; those belong to compilation/bootstrap.
- `HARNESS_BOOTSTRAP.md` is canonical for host discovery, capability establishment, store selection, adapter compilation/installation, recovery, and verification.
- Generated host adapters are derived/reproducible outputs. Do not hand-maintain them as a second workflow specification.
- A logical adapter may use multiple physical host-native assets, but only one managed OpenDump realization should be active for a host environment.
- Prefer reliable linkage to current `AGENTS.md` semantics when the host supports it; otherwise compile the full required runtime semantics faithfully.
- Adapter verification must cover semantic realization as well as the concrete store binding.
- Supporting a new AI host should normally change bootstrap/compiler knowledge, not fork `AGENTS.md` around that host.
- Keep the public template host-neutral: no product-specific adapters, rule directories, skills, bridges, project-instruction payloads, or environment-specific binding manifests.
- A GitHub store may be user-owned or otherwise explicitly authorized by the user; use **user-controlled** for both cases.
- The canonical task-state schema is `private.md`, `business.md`, `private-completed.md`, and `business-completed.md` (or equivalent logical collections in artifact mode).
- `private.md` and `business.md` must preserve `Backlog` and `In progress` sections.
- Source-ingestion mechanisms such as watched directories, upload queues, staging folders, acknowledgement queues, and processed-source archives are integrations, not canonical store state.
