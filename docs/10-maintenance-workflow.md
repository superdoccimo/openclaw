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

## Daily Note Rule

Use `daily/TEMPLATE.md` for rough findings.

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

Promote a daily note to `docs/` when it becomes reusable.

Reusable means at least one of these is true:

- the same failure pattern happened more than once
- the finding changes how heartbeat or dashboard should behave
- the finding affects security boundaries
- the finding explains a common misleading signal
- the finding can become a runbook or notification example

## Writing Style

- Prefer field notes over polished marketing text
- State the failure layer clearly
- Separate observation from assessment
- Include sanitized evidence
- Use public role names only
- Keep product-specific preference practical, not absolute

## Versioning

Use `CHANGELOG.md` for structural changes to the repository. Use daily notes for operational discoveries.
