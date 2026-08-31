# opendump

Open-source place to dump and track personal and business tasks with an AI agent.

OpenDump is host-neutral: the task model stays the same while the bound store may be GitHub, durable local files, or a durable host artifact.

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

3. The agent identifies its host, persistent instruction surface, and actual capabilities.
4. It selects exactly one durable store, materializes/validates the canonical task schema, generates one host-native adapter containing the concrete binding, reads it back, independently verifies the store, and loads live tasks.
5. Only then is initialization `READY`.

If no repository exists yet and the host can create one, cold start may create a normal repository and write the canonical files; a dedicated template-creation API is not required.

## Store modes

OpenDump binds exactly one mode/location per host adapter:

| Mode | Store |
|---|---|
| `github` | User-controlled GitHub repository (preferred for sync/history) |
| `local-files` | Durable local markdown files |
| `artifact` | Durable host artifact with equivalent task collections |
| `unsupported` | No reliable durable store is available |

The adapter is the **sole persistent binding record** for the active mode and concrete location in that host environment. OpenDump does not duplicate repository names, branches, paths, or host identity into the task store.

If the bound store becomes inaccessible, the agent reports the problem rather than silently switching modes.

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

Normal sessions do **not** run the bootstrap protocol. They read the installed adapter, load live task state, and follow `AGENTS.md`. Bootstrap is used for initial setup, recovery, or an explicit store change.

## Minimal store schema

Canonical task state:

| File | Purpose |
|---|---|
| `private.md` | Personal Backlog + In progress |
| `business.md` | Business Backlog + In progress |
| `private-completed.md` | Completed personal tasks |
| `business-completed.md` | Completed business tasks |

For file/GitHub stores these protocol files accompany the task state:

| File | Purpose |
|---|---|
| `AGENTS.md` | Runtime task workflow semantics |
| `HARNESS_BOOTSTRAP.md` | Cold-start, recovery, binding, adapter, and verification protocol |

That's the canonical OpenDump footprint. No host-specific adapters, environment-binding manifests, inbox directories, processed directories, upload queues, or staging folders belong in the public template.

## Host-neutral template

The public template intentionally ships no product-specific adapter, rule, skill, bridge, or project-instruction payload.

Cold start discovers the host's actual persistent instruction mechanism and generates the adapter downstream. `AGENTS.md` remains part of the template because it is the portable runtime specification, not evidence of any particular host.

## Lifecycle

1. **Capture** — new trackable work → Backlog, persisted in the same turn.
2. **Progress** — move to In progress and append dated notes.
3. **Done** — remove from the active list and append to the matching completed archive with `YYYY-MM-DD`.
4. **Status** — broad status questions open with all current Backlog + In progress tasks, Private then Business.

Full runtime semantics: [`AGENTS.md`](./AGENTS.md).

Initialization/recovery protocol: [`HARNESS_BOOTSTRAP.md`](./HARNESS_BOOTSTRAP.md).

## Privacy

The public template contains no real tasks. Treat your own store as sensitive; a private GitHub repository is usually appropriate for personal or confidential work.

Source material is not retained in the OpenDump store by default.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). Changes that reduce ambiguity or simplify cross-host behavior are especially welcome.

## License

MIT — see [LICENSE](./LICENSE).
