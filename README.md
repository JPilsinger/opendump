# opendump

Open-source place to dump and track tasks. Your AI agent (Cursor, Claude, ChatGPT, OpenCode, …) catches, progresses, and archives them across harnesses.

## About

opendump is a harness-portable task dump: you talk to your agent, it captures personal and work tasks, and persists them to **your** GitHub repo when you want sync — or to local files / a durable artifact when you do not. Drop a quick screenshot, snap a photo on your phone, or point at files and other content — the agent reads the image or file and extracts the to-dos accordingly.

`JPilsinger/opendump` is the **public GitHub template and workflow reference**, not a personal task database. Personal and shared tasks live in a repo **you** create with **Use this template**, or another explicitly locked durable store.

**Preferred store:** your own GitHub template instance (sync + history + shareable). Cold start can also lock local files or a durable artifact when GitHub is unavailable or explicitly declined. Chat-only harnesses cannot track reliably and must say so.

## Setup

### Preferred: your own repo from the template

1. Open `JPilsinger/opendump` on GitHub → **Use this template** → create **your** repository (private is fine).
2. In your host harness chat, send:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

3. The agent reviews the public template for workflow rules, identifies its own harness/instruction surface and actual capabilities, then binds exactly one **user-owned** store.
4. Cold start is successful only after the store and host adapter are both installed/read back, their binding agrees, and live open tasks have been loaded.
5. Talk normally — trackable work is auto-captured into the verified store.

### Why copied template files say `template-seed`

GitHub's template flow copies configuration and adapter files verbatim. They therefore intentionally start as **uninitialized seed metadata** rather than claiming that every copy is the public upstream.

During cold start the agent determines repository role from the **actual repository identity**:

- exact `JPilsinger/opendump` → public upstream; task writes forbidden;
- any different user/org repository copied from the template → uninitialized instance; replace seed metadata with its concrete locked binding.

This prevents a newly templated personal repo from being mistaken for the read-only upstream.

### One-liner only

You can send the one-liner without creating a repo first. The deterministic cold-start protocol prefers GitHub. If the harness can create a user-owned template instance itself, it does so. If GitHub requires a user action the harness cannot perform (for example authentication/connection/template creation), initialization is explicitly **blocked** until that action is completed rather than silently falling back to a weaker store.

## Without GitHub

If GitHub is explicitly declined or not a feasible path, the same protocol can bind:

1. `local-files` — a durable markdown tree mirroring the opendump layout;
2. `artifact` — a durable harness artifact mirroring the same task sections;
3. `unsupported` — no reliable tracking when the host has no durable writable store/instruction arrangement.

The agent must lock exactly one mode and concrete location. It must never leave persistent instructions saying “GitHub or local” and decide differently at runtime.

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

## What you get

| File | Purpose |
|------|---------|
| `AGENTS.md` | Canonical normal task workflow + store-mode semantics |
| `HARNESS_BOOTSTRAP.md` | Normative initialization state machine, capability gates, binding and verification |
| `opendump.config.md` | Template seed before initialization; concrete store binding afterwards |
| `private.md` / `business.md` | Open personal and work tasks |
| `private-completed.md` / `business-completed.md` | Completed-task archives |
| `inbox/` | Optional drop folder for notes, screenshots, voice |
| `CLAUDE.md`, `.cursor/…` | Derived harness adapters / seed bridges |

## Lifecycle (short)

1. **Capture** — New trackable work → Backlog in the locked store, persist same turn
2. **Progress** — Dated notes; move to In progress
3. **Done** — Move to the matching completed archive with `YYYY-MM-DD`
4. **Status** — “What’s up?” opens with all open Private + Business tasks

Full task semantics: [`AGENTS.md`](./AGENTS.md). Initialization protocol: [`HARNESS_BOOTSTRAP.md`](./HARNESS_BOOTSTRAP.md).

## Privacy

Task files stay empty in the public template. Treat your instance as sensitive. Prefer a **private** GitHub repo for personal or family use.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). Fork and adapt freely; ideas that reduce ambiguity or improve cross-harness initialization are especially welcome.

## License

MIT — see [LICENSE](./LICENSE).
