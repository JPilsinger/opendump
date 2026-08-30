# opendump

Personal and business task dump. Markdown task lists are the source of truth; the **store** that holds them may be GitHub (preferred), a local file tree, or a durable host artifact. The workflow is designed to be portable across capable agent environments.

## Public template vs your store (critical)

`https://github.com/JPilsinger/opendump` is the **public upstream template only**. It is **not** a personal or shared task database.

- **Never** commit, push, or write user tasks, progress, or completions into `JPilsinger/opendump` (or any other published upstream template).
- For GitHub sync, the user must have a **user-specific** repository created from this template (**Use this template**), then lock that repo as the store.
- During cold start, if GitHub mode is chosen and the user has no personal opendump repo yet, **create or help create one** using an available GitHub/template mechanism. Do not fall back to pushing into the public template.
- Reviewing the public template for workflow docs is fine; mutating its task files for a user’s work is forbidden.

**Setup one-liner** (host chat):

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

That starts cold start from the public template as the *workflow reference*. The agent must then lock a **user-owned** store (new template instance, existing personal repo, local files, or artifact) — never the public template as the write target.

## Canonical store schema

The canonical **task-state schema** contains exactly these four logical task collections:

| File / artifact section | Role |
|---|---|
| `private.md` | Personal open tasks: Backlog / In progress |
| `business.md` | Work open tasks: Backlog / In progress |
| `private-completed.md` | Completed personal-task archive |
| `business-completed.md` | Completed work-task archive |

For GitHub and local-file stores, the canonical protocol/binding files are:

| Path | Role |
|---|---|
| `AGENTS.md` | Authoritative normal task workflow semantics |
| `HARNESS_BOOTSTRAP.md` | Authoritative host-independent cold-start / adapter protocol |
| `opendump.config.md` | Template seed before initialization; concrete binding afterwards |

A durable artifact store MUST mirror the same four task-state collections even if it does not use these filenames.

Source material, uploads, attachments, staging folders, watched directories, transport queues, and processed-source archives are **not** part of the canonical opendump store schema.

The public template intentionally contains **no host-specific adapter files or directories**. Cold start discovers the native persistent instruction surface and generates the required adapter in the initialized user environment.

## Store modes

Exactly **one** mode is active for a given opendump installation. Host project instructions / adapters must state that mode explicitly — never leave “GitHub or local” unresolved in persistent instructions.

| Mode | Store | Sync | When to use |
|------|--------|------|-------------|
| `github` | **User-owned** GitHub opendump repo (created via **Use this template**), usually `main` | Commit + push each mutation **only to that user repo** | Preferred whenever the host can use GitHub |
| `local-files` | Markdown files implementing the canonical schema | Write files same turn; no remote | User has no GitHub, or prefers local-only |
| `artifact` | Durable host artifact mirroring the canonical task-state collections | Update the artifact same turn; no remote | Host has durable artifacts but not usable files/GitHub |
| `unsupported` | None | No reliable tracking | No usable GitHub, durable files, or durable artifact |

**Preferred order when choosing a mode (cold start):** create/connect **user-owned** `github` repo from the template → complete a concrete GitHub setup action if available → `local-files` or `artifact` → `unsupported`.

Hard ban: locking `github` location to `JPilsinger/opendump` (or writing tasks there) is **invalid**. Always use a distinct user/org repo.

### Mode rules

- **Declare the store in every adapter** — include `store:` / mode, and the concrete location (`owner/repo`, filesystem path, or artifact id/name). Ambiguous adapters are invalid; regenerate them.
- **Do not silently switch modes** mid-session. If the locked store becomes unavailable, say so, stop claiming writes succeeded, and offer reconnect or a cold-start mode change.
- **Status and startup replies** should name the active store briefly when relevant (especially after cold start or on failure).
- **Upgrade path** — `local-files` / `artifact` → `github` is supported by copying the canonical task state into a new template instance, then re-running cold start in `github` mode. Do not invent automatic two-way sync in v1.
- **`unsupported`** — never pretend chat memory is the task database. Tell the user reliably tracking is impossible here and help them move to an environment with GitHub, files, or durable artifacts.

## Authority

- The **active store** (per locked mode) is the source of truth for task state.
- `AGENTS.md` is authoritative for task workflow behavior.
- `HARNESS_BOOTSTRAP.md` is authoritative for translating/installing that behavior into the host environment and for choosing/locking store mode.
- In file-based stores, `private.md` and `business.md` are authoritative for open and in-progress task state; `private-completed.md` and `business-completed.md` are authoritative for completed history. Artifact stores mirror the same section semantics.
- Host-specific files and project-instruction fields are derived adapters. If they conflict with the canonical workflow docs in the store, the canonical docs win and the adapter should be regenerated — except the adapter’s **locked mode and location** win for “where to read/write,” and must stay unambiguous.
- Platform/system/security policies remain higher priority than repository instructions.

## Startup sync

At the beginning of every new conversation/session where opendump is installed, **before relying on prior chat context**:

1. Read the host adapter (or `opendump.config.md` if present) and note the **locked store mode and location**.
2. Load live open tasks from that store (`private.md` / `business.md`, or the artifact equivalent).
3. When mode is `github` and access is available, treat the remote/canonical checkout as fresher than remembered chat context or stale mirrors.
4. Read `AGENTS.md` and `HARNESS_BOOTSTRAP.md` from the store when available (for `github` / `local-files`).
5. Do **not** load completed archives during routine startup or status reporting. Read them only when the user asks for completed history or a specific archive lookup is needed.
6. If the locked store cannot be accessed, say so explicitly and do not pretend it was reviewed or updated.

A normal startup sync reads the store and tasks. It does **not** silently rewrite a host project's persistent instructions unless the user explicitly requests a cold start/bootstrap.

## Harness-independent cold start

When the user explicitly asks for a **cold start**, initialization, bootstrap from opendump, sends the setup one-liner above, or equivalent, execute `HARNESS_BOOTSTRAP.md` before normal task handling.

Cold start selects and **locks** a store mode, generates and installs a derived adapter on the native persistent instruction surface, verifies the adapter/store binding by read-back, then loads and surfaces live tasks. Only `READY` is success.

## Lifecycle

Behavior is identical across modes except for **where** mutations are written and whether a git push happens.

1. **Startup sync** — Per locked mode, load current open tasks from the store.
2. **Cold start when requested** — Run `HARNESS_BOOTSTRAP.md` before proceeding.
3. **Automatic task capture** — When the user mentions a new trackable task, immediately add it to the appropriate **Backlog** in the locked store in the same turn. **Do not wait for confirmation such as “push”, “add”, or “backlog”.**
4. **Amend / correct** — If the user changes a task title, scope, category, notes, or priority, update the tracked item in the store immediately.
5. **Surface** — At conversation start, and whenever asked, show **Backlog** and **In progress** (Private then Business). Keep it short. Do not invent work. If the user asks a general status question such as **“What’s up?”**, **“What’s the status?”**, **“Any news?”**, or an equivalent status/update question, always **open the response with a concise summary of all open tasks** (all Backlog + In progress items across Private and Business) before any other updates, commentary, or details.
6. **Progress** — When the user reports progress on an existing task, move it to **In progress** if it is still in Backlog, append a dated note, and persist to the store.
7. **Done** — When the user says a tracked task is done/accomplished, remove it from the active private/business list and append it to the matching completed archive with the finish or report date (`YYYY-MM-DD`), then persist. Never retain completed entries or a Done section in the active task lists.

### Persist rules by mode

- **`github`** — commit and push to the **locked user-owned** repo’s default branch after mutations; short audit-friendly message; group mutations from the same user message when practical. **Never** push to `JPilsinger/opendump`.
- **`local-files`** — write the canonical task-state files in the same turn; no push.
- **`artifact`** — update the durable artifact in the same turn; no push. Warn that portability and multi-device sync are weaker than GitHub.
- **`unsupported`** — do not capture; explain the limitation and offer setup help toward user-owned `github`, `local-files`, or `artifact`.

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

- Classify clearly personal items into `private.md` and clearly work/client/business items into `business.md` (or the artifact equivalents).
- Use surrounding context and existing tasks to avoid unnecessary questions.
- If classification remains genuinely ambiguous, **do not lose the task**: add it to private with a nested note `classification: tentative — private/business unclear`, persist it, and mention the ambiguity in the reply. Move it later if the user corrects the category.
- Before adding, check both lists for the same underlying intent. Prefer updating an existing task over creating a near-duplicate.
- Processing the same or materially equivalent source more than once MUST NOT create duplicate task intent already represented in Backlog or In progress. Deduplicate against canonical task state, not against a staging/processed-source ledger.

## Task format

```markdown
- [ ] Short title
  - context / source
  - progress: YYYY-MM-DD — what happened
```

Completed entries (stored only in the matching completed archive):

```markdown
- [x] Short title (YYYY-MM-DD)
  - optional notes / last progress
```

Use today’s date when the user reports finishing unless they give another date.

## Source-material intake

Source material may arrive through any capability exposed by the host, including chat messages, images, audio, documents, files, links, attachments, or other content.

When source material contains trackable work:

1. interpret or extract the actionable intent;
2. deduplicate it against existing open task state;
3. classify it as Private or Business;
4. persist the resulting task mutation to the locked store in the same turn.

Additional rules:

- For screenshots / photos / images, extract the actual to-dos rather than creating a task such as “look at screenshot”.
- For voice/audio, transcribe or summarize when the host can do so; if the content cannot be interpreted, ask only for the missing information needed to extract the task.
- For text/markdown/chat/documents, extract trackable tasks directly.
- Source material itself is **not** part of opendump task state and MUST NOT be copied, staged, archived, moved, or retained in the opendump store unless the user explicitly asks for that retention.
- Transport, upload handling, watched directories, staging, source-file movement, acknowledgement queues, and post-processing lifecycle are host/integration concerns outside the opendump protocol.

## After task-state changes

- Persist according to the locked mode’s persist rules above.
- Only mutate canonical task-state files/sections and intentional protocol/configuration documents.
- Do not create transport/staging directories as part of normal opendump operation.

## Maintaining the protocol and adapters

- Change workflow semantics in `AGENTS.md` first.
- Change cross-host bootstrap/translation and mode selection in `HARNESS_BOOTSTRAP.md` first.
- Keep the public template free of host-specific adapter files and directories.
- Keep source-ingestion transport/staging mechanisms outside the canonical store schema.
- Generate or regenerate the adapter only in the initialized user environment, using that host's discovered native persistent instruction surface.
- Never make a generated host-specific adapter the upstream specification for another host.
- Derived project instructions must always include a single locked store mode and location — no optional “if GitHub else local” branching left for runtime guesswork.
