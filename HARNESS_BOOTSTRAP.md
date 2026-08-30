# Harness bootstrap protocol

This file defines the harness-independent initialization protocol for opendump. `AGENTS.md` defines normal task workflow semantics. Cold start is a **formal protocol**: the host agent may use its intelligence to discover facts about its own environment, but the protocol determines what those facts imply and what counts as success.

## Normative language

The terms **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are normative.

- **MUST / MUST NOT** — required for correctness.
- **SHOULD / SHOULD NOT** — default behavior; deviate only for a concrete, explainable reason.
- **MAY** — genuinely optional.

Initialization prose and examples never override normative requirements.

## Authority and precedence

1. Platform/system/security policies always have highest priority.
2. `AGENTS.md` is authoritative for normal task workflow semantics and store-mode rules.
3. `HARNESS_BOOTSTRAP.md` is authoritative for initialization, capability detection, store selection, adapter installation, and verification.
4. The locked user-owned store is authoritative for task state.
5. Host-native instruction files/fields are derived adapters. Their locked `store` and `location` determine where that host reads/writes, but they MUST NOT redefine workflow semantics.

If a derived adapter conflicts with canonical workflow semantics, regenerate it. If its store binding is absent, ambiguous, or invalid, initialization is not complete.

## Cold-start trigger

Run this protocol when the user explicitly requests cold start, initialization, bootstrap, or sends the preferred one-liner:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

Equivalent requests naming another opendump checkout/reference also count.

When the reference is `JPilsinger/opendump`, it is the public workflow/template reference only. User tasks MUST NOT be written there.

## Core principle

**Intelligence discovers facts. The protocol decides what those facts imply.**

The agent SHOULD reason freely when identifying its host harness, native instruction surface, available tools, repository context, and persistence capabilities. Once those facts are established, state transitions, store selection, success criteria, and failure behavior are deterministic.

## Initialization state machine

Cold start MUST progress through these states in order. A state may perform multiple operations, but a later state MUST NOT be claimed until the current state's postconditions are satisfied.

| State | Required outcome |
|---|---|
| `UNINITIALIZED` | No verified active binding has yet been accepted for this cold start |
| `ENVIRONMENT_IDENTIFIED` | Host harness and native persistent-instruction surface are identified, or explicitly unknown |
| `CAPABILITIES_ESTABLISHED` | Relevant persistence capabilities are established as facts |
| `STORE_SELECTED` | Exactly one store mode is selected by the deterministic selection procedure |
| `STORE_BOUND` | A concrete store exists and satisfies the mode's binding gate |
| `ADAPTER_INSTALLED` | Host persistent instructions contain exactly one concrete opendump binding |
| `VERIFIED` | Store and adapter are read back and match the intended binding |
| `READY` | Live open tasks are loaded from the verified store |

Two explicit non-success terminal states also exist:

- `BLOCKED` — the environment could support initialization, but a required external/user action or unavailable permission prevents the next required transition.
- `UNSUPPORTED` — no durable writable store/instruction arrangement exists that can provide reliable tracking.

`READY` is the **only successful terminal state**. There is no partial-success state. Producing instructions for manual installation, creating a repository without installing an adapter, or writing an adapter without read-back verification MUST NOT be described as successful initialization.

## Phase 1 — environment identification

The agent MUST identify the host harness and its native persistent instruction surface using the first conclusive evidence available in this order:

1. Platform/system-provided host identity.
2. Native persistent-instruction APIs or surfaces exposed by the host.
3. Host-specific workspace conventions.
4. Available tool/connector signatures.
5. Current official host documentation when external lookup is available and materially needed.
6. User clarification only when identity remains materially ambiguous and the ambiguity blocks correct installation.

The agent MUST NOT infer a specific harness solely from a portable file such as `AGENTS.md`, `README.md`, or another file that may exist in multiple hosts.

Required normalized facts:

```yaml
host:
  name: <identified harness or unknown>
  instruction_surface: <native persistent surface or none>
  instruction_readable: <true|false>
  instruction_writable: <true|false>
```

Transition to `ENVIRONMENT_IDENTIFIED` only after these facts are established. An unfamiliar harness is not an error; continue by capability rather than vendor name.

## Phase 2 — capability establishment

Determine capabilities by observing actual tools, permissions, and durable surfaces. Do not assume a capability merely because a product normally supports it.

Normalize at least these facts when relevant:

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

A capability is `true` only when the current environment/session exposes a usable way to perform it. When a capability cannot be proven and is required for a candidate mode, treat it as unavailable until proven otherwise.

`github.setup_available` is `true` only when the current host/session exposes a **concrete, actionable path** by which the user can enable the missing GitHub capability without changing to another harness, such as connecting/authenticating GitHub, approving repository access, creating/selecting a template instance through an exposed UI, or completing an equivalent host-supported setup action. If true, `setup_user_action` MUST name that exact action. Mere theoretical GitHub support or generic advice to "set up GitHub" is not enough.

Transition to `CAPABILITIES_ESTABLISHED` only after enough facts exist to execute store selection mechanically.

## Repository-role determination

Repository role MUST be derived from the **actual repository identity**, not solely from inherited template files.

Let `UPSTREAM = JPilsinger/opendump`.

1. If the current repository full name equals `UPSTREAM`, its role is `public-upstream-template` and user task writes are forbidden.
2. If the current repository full name does **not** equal `UPSTREAM` and it contains inherited seed metadata such as `role: template-seed`, `initialized: false`, or legacy `role: public-upstream-template`, treat that metadata as an **uninitialized template copy**, not evidence that the new repository is the public upstream.
3. During successful initialization of a user-owned file/GitHub store, replace inherited seed/legacy metadata with the actual locked binding.
4. Never use repository name similarity, fork/template ancestry, or copied adapter text as sufficient evidence that a repository is the public upstream; compare concrete repository identity when available.

This rule exists because GitHub template creation copies files verbatim.

## Existing-binding rule

Before selecting a new store, inspect the host adapter and `opendump.config.md` when accessible.

An existing binding is valid only if all are true:

- exactly one `store` mode is declared;
- exactly one concrete `location` is declared when the mode requires one;
- the declared store satisfies its binding gate below;
- `github` location is not `JPilsinger/opendump`;
- host adapter and store config do not materially disagree.

If a valid existing binding is present, reuse it. Do not select another store merely because another mode is preferable.

If a binding exists but is invalid or contradictory, do not silently choose one side. Treat cold start as uninitialized and establish a new single binding using this protocol; preserve task data from the old store unless the user explicitly requests migration/deletion.

## Phase 3 — deterministic store selection

If no valid existing binding is present, select the store using this procedure. Do not ask the user to choose merely because multiple technical modes are available. A user preference stated before or during initialization overrides the default precedence if it names a viable mode.

### GitHub candidate resolution

A GitHub repository is a resolvable opendump candidate only when at least one of these is true:

1. the user explicitly supplied that repository/location;
2. the current repository is a non-upstream template copy containing the expected opendump layout/seed metadata;
3. an accessible repository contains an initialized opendump config whose `upstream` is `JPilsinger/opendump` and whose declared location matches itself.

Do not guess a store from repository name similarity alone. If more than one candidate remains and none is distinguished by an existing valid binding or explicit user reference, enter `BLOCKED` with reason `github_store_selection_required` and present the concrete candidates.

### Selection algorithm

Evaluate in order:

1. **Explicit viable user preference** — If the user explicitly requested `github`, `local-files`, or `artifact` and that mode is currently viable, select it and skip lower-priority alternatives.
2. **Resolvable user-owned GitHub store** — Else if GitHub read/write capability exists and exactly one user-owned opendump repository is resolvable under the candidate rules above, select `github`.
3. **Create user-owned GitHub store** — Else if the environment can create a user-owned repository from the template and then read/write it, create it and select `github`.
4. **Concrete GitHub setup path exists** — Else if `github.setup_available == true` and the user has not explicitly declined GitHub, enter `BLOCKED` with the concrete `setup_user_action`. Do not silently fall through to a weaker mode.
5. **Durable local files** — Else if durable file read/write capability exists, select `local-files`.
6. **Durable artifact** — Else if a durable artifact can be read and written, select `artifact`.
7. **No durable store** — Else enter `UNSUPPORTED`.

`github` MUST never bind to `JPilsinger/opendump` or another known published upstream template as the user's task database.

Transition to `STORE_SELECTED` only when exactly one mode is determined.

## Phase 4 — store binding gates

A selected mode is not yet bound. Materialize/resolve the store and enforce the relevant gate.

### `github` binding gate

All MUST be true:

- concrete repository full name is known;
- repository exists;
- repository is readable in the current environment;
- repository is writable in the current environment;
- repository is not `JPilsinger/opendump`;
- required task files/layout exist or are created from the template;
- concrete branch/default ref is known or resolvable.

Canonical location format:

```text
<owner>/<repo>@<branch>
```

### `local-files` binding gate

All MUST be true:

- concrete directory/path is known;
- required opendump markdown layout exists or is created;
- task files are readable;
- task files are writable;
- location is durable beyond the current response/session according to the host's file model.

### `artifact` binding gate

All MUST be true:

- concrete artifact identity/name is known;
- artifact exists or is created;
- artifact is readable;
- artifact is writable;
- artifact is durable according to the host;
- it contains/mirrors Private + Business Backlog/In progress and completed archive semantics.

### `unsupported`

No fake store/location may be created. Reliable capture MUST remain disabled.

Transition to `STORE_BOUND` only after the selected mode's gate passes.

## Phase 5 — record the binding

For `github` and `local-files`, the store MUST contain `opendump.config.md` with the actual binding. Replace template seed or legacy upstream metadata during initialization.

Canonical form:

```markdown
# opendump config

- initialized: true
- store: github
- location: <owner>/<repo>@<branch>
- harness: <identified harness>
- locked: YYYY-MM-DD
- upstream: JPilsinger/opendump
```

Use the equivalent concrete path for `local-files`. For `artifact`, the adapter MUST record the artifact identity; record equivalent metadata inside the artifact when the host supports it without harming the task layout.

Credentials, access tokens, private keys, cookies, and connector secrets MUST NOT be written into configuration or persistent instructions.

## Phase 6 — install the host adapter

Locate the host's native persistent instruction surface discovered in Phase 1.

The installed adapter MUST:

- identify itself as opendump-derived;
- reference `AGENTS.md` / `HARNESS_BOOTSTRAP.md` when those canonical files are accessible, or faithfully mirror their minimum semantics when they are not;
- declare exactly one `store` mode;
- declare exactly one concrete `location` for `github`, `local-files`, or `artifact`;
- contain no runtime branching such as "GitHub if available, otherwise local";
- require startup reads from the bound store;
- require same-turn task persistence to the bound store;
- forbid writes to `JPilsinger/opendump` as a user task store;
- state that unavailable store access must be reported rather than silently switching modes;
- preserve unrelated existing host/project instructions.

When the instruction surface is shared, use one replaceable managed block:

```markdown
<!-- OPENDUMP:BEGIN store=<mode> location=<concrete location> -->
...derived instructions for this binding only...
<!-- OPENDUMP:END -->
```

Repeated cold start MUST replace/update the existing managed block rather than append duplicates.

If the agent can generate but cannot write the native instruction surface, it MUST provide the exact payload for manual installation and enter:

```text
BLOCKED
reason: manual_adapter_installation_required
```

Generating the payload is not installation. After the user installs it, cold start resumes at this phase and MUST verify the installed content before success.

Transition to `ADAPTER_INSTALLED` only after the persistent surface actually contains the intended adapter.

## Phase 7 — mandatory verification

Verification is a hard gate, not a best-effort step.

The agent MUST:

1. Read the native persistent instruction surface back after installation.
2. Parse/read the installed opendump binding.
3. Confirm exactly one mode and one concrete location.
4. Confirm the read-back binding equals the intended binding.
5. Read `opendump.config.md` for file/GitHub modes and confirm it agrees with the adapter.
6. Re-read enough of the bound store to prove current read access.
7. Where write verification has not already occurred during materialization/config update, perform a non-destructive write-capability verification appropriate to the host or rely on a successful configuration write just performed.

If any verification fails, do not claim installation succeeded. Repair and re-verify when possible; otherwise enter `BLOCKED` with the failing gate and reason.

Transition to `VERIFIED` only after all applicable checks pass.

## Phase 8 — load live state and become ready

After verification:

1. Read `private.md` and `business.md` (or artifact equivalents).
2. Do not load completed archives during routine initialization unless needed to resolve a specific conflict/migration.
3. Surface current Backlog + In progress succinctly.
4. Continue normal operation under `AGENTS.md`.

Only now transition to `READY`.

A successful initialization result is equivalent to:

```yaml
opendump_initialization:
  status: ready
  harness: <host>
  instruction_surface: <surface>
  store: <github|local-files|artifact>
  location: <concrete location>
  store_verified: true
  adapter_verified: true
  tasks_loaded: true
```

The user-facing response MAY be conversational and does not need to print this YAML verbatim, but it MUST communicate the concrete store and successful verification accurately.

## Blocked and unsupported results

When blocked, state the exact phase/reason and the minimum required next action. Example semantic result:

```yaml
opendump_initialization:
  status: blocked
  phase: ADAPTER_INSTALLED
  reason: manual_adapter_installation_required
```

When unsupported, explicitly state that reliable task tracking is unavailable in the current harness and do not pretend chat memory is the database.

Do not convert `BLOCKED` or `UNSUPPORTED` into a weaker silent success.

## Startup behavior after initialization

Normal sessions do not rerun cold start automatically.

At startup:

1. Read the installed host adapter and identify the locked store/location.
2. Read live open tasks from that store.
3. When file/GitHub canonical workflow docs are accessible, follow current `AGENTS.md` semantics.
4. If the bound store is inaccessible, report the failure and stop claiming persistence. Do not switch stores silently.
5. Run this full cold-start protocol again only when explicitly requested, when the binding is invalid, or when the user requests a mode/location change.

## Known harness implementation notes

These are implementation hints, not alternate decision algorithms. The state machine and capability gates above remain authoritative.

### Claude Code

Use project `CLAUDE.md` / `.claude/CLAUDE.md`. When the opendump store is the workspace, a small bridge may import canonical files and state the binding. When installing into another project, preserve unrelated instructions and update a dedicated managed block.

### Claude Projects / Claude for Work

Use Project Instructions as the native persistent surface. If programmatic writes are unavailable, provide the exact managed block and enter `BLOCKED` until the user installs it and it can be verified.

### Cursor

Use an always-applied `.cursor/rules/opendump.mdc` rule and keep the opendump skill mirror aligned. The generated user-instance adapter must contain one concrete store binding.

### OpenCode

OpenCode can consume project `AGENTS.md`. If the opendump store is the project root, use canonical files plus the concrete config binding. If another project is active, preserve unrelated target-project guidance and install a managed opendump block there.

### ChatGPT Projects

Use Project Instructions as the persistent instruction surface when available. Determine GitHub/files/artifact capability from actually exposed tools/connections rather than assuming product-wide capability. If Project Instructions cannot be written programmatically, manual payload generation leaves initialization `BLOCKED` until installed and verified.

### Artifact-first harnesses

When durable artifacts are the selected mode, one concrete artifact identity must be bound and verified. Do not represent ephemeral chat content as a durable artifact.

### Unknown/future harnesses

Do not force another vendor's conventions onto an unfamiliar host. Identify the native persistent instruction surface and capabilities, then execute the same state machine. Harness identity may remain descriptive/unknown as long as the actual instruction surface and persistence capabilities are established sufficiently for correct binding.

## Minimum semantics every generated adapter must preserve

- Exactly one bound store mode and concrete location.
- Public upstream is never a user task write target.
- Startup reads live tasks from the bound store.
- Same-turn task mutations persist only to that store.
- No silent mode switching.
- Explicit cold start executes this state machine.
- Manual adapter text is not considered installed until persisted and verified.
- Failure to access the bound store is reported accurately.
- Normal task semantics come from `AGENTS.md`.

## Migration / mode change

A mode/location change is an explicit re-initialization operation.

1. Preserve existing task data.
2. Materialize/copy it into the target store.
3. Re-run store binding, adapter installation, and verification for the new target.
4. Replace the old binding only after the new one verifies successfully.
5. Never operate two authoritative stores in parallel.

## Maintenance rule

Change normal task semantics in `AGENTS.md`. Change initialization semantics in this file. Then refresh committed derived adapters (`CLAUDE.md`, Cursor rule/skill, and other future adapters). Vendor-specific notes MUST remain downstream implementation guidance, never a competing initialization algorithm.
