# opendump

Open-source place to dump tasks. Markdown lists in GitHub are the source of truth; your agent (Cursor, Claude, ChatGPT, OpenCode, …) catches, progresses, and archives them across harnesses.

## Use as a template

1. On GitHub: **Use this template** → create your own repository (keep it private if your tasks are personal).
2. Clone your new repo over SSH and open it in your preferred agent harness.
3. Tell the agent to **cold start** / **bootstrap opendump** so it installs a native adapter for that harness.
4. Talk normally — trackable work is auto-captured into `private.md` / `business.md` and pushed.

Do not commit your personal tasks back to the public template.

## What you get

| File | Purpose |
|------|---------|
| `AGENTS.md` | Canonical workflow (auto-capture, progress, done archives) |
| `HARNESS_BOOTSTRAP.md` | Cross-harness cold-start / adapter protocol |
| `private.md` / `business.md` | Open personal and work tasks |
| `private-completed.md` / `business-completed.md` | Completed-task archives |
| `inbox/` | Optional drop folder for notes, screenshots, voice |
| `CLAUDE.md`, `.cursor/…` | Derived adapters for Claude Code and Cursor |

## Lifecycle (short)

1. **Capture** — New trackable work → Backlog, commit + push same turn  
2. **Progress** — Dated notes; move to In progress  
3. **Done** — Move to the matching `*-completed.md` archive with `YYYY-MM-DD`  
4. **Status** — “What’s up?” opens with all open Private + Business tasks  

Full semantics: [`AGENTS.md`](./AGENTS.md). Harness install: [`HARNESS_BOOTSTRAP.md`](./HARNESS_BOOTSTRAP.md).

## Privacy

Task files start empty. Treat your instance as sensitive if you put real life/work there. Prefer a **private** GitHub repo for personal use; the public template stays scrubbed.

## Contributing

Improvements to workflow docs and adapters belong in the public template. Keep example task lists empty of real personal data.

## License

MIT — see [LICENSE](./LICENSE).
