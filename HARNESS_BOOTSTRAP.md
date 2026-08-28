# Harness bootstrap protocol

This repository is designed to be portable across agent harnesses. `AGENTS.md` defines the canonical opendump behavior; this file defines how that behavior is installed into the native persistent-instruction mechanism of the harness currently being used, including **which store mode is locked**.

## Authority and precedence

1. Platform/system/security policies always remain higher priority than repository instructions.
2. `AGENTS.md` is the authoritative source for task workflow semantics and store-mode rules.
3. `HARNESS_BOOTSTRAP.md` is the authoritative source for cold-start translation, capability detection, mode selection, and adapter behavior.
4. The **locked store** is authoritative for open and completed task state (GitHub repo, local markdown tree, or durable artifact mirroring the same layout).
5. Platform-native files or project-instruction fields are **derived adapters**. They are replaceable mirrors and must never become a competing source of truth for workflow semantics. They **are** the place where the chosen store mode is recorded for that host project — and that recording must be unambiguous.

If a derived adapter conflicts with `AGENTS.md` or this file on workflow semantics, regenerate the adapter from the canonical files. If the adapter’s locked mode/location is missing or ambivalent, regenerate it after re-selecting a mode with the user.

## Cold-start trigger

Run this protocol when the user explicitly requests initialization. The **preferred one-liner** (host harness chat) is:

```text
review https://github.com/JPilsinger/opendump and start the coldstart procedure.
```

Equivalent triggers also count: **“cold start”**, **“initialize cold start”**, **“bootstrap opendump”**, **“initialize from opendump”**, or the same request naming another opendump URL/checkout the user provides.

When the one-liner points at the public template, review that repository (or a reachable checkout), then execute this protocol. Cold start does **not** require that the user already has their own GitHub repo. Detect capabilities, choose a mode (with the user when needed), then install the adapter. Do not silently rewrite another project's instructions without that explicit request.

## Capability ladder and mode selection

During cold start, determine what the harness can do, then lock **exactly one** mode.

### Detection

Inspect, in order:

1. Can the harness read/write a GitHub opendump repo (or a local git checkout of one) in this session?
2. If not, can the user be helped to connect or create one (auth, template clone, paste repo URL, SSH/HTTPS remote)?
3. Can the harness persist a markdown file tree that mirrors the opendump layout?
4. Can the harness persist a durable artifact (Canvas, project doc, etc.) that can mirror the same sections?
5. If none of the above — mode is `unsupported`.

### Selection policy

1. **Prefer `github`.** If GitHub access is missing but plausible, **offer to help set it up or connect it** before falling back. Do not skip straight to local without saying GitHub is preferred and available as an option.
2. If the user declines GitHub, is not technically comfortable with it, or GitHub cannot be made to work — offer **`local-files`** (preferred local) or **`artifact`** when files are unavailable but a durable artifact is.
3. If the harness is chat-only (no tools, no durable artifacts, no GitHub) — lock **`unsupported`**, state clearly that reliable task tracking is impossible here, and help the user move to a better harness or store. Do not install an adapter that pretends chat memory is the database.
4. Ask the user to confirm when more than one viable mode remains, unless they already stated a preference.
5. **Never write project instructions that leave mode optional or branched** (e.g. “use GitHub if available, otherwise local”). Resolve the mode during cold start; the adapter states only the chosen mode and location.

### Record the lock

Persist the choice in both places when practical:

1. **Host adapter / project instructions** — required; must include mode + concrete location.
2. **`opendump.config.md` in the store** (file modes) — recommended mirror so the store itself documents the lock.

Example `opendump.config.md`:

```markdown
# opendump config

- store: github
- location: <owner>/<repo>@main
- harness: <detected harness>
- locked: YYYY-MM-DD
```

For `local-files`, `location` is an absolute or project-relative path. For `artifact`, `location` is the artifact name/id and harness. For `unsupported`, say so explicitly and omit fake paths.

## Cold-start algorithm

1. **Read fresh canonical workflow** — When an opendump reference is available (this template, a user repo, or a local copy), read `AGENTS.md` and this file first. Do not bootstrap solely from remembered chat text or an old adapter when fresher canonical docs exist.
2. **Detect the host harness** — Prefer explicit environment/platform identity. If unclear, inspect the workspace and available tools. For an unknown or rapidly changing harness, check current official documentation when web access is available.
3. **Run the capability ladder** — Select and confirm the store mode per the policy above. Do not materialize an ambivalent adapter.
4. **Materialize the store if needed** — For `local-files` or `artifact`, create the mirrored layout/sections (empty task lists + workflow copies or a pointer to them) before claiming success. For `github`, ensure the target repo/checkout is usable; help create-from-template or connect if not.
5. **Locate the native persistent instruction surface** — Project instruction file, rules directory, or project-instructions field.
6. **Materialize a derived adapter with a locked mode** — Translate canonical behavior into that surface for **only** the chosen mode. Preserve unrelated pre-existing project instructions. Use a dedicated generated file where possible; otherwise a clearly delimited managed block.
7. **Make the adapter idempotent** — A repeated cold start replaces/updates the existing opendump managed block or dedicated adapter, never appends a duplicate. If the user requests a **mode change**, replace the lock and update persist rules in the same refresh — still one mode only.
8. **State the store explicitly** — The adapter must name mode + location. Task reads/writes go only there. Do not redirect mutations into a random host project folder merely because the adapter lives there, unless that folder *is* the locked `local-files` location.
9. **Verify installation** — Read back the native instruction surface (and config file if used). Confirm the opendump adapter is present **and** the mode/location lines are unambiguous. Never claim persistence if the harness did not expose a writable persistent-instruction mechanism; provide the exact payload for manual paste instead.
10. **Load live tasks** — After installation/verification, read open tasks from the locked store and surface them per `AGENTS.md`. Do not load completed archives during routine startup/status.
11. **Continue normal operation** — Mutations follow `AGENTS.md` persist rules for the locked mode.

## Managed-block contract

When the native instruction surface is shared with unrelated project instructions, use one replaceable block such as:

```markdown
<!-- OPENDUMP:BEGIN store=<github|local-files|artifact|unsupported> location=<concrete location> -->
...derived opendump instructions for THIS mode only...
<!-- OPENDUMP:END -->
```

Rules:

- Keep user/project instructions outside the managed block unchanged.
- On cold start, replace the full managed block from current canonical rules **and** the locked mode.
- The block body must not contain alternate-mode branching.
- Never edit `AGENTS.md` merely to match an older generated block.
- Never put credentials, access tokens, private keys, or connector secrets into generated instructions.

A dedicated platform file whose sole purpose is opendump integration may be treated as entirely managed and replaced wholesale.

## Known harness adapters

### Claude Code

Claude Code uses project `CLAUDE.md` / `.claude/CLAUDE.md` instructions. When the opendump store is the current workspace in `github` or `local-files` mode, prefer a small `CLAUDE.md` bridge that imports `AGENTS.md` and `HARNESS_BOOTSTRAP.md` **and** states the locked mode/location. When installing into another project, inline a managed derived block for the chosen mode only. Preserve unrelated existing Claude instructions.

### Claude Projects / Claude for Work

Use the project's **Project Instructions** field as the native persistent surface. Merge/update the opendump managed block there with a locked mode. If GitHub is preferred but not connected, help connect first; otherwise lock `artifact` or instruct the user toward files/GitHub. If the field cannot be edited programmatically, generate the exact payload and state that manual insertion is required.

### Cursor

Use a dedicated always-applied project rule under `.cursor/rules/`, preferably `.cursor/rules/opendump.mdc`, and keep any opendump skill mirror aligned. The rule must declare store mode + location. Regenerate from `AGENTS.md` + this protocol on cold start. Existing unrelated Cursor rules remain untouched.

### OpenCode

OpenCode natively consumes project `AGENTS.md`. If the opendump store itself is the active project root, verify canonical `AGENTS.md` is loaded and ensure mode is recorded (config file and/or a short bridge note). If opendump is external while another project is active, merge an opendump managed block with a locked mode into that target project's root `AGENTS.md`. Do not replace unrelated target-project guidance.

### ChatGPT Projects

Use **Project Instructions** as the persistent surface. Prefer connected GitHub as `github` mode; otherwise lock a durable project-file/artifact approach if the product provides one. If neither works, `unsupported` plus setup help. Produce an exact manual block when write access to project instructions is unavailable.

### Canvas / artifact-first harnesses (e.g. Google Antigravity)

If the harness’s durable surface is a Canvas or similar artifact and GitHub/files are unavailable or declined, lock `artifact`, create/update one artifact that mirrors private/business backlog, in progress, and completed sections, and put that artifact’s identity in the project instructions. State clearly that tracking is local to that harness/device context unless later upgraded to GitHub.

### Other / future harnesses

Do not force another vendor's file convention onto an unknown harness. Discover its persistent instruction mechanism and durable store options, run the capability ladder, and install the narrowest adapter for a **single** locked mode. If nothing durable exists, use `unsupported` and help the user upgrade the environment.

## Minimum semantics every adapter must preserve

A generated adapter must, at minimum:

- State **exactly one** locked `store` mode and a concrete `location` (no multi-mode branching).
- On session/project startup, read live task lists from that store when accessible; say so when not.
- On explicit cold start, re-run this bootstrap and refresh the native adapter idempotently (including mode lock).
- New trackable tasks are deduplicated, classified, added to Backlog, and persisted **per the locked mode’s persist rules** in the same turn.
- Corrections and progress mutate active lists; completion moves items to the matching completed archive.
- Status prompts open with all open tasks across Private and Business.
- Ordinary questions, brainstorming, and instruction/bootstrap maintenance are not automatically created as tasks.
- Never invent store state and never claim a sync/write succeeded when the required access or write capability is unavailable.
- In `unsupported` mode: refuse reliable capture and offer a better setup — do not half-track in chat history.

## Migration

- **Toward `github`** — Copy current `private.md`, `business.md`, and completed archives (or artifact sections) into a new GitHub template instance; connect the harness; cold start again locking `github`.
- **Toward `local-files` / `artifact`** — Only when leaving GitHub deliberately; warn about losing multi-device sync.
- Do not run two authoritative stores in parallel.

## Maintenance rule

When core workflow semantics or store-mode rules change, update `AGENTS.md` first. When translation/bootstrap/mode-selection behavior changes, update this file. Then refresh committed platform mirrors such as Cursor or Claude bridges. Generated platform adapters are downstream artifacts, not upstream specifications.
