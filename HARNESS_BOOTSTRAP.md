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
5. The installed host-native adapter is the **sole persistent binding record** for that host environment. Its `store` and `location` determine where that host reads/writes, but it MUST NOT redefine workflow semantics.

If a generated adapter conflicts with canonical workflow semantics, regenerate it. If its store binding is absent, ambiguous, or invalid, initialization is not complete.

The opendump store MUST NOT contain a second environment-specific binding manifest. Repository names, filesystem paths, host identity, branch names, and adapter bindings belong to the host adapter or are discovered from the current environment, not duplicated into canonical task state.

## Cold-start trigger

Run this protocol when the user explicitly requests cold start, initialization, bootstrap, or sends the preferred one-liner:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

Equivalent requests naming another opendump checkout/reference also count.

When the reference is `JPilsinger/opendump`, it is the public workflow/template reference only. User tasks MUST NOT be written there.

## Core principle

**Intelligence discovers facts. The protocol decides what those facts imply.**

The agent SHOULD reason freely when identifying its host environment, native persistent instruction surface, available tools, repository context, and persistence capabilities. Once those facts are established, state transitions, store selection, success criteria, and failure behavior are deterministic.

## Harness-neutrality invariant

The public opendump template MUST remain host-neutral.

- The template MUST NOT ship files, directories, skills, rules, bridges, project instructions, or other adapters whose purpose is to target a specific agent product or harness.
- Host-specific adapter paths and syntax MUST be discovered at cold start from the actual environment, not preselected by the template.
- A host-specific adapter MAY be generated into a user-owned initialized environment when that is the native persistent instruction mechanism of the detected host.
- Generated host adapters are downstream artifacts. They MUST NOT become canonical specifications for opendump or another host.
- The absence of a preinstalled host adapter in the public template is intentional and MUST NOT be treated as missing configuration.

`AGENTS.md` is intentionally retained as a portable canonical runtime specification. Its presence MUST NOT be used as evidence of any particular host identity.

## Canonical task-state schema

For `github` and `local-files`, the canonical task-state schema consists of exactly these four files:

```text
private.md
business.md
private-completed.md
business-completed.md
```

For `artifact`, the artifact MUST expose equivalent logical collections for Private/Business open state and Private/Business completed history.

For file/GitHub stores, `AGENTS.md` and `HARNESS_BOOTSTRAP.md` accompany the task state as canonical protocol documents. They are not task-state collections and they do not record the active host binding.

Source-material transport or lifecycle mechanisms — including watched directories, upload queues, staging folders, source archives, acknowledgement ledgers, and processed-source directories — are not part of the canonical opendump schema and MUST NOT be required for a binding to be valid.

## Initialization state machine

Cold start MUST progress through these states in order. A later state MUST NOT be claimed until the current state's postconditions are satisfied.

| State | Required outcome |
|---|---|
| `UNINITIALIZED` | No verified active binding has yet been accepted for this cold start |
| `ENVIRONMENT_IDENTIFIED` | Host and native persistent-instruction surface are identified, or explicitly unknown |
| `CAPABILITIES_ESTABLISHED` | Relevant persistence capabilities are established as facts |
| `STORE_SELECTED` | Exactly one store mode is selected by the deterministic selection procedure |
| `STORE_BOUND` | A concrete store exists and satisfies the mode's binding gate |
| `ADAPTER_INSTALLED` | Host persistent instructions contain exactly one concrete opendump binding |
| `VERIFIED` | The adapter is read back and the bound store is independently verified |
| `READY` | Live open tasks are loaded from the verified store |

Two explicit non-success terminal states also exist:

- `BLOCKED` — the environment could support initialization, but a required external/user action or unavailable permission prevents the next required transition.
- `UNSUPPORTED` — no durable writable store/instruction arrangement exists that can provide reliable tracking.

`READY` is the **only successful terminal state**. There is no partial-success state.

## Phase 1 — environment identification

The agent MUST identify the host and its native persistent instruction surface using the first conclusive evidence available in this order:

1. Platform/system-provided host identity.
2. Native persistent-instruction APIs or surfaces exposed by the host.
3. Host-specific workspace conventions discovered in the current environment.
4. Available tool/connector signatures.
5. Current official host documentation when external lookup is available and materially needed.
6. User clarification only when identity remains materially ambiguous and the ambiguity blocks correct installation.

The agent MUST NOT infer a specific host solely from a portable file such as `AGENTS.md`, `README.md`, or another file that may exist in multiple environments.

Required normalized facts:

```yaml
host:
  name: <identified host or unknown>
  instruction_surface: <native persistent surface or none>
  instruction_readable: <true|false>
  instruction_writable: <true|false>
```

### Instruction-surface selection

If the host exposes multiple persistent instruction surfaces, select the **narrowest durable project-scoped surface that reliably applies to the sessions in which opendump is expected to operate**.

- Prefer project/workspace scope over account/global scope when both satisfy the requirement.
- Preserve unrelated existing instructions on shared surfaces.
- Do not install duplicate opendump adapters into multiple surfaces merely because they are available.
- If multiple surfaces are equally suitable and choosing incorrectly could materially change scope or behavior, enter `BLOCKED` with reason `instruction_surface_selection_required` and present the concrete alternatives.

Transition to `ENVIRONMENT_IDENTIFIED` only after these facts are established. An unfamiliar host is not an error; continue by capability rather than product name.

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

`github.setup_available` is `true` only when the current host/session exposes a **concrete, actionable path** by which the user can enable the missing GitHub capability without changing to another host. If true, `setup_user_action` MUST name that exact action. Mere theoretical support or generic advice to "set up GitHub" is not enough.

Transition to `CAPABILITIES_ESTABLISHED` only after enough facts exist to execute store selection mechanically.

## Repository-role determination

Repository role MUST be derived from the **actual repository identity**, never from copied metadata.

Let `UPSTREAM = JPilsinger/opendump`.

1. If the current repository full name equals `UPSTREAM`, its role is `public-upstream-template` and user task writes are forbidden.
2. Any different repository with the canonical opendump structural signature is not the public upstream merely because its files were copied from the template.
3. Never use repository name similarity, fork/template ancestry, copied text, or generated adapter paths as sufficient evidence that a repository is the public upstream; compare concrete repository identity when available.

The public template requires no seed manifest. GitHub template creation may copy the canonical files verbatim; actual repository identity determines whether the checkout is the upstream reference or a candidate user-owned store.

## Existing-binding rule

Before selecting a new store, inspect the installed host adapter when available.

An existing binding is valid only if all are true:

- exactly one `store` mode is declared;
- exactly one concrete `location` is declared when the mode requires one;
- the declared store satisfies its binding gate below;
- a `github` location is not `JPilsinger/opendump` or another known published upstream template.

If a valid existing adapter binding is present, reuse it. Do not select another store merely because another mode is preferable.

If an adapter binding exists but is invalid, ambiguous, or points to an inaccessible store, do not silently choose another store. Treat cold start as uninitialized, preserve any recoverable canonical task state, and establish one new binding using this protocol.

## Phase 3 — deterministic store selection

If no valid existing adapter binding is present, select the store using this procedure. Do not ask the user to choose merely because multiple technical modes are available. A user preference stated before or during initialization overrides the default precedence if it names a viable mode.

### GitHub structural signature and candidate resolution

A file/GitHub opendump store has this structural signature:

- `private.md`
- `business.md`
- `private-completed.md`
- `business-completed.md`
- `AGENTS.md`
- `HARNESS_BOOTSTRAP.md`

A GitHub repository is a resolvable opendump candidate only when at least one of these is true:

1. the user explicitly supplied that repository/location;
2. the current repository is non-upstream and has the structural signature;
3. an accessible, user-owned, writable repository has the structural signature and its canonical protocol documents identify it as opendump-compatible.

The installed adapter, when valid, outranks structural discovery. Structural matching MUST NOT override an explicit user reference.

Do not guess a store from repository name similarity alone. If more than one structurally valid candidate remains and none is distinguished by an existing valid adapter or explicit user reference, enter `BLOCKED` with reason `github_store_selection_required` and present the concrete candidates.

### Selection algorithm

Evaluate in order:

1. **Explicit viable user preference** — If the user explicitly requested `github`, `local-files`, or `artifact` and that mode is currently viable, select it and skip lower-priority alternatives.
2. **Resolvable user-owned GitHub store** — Else if GitHub read/write capability exists and exactly one user-owned opendump repository is resolvable, select `github`.
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
- repository is user-owned or otherwise explicitly authorized by the user for task storage;
- repository is not `JPilsinger/opendump` or another known published upstream template;
- the four canonical task-state files exist or are created;
- `AGENTS.md` and `HARNESS_BOOTSTRAP.md` exist or are materialized from the canonical template;
- concrete branch/default ref is known or resolvable.

Canonical location format:

```text
<owner>/<repo>@<branch>
```

### `local-files` binding gate

All MUST be true:

- concrete directory/path is known;
- the four canonical task-state files exist or are created;
- the four task-state files are readable and writable;
- `AGENTS.md` and `HARNESS_BOOTSTRAP.md` exist or are materialized for the file-based installation;
- location is durable beyond the current response/session according to the host's file model.

### `artifact` binding gate

All MUST be true:

- concrete artifact identity/name is known;
- artifact exists or is created;
- artifact is readable;
- artifact is writable;
- artifact is durable according to the host;
- it mirrors Private + Business Backlog/In progress and Private + Business completed history semantics.

Transport/staging sections and environment-specific binding metadata inside the store are not required.

### `unsupported`

No fake store/location may be created. Reliable capture MUST remain disabled.

Transition to `STORE_BOUND` only after the selected mode's gate passes.

## Phase 5 — generate and install the host adapter

The concrete binding determined in Phase 4 is persisted **only in the host-native adapter**. Do not write a duplicate binding manifest into the opendump store.

Use the native persistent instruction surface discovered in Phase 1. The generated adapter MUST:

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

When the instruction surface is shared, use one replaceable managed block when the host's syntax permits it:

```markdown
<!-- OPENDUMP:BEGIN store=<mode> location=<concrete location> -->
...derived instructions for this binding only...
<!-- OPENDUMP:END -->
```

If the host uses a different native structure, preserve the same idempotence and single-binding semantics using that structure.

Repeated cold start MUST replace/update the existing opendump adapter rather than append duplicates.

If the agent can generate but cannot write the native instruction surface, it MUST provide the exact payload for manual installation and enter:

```text
BLOCKED
reason: manual_adapter_installation_required
```

Generating the payload is not installation. After the user installs it, cold start resumes at this phase and MUST verify the installed content before success.

Transition to `ADAPTER_INSTALLED` only after the persistent surface actually contains the intended adapter.

## Phase 6 — mandatory verification

Verification is a hard gate, not a best-effort step.

The agent MUST:

1. Read the native persistent instruction surface back after installation.
2. Parse/read the installed opendump binding.
3. Confirm exactly one mode and one concrete location.
4. Confirm the read-back binding equals the intended binding from Phase 4.
5. Resolve that location independently and re-read enough of the bound store to prove current read access.
6. Confirm the canonical task-state schema is present for the selected mode.
7. For GitHub/file modes, confirm the canonical protocol documents are accessible.
8. Verify current write capability using a successful materialization/write already performed, authoritative permission evidence, or a safe reversible probe appropriate to the host.
9. Confirm the store does not need any secondary binding/config manifest for normal operation.

If any verification fails, do not claim installation succeeded. Repair and re-verify when possible; otherwise enter `BLOCKED` with the failing gate and reason.

Transition to `VERIFIED` only after all applicable checks pass.

## Phase 7 — load live state and become ready

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

When blocked, state the exact state/reason and the minimum required next action. Example semantic result:

```yaml
opendump_initialization:
  status: blocked
  phase: ADAPTER_INSTALLED
  reason: manual_adapter_installation_required
```

When unsupported, explicitly state that reliable task tracking is unavailable in the current host and do not pretend chat memory is the database.

Do not convert `BLOCKED` or `UNSUPPORTED` into a weaker silent success.

## Startup behavior after initialization

Normal sessions do not rerun cold start automatically.

At startup:

1. Read the installed host adapter and identify the locked store/location.
2. Read live open tasks from that store.
3. When file/GitHub canonical workflow docs are accessible, follow current `AGENTS.md` semantics.
4. If the adapter is missing, ambiguous, or invalid, do not infer the active store from a copied config file; run cold start or report the missing binding.
5. If the bound store is inaccessible, report the failure and stop claiming persistence. Do not switch stores silently.
6. Run this full cold-start protocol again only when explicitly requested, when the binding is invalid, or when the user requests a mode/location change.

## Minimum semantics every generated adapter must preserve

- Exactly one bound store mode and concrete location.
- The adapter is the sole persistent binding record for that host environment.
- Public upstream is never a user task write target.
- Startup reads live tasks from the bound store.
- Same-turn task mutations persist only to that store.
- No silent mode switching.
- Explicit cold start executes this state machine.
- Manual adapter text is not considered installed until persisted and verified.
- Failure to access the bound store is reported accurately.
- Normal task semantics come from `AGENTS.md`.
- Source-material transport/staging is not part of canonical task state.

## Migration / mode change

A mode/location change is an explicit re-initialization operation.

1. Preserve existing canonical task state.
2. Materialize/copy it into the target store.
3. Re-run store binding, adapter installation, and verification for the new target.
4. Replace the old adapter binding only after the new target verifies successfully.
5. Never operate two authoritative stores in parallel.

No store-side binding manifest needs migration because environment-specific binding state belongs only to the adapter.

## Maintenance rule

- Change normal task semantics in `AGENTS.md`.
- Change initialization semantics in this file.
- Keep the public template free of committed host-specific adapters and environment-specific binding manifests.
- Keep source-ingestion transport/staging mechanisms outside the canonical store schema.
- When an initialized user environment contains a generated adapter, regenerate that downstream adapter from the canonical documents after relevant semantic changes.
- Never copy a generated adapter into the public template as a compatibility shortcut.
