---
name: opendump
description: >-
  Harness-portable personal/business task dump with a locked store mode
  (github preferred; local files or artifacts as fallbacks). Use at the
  start of a chat in this project, on explicit cold-start/bootstrap requests,
  whenever the user mentions a new trackable task, or when they ask what is
  open, in progress, done, private, or business.
---

# opendump

## Locked store (this template checkout)

- store: `github`
- location: this repository on `main`
- Persist: commit + push same turn

1. At conversation start, honor the locked mode above. Read `opendump.config.md` if present, then `AGENTS.md` and `HARNESS_BOOTSTRAP.md`, then `private.md` and `business.md`. Do not rely on stale chat context if the store is available.
2. On explicit **cold start / initialization / bootstrap**, execute `HARNESS_BOOTSTRAP.md`: capability ladder → confirm exactly one mode → update the native adapter so mode/location are unambiguous (no “if GitHub else local” branching) → verify → load live tasks.
3. This Cursor skill/rule pair is derived. Do not use it as the upstream specification for Claude, ChatGPT, OpenCode, or another harness.
4. New trackable task → immediately add to Backlog in `private.md` / `business.md` in the locked store and persist per mode. No separate confirmation step.
5. Before adding, deduplicate against Backlog + In progress in both lists. Corrections update the existing item.
6. Classify from context. If still genuinely unclear, capture in `private.md` with `classification: tentative — private/business unclear`, persist, and flag the ambiguity in the reply.
7. Do not auto-create tasks from ordinary questions, brainstorming, hypotheticals, status queries, repository/instruction/bootstrap maintenance, or progress/done updates to existing tasks.
8. Surface Backlog + In progress (Private then Business) at conversation start and when asked. Status prompts like “What’s up?”, “What’s the status?”, and “Any news?” must open with all open tasks.
9. Progress → dated note and move to In progress if needed. Done → remove from the active file and append to the matching completed archive with `YYYY-MM-DD`. Never keep Done sections or completed entries in active lists. Persist every mutation per locked mode.
10. Do not load completed archives during routine startup/status; read only for explicit completed-history lookups.
11. Never commit raw `inbox/` / `processed/` captures. In `github` mode keep commits short and audit-friendly; group changes from the same user message when practical.
12. If the locked store is unavailable, do not silently switch modes; say so and offer reconnect or a cold-start mode change. In `unsupported` contexts, do not pretend chat memory is the task database.
