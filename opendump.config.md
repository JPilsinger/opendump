# opendump config

- role: public-upstream-template
- store: none (not a personal task store)
- upstream: JPilsinger/opendump
- task-writes: forbidden
- locked: 2026-08-28

This tree is the **public template**. Agents must not capture or push user tasks here.

For a real installation, cold start must lock a **user-owned** store:

- `github` → repo created via **Use this template** (never `JPilsinger/opendump` as the write target)
- or `local-files` / `artifact` per `HARNESS_BOOTSTRAP.md`

After you create your own instance, replace this file with your lock, for example:

```markdown
# opendump config

- store: github
- location: <your-user>/<your-opendump-repo>@main
- harness: <harness>
- locked: YYYY-MM-DD
```
