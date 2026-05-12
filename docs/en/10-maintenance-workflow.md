# Maintenance Workflow

This repository is expected to change often. The goal is not to make every note perfect on the first write. The goal is to keep discoveries easy to add, easy to review, and safe to publish.

## Update Flow

```text
daily note
  -> repeated pattern
  -> stable docs page
  -> example notification or state format
  -> README link if it becomes a major topic
```

## Language Rule

Canonical docs are English.

Japanese content is intentionally lighter:

- `README.ja.md` for the Japanese entrypoint
- `docs/ja/` for summaries, not full translations
- `daily/ja/` for frequent Japanese operation notes

Do not maintain full parallel English and Japanese translations unless a page becomes important enough to justify it.

## Daily Note Rule

Use `daily/TEMPLATE.md` or `daily/ja/TEMPLATE.md` for rough findings.

Daily notes should contain:

- what was observed
- what was misleading
- what layer actually failed
- what should be checked next time
- whether the finding should be promoted later

Daily notes should not contain:

- real host names
- real IP addresses
- real domains
- channel IDs
- raw access logs
- token, cookie, credential, or secret values
- executable scripts copied from private operations

## Promotion Rule

Promote a daily note to `docs/en/` when it becomes reusable.

Reusable means at least one of these is true:

- the same failure pattern happened more than once
- the finding changes how heartbeat or dashboard should behave
- the finding affects security boundaries
- the finding explains a common misleading signal
- the finding can become a runbook or notification example

## Writing Style

- prefer field notes over marketing text
- state the failure layer clearly
- separate observation from assessment
- include sanitized evidence
- use public role names only
- keep product-specific preferences practical, not absolute

## Versioning

Use `CHANGELOG.md` for structural changes to the repository. Use daily notes for operational discoveries.
