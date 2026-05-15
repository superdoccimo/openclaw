# Event Store Recovery Semantics

`events.json` is operational history. It should not be treated as a mutable
status table.

This distinction matters because a healthy recovery can still leave historical
error records behind. If a dashboard reads those raw records as current status,
the operator sees a persistent failure even after the agent or peer has already
recovered.

## Core Rule

Keep the historical trail, but alert on unresolved state.

Useful dashboard fields:

- `historicalErrors`: recent raw events with `status=error` or
  `severity=error`
- `unresolvedErrors`: errors that have not been cleared by a later recovery
  event
- `errors`: optional backward-compatible alias for `unresolvedErrors`
- `latestAt`: latest event timestamp, independent of error status

Do not delete old error events just to make a dashboard green. The old event is
evidence. The recovery event is also evidence. The dashboard should understand
both.

## Recovery Events

A recovery event should be machine-readable, not only a human sentence.

Recommended shape:

```json
{
  "source": "OpenClaw-Control",
  "type": "watch.remote",
  "severity": "info",
  "status": "ok",
  "summary": "[OpenClaw-Control/remote] peer watch recovered",
  "tags": ["control", "watch", "remote", "recovery"],
  "data": {
    "resolutionKey": "remote-peer-watch",
    "recovery": true,
    "deliveryOk": true
  }
}
```

Use a stable `data.resolutionKey` when possible.

Fallback subject fields can be useful when older producers already emit them:

- `resolves`
- `incidentKey`
- `errorKey`
- `watch`
- `target`
- `name`

The recovery matcher should normally group by:

```text
source + type + subject
```

If no subject exists, grouping by `source + type` is acceptable for simple
single-purpose watches, but it becomes too coarse when one producer emits
multiple independent incidents of the same type.

## Source Of Truth

Copied deployments often carry stale state files from another host. That can
make a healthy agent look broken.

Every generated or persisted operational state should declare ownership:

```json
{
  "_meta": {
    "schemaVersion": 2,
    "sourceOfTruth": "OpenClaw-Control:/path/to/state.json",
    "updatedByRole": "OpenClaw-Control",
    "description": "Role states are host-owned. Do not treat archived foreign roles as current status."
  }
}
```

For multi-agent operations, avoid ambiguous keys such as `remote` or
`agent-update` without owner metadata. Prefer explicit role names, source of
truth paths, and direction:

```text
OpenClaw-Control updates OpenClaw-A
OpenClaw-A reports its own local heartbeat
```

If a state file contains a role owned by another host, archive it as stale
foreign state rather than presenting it as current status.

## Dashboard Behavior

Good dashboard behavior:

- show unresolved error count as the alert number
- keep historical error count visible as context
- label values as `unresolved`, `historical`, or `latest`
- show the latest recovery event when it cleared a previous alert
- make the event store path and agent data directory visible in diagnostics

Bad dashboard behavior:

- label historical errors as current errors
- hide historical errors after recovery
- use `events.length` or raw error count as an alert badge
- copy another agent's default event path without changing labels and state
  ownership
- rerun heavy checks just because an old event is red

## Heartbeat And Review Use

Heartbeat should not panic on historical errors alone.

When a heartbeat sees `historicalErrors > 0` and `unresolvedErrors = 0`, it
should treat the situation as recovered history. The useful work is to check
whether the recovery taught a reusable lesson:

- Was the detector too strict?
- Was the dashboard label ambiguous?
- Did a service environment differ from the interactive shell?
- Did the copied deployment have stale source-of-truth metadata?
- Should a runbook or public field note be updated?

Weekly review can still inspect historical errors. The goal is not to keep the
dashboard green by hiding evidence. The goal is to separate current action from
learning material.

## Publication Boundary

When publishing this pattern, keep the event semantics and remove private
details.

Safe to publish:

- field names and matching rules
- sanitized example events
- dashboard behavior principles
- source-of-truth ownership pattern

Do not publish:

- real hostnames or SSH aliases
- private paths that identify an operator
- raw event payloads containing channel IDs, tokens, or command output
- screenshots that expose private dashboard data

The reusable lesson is that event history, current alert state, and recovery
evidence are three different things.
