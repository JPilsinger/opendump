---
name: opendump
description: >-
  Public opendump template: cold-start into a user-owned GitHub repo (Use this
  template), or local/artifact stores. Never push personal tasks to
  JPilsinger/opendump. Use on cold-start one-liners and task capture/status.
---

# opendump

## Public template — no personal task writes

- upstream: `JPilsinger/opendump`
- **Never** commit/push user tasks here
- Preferred GitHub path: **Use this template** → user-owned repo → lock that repo as `store: github`

1. On the one-liner `review https://github.com/JPilsinger/opendump and start the coldstart procedure.` (or equivalent), execute `HARNESS_BOOTSTRAP.md`. Review this template for workflow only.
2. Capability ladder → confirm **exactly one** mode → if `github`, create/connect a **user-specific** template instance (not this public repo) → write host adapter with that mode/location only → verify → load tasks from the user store.
3. This Cursor skill/rule pair is derived. Do not use it as the upstream specification for other harnesses.
4. New trackable task → Backlog in the **locked user-owned store** only; persist per mode. No separate confirmation step.
5. Deduplicate against that store. Corrections update the existing item.
6. Classify from context; if unclear, capture under private with `classification: tentative — private/business unclear`, persist, and flag it.
7. Do not auto-create tasks from ordinary questions, brainstorming, hypotheticals, status queries, or bootstrap maintenance.
8. Surface Backlog + In progress from the locked store. Status prompts must open with all open tasks.
9. Progress / done → locked store only. Push only to the user’s `github` repo.
10. Do not load completed archives during routine startup/status.
11. Never commit raw `inbox/` / `processed/` captures. Never push user data to `JPilsinger/opendump`.
12. If the locked store is unavailable, do not silently switch to the public template; say so and offer reconnect or cold-start mode change.
