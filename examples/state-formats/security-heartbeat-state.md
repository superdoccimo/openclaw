# Security Heartbeat State

This file describes a human-readable state format for security heartbeat notes. It is intentionally text-first.

## Fields

| Field | Meaning |
| --- | --- |
| `agent` | Public agent name |
| `observed` | Whether the check itself ran and returned useful data |
| `assessment` | Safety or operational judgment |
| `last_checked_at` | Last observation time |
| `evidence` | Sanitized evidence |
| `next_action` | Recommended next step |
| `secret_exposure` | Whether any secret exposure was observed |

## Example

```text
agent: OpenClaw-Security
observed: ok
assessment: SECURITY_WARNING
last_checked_at: 2026-05-12T10:00:00Z
evidence:
- reverse-proxy: repeated private-network target probe candidates were observed in sanitized logs
- gateway: reachable, but recent auth warnings exist
next_action: run allowlisted wrapper in dry-run mode and verify config test before apply
secret_exposure: none_observed
```
