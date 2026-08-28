# Harness bootstrap protocol

This repository is designed to be portable across agent harnesses. `AGENTS.md` defines the canonical opendump behavior; this file defines how that behavior is installed into the native persistent-instruction mechanism of the harness currently being used.

## Authority and precedence

1. Platform/system/security policies always remain higher priority than repository instructions.
2. `AGENTS.md` is the authoritative source for task workflow semantics.
3. `HARNESS_BOOTSTRAP.md` is the authoritative source for cold-start translation and adapter behavior.
4. `private.md` and `business.md` are authoritative for open and in-progress task state; `private-completed.md` and `business-completed.md` are the respective completed-task archives.
5. Platform-native files or project-instruction fields are **derived adapters**. They are replaceable mirrors and must never become a competing source of truth.

If a derived adapter conflicts with `AGENTS.md` or this file, regenerate the adapter from the canonical files.

## Cold-start trigger

Run this protocol only when both conditions are true:

- the user has granted the current harness access to **this** opendump repository (or supplied a local checkout/reference to it); and
- the user explicitly requests initialization, for example **“cold start”**, **“initialize cold start”**, **“bootstrap opendump”**, **“initialize from opendump”**, or equivalent language.

Repository access alone does not silently rewrite another project's instructions.

## Cold-start algorithm

1. **Read fresh canonical state** — Read the latest `main` branch, starting with `AGENTS.md` and this file, then `private.md` and `business.md`. Do not bootstrap from remembered chat text or an old generated adapter when the canonical repository is available.
2. **Detect the host harness** — Prefer explicit environment/platform identity. If unclear, inspect the current workspace and available platform tools for native instruction mechanisms. For an unknown or rapidly changing harness, check its current official documentation when web access is available rather than assuming an old convention.
3. **Locate the native persistent instruction surface** — Use a project-scoped mechanism whenever the harness provides one: a native project instruction file, rules directory, or project-instructions field.
4. **Materialize a derived adapter** — Translate the canonical behavior into that native surface. Preserve all unrelated pre-existing project instructions. Use a dedicated generated file where possible; otherwise use a clearly delimited managed block.
5. **Make the adapter idempotent** — A repeated cold start must replace/update the existing opendump managed block or dedicated adapter, never append a duplicate copy.
6. **Keep the remote source explicit** — The adapter must state that task reads/writes go to **this** opendump repository on `main` (the user's template instance or fork). Do not redirect task mutations into the host project's repository merely because that is where the adapter lives.
7. **Verify installation** — Read back the native instruction surface, or use the harness's inspection command/tool when available, and confirm that the opendump adapter is present. Never claim persistence if the harness did not expose a writable persistent-instruction mechanism.
8. **Load live tasks** — After installation/verification, read `private.md` and `business.md`, then surface the current open tasks as required by `AGENTS.md`. Do not load `private-completed.md` or `business-completed.md` during routine startup or status reporting.
9. **Continue normal operation** — New tasks, corrections, progress, and completions are written back to the authoritative GitHub repository according to `AGENTS.md`.

## Managed-block contract

When the native instruction surface is shared with unrelated project instructions, use one replaceable block such as:

```markdown
<!-- OPENDUMP:BEGIN managed source=<owner>/<repo>@main -->
...derived opendump instructions...
<!-- OPENDUMP:END -->
```

Replace `<owner>/<repo>` with the GitHub owner and name of **this** opendump instance.

Rules:

- Keep user/project instructions outside the managed block unchanged.
- On cold start, replace the full managed block from current canonical rules.
- Never edit `AGENTS.md` merely to match an older generated block.
- Never put credentials, access tokens, private keys, or connector secrets into generated instructions.

A dedicated platform file whose sole purpose is opendump integration may be treated as entirely managed and replaced wholesale.

## Known harness adapters

### Claude Code

Claude Code uses project `CLAUDE.md` / `.claude/CLAUDE.md` instructions. When the canonical repository is the current workspace, prefer a small `CLAUDE.md` bridge that imports `AGENTS.md` and `HARNESS_BOOTSTRAP.md`. When the opendump repository is available at another local path, import those files by relative path if practical; otherwise inline a managed derived block containing the complete operational behavior. Preserve any unrelated existing Claude instructions.

### Claude Projects / Claude for Work

Use the project's **Project Instructions** field as the native persistent surface. If the current harness exposes a tool/API that can edit project instructions, merge/update the opendump managed block there. If it cannot programmatically edit that field, generate the exact managed-block payload for the user and state that manual insertion is required; do not pretend it was persisted.

### Cursor

Use a dedicated always-applied project rule under `.cursor/rules/`, preferably `.cursor/rules/opendump.mdc`, and keep any opendump skill mirror aligned if the project uses one. Regenerate the dedicated rule from `AGENTS.md` + this protocol on cold start. Existing unrelated Cursor rules remain untouched.

### OpenCode

OpenCode natively consumes project `AGENTS.md`. If the canonical repository itself is the active project root, no translated duplicate is necessary: verify that the canonical `AGENTS.md` is being loaded. If opendump is only an external/reference repository while another project is active, merge an opendump managed block into that target project's root `AGENTS.md` so the active OpenCode session receives the behavior. Do not replace unrelated target-project guidance.

### ChatGPT Projects

Use the ChatGPT project's **Project Instructions** as the persistent surface and keep GitHub as the live authoritative data source. If the current ChatGPT environment exposes a writable project-instructions action, merge/update the managed block there. If no such write action is available, produce the exact block for manual insertion and clearly report the limitation. Connected GitHub access should still be used to read fresh canonical rules and task state when available.

### Other / future harnesses

Do not force another vendor's file convention onto an unknown harness. Discover the harness's current project-scoped persistent instruction mechanism and create the narrowest native adapter that preserves the semantics in `AGENTS.md` and this file. If no native persistent mechanism exists, use the closest project-scoped instruction facility available; if none is writable, return an exact bootstrap payload and state that persistence could not be automated.

## Minimum semantics every adapter must preserve

A generated adapter must, at minimum, carry these behaviors:

- This opendump repository on `main` is the source of truth.
- On session/project startup, read fresh canonical instructions and live task lists when repository access is available.
- On explicit cold start, re-run this bootstrap and refresh the native adapter idempotently.
- New trackable tasks are automatically deduplicated, classified, added to Backlog, and pushed in the same turn.
- Corrections and progress mutate the active task lists and are pushed. Completion removes the item from its active list and appends it to the matching `private-completed.md` or `business-completed.md` archive.
- Status prompts open with all open tasks across Private and Business.
- Ordinary questions, brainstorming, and repository-instruction maintenance are not automatically created as tasks.
- Never invent repository state and never claim a sync/write succeeded when the required access or native write capability is unavailable.

## Maintenance rule

When core workflow semantics change, update `AGENTS.md` first. When translation/bootstrap behavior changes, update this file. Then refresh committed platform mirrors such as Cursor or Claude bridges. Generated platform adapters are downstream artifacts, not upstream specifications.
