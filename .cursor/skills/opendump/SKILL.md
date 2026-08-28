---
name: opendump
description: >-
  Harness-portable GitHub-synced personal/business task dump. Use at the
  start of a chat in this project, on explicit cold-start/bootstrap requests,
  whenever the user mentions a new trackable task, or when they ask what is
  open, in progress, done, private, or business.
---

# opendump

1. At conversation start, review the latest state of this repository on `main`. Read `AGENTS.md` and `HARNESS_BOOTSTRAP.md` first; they are canonical for workflow and harness translation. Also read `private.md` and `business.md`. Do not rely on stale chat context if GitHub is available.
2. If the user has provided access to the authoritative repo and explicitly requests a **cold start / initialization / bootstrap**, execute `HARNESS_BOOTSTRAP.md`. Detect the current harness, update its native persistent instruction adapter idempotently, preserve unrelated instructions, verify persistence when possible, then load live tasks.
3. This Cursor skill/rule pair is derived. Do not use it as the upstream specification for Claude, ChatGPT, OpenCode, or another harness.
4. New trackable task → immediately add to Backlog in `private.md` / `business.md` in the authoritative repo and push in the same turn. No separate confirmation step.
5. Before adding, deduplicate against Backlog + In progress in both lists. Corrections update the existing item.
6. Classify from context. If still genuinely unclear, capture in `private.md` with `classification: tentative — private/business unclear`, push, and flag the ambiguity in the reply.
7. Do not auto-create tasks from ordinary questions, brainstorming, hypotheticals, status queries, repository/instruction/bootstrap maintenance, or progress/done updates to existing tasks.
8. Surface Backlog + In progress (Private then Business) at conversation start and when asked. Status prompts like “What’s up?”, “What’s the status?”, and “Any news?” must open with all open tasks.
9. Progress → dated note and move to In progress if needed. Done → remove from the active file and append to the matching `private-completed.md` or `business-completed.md` archive with `YYYY-MM-DD`. Never keep Done sections or completed entries in `private.md` / `business.md`. Push every mutation.
10. Do not load `private-completed.md` or `business-completed.md` during routine startup/status; read it only for explicit completed-history or archive lookups.
11. Never commit raw `inbox/` / `processed/` captures. Keep commits short and audit-friendly; group changes from the same user message when practical.
