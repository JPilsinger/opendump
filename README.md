# opendump

Open-source place to dump and track personal and business tasks with an AI agent.

OpenDump is also a concrete example of a broader portable-agentic-workflow architecture: define workflow semantics once, then compile them into a host-specific realization using whatever native agentic primitives the current AI platform provides.

## Architecture

OpenDump has three central control artifacts:

```text
AGENTS.md
    WHAT OpenDump means at runtime
        ↓
HARNESS_BOOTSTRAP.md
    HOW to realize OpenDump on an arbitrary host
        ↓
Host adapter
    THE realized host-specific OpenDump implementation
```

`AGENTS.md` is the canonical **host-neutral runtime contract**.

`HARNESS_BOOTSTRAP.md` is the **host-neutral compiler/installer/verifier**.

The generated **host adapter** is the complete host-specific manifestation of `AGENTS.md`, parameterized by the concrete task-store binding. It may be one file or field, or several cooperating host-native assets such as project instructions, rules, skills, persistent settings, or other mechanisms.

Conceptually:

```text
OpenDumpAdapter = Compile(AGENTS.md, HostEnvironment, StoreBinding)
```

The adapter is derived. It is never a second source of OpenDump workflow truth.

When a host can reliably consume current `AGENTS.md` semantics directly, bootstrap prefers to link them rather than copy them. When it cannot, bootstrap faithfully compiles the required semantics into native host assets. Either way, correctness means equivalent observable OpenDump behavior.

## How it works

You talk normally. The agent captures trackable work, progresses it, and archives completed tasks in a verified store.

Source material can arrive however the current host supports it — chat, screenshots, photos, audio, documents, files, links, or attachments. OpenDump stores the resulting **task state**, not the transport/staging lifecycle of that source material.

`JPilsinger/opendump` is the **public template and protocol reference**, never a personal task database.

A **user-controlled store** is owned by you or explicitly authorized by you for OpenDump task storage.

## Setup

Preferred GitHub flow:

1. Create a user-controlled repository. Using GitHub's **Use this template** is the easiest path, but any writable repository can be materialized with the canonical OpenDump files.
2. In your host chat, send:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

3. The bootstrap/compiler identifies the actual host, its durable native agentic primitives, current tools/permissions, and viable task stores.
4. It establishes exactly one concrete store binding.
5. It compiles `AGENTS.md` + host environment + store binding into one logical host adapter, using linked, compiled, or hybrid realization as appropriate.
6. It installs the required host-native assets, reads them back, verifies semantic coverage and the store binding, and loads live tasks.
7. Only then is initialization `READY`.

If no repository exists yet and the host can create one, cold start may create a normal repository and write the canonical files; a dedicated template-creation API is not required.

## The host adapter

The adapter is **more than a store pointer**.

It is the complete host-specific realization of the OpenDump runtime contract. The concrete `store` and `location` are required inputs to that realization, but they are only one part of it.

A logical adapter MAY span multiple physical host assets when required. For example, one host might realize OpenDump through project instructions plus a connector; another through repository rules plus filesystem tools; another might natively load `AGENTS.md` and need only a small managed binding/integration layer.

All are valid if they preserve the same canonical runtime semantics.

## Store modes

OpenDump binds exactly one mode/location per logical host adapter:

| Mode | Store |
|---|---|
| `github` | User-controlled GitHub repository (preferred for sync/history) |
| `local-files` | Durable local markdown files |
| `artifact` | Durable host artifact with equivalent task collections |
| `unsupported` | No reliable durable store is available |

The store binding belongs to the generated host realization, not to canonical task state. OpenDump does not duplicate repository names, branches, paths, or host identity into the task store.

If the bound store becomes inaccessible, the runtime reports the problem rather than silently switching modes.

## Deterministic cold start

`HARNESS_BOOTSTRAP.md` defines initialization as:

```text
UNINITIALIZED
→ ENVIRONMENT_IDENTIFIED
→ CAPABILITIES_ESTABLISHED
→ STORE_SELECTED
→ STORE_BOUND
→ ADAPTER_INSTALLED
→ VERIFIED
→ READY
```

`READY` is the only successful terminal state. `BLOCKED` and `UNSUPPORTED` are explicit non-success states.

The guiding principle is:

> Intelligence discovers facts. The protocol decides what those facts imply.

Verification covers both **binding correctness** and **semantic correctness**: the adapter must point to exactly the intended store and faithfully make the canonical OpenDump runtime contract effective on the host.

Normal sessions do not rerun the compiler protocol. They execute under the installed adapter. The adapter may make `AGENTS.md` effective through direct linkage or through a verified compiled host-native realization.

## Minimal store schema

Canonical task state:

| File | Purpose |
|---|---|
| `private.md` | Personal Backlog + In progress |
| `business.md` | Business Backlog + In progress |
| `private-completed.md` | Completed personal tasks |
| `business-completed.md` | Completed business tasks |

For file/GitHub stores these canonical protocol files accompany the task state:

| File | Purpose |
|---|---|
| `AGENTS.md` | Canonical host-neutral runtime contract |
| `HARNESS_BOOTSTRAP.md` | Host-neutral compiler/bootstrap protocol |

That's the canonical OpenDump footprint. No host-specific adapters, environment-binding manifests, inbox directories, processed directories, upload queues, or staging folders belong in the public template.

## Host-neutral template

The public template intentionally ships no product-specific adapter, rule, skill, bridge, or project-instruction payload.

Cold start discovers the host's actual agentic primitives and generates the adapter downstream. Supporting a new host should improve compilation/discovery logic, not add a host-specific fork of the OpenDump runtime contract.

## Lifecycle

1. **Capture** — new trackable work → Backlog, persisted in the same turn.
2. **Progress** — move to In progress and append dated notes.
3. **Done** — remove from the active list and append to the matching completed archive with `YYYY-MM-DD`.
4. **Status** — broad status questions open with all current Backlog + In progress tasks, Private then Business.

Full runtime contract: [`AGENTS.md`](./AGENTS.md).

Compiler/bootstrap protocol: [`HARNESS_BOOTSTRAP.md`](./HARNESS_BOOTSTRAP.md).

## Privacy

The public template contains no real tasks. Treat your own store as sensitive; a private GitHub repository is usually appropriate for personal or confidential work.

Source material is not retained in the OpenDump store by default.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). Changes that strengthen semantic portability, reduce ambiguity, or improve cross-host compilation are especially welcome.

## License

MIT — see [LICENSE](./LICENSE).
