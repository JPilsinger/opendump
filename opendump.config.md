# opendump config

- store: github
- location: this repository @main
- harness: any (template instance)
- locked: 2026-08-28

This public template’s own adapters assume **github** mode against the user’s instantiated repo. Host projects that cold-start from opendump must lock a single mode in *their* project instructions — see `HARNESS_BOOTSTRAP.md`.
