# Multi-Agent Architecture

Multi-agent OpenClaw operations need explicit responsibility boundaries. Without boundaries, domain decisions, security monitoring, daily operations, research, and update work blend together until no agent has a clear job.

## Public Role Names

| Agent | Responsibility |
| --- | --- |
| `OpenClaw-Control` | control tower, daily operations, dashboard, peer monitoring |
| `OpenClaw-A` | domain-specific work and update assistance |
| `OpenClaw-Security` | anomaly detection, defensive decisions, security findings |
| `OpenClaw-Research` | OSS research, upstream change tracking, discovery summaries |

## Example Relationship

```text
OpenClaw-Control
  monitors OpenClaw-A
  monitors OpenClaw-Security
  monitors OpenClaw-Research
  owns dashboard and global status

OpenClaw-A
  assists updates for OpenClaw-Control
  owns domain-specific workflows

OpenClaw-Security
  monitors exposed surfaces
  reports SECURITY_WARNING / SECURITY_OK
  proposes or applies only bounded remediation

OpenClaw-Research
  tracks upstream changes
  summarizes new operational findings
```

## Separation Rules

- `OpenClaw-Control` observes global state, but does not own every domain decision.
- `OpenClaw-A` owns domain-specific work, but should not become the global monitor.
- `OpenClaw-Security` focuses on security and should not take over market, research, or daily-life workflows.
- `OpenClaw-Research` should primarily investigate and summarize; production changes should remain proposal-only unless explicitly promoted.

## Why Multiple Agents Help

With one agent, it is hard to know whether a failure is host-specific or systemic.

Multiple agents create comparison points:

- OAuth works on one agent but not another.
- Control reports Security as down, while Security's local status says healthy.
- The dashboard is green, but the journal contains timeouts.
- The updater succeeded, but the gateway still holds stale state.

Comparison is an operational signal.

## Update Path Risk

Peer updates are useful, but the updater's own auth state and wrappers become part of the production path.

```text
OpenClaw-A updates OpenClaw-Control
OpenClaw-Control updates OpenClaw-A / OpenClaw-Security / OpenClaw-Research
```

If only one side fixes auth order or Node wrapper alignment, the other side can reintroduce stale state during the next update.
