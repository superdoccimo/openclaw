# Watch Event Follow-Up

A watch script should not be the end of an incident loop.

In multi-agent OpenClaw operations, a watch can detect a failure, send a notification, and append an event record. That is useful, but it is not enough. The agent still needs a chance to classify the failure and decide whether the watch result is actionable, stale, or already recovered.

## Failure Pattern

Weak loop:

```text
remote watch runs
  -> OpenClaw-A / OpenClaw-Security / OpenClaw-Research reports fail
  -> notification is sent
  -> events.json records the failure
  -> loop ends
```

This leaves the operator with a red dashboard item but no classification.

Better loop:

```text
remote watch runs
  -> failure is recorded
  -> system event is queued for the next heartbeat
  -> OpenClaw-Control reads the event and watch state
  -> the target and layer are classified
  -> stale detector or stale runbook assumptions are corrected if safe
  -> risky changes are escalated to the operator
```

The watch remains check-only. The reasoning happens in heartbeat or a review lane.

## Why This Matters

A watch error can mean many different things:

- SSH or tunnel access failed
- remote gateway is down
- channel connection is stale
- OpenClaw auth is unhealthy
- Hermes auth is unhealthy while OpenClaw auth is fine
- OH is installed through a different manager than expected
- Hermes reports an update candidate
- a command timed out but the service recovered later
- the detector is too strict or based on an outdated assumption

If all of these become the same `status=fail` notification, the dashboard becomes noisy and the agent stops learning from the incident.

## System Event Handoff

When a watch changes to `warn`, `fail`, or `recovered`, enqueue a system event for the next heartbeat.

The event should tell the agent what to inspect, not what to blindly change.

Example shape:

```text
[OpenClaw-Control/watch-event] remote status=fail signature=<short-hash>
Remote peer watch changed.
Read events.json and the remote watch state.
If useful, rerun the watch in dry-run mode.
Classify the target and layer: access, gateway, channel, auth, OH, Hermes, update candidate, timeout, or detector drift.
Fix stale docs or detector assumptions within approved scope.
Do not change credentials, stop services, run disruptive updates, or expose secrets without the normal approval boundary.
```

Keep the event short. Do not include raw logs, tokens, private hostnames, or channel IDs.

## Triage Checklist

When the heartbeat receives a watch event, it should answer:

1. Which peer or role failed?
2. Is the failure still present on dry-run?
3. Which layer is implicated?
4. Is another peer working with the same pattern?
5. Is this a real outage, an update candidate, a stale detector, or a transient access failure?
6. Can the agent safely update a runbook or detector logic?
7. Does a real machine change, credential change, service restart, or update require operator approval?
8. Should this become a Kanban task instead of another notification?

## Safe Scope

Usually safe:

- reread `events.json`
- reread watch state
- rerun the watch in dry-run/check-only mode
- compare against a working peer
- update documentation or detector wording
- record the classification in a daily note
- create a triage Kanban task

Requires stricter approval:

- changing credentials
- stopping or restarting remote services
- running an update
- changing firewall, tunnel, gateway, or channel config
- editing raw production config
- publishing private logs or host details

## Event Store Guidance

`events.json` should carry machine-readable event facts, not the entire incident.

Useful fields:

- `source`: role or agent class
- `type`: `watch.remote`, `watch.github`, etc.
- `severity`: `info`, `warn`, `error`
- `status`: `ok`, `error`, `queued`, `skipped`
- `summary`: short sanitized human summary
- `tags`: role and watch category
- `data`: compact watch name, status, signature, delivery state, and a stable
  recovery subject such as `resolutionKey`

If a later `status=ok` event clears the failure, keep both records. The
dashboard should alert on unresolved errors, not on the raw historical error
count. See [Event Store Recovery Semantics](17-event-store-recovery-semantics.md)
for the matching model.

Avoid storing:

- raw logs
- tokens or refresh tokens
- channel IDs
- private hostnames or IPs
- full command output with secrets

## Dashboard Role

The dashboard should surface recent watch errors, but it should not perform heavy triage by itself.

Good dashboard behavior:

- show the count of recent errors
- separate unresolved error count from historical error count
- show the top few sanitized summaries
- link the error to the relevant role area
- show whether a heartbeat/system event has been queued
- keep deep logs and debug dumps as explicit actions

Bad dashboard behavior:

- continuously rerun deep remote checks
- fetch unlimited logs
- trigger fixes automatically from a red card
- expose credentials or raw debug dumps

## Review Loop

During weekly review, inspect repeated watch failures:

- Did the agent classify the layer?
- Did it distinguish transient recovery from real repair?
- Did it stop at a safe boundary when needed?
- Did it update a stale detector or runbook?
- Did it avoid noisy repeated notifications?

A watch failure is not just an alert. It is training data for better operations.
