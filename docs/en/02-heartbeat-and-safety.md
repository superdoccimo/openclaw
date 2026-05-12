# Heartbeat And Safety

Heartbeat should not only answer "did the check run?"

`exit 0` means the check completed. It does not prove the environment is safe or that the real production response path is working.

## Separate Execution From Assessment

Weak signal:

```text
heartbeat finished
status: ok
```

Better signal:

```text
observed: ok
assessment: SECURITY_WARNING
reason: gateway reachable, but auth profile has expired fallback
next_action: repair auth order and restart gateway
```

## Suggested State Model

| Field | Meaning |
| --- | --- |
| `observed` | whether the check produced usable observations |
| `assessment` | safety or operational judgment |
| `evidence` | what the judgment is based on |
| `next_action` | what a human or agent should do next |
| `last_checked_at` | last observation time |

`observed: ok` and `assessment: SECURITY_OK` are different claims.

## Recommended Assessment Values

- `SECURITY_OK`
- `SECURITY_WARNING`
- `SECURITY_UNKNOWN`
- `CHECK_FAILED`
- `STALE`

## What Heartbeat Should Check

- whether OpenClaw CLI is launched from the expected path
- whether the gateway service is active
- whether auth order prioritizes the production profile
- whether channel send/receive paths work
- whether dashboard APIs are too heavy or stale
- whether tunnel / reverse proxy / private mesh access is reachable
- whether recent journal entries show timeouts or credential errors

## What Heartbeat Should Avoid

- running deep status checks at short intervals
- printing token or credential values
- auto-applying fixes without evidence
- turning the dashboard green based only on `exit 0`

## Notification Format

Keep notifications short enough for a human to act on.

```text
[OpenClaw-Control/auth] auth check
- OpenClaw-Control: OpenClaw openai-codex production profile ok
- OpenClaw-A: OpenClaw openai-codex OAuth expired
- OpenClaw-Research: Hermes openai-codex OAuth exhausted
- OpenClaw-Security: ok
assessment: SECURITY_WARNING
next: repair auth order, then restart gateway
```
