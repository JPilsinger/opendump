# opendump

Open-source place to dump and track tasks. Your AI agent catches, progresses, and archives them across capable host environments.

## About

opendump is a harness-portable task dump: you talk to your agent, it captures personal and work tasks, and persists them to **your** GitHub repo when you want sync — or to local files / a durable artifact when you do not. Send a screenshot, snap a photo, attach a document, point at a file, or provide other content — the agent extracts the actionable work through whatever input capabilities the current host exposes.

`JPilsinger/opendump` is the **public GitHub template and workflow reference**, not a personal task database. Personal and shared tasks live in a repo **you** create with **Use this template**, or another explicitly locked durable store.

**Preferred store:** your own GitHub template instance (sync + history + shareable). Cold start can also lock local files or a durable artifact when GitHub is unavailable or explicitly declined. Hosts without any durable writable store cannot track reliably and must say so.

## Setup

### Preferred: your own repo from the template

1. Open `JPilsinger/opendump` on GitHub → **Use this template** → create **your** repository (private is fine).
2. In your host chat, send:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

3. The agent reviews the public template for workflow rules, identifies its own host/instruction surface and actual capabilities, then binds exactly one **user-owned** store.
4. It generates the host-native adapter only after that discovery; the public template does not preinstall product-specific adapters.
5. The generated adapter records exactly one store mode and concrete location. OpenDump does not duplicate that environment-specific binding inside the task store.
6. Cold start is successful only after the adapter has been installed/read back, the bound store has been independently verified, and live open tasks have been loaded.
7. Talk normally — trackable work is auto-captured into the verified store.

### Template copies and repository identity

GitHub's template flow copies the canonical files verbatim. No seed/config manifest is required.

During cold start the agent determines repository role from the **actual repository identity**:

- exact `JPilsinger/opendump` → public upstream; task writes forbidden;
- a different user/org repository with the canonical OpenDump structure → candidate user-owned store, subject to access/ownership verification.

Copied files never make a different repository the public upstream.

### One-liner only

You can send the one-liner without creating a repo first. The deterministic cold-start protocol prefers GitHub. If the host can create a user-owned template instance itself, it does so. If GitHub requires a user action the host cannot perform, initialization is explicitly **blocked** until that action is completed rather than silently falling back to a weaker store.

## Without GitHub

If GitHub is explicitly declined or not a feasible path, the same protocol can bind:

1. `local-files` — durable markdown files implementing the canonical task-state schema;
2. `artifact` — a durable host artifact mirroring the same task-state collections;
3. `unsupported` — no reliable tracking when the host has no durable writable store/instruction arrangement.

The agent must lock exactly one mode and concrete location in the host adapter. It must never leave persistent instructions saying “GitHub or local” and decide differently at runtime.

## Deterministic initialization

`HARNESS_BOOTSTRAP.md` defines cold start as a formal state machine:

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

The host agent is trusted to intelligently discover facts about its own environment. The protocol then deterministically defines what those facts imply: **intelligence discovers facts; the protocol decides the outcome.**

## Harness-neutral template

The public template intentionally ships **no product-specific rule, skill, bridge, project-instruction file, or environment-specific binding manifest**. Cold start discovers the host's native persistent instruction mechanism and generates the adapter there.

That means:

- no host gets privileged treatment in the canonical template;
- no host-specific instructions execute before environment discovery;
- generated adapters cannot drift into competing canonical specifications;
- future hosts follow the same protocol without requiring a template update merely to add their file naming convention;
- repository moves, renames, branches, and host identity are not duplicated into stale store metadata.

`AGENTS.md` remains intentionally: it is the portable canonical runtime specification, not proof of a particular host.

## Minimal store schema

OpenDump stores **task state**, not environment-binding state or source-material transport state.

The canonical task-state collections are:

| File | Purpose |
|------|---------|
| `private.md` | Open personal tasks |
| `business.md` | Open work tasks |
| `private-completed.md` | Completed personal-task archive |
| `business-completed.md` | Completed work-task archive |

For GitHub/local-file stores, these canonical protocol files accompany the task state:

| File | Purpose |
|------|---------|
| `AGENTS.md` | Canonical normal task workflow + store-mode semantics |
| `HARNESS_BOOTSTRAP.md` | Normative host-independent initialization state machine, capability gates, binding and verification |

The **host adapter is the sole persistent binding record** for the active mode and location in that host environment.

Attachments, uploads, screenshots, audio, watched folders, staging queues, and processed-source archives are not part of the canonical store. The host may accept any of those inputs; OpenDump extracts the task intent and persists only the resulting task state unless the user explicitly requests source retention.

Host-specific adapter files are generated downstream during initialization and are not part of the public template.

## Lifecycle (short)

1. **Capture** — New trackable work → Backlog in the locked store, persist same turn
2. **Progress** — Dated notes; move to In progress
3. **Done** — Move to the matching completed archive with `YYYY-MM-DD`
4. **Status** — “What’s up?” opens with all open Private + Business tasks

Full task semantics: [`AGENTS.md`](./AGENTS.md). Initialization protocol: [`HARNESS_BOOTSTRAP.md`](./HARNESS_BOOTSTRAP.md).

## Privacy

Task files stay empty in the public template. Treat your instance as sensitive. Prefer a **private** GitHub repo for personal or family use. Source material is not retained in the OpenDump store by default.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). Fork and adapt freely; ideas that reduce ambiguity or improve cross-host initialization are especially welcome.

## License

MIT — see [LICENSE](./LICENSE).
