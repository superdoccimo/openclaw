# Duty Desk Ticket Bridge

Cross-agent consultation needs a durable place to land before any worker is
asked to act.

A useful OpenClaw pattern is a duty desk ticket bridge: one agent receives
unresolved consultations, turns them into small structured tickets, and routes
the next step as a proposal, review, or human hold. The bridge should be boring,
local, and auditable. It is not a new authority layer.

## Problem Pattern

In multi-agent operations, a confusing issue often moves through chat like this:

```text
agent observes a problem
  -> another agent gives partial advice
  -> a coding agent is asked to help
  -> a reviewer notices safety concerns
  -> the original context is scattered across messages
```

This can work once, but it does not scale. The operator cannot easily tell:

- what was observed
- which facts were confirmed
- which parts were unknown
- which agent owned the next action
- whether the issue required human approval
- whether the result was sent to a worker, a tracker, or a reviewer

The misleading interpretation is that the agents need more autonomy. Often they
need a better intake surface.

## Pattern

Use a duty desk agent as the unresolved-case intake:

```text
OpenClaw-Control / OpenClaw-A / OpenClaw-Security / OpenClaw-Research
  -> unresolved consultation
OpenClaw-Consult
  -> local ticket bridge
ticket store
  -> risk label, status, facts, unknowns, next action
handoff drafts
  -> coding-agent request, review ticket, public issue draft, or human hold
```

The bridge should accept a short title, source role, category, summary, and
optional evidence notes. It should return a short machine-readable result:

```text
ticket=<id>
risk=<low|medium|high|critical>
status=<open|proposal-ready|human-review>
```

That is enough for a heartbeat, dashboard, or operator to know that the issue is
captured.

## Keep Heartbeat Small

The heartbeat prompt should not become the ticketing runbook.

Good heartbeat shape:

```text
If unresolved consultations or approval-waiting work appears, create a local
duty desk ticket. Do not execute external exposure, credential, payment,
service restart, push, merge, or production changes without human review.
```

Put the actual commands, examples, and rollout steps in a runbook outside the
heartbeat bootstrap file.

This keeps the agent's recurring context small and reduces the chance that
important operating boundaries are truncated or diluted by long examples.

## Risk Routing

The first bridge does not need deep static analysis. It only needs conservative
classification.

Route to `human-review` when the consultation mentions:

- tokens, cookies, secrets, credentials, private keys, or auth files
- DNS, firewall, tunnel, Cloudflare, public exposure, or WebSocket exposure
- service restart, package install or update, deploy, push, merge, or release
- payment, billing, trading, subscription, contract, or order execution
- unclear ownership between agents

Low-risk and medium-risk issues can become proposals or handoff drafts. High
and critical issues should be visible but not executed.

## Handoff Drafts

The ticket bridge should create or feed handoff drafts, not skip review.

Useful outputs:

- coding-agent request draft
- issue tracker draft
- review ticket draft
- security reviewer summary
- operator decision log

A coding-agent request should include:

```text
background
target files or docs
confirmed facts
unknowns
safety boundary
forbidden actions
verification expectation
desired output
```

This makes the coding agent a bounded worker, not a hidden administrator.

## Relationship To Other Patterns

The duty desk ticket bridge sits before the proposal-only coding-agent lab.

```text
consultation
  -> ticket bridge
  -> proposal-only coding-agent request
  -> review
  -> implementation, if approved
```

It also fits the NotebookLM source-pack handoff. A ticket can record that a
source pack should be prepared, uploaded, or reviewed without asking an agent to
fight a brittle browser UI.

## Verification

A healthy rollout can be verified without changing production:

```text
harmless consultation creates ticket
risky consultation becomes human-review
ticket appears in local dashboard or list command
handoff draft can be generated from the ticket
heartbeat can point to the runbook without copying it
no service, gateway, public exposure, or repository setting changed
```

If the bridge hangs, writes secrets, creates noisy notifications, or bypasses
human review, fix the bridge before connecting more agents.

## Practical Rule

Do not ask every agent to become a project manager.

Give one consultation role a small local ticket bridge, keep heartbeat short,
classify risky work conservatively, and make every worker handoff visible. The
goal is not to automate every decision; it is to keep consultation, hold,
approval, and record-keeping from disappearing into chat.
