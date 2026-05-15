# Multi-Agent Architecture

Multi-agent OpenClaw operations need explicit responsibility boundaries. Without boundaries, domain decisions, security monitoring, daily operations, research, and update work blend together until no agent has a clear job.

## Public Role Names

| Agent | Responsibility |
| --- | --- |
| `OpenClaw-Control` | control tower, daily operations, dashboard, peer monitoring |
| `OpenClaw-A` | domain-specific work and update assistance |
| `OpenClaw-Security` | anomaly detection, defensive decisions, security findings |
| `OpenClaw-Research` | OSS research, upstream change tracking, discovery summaries |
| `OpenClaw-Consult` | cross-agent technical consultation, evidence sorting, browser-state support, safe handoff shaping |

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

OpenClaw-Consult
  receives technical ambiguity from peers
  sorts evidence, risk, and next action
  supports browser-state and GUI evidence
  shapes safe handoffs to Operator or coding agents
```

## Separation Rules

- `OpenClaw-Control` observes global state, but does not own every domain decision.
- `OpenClaw-A` owns domain-specific work, but should not become the global monitor.
- `OpenClaw-Security` focuses on security and should not take over market, research, or daily-life workflows.
- `OpenClaw-Research` should primarily investigate and summarize; production changes should remain proposal-only unless explicitly promoted.
- `OpenClaw-Consult` helps peers resolve technical ambiguity, but should not become a superior agent or approval gate for every action.

## Why Multiple Agents Help

With one agent, it is hard to know whether a failure is host-specific or systemic.

Multiple agents create comparison points:

- OAuth works on one agent but not another.
- Control reports Security as down, while Security's local status says healthy.
- The dashboard is green, but the journal contains timeouts.
- The updater succeeded, but the gateway still holds stale state.

Comparison is an operational signal.

## Consultation Layer

Once agents become more autonomous, the operator can become the default router
for every cross-role technical question. That does not scale.

A consultation role gives peer agents a place to send problems that do not
belong cleanly to one domain:

- gateway, auth, channel, session, or provider uncertainty
- Node / nvm / systemd / wrapper drift
- dashboard or browser-state evidence
- safe handoff to a coding agent
- confusing output from another agent that needs triage

The consultation role should not override domain expertise. It should organize
facts, evidence, risk, and next action so the originating agent, the operator,
or a coding agent can move without repeating context gathering.

See [16-cross-agent-consultation.md](16-cross-agent-consultation.md).

## Update Path Risk

Peer updates are useful, but the updater's own auth state and wrappers become part of the production path.

```text
OpenClaw-A updates OpenClaw-Control
OpenClaw-Control updates OpenClaw-A / OpenClaw-Security / OpenClaw-Research
```

If only one side fixes auth order or Node wrapper alignment, the other side can reintroduce stale state during the next update.
