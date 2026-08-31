# Host-neutral bootstrap / compiler protocol

This file defines how OpenDump is realized on an arbitrary AI host. `AGENTS.md` is the canonical host-neutral runtime contract; this document is the compiler, installer, recovery, and verification protocol for turning that contract into a working host-specific realization.

The architecture is:

```text
AGENTS.md
    WHAT OpenDump means
        ↓
HARNESS_BOOTSTRAP.md
    HOW to realize it on this host
        ↓
Host adapter
    THE realized host-specific implementation
        ↓
Native host agentic assets + bound task state
```

Conceptually:

```text
OpenDumpAdapter = Compile(AGENTS.md, HostEnvironment, StoreBinding)
```

Correctness is semantic, not textual: executing OpenDump through the installed adapter on the target host MUST preserve the runtime obligations of `AGENTS.md` under the selected store binding, subject only to higher-priority platform/security policy and genuinely unavailable capabilities.

## Normative language

**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are normative.

- **MUST / MUST NOT** — required for correctness.
- **SHOULD / SHOULD NOT** — default behavior; deviate only for a concrete reason.
- **MAY** — optional.

## Core principle

**Intelligence discovers facts. The protocol decides what those facts imply.**

The agent SHOULD reason freely when identifying the host, native agentic primitives, available tools, permissions, repository context, and durable storage. Once those facts are established, state transitions, store selection, compilation requirements, and success criteria are deterministic.

## Core definitions

### Workflow contract

`AGENTS.md` is the sole canonical source of OpenDump runtime semantics.

### Host environment

The observed AI platform and execution context, including persistent instruction mechanisms, rule/skill systems, automatic context loading, tools/connectors, storage primitives, permissions, scope, and other native agentic assets relevant to realizing OpenDump.

### Store binding

The concrete authoritative OpenDump task store selected for this host environment: one mode plus one concrete location.

### Host adapter

The complete **host-specific realization of `AGENTS.md`**, parameterized by the store binding.

A host adapter is a **logical artifact**. It MAY consist of one physical file or field, or several cooperating host-native assets such as project instructions, repository rules, skills, persistent settings, or other native mechanisms.

The adapter is derived compilation output. It MUST NOT become a second source of OpenDump workflow truth.

### Linked, compiled, and hybrid realization

- **Linked** — the host can reliably consume current `AGENTS.md` runtime semantics directly; the adapter supplies activation, binding, and host-specific integration.
- **Compiled** — direct runtime consumption is unavailable; required `AGENTS.md` semantics are faithfully translated into host-native assets.
- **Hybrid** — some semantics are linked while other required behavior is compiled into host-native assets.

Prefer linked realization when it is reliable because it minimizes semantic duplication.

## Authority and invariants

1. Platform/system/security policies have highest priority.
2. `AGENTS.md` is authoritative for OpenDump runtime semantics.
3. `HARNESS_BOOTSTRAP.md` is authoritative for host discovery, compilation, installation, recovery, store selection, and verification.
4. The installed host adapter is authoritative for the host-specific realization and concrete store binding, but MUST NOT redefine canonical runtime semantics.
5. The bound store is authoritative for task state.

A **user-controlled store** is owned by the user or explicitly authorized by the user for OpenDump task storage.

The public template MUST remain host-neutral:

- no committed product-specific adapters, rules, skills, bridges, or project-instruction payloads;
- no store-side environment binding manifest;
- no host identity, repository path, branch, filesystem path, or adapter binding duplicated into canonical task state;
- generated host adapters remain downstream derived artifacts.

`AGENTS.md` is portable workflow source. Its presence MUST NOT be used as evidence of host identity.

## Cold-start trigger

Run this protocol when the user explicitly requests initialization, bootstrap, cold start, recovery of an invalid/missing adapter, adapter recompilation, or a store mode/location change.

Preferred request:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

When the reference is `JPilsinger/opendump`, it is the public protocol/template reference only. User tasks MUST NOT be written there.

## Canonical store schema

For `github` and `local-files`, canonical task state consists of:

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

The completed files are completed-task archives and may contain no entries.

`AGENTS.md` and `HARNESS_BOOTSTRAP.md` accompany file/GitHub task state as canonical protocol documents. They are not task-state collections and MUST NOT record the active environment binding.

For `artifact`, the artifact MUST expose equivalent logical collections for Private/Business open state and Private/Business completed history.

Source transport/lifecycle mechanisms — uploads, watched directories, staging folders, acknowledgement ledgers, source archives, and processed-source directories — are outside the canonical OpenDump schema.

## Initialization state machine

Cold start MUST progress in this order:

| State | Required outcome |
|---|---|
| `UNINITIALIZED` | No verified host realization accepted for this run |
| `ENVIRONMENT_IDENTIFIED` | Host and applicable native agentic primitives established |
| `CAPABILITIES_ESTABLISHED` | Relevant capabilities and permissions established as facts |
| `STORE_SELECTED` | Exactly one store mode selected |
| `STORE_BOUND` | Concrete store/location satisfies its binding gate |
| `ADAPTER_INSTALLED` | One logical host adapter is installed in the required native assets |
| `VERIFIED` | Adapter realization, binding, and store access are verified |
| `READY` | Live open task state is loaded under the verified realization |

Non-success terminal states:

- `BLOCKED` — correctness requires an external/user action or unavailable permission.
- `UNSUPPORTED` — the host cannot provide the durable agentic/storage primitives required for a faithful OpenDump realization.

`READY` is the only successful terminal state. A later state MUST NOT be claimed until the current state's postconditions hold.

## Phase 1 — environment and agentic-primitive discovery

Identify the host using the first conclusive evidence available, in this order:

1. platform/system-provided host identity;
2. native persistent agentic APIs/surfaces;
3. host-specific workspace conventions observed in the current environment;
4. tool/connector signatures;
5. current official host documentation when materially needed;
6. user clarification only when remaining ambiguity prevents correct realization.

Do not infer host identity solely from portable repository files.

Discover the smallest set of durable host-native primitives that can make OpenDump effective in the sessions where it is expected to operate. Relevant primitives MAY include:

- project/workspace instructions;
- repository-scoped instructions;
- rule systems;
- skills;
- persistent settings/configuration;
- native automatic loading of `AGENTS.md` or equivalent context;
- tools/connectors required by the workflow;
- other durable agentic assets exposed by the host.

Normalize at least enough information to reason about:

```yaml
host:
  name: <identified host or unknown>
  adapter_surfaces:
    - kind: <native agentic asset/surface>
      scope: <project|workspace|repository|account|other>
      readable: <true|false>
      writable: <true|false>
  canonical_contract_runtime_access: <true|false>
```

### Adapter-surface selection

Select the **smallest durable set of native assets that reliably realizes the OpenDump runtime contract**.

Use these preferences when applicable:

1. native reliable consumption of current `AGENTS.md` semantics;
2. the host's designated project/workspace instruction mechanism;
3. other project/workspace-scoped durable mechanisms;
4. repository-scoped mechanisms when OpenDump sessions are reliably anchored there;
5. broader account/global mechanisms only when narrower assets do not reliably apply.

Also:

- preserve unrelated instructions/configuration on shared assets;
- one logical OpenDump adapter MAY span multiple physical assets when required;
- do not create multiple competing logical OpenDump adapters merely because several surfaces are available;
- do not choose a narrower surface that fails to apply to expected sessions;
- do not enter `BLOCKED` merely because multiple reasonable realizations exist — use host knowledge and these preferences;
- enter `BLOCKED` with reason `adapter_surface_selection_required` only when the agent cannot establish whether remaining candidates persist/apply correctly and choosing could make the workflow unreliable.

Transition to `ENVIRONMENT_IDENTIFIED` once the host and applicable realization primitives are established.

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

Transition to `CAPABILITIES_ESTABLISHED` once enough facts exist to execute store selection mechanically and compile a faithful adapter.

## Repository role

Let `UPSTREAM = JPilsinger/opendump`.

Repository role is determined from actual repository identity, never copied metadata:

1. If the current repository full name equals `UPSTREAM`, it is the public upstream template and user task writes are forbidden.
2. Any different repository may be a user-controlled OpenDump store if it satisfies the structural/binding rules below.
3. Repository name similarity, copied files, fork/template ancestry, or generated adapter paths are not sufficient evidence of upstream identity.

The public template requires no seed/config manifest.

## Existing adapter and binding

Before selecting a new store, inspect the installed logical host adapter when available.

Evaluate **binding validity** separately from **realization validity**.

A reusable binding requires:

- exactly one authoritative `store` mode;
- exactly one concrete `location`;
- the declared store currently satisfies its binding gate;
- a GitHub location is not `JPilsinger/opendump` or another known published upstream template.

If the binding is valid, preserve it even when the adapter's semantic realization needs recompilation. Do not select a different store merely because the adapter assets are stale, incomplete, or need repair.

A reusable adapter additionally requires that its host-native realization still satisfies the semantic verification requirements in Phase 6.

If the binding is missing, ambiguous, invalid, or inaccessible, do not silently fall back. Preserve recoverable canonical task state and establish one new binding using this protocol.

## Phase 3 — deterministic store selection

If no reusable binding exists, select one mode using this procedure. A viable explicit user preference overrides defaults.

### GitHub structural signature

A file/GitHub OpenDump store has this signature:

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
3. an accessible user-controlled writable repository has the structural signature and canonical protocol docs identify it as OpenDump-compatible.

A valid installed binding outranks structural discovery. An explicit user reference outranks structural matching.

Do not guess from repository name similarity. If multiple structurally valid candidates remain with no valid binding or explicit reference distinguishing them, enter:

```text
BLOCKED
reason: github_store_selection_required
```

and present the concrete candidates.

### Selection algorithm

Evaluate in order:

1. **Explicit viable preference** — if the user explicitly requested `github`, `local-files`, or `artifact` and it is viable, select it.
2. **Resolvable GitHub store** — else if GitHub read/write exists and exactly one user-controlled candidate is resolvable, select `github`.
3. **Create GitHub store** — else if the host can create a repository and then read/write it, create a user-controlled repository and materialize the canonical OpenDump files. Prefer the host's native GitHub-template mechanism when `template_create == true`; otherwise create a normal repository and write the six canonical files.
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

## Phase 5 — compile and install the host adapter

Compile one logical OpenDump adapter from:

```text
AGENTS.md + observed host environment + Phase 4 store binding
```

### 5.1 Build a realization plan

Map every runtime requirement in `AGENTS.md` that is necessary for execution to a concrete host realization.

The plan MAY be transient. Do not create another persistent manifest merely to record it.

Conceptually:

```text
OpenDump requirement             Host realization
------------------------------   --------------------------------
runtime contract active        → native AGENTS loading / project instructions
single authoritative store     → managed binding in native adapter asset
load live open task state      → host tool/connector capability
same-turn persistence          → host write capability + persistent instructions
status/task lifecycle behavior → linked or compiled AGENTS semantics
failure on inaccessible store  → linked or compiled AGENTS semantics
```

The completeness boundary is the canonical `AGENTS.md` contract itself. Do **not** maintain a separate hand-authored “minimum adapter semantics” specification.

### 5.2 Choose realization strategy

Prefer **linked** realization when the host can reliably consume current `AGENTS.md` semantics in every expected OpenDump session.

Use **compiled** realization when direct linkage is unavailable: translate the required runtime semantics into host-native assets faithfully.

Use **hybrid** realization when appropriate.

The adapter MUST NOT omit a runtime requirement merely because the host expresses it differently. If a required semantic cannot be realized faithfully, do not claim successful installation.

### 5.3 Install the logical adapter

The installed adapter MUST:

- identify itself as OpenDump-derived where the host permits identification;
- make the canonical `AGENTS.md` runtime contract effective through linked, compiled, or hybrid realization;
- contain exactly one authoritative store mode/location for this host environment;
- contain no silent runtime store fallback unless the canonical workflow contract explicitly defines such behavior;
- activate reliably in the sessions where OpenDump is expected to operate;
- preserve unrelated existing host/project instructions and configuration;
- remain one managed logical realization even when multiple physical native assets are required.

When a shared textual surface is used and the syntax permits it, use a replaceable managed region such as:

```markdown
<!-- OPENDUMP:BEGIN -->
...host-specific realization and binding...
<!-- OPENDUMP:END -->
```

Binding metadata MAY appear inside that managed region, but the region's purpose is the OpenDump host realization as a whole, not merely store binding.

Repeated cold start/recompilation MUST update or reconcile the existing logical adapter rather than append competing variants.

If one or more required native assets can be generated but cannot be written by the agent, provide the exact manual installation payload(s) and enter:

```yaml
opendump_initialization:
  status: blocked
  state: STORE_BOUND
  reason: manual_adapter_installation_required
```

Generated payload is not installation. After the user installs all required assets, resume Phase 5 and verify the persisted realization.

Transition to `ADAPTER_INSTALLED` only after all physical assets required by the intended logical adapter actually exist in their persistent host-native locations.

## Phase 6 — mandatory semantic and binding verification

Verification is a hard gate. Persisted assets alone do not prove a working OpenDump realization.

The agent MUST verify:

1. **Read-back** — all managed native assets that constitute the logical adapter can be read/observed after installation where the host permits read-back.
2. **Uniqueness** — exactly one logical OpenDump realization is active; no conflicting duplicate adapter/binding is present.
3. **Activation** — the selected host-native assets will actually apply in the sessions where OpenDump is expected to operate.
4. **Binding** — exactly one store mode/location is authoritative and equals the intended Phase 4 binding.
5. **Semantic coverage** — every runtime requirement in `AGENTS.md` necessary for execution is either reliably linked to the canonical contract or faithfully compiled into the installed host-native realization.
6. **Linked-contract validity** — for linked/hybrid realization, current `AGENTS.md` is reliably available/effective through the claimed native linkage.
7. **Compiled-contract validity** — for compiled/hybrid realization, the translated assets faithfully preserve the required semantics and are treated as derived output rather than independent source.
8. **Store read access** — independently resolve the bound location and prove current read access.
9. **State shape** — confirm the canonical task-state shape for the selected mode.
10. **Store write capability** — verify current write capability using an already-successful materialization/write, authoritative permission evidence, or a safe reversible probe appropriate to the host.

A semantic coverage matrix MAY be reasoned about transiently, for example:

| Canonical obligation | Host realization | Strategy | Verified |
|---|---|---|---|
| runtime contract active | project instructions + `AGENTS.md` | linked | yes |
| store binding | managed project instruction | compiled | yes |
| task reads/writes | GitHub connector | runtime capability | yes |

Do not persist this matrix unless the host requires it for its native adapter representation.

If verification fails, repair/recompile and re-verify when possible; otherwise enter `BLOCKED` with the last satisfied state and concrete reason. Never claim installation succeeded without verification.

Transition to `VERIFIED` only after all applicable checks pass.

## Phase 7 — load live state and become ready

After verification:

1. execute under the installed adapter realization;
2. load `private.md` and `business.md` (or artifact equivalents) from the verified bound store;
3. do not load completed archives unless needed for a specific conflict/migration;
4. surface current Backlog + In progress succinctly.

Only now transition to `READY`.

Semantic success result:

```yaml
opendump_initialization:
  status: ready
  state: READY
  host: <host>
  adapter:
    strategy: <linked|compiled|hybrid>
    surfaces: <host-native assets>
  store: <github|local-files|artifact>
  location: <concrete location>
  store_verified: true
  adapter_verified: true
  semantics_verified: true
  tasks_loaded: true
```

The user-facing response may be conversational but MUST accurately communicate the concrete store and successful host realization.

## Blocked and unsupported results

When blocked, report the last satisfied state, concrete reason, and minimum next action.

When unsupported, explicitly state which required durable agentic/storage primitive cannot be realized. Do not treat chat memory as the task database and do not silently compile a weaker workflow.

Never convert `BLOCKED` or `UNSUPPORTED` into a weaker silent success.

## Normal runtime after initialization

Normal sessions do not rerun this compiler protocol.

The installed host adapter MUST make the canonical OpenDump runtime semantics effective in each expected session through its verified linked/compiled/hybrid realization. Runtime then resolves the adapter's authoritative store binding and loads live task state according to `AGENTS.md`.

Do not require every host to perform an explicit file read of `AGENTS.md` if the host already realizes the contract faithfully through another verified native mechanism.

If the adapter is missing, conflicting, known stale, or invalid, or if the bound store is inaccessible, report the failure and invoke recovery/recompilation when appropriate rather than guessing or silently switching.

Run full cold start/recompilation again when explicitly requested, when the adapter/binding is missing or invalid, when an embedded realization is known stale, for recovery, or for a requested mode/location change.

## Adapter update and regeneration

The host adapter is a build artifact.

When `AGENTS.md` runtime semantics change:

- a linked realization MAY continue using the current contract if its linkage is still reliable;
- a compiled/hybrid realization MUST be regenerated for any changed semantics that are embedded in host-native assets.

When host-native primitives or runtime bindings change, re-evaluate the realization and recompile as needed.

Where useful and supported by the host, generated adapter assets MAY include source provenance/revision metadata to help detect staleness. Such metadata is advisory compilation provenance, not semantic authority.

## Migration / mode change

A mode/location change is explicit re-initialization:

1. preserve existing canonical task state;
2. materialize/copy it into the target store;
3. run target binding, adapter compilation/installation, and verification;
4. replace the old adapter binding only after the target verifies;
5. never operate two authoritative stores in parallel.

There is no store-side binding manifest to migrate.

## Maintenance

- Change OpenDump runtime semantics only in `AGENTS.md`.
- Change host discovery, compilation, installation, recovery, store selection, or verification here.
- Keep the public template free of committed host-specific adapter assets and environment-specific binding manifests.
- Keep source-ingestion transport/staging outside the canonical store schema.
- Treat every generated adapter as derived/reproducible output.
- Never promote a generated host adapter into a competing canonical workflow specification.
- Supporting a new AI host SHOULD require better host discovery/compilation logic, not host-specific forks of `AGENTS.md`.
