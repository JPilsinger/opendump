# opendump

Personal and business task dump. The locked store is the source of truth for task state; the installed host adapter tells the current host where that store is.

## Public template vs user-controlled store

`JPilsinger/opendump` is the public template and protocol reference. It is never a user's task database.

A **user-controlled store** is a store owned by the user or explicitly authorized by the user for opendump task storage.

- Never write user tasks, progress, or completions to `JPilsinger/opendump` or another known published upstream template.
- For GitHub sync, bind a distinct user-controlled repository.
- Reviewing the public template for protocol docs is allowed; mutating its task files for user work is not.

Preferred cold-start request:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

Cold start is defined by `HARNESS_BOOTSTRAP.md`.

## Canonical store schema

The canonical task-state schema contains exactly four logical collections:

| File / artifact section | Role |
|---|---|
| `private.md` | Personal open tasks |
| `business.md` | Work open tasks |
| `private-completed.md` | Completed personal tasks |
| `business-completed.md` | Completed work tasks |

For file/GitHub stores:

- `private.md` and `business.md` MUST expose `Backlog` and `In progress` sections.
- `private-completed.md` and `business-completed.md` are append-only completed-task archives under normal operation.
- `AGENTS.md` accompanies the task state as the canonical runtime protocol.
- `HARNESS_BOOTSTRAP.md` accompanies the task state as the cold-start/recovery protocol, but is not part of normal startup reads.

Artifact stores MUST provide equivalent logical collections even if they do not use these filenames.

Environment-specific binding metadata is not part of the store. The installed host adapter is the sole persistent record of the active `store` and concrete `location` for that host environment.

Source material, uploads, attachments, watched folders, staging queues, acknowledgement ledgers, and processed-source archives are not part of the canonical store.

## Store modes

Exactly one mode is active for a given host binding.

| Mode | Store | Persistence |
|---|---|---|
| `github` | User-controlled GitHub repository | Commit/push task mutations to the bound repository |
| `local-files` | Durable local markdown files | Write task mutations in the same turn |
| `artifact` | Durable host artifact | Update the artifact in the same turn |
| `unsupported` | None | No reliable task tracking |

Mode rules:

- Every installed adapter MUST declare exactly one mode and one concrete location.
- The adapter MUST NOT contain runtime fallback branching such as “GitHub if available, otherwise local”.
- Do not silently switch modes or locations. A mode/location change requires cold start/re-initialization.
- If the bound store becomes inaccessible, report the failure and stop claiming persistence.
- `github` MUST never bind to `JPilsinger/opendump` or another known published upstream template.

## Authority

1. Platform/system/security policies have highest priority.
2. `AGENTS.md` defines normal task workflow semantics.
3. The installed host adapter defines the concrete store mode/location for that host environment.
4. The bound store is authoritative for task state.

Host-specific adapters are derived. If an adapter conflicts with runtime semantics, regenerate it; its concrete binding remains authoritative only for where this host reads/writes.

## Normal startup

At the beginning of a session where opendump is installed:

1. Read the installed host adapter and resolve its single locked store/location.
2. If the adapter is missing, ambiguous, or invalid, run cold start when appropriate or report the missing binding. Do not infer the binding from store-side metadata.
3. Read `private.md` and `business.md` (or artifact equivalents) from the bound store.
4. For file/GitHub modes, read current `AGENTS.md` from the bound store when accessible.
5. In GitHub mode, prefer the bound repository's current remote state over remembered chat context or stale mirrors.
6. Do not read completed archives during routine startup/status reporting unless a specific archive lookup is needed.
7. If the bound store cannot be accessed, report the failure and stop claiming persistence. Do not switch stores silently.

Normal startup does not read `HARNESS_BOOTSTRAP.md` and does not rewrite the adapter. Use the bootstrap protocol only for explicit cold start, invalid/missing binding, recovery, or a requested mode/location change.

## Lifecycle

Behavior is identical across modes except for where mutations are persisted.

1. **Capture** — When the user expresses new trackable work, add it to the appropriate Backlog in the same turn.
2. **Amend** — When the user changes a task's title, scope, category, notes, or priority, update the existing task immediately.
3. **Surface** — At conversation start and whenever asked, show Backlog + In progress, Private then Business. For broad status questions such as “What's up?”, “What's the status?”, or “Any news?”, open with a concise summary of all open tasks before other details.
4. **Progress** — When the user reports progress, move the task to In progress if needed, append a dated progress note, and persist.
5. **Done** — Remove the task from the active file/section and append it to the matching completed archive with the completion/report date (`YYYY-MM-DD`). Do not keep completed tasks in active lists.

### Persistence by mode

- `github` — commit and push task mutations to the bound user-controlled repository; use short audit-friendly commit messages and group mutations from the same user message when practical.
- `local-files` — write the canonical task-state files in the same turn.
- `artifact` — update the durable artifact in the same turn.
- `unsupported` — do not capture tasks; explain that reliable persistence is unavailable.

## What counts as a new task

Capture actionable work that should remain tracked beyond the current exchange, including explicit to-dos and statements such as “I need to…”, “we need to…”, “remember to…”, or “add a task…”.

Do not create a new task for:

- ordinary questions, explanations, brainstorming, or hypothetical ideas with no action commitment;
- status queries;
- progress/done updates that clearly refer to an existing task;
- repository/protocol/bootstrap maintenance unless the user explicitly asks to track it;
- work already represented in Backlog or In progress.

If an explicit task can be fully handled in the current turn but the user framed it as something to track, capture it. Otherwise ordinary one-off assistant work is not automatically a task.

## Classification and deduplication

- Clearly personal work goes to Private; clearly work/client/business work goes to Business.
- Use surrounding context and existing tasks to avoid unnecessary questions.
- If classification remains genuinely ambiguous, preserve the task in Private with a nested note `classification: tentative — private/business unclear`, persist it, and mention the ambiguity. Move it later if corrected.
- Before adding a task, check both open collections for the same underlying intent. Prefer updating an existing task over creating a near-duplicate.
- Processing the same or materially equivalent source more than once MUST NOT create duplicate task intent already represented in Backlog or In progress.

## Task format

Open task:

```markdown
- [ ] Short title
  - context / source
  - progress: YYYY-MM-DD — what happened
```

Completed task:

```markdown
- [x] Short title (YYYY-MM-DD)
  - optional notes / last progress
```

Use today's date when the user reports finishing unless they provide another date.

## Source-material intake

Source material may arrive through any capability exposed by the host, including chat messages, images, audio, documents, files, links, or attachments.

When source material contains trackable work:

1. interpret/extract the actionable intent;
2. deduplicate it against existing open task state;
3. classify it as Private or Business;
4. persist the resulting task mutation to the locked store in the same turn.

Additional rules:

- For screenshots/photos/images, extract the actual to-dos rather than creating a task such as “look at screenshot”.
- For voice/audio, transcribe or summarize when possible; if the content cannot be interpreted, ask only for the missing information needed to extract the task.
- For text/markdown/chat/documents, extract trackable tasks directly.
- Source material itself is not opendump task state and MUST NOT be copied, staged, archived, moved, or retained in the opendump store unless the user explicitly requests retention.
- Transport, upload handling, watched directories, staging, source-file movement, acknowledgement queues, and post-processing lifecycle are host/integration concerns.

## Maintaining the protocol

- Change normal task semantics in `AGENTS.md`.
- Change initialization, recovery, store selection, adapter generation, or verification semantics in `HARNESS_BOOTSTRAP.md`.
- Keep the public template host-neutral: no committed host-specific adapters or environment-specific binding manifests.
- Keep source-ingestion transport/staging outside the canonical store schema.
- Generated adapters belong only in initialized host environments and MUST preserve exactly one store mode/location.
