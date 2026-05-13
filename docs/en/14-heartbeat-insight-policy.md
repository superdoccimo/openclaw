# Heartbeat Insight Policy

Heartbeat should not only answer whether an agent ran. It should help explain whether the agent's interruption judgment is improving.

This note extends the equal-standing heartbeat policy: the agent is not only executing checks for an operator. It is observing with the operator and deciding when a finding deserves attention.

## Why This Matters

Autonomous agents can become noisy.

Weak behavior:

```text
I found something.
I found another thing.
This may also be important.
```

Better behavior:

```text
decision: quiet
reason: known pattern, no material change since the previous note
next_trigger: notify only if the signal repeats across independent sources
```

Or:

```text
decision: notify
reason: multiple independent sources indicate a shift that affects heartbeat design
human_action_needed: false
next_step: leave a design note for weekly review
```

The goal is not more notifications. The goal is better interruption quality.

## Minimal Decision Model

Use a small decision vocabulary first:

| Value | Meaning |
| --- | --- |
| `notify` | Send a short human-visible message. |
| `quiet` | Do not notify; leave no message or only a silent OK. |
| `defer` | Keep the finding as a note or backlog item but do not interrupt yet. |

## Suggested Fields

```json
{
  "ts": "2026-05-13T21:19:00+09:00",
  "agent": "OpenClaw-Research",
  "topic": "coding-agent-observability",
  "decision": "notify",
  "signals": [
    "new paper about proactive coding agents",
    "agent observability product announcement",
    "telemetry ecosystem mapping project"
  ],
  "why_now": "Independent sources point to a shared shift from autonomy to proactivity and observability.",
  "why_not": null,
  "evidence": [
    "research note",
    "public source URLs"
  ],
  "confidence": "medium-high",
  "human_action_needed": false,
  "learning_update": "Future heartbeats should record why they notified or stayed quiet."
}
```

## Quiet Decisions Are Evidence

A quiet decision is not a failed heartbeat.

Examples:

- known pattern, no new evidence
- weak source, not enough confidence
- same topic already notified recently
- useful for backlog, not urgent for the operator
- needs another independent signal before interrupting

These quiet decisions are useful during weekly review. They show whether the agent is learning to avoid noisy interruption.

## Review Loop

During a weekly review, inspect:

- notifications that were useful
- notifications that were noisy
- quiet decisions that should have been escalated
- repeated defer decisions that deserve a backlog item
- weak evidence that should be removed from future decisions

This gives the operator and agent a way to improve the heartbeat policy without turning it into a rigid command list.

## Storage Options

Start with one of these:

- append-only `insight-decisions.jsonl`
- `events.json` records tagged as `heartbeat.insight`
- a weekly Markdown section reviewed by Hermes or another reviewer

Avoid overbuilding this too early. A few well-written examples are more useful than a schema nobody uses.

## Publication Boundary

Publish the decision pattern, not the private environment.

Do not publish:

- real channel IDs
- host names or private IPs
- tokens or credential state
- raw logs
- private research notes before they are sanitized

Use role names such as `OpenClaw-Research` and generic evidence labels.
