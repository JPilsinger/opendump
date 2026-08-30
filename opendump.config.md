# opendump config

- role: template-seed
- initialized: false
- store: none
- upstream: JPilsinger/opendump

This file is copied verbatim by GitHub's **Use this template** flow. It therefore MUST NOT be treated as proof that the current repository is the public upstream.

Repository role is determined structurally during cold start:

- if the actual current repository is `JPilsinger/opendump`, it is the public upstream template and user task writes are forbidden;
- if the actual current repository is different, this seed means the template copy is uninitialized and must be replaced with its real locked binding.

A successfully initialized file/GitHub instance replaces this seed with, for example:

```markdown
# opendump config

- initialized: true
- store: github
- location: <your-user>/<your-opendump-repo>@main
- harness: <harness>
- locked: YYYY-MM-DD
- upstream: JPilsinger/opendump
```

See `HARNESS_BOOTSTRAP.md` for the normative repository-role, binding, and verification rules.
