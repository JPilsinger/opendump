# opendump

Open-source place to dump and track tasks. Markdown lists are the source of truth; your agent (Cursor, Claude, ChatGPT, OpenCode, …) catches, progresses, and archives them across harnesses.

## About

opendump is a harness-portable task dump: you talk to your agent, it captures personal and work tasks into a simple markdown layout, and persists them to GitHub when it can — or to local files / a durable artifact when it cannot. Use the public repo as a GitHub template, or cold-start from the URL alone.

**Preferred store:** GitHub (sync + history). Cold start can also lock **local files** or a **durable artifact** (e.g. Canvas) if you do not use GitHub. Chat-only harnesses cannot track reliably — the agent should say so and help you set up something better.

## Setup (one-liner)

In your host harness chat, send:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

That is enough to begin. The agent should review the template, run the cold-start protocol, lock a single store mode, and install host project instructions.

Optional GitHub path if you want your own remote from day one:

1. **Use this template** → create your repo (private is fine for personal tasks).
2. Clone it and open it in the harness — or keep working elsewhere and point cold start at your repo.
3. Send the same one-liner (or cold-start against your clone).
4. Talk normally — trackable work is auto-captured and persisted per the locked mode.

Do not commit your personal tasks back to the public template.

## Without GitHub

The same one-liner still applies. The agent should:

1. Prefer GitHub and offer to help connect or create a repo.
2. If you decline or cannot use GitHub — set up a **local markdown mirror** or a **durable artifact** with the same task sections, and lock that mode in project instructions (no “GitHub or local” ambivalence).
3. If the chat cannot persist anything — explain that tracking will not work there and help you move to a harness with files, artifacts, or GitHub.

## What you get

| File | Purpose |
|------|---------|
| `AGENTS.md` | Canonical workflow + store-mode rules |
| `HARNESS_BOOTSTRAP.md` | Cold-start, capability ladder, adapter install |
| `opendump.config.md` | Locked store mode for this instance |
| `private.md` / `business.md` | Open personal and work tasks |
| `private-completed.md` / `business-completed.md` | Completed-task archives |
| `inbox/` | Optional drop folder for notes, screenshots, voice |
| `CLAUDE.md`, `.cursor/…` | Derived adapters for Claude Code and Cursor |

## Lifecycle (short)

1. **Capture** — New trackable work → Backlog, persist same turn (push only in `github` mode)  
2. **Progress** — Dated notes; move to In progress  
3. **Done** — Move to the matching completed archive with `YYYY-MM-DD`  
4. **Status** — “What’s up?” opens with all open Private + Business tasks  

Full semantics: [`AGENTS.md`](./AGENTS.md). Harness install: [`HARNESS_BOOTSTRAP.md`](./HARNESS_BOOTSTRAP.md).

## Privacy

Task files start empty. Treat your instance as sensitive if you put real life/work there. Prefer a **private** GitHub repo for personal use; the public template stays scrubbed.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). Fork and adapt freely; ideas that improve the shared template are welcome.

## License

MIT — see [LICENSE](./LICENSE).
