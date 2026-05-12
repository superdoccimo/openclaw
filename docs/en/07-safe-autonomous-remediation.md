# Safe Autonomous Remediation

Autonomous remediation requires explicit boundaries.

The operating rule is: broad read-only access, narrow write access, apply through wrappers, test before apply, rollback required, no secrets in output.

## Levels

| Level | Capability | Example |
| --- | --- | --- |
| L0 | observe only | status, journal, cache age |
| L1 | propose | runbook and patch plan |
| L2 | bounded apply | run an allowlisted wrapper |
| L3 | operator-approved apply | change config after explicit human approval |

Do not start at L2 or L3. Promote only after a recurring pattern is understood.

## Requirements Before Apply

- target is allowlisted
- current state is recorded
- dry-run or config test passes
- rollback path exists
- secrets are not displayed
- result is recorded in the daily log

## Bad Automation

```text
agent found suspicious request
agent edits nginx config directly
agent restarts service
dashboard turns green
```

This hides the evidence, rollback path, and config validity.

## Better Automation

```text
agent found repeated deny candidate
agent writes sanitized finding
agent runs wrapper --dry-run
agent runs nginx -t or equivalent test
agent applies allowlisted rule through wrapper
agent records result and rollback command
agent reports SECURITY_WARNING -> SECURITY_OK only after verification
```

## Wrapper Principle

Do not let an agent edit raw config directly. Constrain changes through operator-designed wrappers.

A wrapper should provide:

- allowlist
- dry-run
- config test
- backup
- rollback
- structured output
- secret masking
