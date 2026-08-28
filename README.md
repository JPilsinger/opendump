# opendump

Open-source place to dump and track tasks. Markdown lists are the source of truth; your agent (Cursor, Claude, ChatGPT, OpenCode, …) catches, progresses, and archives them across harnesses.

**Preferred store:** GitHub (sync + history). Cold start can also lock **local files** or a **durable artifact** (e.g. Canvas) if you do not use GitHub. Chat-only harnesses cannot track reliably — the agent should say so and help you set up something better.

## Use as a template (GitHub)

1. On GitHub: **Use this template** → create your own repository (keep it private if your tasks are personal).
2. Clone your new repo over SSH and open it in your preferred agent harness.
3. Tell the agent to **cold start** / **bootstrap opendump** so it installs a native adapter with **store mode locked to github**.
4. Talk normally — trackable work is auto-captured into `private.md` / `business.md` and pushed.

Do not commit your personal tasks back to the public template.

## Without GitHub

Say **cold start** in your harness. The agent should:

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

Improvements to workflow docs and adapters belong in the public template. Keep example task lists empty of real personal data.

## License

MIT — see [LICENSE](./LICENSE).
