# opendump

Open-source place to dump and track tasks. Your AI agent (Cursor, Claude, ChatGPT, OpenCode, …) catches, progresses, and archives them across harnesses.

## About

opendump is a harness-portable task dump: you talk to your agent, it captures personal and work tasks, and persists them to **your** GitHub repo when you want sync — or to local files / a durable artifact when you do not. Drop a quick screenshot that contains new tasks, or point at files and other content — the agent reads them and captures the work accordingly.

This repository (`JPilsinger/opendump`) is the **public GitHub template**, not your task database. Personal and shared tasks live in a repo **you** create with **Use this template**.

**Preferred store:** your own GitHub template instance (sync + history + shareable). Cold start can also lock **local files** or a **durable artifact** (e.g. Canvas). Chat-only harnesses cannot track reliably — the agent should say so and help you set up something better.

## Setup

### Preferred: your own repo from the template

1. Open https://github.com/JPilsinger/opendump → **Use this template** → create **your** repository (private is fine).
2. In your host harness chat, send:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

3. The agent reviews the public template for workflow rules, then locks **your** repo (or helps create it if you skipped step 1). It must **never** push your tasks to the public template.
4. Talk normally — trackable work is auto-captured into your store.

### One-liner only

You can send the one-liner without creating a repo first. The agent should still prefer GitHub and **create or guide a user-specific template instance** before capturing tasks — not write into `JPilsinger/opendump`.

## Without GitHub

The same one-liner still applies. The agent should:

1. Prefer a **user-owned** GitHub repo from this template and offer to help create/connect it.
2. If you decline or cannot use GitHub — set up a **local markdown mirror** or a **durable artifact** with the same task sections, and lock that mode in project instructions (no “GitHub or local” ambivalence).
3. If the chat cannot persist anything — explain that tracking will not work there and help you move to a harness with files, artifacts, or GitHub.

## What you get

| File | Purpose |
|------|---------|
| `AGENTS.md` | Canonical workflow + store-mode rules |
| `HARNESS_BOOTSTRAP.md` | Cold-start, capability ladder, adapter install |
| `opendump.config.md` | Role/config for this tree (template vs personal instance) |
| `private.md` / `business.md` | Open personal and work tasks (empty in the public template) |
| `private-completed.md` / `business-completed.md` | Completed-task archives |
| `inbox/` | Optional drop folder for notes, screenshots, voice |
| `CLAUDE.md`, `.cursor/…` | Derived adapters for Claude Code and Cursor |

## Lifecycle (short)

1. **Capture** — New trackable work → Backlog in **your** store, persist same turn (push only to your `github` repo)  
2. **Progress** — Dated notes; move to In progress  
3. **Done** — Move to the matching completed archive with `YYYY-MM-DD`  
4. **Status** — “What’s up?” opens with all open Private + Business tasks  

Full semantics: [`AGENTS.md`](./AGENTS.md). Harness install: [`HARNESS_BOOTSTRAP.md`](./HARNESS_BOOTSTRAP.md).

## Privacy

Task files stay empty in the public template. Treat **your** instance as sensitive. Prefer a **private** GitHub repo for personal or family use.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). Fork and adapt freely; ideas that improve the shared template are welcome.

## License

MIT — see [LICENSE](./LICENSE).
