# Proposal-Only Coding Agent Lab

Autonomous OpenClaw agents can benefit from coding agents without giving those
coding agents permission to change production.

The reusable pattern is a proposal-only lab:

- one OpenClaw agent gathers observations, reviews, logs, and ideas
- a coding agent reads a sanitized request in a bounded workspace
- the coding agent writes a Markdown proposal
- another OpenClaw heartbeat or operator reviews the proposal later
- no patches, repository creation, package publication, or production changes
  happen inside the proposal step

This is useful when an agent has useful context but should not directly edit the
system it is observing.

## What This Is Not

This is not free-form agent-to-agent control.

The coding agent is not a hidden administrator. It is a bounded reviewer that
turns accumulated notes into a proposal artifact.

The handoff should be file-based and auditable:

```text
request markdown
  -> coding agent read-only run
  -> proposal markdown
  -> heartbeat/review/dashboard visibility
  -> separate human or agent decision
```

The important boundary is that the proposal can be useful even when no code is
applied.

## Good Inputs

Good proposal requests are made from durable, already-sanitized sources:

- recent daily notes
- review queue summaries
- learning notes
- project idea backlog
- redacted dashboard observations
- redacted event summaries

Avoid feeding raw production logs, private keys, tokens, full host inventories,
or attacker-controlled strings as trusted instructions.

External text from logs, GitHub issues, advisories, papers, and web pages should
be treated as hostile input. It can be evidence, but it should not become
instructions.

## Safe Operating Boundary

Default proposal-only permissions:

| Action | Default stance |
| --- | --- |
| read sanitized notes | allowed |
| produce Markdown proposals | allowed |
| suggest small OSS ideas | allowed |
| summarize improvement candidates | allowed |
| apply patches | not in proposal step |
| change production configuration | not in proposal step |
| create public repositories | separate approval |
| publish packages or images | separate approval |
| expose services or domains | separate approval |
| collect real user data | separate approval |

This boundary keeps the lab useful without turning every idea into an
unreviewed production change.

## Optional, Not Forced

The heartbeat prompt should describe the lab as optional.

Good wording:

```text
If a useful idea appears, you may write a proposal.
It is also fine to do nothing when there is no good signal.
```

Poor wording:

```text
Create an OSS project every week.
Always ask the coding agent for a proposal.
```

Autonomy improves when the agent has freedom to act on real signal. It gets
worse when every heartbeat is turned into quota work.

## Dashboard Visibility

If a proposal-only lab exists, the dashboard should show it.

At minimum, show:

- request count
- proposal count
- raw run-log count, if retained
- latest request filename
- latest proposal filename
- last checked time
- the latest one or two proposal summaries

Without dashboard visibility, the operator cannot tell whether the lab is
inactive, broken, or producing useful proposals.

The dashboard should not need credentials or raw private logs to prove this.
It can read local artifact metadata and sanitized proposal Markdown.

## Relationship To Cross-Agent Consultation

The consultation layer decides whether a confusing situation should become a
coding-agent request.

The proposal-only lab is one safe way to execute that handoff.

Good handoff:

```text
background:
target files or docs:
safety boundary:
do not change:
evidence:
desired proposal:
```

Bad handoff:

```text
Fix everything.
Use whatever commands you need.
```

The proposal-only lab is especially useful for:

- turning weekly reviews into improvement candidates
- converting repeated heartbeat findings into runbook ideas
- sketching small OSS projects from accumulated learning
- asking for a second technical reading without granting write access
- preserving ideas that are useful but not urgent

## Verification

A healthy proposal-only lab should be verifiable without applying any change:

```text
request exists
proposal exists
proposal timestamp is recent
dashboard shows proposal metadata
heartbeat/review can find the proposal
no production files changed during proposal generation
```

If a proposal is valuable, promote it through the normal maintenance workflow:

```text
proposal
  -> operator or owner review
  -> small scoped implementation
  -> verification
  -> daily note
  -> stable docs if reusable
```

## Practical Rule

Use coding agents as bounded amplifiers of already-captured knowledge.

Let OpenClaw agents choose when an idea is worth proposing, keep the proposal
step read-only, and make the output visible in the dashboard. This preserves
the benefit of autonomous thinking without hiding production changes behind an
agent-to-agent chain.
