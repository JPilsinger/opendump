# Host-neutral bootstrap protocol

This file defines opendump initialization, recovery, store selection, adapter generation, and verification. `AGENTS.md` defines normal runtime task semantics.

Cold start is a formal protocol: the host agent may use its intelligence to discover facts about its environment, but this protocol determines what those facts imply and what counts as success.

## Normative language

**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are normative.

- **MUST / MUST NOT** — required for correctness.
- **SHOULD / SHOULD NOT** — default behavior; deviate only for a concrete reason.
- **MAY** — optional.

## Authority and invariants

1. Platform/system/security policies have highest priority.
2. `AGENTS.md` is authoritative for normal runtime task semantics.
3. `HARNESS_BOOTSTRAP.md` is authoritative for initialization, recovery, store selection, adapter generation, and verification.
4. The installed host adapter is the sole persistent binding record for that host environment.
5. The bound store is authoritative for task state.

A **user-controlled store** is owned by the user or explicitly authorized by the user for opendump task storage.

The public template MUST remain host-neutral:

- no committed product-specific adapters, rules, skills, bridges, or project-instruction payloads;
- no store-side environment binding manifest;
- no host identity, repository path, branch, filesystem path, or adapter binding duplicated into canonical task state;
- host-specific adapters are generated only in initialized host environments and remain derived artifacts.

`AGENTS.md` is a portable runtime specification. Its presence MUST NOT be used as evidence of host identity.

## Cold-start trigger

Run this protocol when the user explicitly requests initialization, bootstrap, cold start, recovery of an invalid/missing binding, or a store mode/location change.

Preferred request:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

When the reference is `JPilsinger/opendump`, it is the public protocol/template reference only. User tasks MUST NOT be written there.

## Core principle

**Intelligence discovers facts. The protocol decides what those facts imply.**

The agent SHOULD reason freely when identifying the host, persistent instruction surface, repository context, available tools, permissions, and durable storage capabilities. Once those facts are established, transitions and success criteria are deterministic.

## Canonical store schema

For `github` and `local-files`, the canonical task-state schema consists of:

```text
private.md
business.md
private-completed.md
business-completed.md
```

`private.md` and `business.md` MUST expose both:

```text
## Backlog
## In progress
```

The two completed files are completed-task archives. They may contain no entries.

`AGENTS.md` and `HARNESS_BOOTSTRAP.md` accompany file/GitHub task state as canonical protocol documents. They are not task-state collections and MUST NOT record the active environment binding.

For `artifact`, the artifact MUST expose equivalent logical collections for Private/Business open state (Backlog + In progress) and Private/Business completed history.

Source transport/lifecycle mechanisms — uploads, watched directories, staging folders, acknowledgement ledgers, source archives, and processed-source directories — are outside the canonical store schema.

## Initialization state machine

Cold start MUST progress in this order:

| State | Required outcome |
|---|---|
| `UNINITIALIZED` | No verified binding accepted for this run |
| `ENVIRONMENT_IDENTIFIED` | Host and persistent instruction surface established |
| `CAPABILITIES_ESTABLISHED` | Relevant persistence capabilities established as facts |
| `STORE_SELECTED` | Exactly one store mode selected |
| `STORE_BOUND` | Concrete store/location satisfies its binding gate |
| `ADAPTER_INSTALLED` | Persistent host instructions contain exactly one concrete opendump binding |
| `VERIFIED` | Adapter read back and bound store independently verified |
| `READY` | Live open task state loaded |

Non-success terminal states:

- `BLOCKED` — correctness requires an external/user action or unavailable permission.
- `UNSUPPORTED` — no durable writable store/instruction arrangement can provide reliable tracking.

`READY` is the only successful terminal state. A later state MUST NOT be claimed until the current state's postconditions hold.

## Phase 1 — environment identification

Identify the host and native persistent instruction surface using the first conclusive evidence available, in this order:

1. platform/system-provided host identity;
2. native persistent-instruction APIs/surfaces;
3. host-specific workspace conventions observed in the current environment;
4. tool/connector signatures;
5. current official host documentation when materially needed;
6. user clarification only when remaining ambiguity prevents correct installation.

Do not infer host identity solely from portable repository files.

Normalize at least:

```yaml
host:
  name: <identified host or unknown>
  instruction_surface: <persistent surface or none>
  instruction_readable: <true|false>
  instruction_writable: <true|false>
```

### Instruction-surface selection

If multiple persistent instruction surfaces exist, select the **narrowest durable surface that reliably applies wherever opendump is expected to operate**.

Use these deterministic preferences when applicable:

1. the host's explicitly designated project/workspace instruction surface;
2. another project/workspace-scoped persistent surface;
3. a repository-scoped persistent surface when opendump sessions are reliably anchored to that repository;
4. a broader account/global surface only when narrower surfaces do not reliably apply.

Also:

- preserve unrelated instructions on shared surfaces;
- install only one opendump adapter;
- do not choose a narrower surface that fails to apply to expected sessions;
- do not enter `BLOCKED` merely because two reasonable surfaces exist — use host knowledge and the preferences above to resolve the choice;
- enter `BLOCKED` with reason `instruction_surface_selection_required` only when the agent cannot establish whether the remaining candidates will persist/apply correctly and choosing could make the binding unreliable.

Transition to `ENVIRONMENT_IDENTIFIED` once the host/surface facts are established.

## Phase 2 — capability establishment

Determine capabilities from actual tools, permissions, and durable surfaces. Do not assume capability because a product normally supports it.

Normalize when relevant:

```yaml
github:
  repo_read: <true|false>
  repo_write: <true|false>
  repo_create: <true|false>
  template_create: <true|false>
  setup_available: <true|false>
  setup_user_action: <concrete action or none>

files:
  durable_read: <true|false>
  durable_write: <true|false>

artifact:
  durable_read: <true|false>
  durable_write: <true|false>
```

A capability is `true` only when the current environment/session exposes a usable way to perform it. Treat unproven required capabilities as unavailable until proven otherwise.

`github.setup_available` is `true` only when the current host/session exposes a concrete action that can enable the missing GitHub capability without changing hosts. If true, `setup_user_action` MUST name that action. Generic advice to “set up GitHub” is insufficient.

Transition to `CAPABILITIES_ESTABLISHED` once enough facts exist to execute store selection mechanically.

## Repository role

Let `UPSTREAM = JPilsinger/opendump`.

Repository role is determined from actual repository identity, never copied metadata:

1. If the current repository full name equals `UPSTREAM`, it is the public upstream template and user task writes are forbidden.
2. Any different repository may be a user-controlled opendump store if it satisfies the structural/binding rules below.
3. Repository name similarity, copied files, fork/template ancestry, or generated adapter paths are not sufficient evidence of upstream identity.

The public template requires no seed/config manifest.

## Existing binding

Before selecting a new store, inspect the installed host adapter when available.

An existing binding is valid only if:

- exactly one `store` mode is declared;
- exactly one concrete `location` is declared;
- the declared store satisfies its current binding gate;
- a GitHub location is not `JPilsinger/opendump` or another known published upstream template.

If valid, reuse it. Do not select another store merely because another mode is preferred.

If the adapter binding is invalid, ambiguous, or inaccessible, do not silently fall back. Treat the run as uninitialized, preserve any recoverable canonical task state, and establish one new binding.

## Phase 3 — deterministic store selection

If no valid existing binding exists, select one mode using this procedure. A viable explicit user preference overrides defaults.

### GitHub structural signature

A file/GitHub opendump store has this signature:

```text
private.md
business.md
private-completed.md
business-completed.md
AGENTS.md
HARNESS_BOOTSTRAP.md
```

The open task files must also contain the required `Backlog` and `In progress` sections.

A GitHub repository is a resolvable candidate when at least one is true:

1. the user explicitly supplied that repository/location;
2. the current repository is non-upstream and has the structural signature;
3. an accessible user-controlled writable repository has the structural signature and canonical protocol docs identify it as opendump-compatible.

A valid installed adapter outranks structural discovery. An explicit user reference outranks structural matching.

Do not guess from repository name similarity. If multiple structurally valid candidates remain with no valid adapter or explicit reference distinguishing them, enter:

```text
BLOCKED
reason: github_store_selection_required
```

and present the concrete candidates.

### Selection algorithm

Evaluate in order:

1. **Explicit viable preference** — if the user explicitly requested `github`, `local-files`, or `artifact` and it is viable, select it.
2. **Resolvable GitHub store** — else if GitHub read/write exists and exactly one user-controlled candidate is resolvable, select `github`.
3. **Create GitHub store** — else if the host can create a repository and then read/write it, create a user-controlled repository and materialize the canonical opendump files. Prefer the host's native GitHub-template mechanism when `template_create == true`; otherwise create a normal repository and write the six canonical files.
4. **Concrete GitHub setup path** — else if `github.setup_available == true` and the user has not explicitly declined GitHub, enter `BLOCKED` and report the exact `setup_user_action`. Do not silently fall through.
5. **Durable local files** — else if durable file read/write exists, select `local-files`.
6. **Durable artifact** — else if a durable artifact can be read/written, select `artifact`.
7. **No durable store** — else enter `UNSUPPORTED`.

GitHub MUST never bind to `JPilsinger/opendump` or another known published upstream template.

Transition to `STORE_SELECTED` only when exactly one mode is determined.

## Phase 4 — store binding gates

Materialize/resolve the selected store and enforce its gate.

### `github`

All MUST be true:

- concrete repository full name is known;
- repository exists and is readable/writable in the current environment;
- repository is user-controlled;
- repository is not a known published upstream template;
- concrete branch/default ref is known/resolvable;
- the four canonical task-state files exist or are created;
- `private.md` and `business.md` expose `Backlog` and `In progress`;
- `AGENTS.md` and `HARNESS_BOOTSTRAP.md` exist or are materialized from the canonical template.

Canonical location:

```text
<owner>/<repo>@<branch>
```

### `local-files`

All MUST be true:

- concrete durable directory/path is known;
- the four canonical task-state files exist or are created and are readable/writable;
- `private.md` and `business.md` expose `Backlog` and `In progress`;
- `AGENTS.md` and `HARNESS_BOOTSTRAP.md` exist or are materialized.

### `artifact`

All MUST be true:

- concrete artifact identity is known;
- artifact exists or is created;
- artifact is durable, readable, and writable;
- it exposes equivalent Private/Business Backlog + In progress and completed-history collections.

No transport/staging structure or store-side environment-binding metadata is required.

Transition to `STORE_BOUND` only after the gate passes.

## Phase 5 — generate and install the host adapter

Persist the Phase 4 binding only in the selected native host instruction surface.

The generated adapter MUST:

- identify itself as opendump-derived;
- reference `AGENTS.md` / `HARNESS_BOOTSTRAP.md` when accessible, or preserve their required semantics when they are not;
- declare exactly one `store` mode and one concrete `location`;
- contain no runtime fallback branching;
- require startup reads from the bound store;
- require same-turn task persistence to that store;
- forbid `JPilsinger/opendump` and other known published upstreams as user task stores;
- require inaccessible store bindings to be reported instead of silently switched;
- preserve unrelated existing host/project instructions.

When syntax permits, use one replaceable managed block:

```markdown
<!-- OPENDUMP:BEGIN store=<mode> location=<concrete location> -->
...derived instructions for this binding only...
<!-- OPENDUMP:END -->
```

Repeated cold start MUST update/replace the existing adapter rather than append duplicates.

If the adapter can be generated but the instruction surface cannot be written, provide the exact manual payload and enter:

```yaml
opendump_initialization:
  status: blocked
  state: STORE_BOUND
  reason: manual_adapter_installation_required
```

Manual payload generation is not installation. After the user installs it, resume Phase 5 and verify the persisted content.

Transition to `ADAPTER_INSTALLED` only after the intended adapter actually exists on the persistent instruction surface.

## Phase 6 — mandatory verification

Verification is a hard gate. The agent MUST:

1. read the native persistent instruction surface back;
2. parse the installed opendump binding;
3. confirm exactly one mode and concrete location;
4. confirm the read-back binding equals the intended Phase 4 binding;
5. independently resolve that location and prove current store read access;
6. confirm the canonical task-state shape for the selected mode;
7. for file/GitHub modes, confirm `AGENTS.md` and `HARNESS_BOOTSTRAP.md` are accessible;
8. verify current store write capability using an already-successful materialization/write, authoritative permission evidence, or a safe reversible probe appropriate to the host.

If verification fails, repair and re-verify when possible; otherwise enter `BLOCKED` with the current state and concrete reason. Never claim installation succeeded without verification.

Transition to `VERIFIED` only after all applicable checks pass.

## Phase 7 — load live state and become ready

After verification:

1. read `private.md` and `business.md` (or artifact equivalents);
2. do not load completed archives unless needed for a specific conflict/migration;
3. surface current Backlog + In progress succinctly;
4. continue under `AGENTS.md` runtime semantics.

Only now transition to `READY`.

Semantic success result:

```yaml
opendump_initialization:
  status: ready
  state: READY
  host: <host>
  instruction_surface: <surface>
  store: <github|local-files|artifact>
  location: <concrete location>
  store_verified: true
  adapter_verified: true
  tasks_loaded: true
```

The user-facing response may be conversational but MUST accurately communicate the concrete store and successful verification.

## Blocked and unsupported results

When blocked, report the last satisfied state, concrete reason, and minimum next action.

When unsupported, explicitly state that reliable task tracking is unavailable in the current host. Do not treat chat memory as the task database.

Never convert `BLOCKED` or `UNSUPPORTED` into a weaker silent success.

## Normal startup after initialization

Normal sessions do not rerun this protocol.

At startup:

1. read the installed host adapter and resolve its locked store/location;
2. read live open tasks from that store;
3. for file/GitHub modes, read current `AGENTS.md` when accessible;
4. do not read this bootstrap document unless cold start/recovery is actually required;
5. if the adapter is missing/invalid or the store is inaccessible, report the failure rather than guessing or silently switching.

Run full cold start again only when explicitly requested, when the binding is missing/invalid, for recovery, or for a requested mode/location change.

## Minimum adapter semantics

Every generated adapter MUST preserve:

- exactly one store mode and concrete location;
- adapter as sole persistent binding record for that host environment;
- public upstream never used as user task store;
- startup reads live tasks from the bound store;
- same-turn task mutations persist only to that store;
- no silent mode switching;
- explicit cold start invokes this protocol;
- manual adapter text is not installed until persisted and verified;
- inaccessible bound store is reported accurately;
- normal task semantics come from `AGENTS.md`;
- source transport/staging is outside canonical task state.

## Migration / mode change

A mode/location change is explicit re-initialization:

1. preserve existing canonical task state;
2. materialize/copy it into the target store;
3. run target binding, adapter installation, and verification;
4. replace the old adapter binding only after the target verifies;
5. never operate two authoritative stores in parallel.

There is no store-side binding manifest to migrate.

## Maintenance

- Change runtime task semantics in `AGENTS.md`.
- Change initialization/recovery semantics here.
- Keep the public template free of committed host-specific adapters and environment-specific binding manifests.
- Keep source-ingestion transport/staging outside the canonical store schema.
- Regenerate downstream adapters after relevant protocol changes.
- Never copy a generated adapter into the public template as a compatibility shortcut.
