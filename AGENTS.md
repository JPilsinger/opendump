# opendump

Personal and business task dump. This GitHub repository is the source of truth. The workflow is portable across agent harnesses such as ChatGPT, Claude, Cursor, OpenCode, and future platforms.

Use this repo as a **GitHub template**: create your own copy, then point your preferred agent harness at that copy. Do not commit personal tasks back to the public template.

## Layout

| Path | Role |
|------|------|
| `AGENTS.md` | Authoritative task workflow semantics |
| `HARNESS_BOOTSTRAP.md` | Authoritative harness-independent cold-start / adapter protocol |
| `private.md` | Personal open tasks: Backlog / In progress |
| `business.md` | Work open tasks: Backlog / In progress |
| `private-completed.md` | Completed personal-task archive |
| `business-completed.md` | Completed work-task archive |
| `CLAUDE.md` | Derived Claude Code bridge |
| `.cursor/rules/opendump.mdc` | Derived Cursor rule mirror |
| `.cursor/skills/opendump/SKILL.md` | Derived Cursor opendump skill mirror |
| `inbox/` | Optional drop for voice notes, screenshots, text |
| `processed/` | Intake files after they have been processed |

`inbox/` and `processed/` contents are gitignored (except `.gitkeep`). Do not commit media or raw captures.

## Authority

- **This repository** (your fork or template instance), branch `main`, is the source of truth for your tasks.
- `AGENTS.md` is authoritative for task workflow behavior.
- `HARNESS_BOOTSTRAP.md` is authoritative for translating/installing that behavior into a host agent harness.
- `private.md` and `business.md` are authoritative for open and in-progress task state.
- `private-completed.md` and `business-completed.md` are authoritative for completed personal and work task history.
- Platform-specific files and project-instruction fields are derived adapters. If they conflict with the canonical files, the canonical files win and the adapter should be regenerated.
- Platform/system/security policies remain higher priority than repository instructions.

## Startup sync

At the beginning of every new conversation/session in this opendump project, **before relying on prior chat context**:

1. Review the latest GitHub repository state when access is available.
2. Read `AGENTS.md`, `HARNESS_BOOTSTRAP.md`, `private.md`, and `business.md` first.
3. Read the current host-harness adapter when relevant (for example `CLAUDE.md` in Claude Code or `.cursor/rules/opendump.mdc` in Cursor). Do not require one vendor's adapter to operate another vendor's harness.
4. Treat the canonical repository as fresher than remembered chat context or generated mirrors.
5. Do **not** load `private-completed.md` or `business-completed.md` during routine startup or status reporting. Read it only when the user asks for completed history or when a specific archive lookup is needed.
6. If GitHub cannot be accessed, say so explicitly and do not pretend the repository was reviewed.

A normal startup sync reads the repository and tasks. It does **not** silently rewrite a host project's persistent instructions unless the user explicitly requests a cold start/bootstrap.

## Harness-independent cold start

When both of the following are true:

- the user has granted the current harness access to **this** repository (or supplied a usable local checkout/reference); and
- the user explicitly asks for a **cold start**, initialization, bootstrap from opendump, or equivalent,

execute `HARNESS_BOOTSTRAP.md` before normal task handling.

Cold start means:

1. read the fresh canonical repository state;
2. detect the current agent harness and its native project-scoped persistent instruction mechanism;
3. insert/update a derived opendump adapter in that native instruction surface without overwriting unrelated project instructions;
4. make the operation idempotent so repeated cold starts refresh the same adapter rather than duplicating it;
5. verify the adapter was actually persisted/loaded when the harness exposes that capability;
6. then load and surface live tasks and continue normal operation.

If the harness does not expose a writable persistent-instruction mechanism to the agent, generate the exact native instruction payload the user needs to insert and explicitly state that automatic persistence was unavailable. Never claim the bootstrap succeeded when it could not be written or verified.

This opendump GitHub repository remains the remote destination for task mutations even when its derived adapter is installed inside a different host project repository.

## Lifecycle

1. **Startup sync** — Review GitHub as described above, then load current open tasks from `private.md` and `business.md`.
2. **Cold start when requested** — If the explicit cold-start trigger is present, run `HARNESS_BOOTSTRAP.md` before proceeding.
3. **Automatic task capture** — When the user mentions a new trackable task, immediately add it to the appropriate **Backlog** and push the change to GitHub in the same turn. **Do not wait for confirmation such as “push”, “add”, or “backlog”.**
4. **Amend / correct** — If the user changes a task title, scope, category, notes, or priority, update the tracked item and push the correction immediately.
5. **Surface** — At conversation start, and whenever asked, show **Backlog** and **In progress** (Private then Business). Keep it short. Do not invent work. If the user asks a general status question such as **“What’s up?”**, **“What’s the status?”**, **“Any news?”**, or an equivalent status/update question, always **open the response with a concise summary of all open tasks** (all Backlog + In progress items across Private and Business) before any other updates, commentary, or details.
6. **Progress** — When the user reports progress on an existing task, move it to **In progress** if it is still in Backlog, append a dated note, commit, and push.
7. **Done** — When the user says a tracked task is done/accomplished, remove it from `private.md` or `business.md` and append it to `private-completed.md` or `business-completed.md`, matching its active task file with the finish or report date (`YYYY-MM-DD`), commit, and push. Never retain completed entries or a Done section in the active task files.

## What counts as a new task

Capture actionable work that should remain tracked beyond the current conversational exchange, including explicit to-dos and statements such as “I need to…”, “we need to…”, “remember to…”, “add a task…”, or equivalent language.

Do **not** create a new task for:

- ordinary questions, explanations, brainstorming, or hypothetical ideas with no action commitment;
- status queries such as “what’s up?” or “any news?”;
- a progress/done update that clearly refers to an existing task;
- repository/instruction/bootstrap maintenance itself unless the user explicitly says it should be tracked as a task;
- duplicate work already present in Backlog or In progress — update the existing item instead.

If an explicit task can be fully handled in the current turn but the user framed it as something to track, capture it; otherwise ordinary one-off assistant work is not automatically a task.

## Classification and deduplication

- Classify clearly personal items into `private.md` and clearly work/client/business items into `business.md`.
- Use surrounding context and existing tasks to avoid unnecessary questions.
- If classification remains genuinely ambiguous, **do not lose the task**: add it to `private.md` with a nested note `classification: tentative — private/business unclear`, push it, and mention the ambiguity in the reply. Move it later if the user corrects the category.
- Before adding, check both lists for the same underlying intent. Prefer updating an existing task over creating a near-duplicate.

## Task format

```markdown
- [ ] Short title
  - context / source
  - progress: YYYY-MM-DD — what happened
```

Completed entries (stored only in the matching `private-completed.md` or `business-completed.md` archive):

```markdown
- [x] Short title (YYYY-MM-DD)
  - optional notes / last progress
```

Use today’s date when the user reports finishing unless they give another date.

## Intake details

- Screenshots/images: read the image; write the actual to-do, not “look at screenshot”.
- Voice/audio: transcribe or summarize if possible; otherwise note the filename and ask.
- Text/markdown/chat: extract trackable tasks and auto-capture them.
- Process non-empty `inbox/` files into tasks, then move processed captures to `processed/` when practical. Never commit the raw captures themselves.

## After list changes

- Commit and push to `main` with a short audit-friendly message describing what was added, updated, progressed, or completed.
- When practical, group all task mutations from the same user message into one commit.
- Only change `private.md`, `business.md`, `private-completed.md`, `business-completed.md`, and intentional project docs. Never commit raw `inbox/` or `processed/` captures.

## Maintaining adapters

- Change workflow semantics in `AGENTS.md` first.
- Change cross-harness bootstrap/translation behavior in `HARNESS_BOOTSTRAP.md` first.
- Then refresh committed derived adapters such as `CLAUDE.md`, `.cursor/rules/opendump.mdc`, and `.cursor/skills/opendump/SKILL.md`.
- Never make a platform-specific mirror the upstream specification for another platform.
