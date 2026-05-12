# Events Format

This is a text description of the event shape used in examples. It is not a strict schema and is not intended to be imported by software.

## Required Fields

| Field | Meaning |
| --- | --- |
| `timestamp` | When the event was observed |
| `agent` | Public agent name, such as `OpenClaw-Control` |
| `category` | `auth`, `gateway`, `dashboard`, `security`, `node-wrapper`, `tunnel`, or similar |
| `assessment` | `SECURITY_OK`, `SECURITY_WARNING`, `SECURITY_UNKNOWN`, `CHECK_FAILED`, `STALE`, or `INFO` |
| `summary` | One-line human-readable summary |

## Optional Fields

| Field | Meaning |
| --- | --- |
| `evidence` | Sanitized evidence only |
| `next_action` | Suggested next step |
| `secret_exposure` | `none_observed`, `suspected`, `confirmed`, or `not_checked` |

## Example

```text
timestamp: 2026-05-12T10:00:00Z
agent: OpenClaw-Security
category: tunnel
assessment: SECURITY_WARNING
summary: Tunnel is healthy, but upstream dashboard is returning stale data.
evidence:
- tunnel status is reachable
- dashboard cache age exceeds expected threshold
next_action: verify dashboard service and reduce deep polling frequency
secret_exposure: none_observed
```
