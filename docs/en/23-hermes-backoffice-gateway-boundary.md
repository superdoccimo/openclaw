# Hermes Back-Office And Gateway Boundary

Hermes is most useful in an OpenClaw deployment when it does back-office work:
review, memory, kanban, scheduled summaries, proposals, and operator-facing
evidence preparation.

OpenClaw should remain the primary chat and notification gateway unless there is
a deliberate reason to split the gateway layer.

## Problem

OpenClaw and Hermes may both support messaging integrations. That can create a
misleading design temptation:

```text
same chat account
same bot token
same channel
two independent pollers or gateways
```

This is fragile. Two tools can compete for updates, acknowledgements, rate
limits, session state, or operator expectations. When a message disappears or a
reply is delayed, the operator no longer knows which layer owned the event.

The safer default is not "enable every gateway everywhere." The safer default is
to assign one tool to the chat gateway and one tool to the review loop.

## Recommended Split

Use this boundary first:

| Layer | Primary role |
| --- | --- |
| OpenClaw gateway | chat intake, notifications, visible operator replies |
| Hermes gateway or scheduler | back-office cron, reviews, memory, kanban, proposal generation |
| Dashboard | evidence display across both layers |
| Operator | approval, risky execution, external publication decisions |

In this model, Hermes can still be active and valuable without owning the chat
bot. It can produce local review artifacts, kanban items, weekly summaries, and
handoff drafts that OpenClaw or the operator can read.

## Avoid Shared Messaging Ownership

Do not let OpenClaw and Hermes independently use the same messaging account or
bot token unless the design has been reviewed and the ownership boundary is
explicit.

If Hermes needs direct messaging later, prefer one of these safer shapes:

- separate bot identity
- separate channel
- explicit read-only intake
- explicit outbound-only notification path
- one gateway disabled while the other gateway owns polling

The important rule is that exactly one component should be responsible for
polling, acknowledgement, and operator-visible response semantics for a given
chat identity.

## Safe First Hermes Bridge

A safe first bridge is a local scheduled review that writes artifacts, not
messages:

```text
Hermes cron
  -> no-agent local review command
  -> local Markdown snapshot
  -> dashboard evidence
  -> OpenClaw or Operator decides whether to notify
```

This pattern is useful because it proves that Hermes is doing work without
increasing the external communication surface.

Good first outputs:

- latest unresolved duty desk tickets
- recent heartbeat findings
- research note summary
- security review summary
- proposal candidates
- stale item list
- weekly review material

The first bridge should not restart services, update packages, push code, merge
branches, rotate credentials, expose sockets, or send trading, billing, or
contract actions.

## Sandbox Status Can Mislead

Some CLI status commands depend on host-level process state, systemd, lock
files, or local databases. When they run inside a restricted coding-agent
sandbox, they can report a false negative even though the host service is
healthy.

Treat these checks as suspect when they conflict:

```text
CLI inside sandbox says gateway is not running
dashboard API shows scheduler running
host-level systemd shows service active
gateway list from the host shows a current process
recent review artifacts are being written
```

Do not convert a sandbox-only false negative into a service restart. Verify with
read-only host-level evidence first.

Useful evidence sources:

- scheduler status
- gateway list
- user service status
- recent journal lines
- dashboard API
- latest artifact timestamp
- active job count

The verification should answer two different questions:

```text
Is the gateway or scheduler process alive?
Is it doing useful work for this agent role?
```

A running gateway with zero active jobs is not the same problem as a stopped
gateway.

## Dashboard Implications

The dashboard should show Hermes as a back-office worker, not only as a chat
gateway.

Useful fields:

- scheduler running or stopped
- active cron job count
- latest review artifact path
- latest review artifact timestamp
- kanban item count
- last successful local review
- messaging platforms enabled or intentionally disabled

If messaging is disabled but local reviews are current, the dashboard should not
show this as a chat outage. It should show Hermes as back-office active and
messaging disabled by design.

## Escalation Policy

Escalate to the operator before changing the gateway boundary when any of these
are involved:

- shared bot identity
- shared token or credential file
- new polling process
- external WebSocket or public callback
- DNS, firewall, tunnel, or reverse proxy change
- package update or service restart
- push, merge, release, or public publication

Hermes can prepare proposals for these changes. It should not silently perform
them as part of a review loop.

## Verification Checklist

Before treating the integration as healthy, verify:

- OpenClaw owns the active chat gateway for the relevant channel.
- Hermes messaging integrations are either deliberately disabled or use separate
  identities.
- Hermes has at least one useful back-office job or review path.
- The latest Hermes artifact is visible in the dashboard.
- Sandbox CLI output is not the only evidence for service state.
- No secrets, channel IDs, hostnames, or raw production logs are copied into
  public docs.

## Practical Rule

Do not compete for the same chat surface.

Let OpenClaw own operator-visible messaging, let Hermes own review and memory
work first, and connect them through local artifacts, dashboards, and explicit
handoff proposals. Add direct Hermes messaging only after the ownership boundary
is clear.
