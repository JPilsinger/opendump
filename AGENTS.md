# opendump

`AGENTS.md` is the canonical **host-neutral runtime contract** for the OpenDump workflow. It defines what OpenDump means at runtime, independent of how any particular AI host realizes it.

During cold start, `HARNESS_BOOTSTRAP.md` compiles this contract together with the observed host environment and the concrete store binding into one logical **host adapter**. The adapter may link this contract directly or faithfully translate it into one or more host-native agentic assets. The adapter is derived; it is never a second source of workflow truth.

Conceptually:

```text
OpenDumpAdapter = Compile(AGENTS.md, HostEnvironment, StoreBinding)
```

## Normative contract

Requirements in this file are runtime semantics. **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** have their usual normative meanings. Imperative rules are requirements unless explicitly described as optional.

Host-specific asset paths, rule formats, skill formats, project settings, connector setup, installation mechanics, and adapter serialization do not belong in this contract. They belong to bootstrap/compilation.

## Public template vs user-controlled store

`JPilsinger/opendump` is the public template and protocol reference. It is never a user's task database.

A **user-controlled store** is a store owned by the user or explicitly authorized by the user for OpenDump task storage.

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
- `AGENTS.md` accompanies the task state as the canonical runtime contract.
- `HARNESS_BOOTSTRAP.md` accompanies the task state as the bootstrap/compiler protocol, but is not part of normal runtime execution.

Artifact stores MUST provide equivalent logical collections even if they do not use these filenames.

Environment-specific adapter configuration and bindings are not task state. Source material, uploads, attachments, watched folders, staging queues, acknowledgement ledgers, and processed-source archives are also outside the canonical store.

## Store modes

Exactly one mode/location is authoritative for a given installed OpenDump adapter.

| Mode | Store | Persistence |
|---|---|---|
| `github` | User-controlled GitHub repository | Commit/push task mutations to the bound repository |
| `local-files` | Durable local markdown files | Write task mutations in the same turn |
| `artifact` | Durable host artifact | Update the artifact in the same turn |
| `unsupported` | None | No reliable task tracking |

Runtime store rules:

- The installed adapter MUST expose exactly one authoritative store mode and one concrete location.
- The adapter MUST NOT silently choose a different store at runtime.
- A mode/location change requires explicit re-initialization.
- If the bound store becomes inaccessible, report the failure and stop claiming persistence.
- `github` MUST never bind to `JPilsinger/opendump` or another known published upstream template.

## Authority

1. Platform/system/security policies have highest priority.
2. `AGENTS.md` is the sole canonical source of OpenDump runtime semantics.
3. The installed host adapter is the derived host-specific realization of this contract and is authoritative for host-specific realization details and concrete runtime bindings.
4. The bound store is authoritative for current task state.

The adapter may determine **how** this contract is realized on the host and **where** the bound store lives. It MUST NOT redefine **what OpenDump means**.

One logical adapter MAY span multiple physical host-native assets. Those assets collectively form one managed realization and MUST NOT become independent competing OpenDump specifications.

## Runtime activation and startup

At the beginning of every session where OpenDump is installed, the host adapter MUST make the canonical OpenDump runtime contract effective.

The realization MAY be:

- **linked** — the host reliably consumes current `AGENTS.md` semantics directly; or
- **compiled** — the adapter faithfully embeds/translates the runtime semantics into host-native assets because direct linkage is unavailable; or
- **hybrid** — some semantics are linked and the remaining host-specific requirements are compiled.

Regardless of realization strategy, normal startup MUST result in:

1. exactly one authoritative store mode/location resolved from the installed adapter;
2. current `private.md` and `business.md` task state (or artifact equivalents) loaded from that store;
3. the canonical runtime semantics in this file effective for the session;
4. the persistence path required by the bound mode treated as authoritative;
5. completed archives left unloaded unless a specific archive lookup is needed;
6. inaccessible or invalid bindings reported accurately rather than silently replaced.

In GitHub mode, current bound-repository state is authoritative over remembered chat context or stale mirrors.

Normal runtime does not execute `HARNESS_BOOTSTRAP.md`. Bootstrap is used for initialization, recovery, adapter recompilation, or an explicit mode/location change.

## Lifecycle

Behavior is identical across store modes except for where mutations are persisted.

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

A runtime implementation MUST NOT claim a task mutation succeeded until the required store mutation for the active mode succeeded.

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
4. persist the resulting task mutation to the authoritative store in the same turn.

Additional rules:

- For screenshots/photos/images, extract the actual to-dos rather than creating a task such as “look at screenshot”.
- For voice/audio, transcribe or summarize when possible; if the content cannot be interpreted, ask only for the missing information needed to extract the task.
- For text/markdown/chat/documents, extract trackable tasks directly.
- Source material itself is not OpenDump task state and MUST NOT be copied, staged, archived, moved, or retained in the OpenDump store unless the user explicitly requests retention.
- Transport, upload handling, watched directories, staging, source-file movement, acknowledgement queues, and post-processing lifecycle are host/integration concerns.

## Adapter implications

Because this file is the canonical runtime contract:

- every runtime requirement necessary for execution MUST either be reliably available to the host through direct linkage or be faithfully compiled into the installed adapter;
- generated adapter assets MUST be treated as disposable/reproducible outputs, not edited as independent workflow specifications;
- a host-specific adapter MAY use any native agentic primitive required for faithful execution;
- if an embedded/compiled adapter is known to be stale relative to this contract, it MUST be recompiled before it is treated as current;
- supporting a new host SHOULD require changes to bootstrap/compiler knowledge, not host-specific changes to this runtime contract.

## Maintaining OpenDump

- Change OpenDump runtime semantics only in `AGENTS.md`.
- Change host discovery, capability establishment, store selection, compilation, adapter installation, or verification in `HARNESS_BOOTSTRAP.md`.
- Keep the public template host-neutral: no committed host-specific adapters or environment-specific binding manifests.
- Keep source-ingestion transport/staging outside the canonical store schema.
- Regenerate embedded/compiled downstream adapters after relevant `AGENTS.md` changes; linked adapters MAY continue using the current contract when their linkage remains valid.
