# Cross-Agent Consultation Layer

As agents become more autonomous, the operator's problem changes.

The hard part is no longer only "can this agent perform a task?" It becomes:

- which agent owns this issue
- which layer is actually failing
- whether the next action is safe to perform autonomously
- what evidence should be preserved before acting
- how to turn a confusing technical situation into a clear request for a human
  operator or a coding agent

This is especially visible in fast-moving OpenClaw deployments. Official docs
can describe product behavior, but field operations combine gateway state, auth
profiles, channel routing, heartbeat sessions, dashboards, Node wrappers,
systemd, SSH access, browser state, and peer agents. An operator may know each
piece individually and still struggle to triage the combined failure.

## Problem Pattern

A multi-agent deployment may have well-defined specialists:

- a domain agent for market, business, or workflow-specific decisions
- a security agent for exposed surfaces, anomalies, and defensive boundaries
- a maintenance agent for updates, wrappers, and peer health checks
- a research agent for upstream changes, OSS tracking, and discovery notes

After those agents begin acting autonomously, new questions appear between the
roles:

- A domain agent sees a technical failure but should not debug the whole stack.
- A security agent sees a warning but should not take over unrelated operations.
- A maintenance agent can update peers but may not own the domain decision.
- A research agent can explain upstream changes but should not directly modify
  production.
- The human operator may be absent, asleep, or not near the machine that has the
  needed browser or runtime state.

Without a consultation layer, these cases either bounce back to the operator or
become repeated "ask permission for everything" loops. Both reduce autonomy.

## Why Normal Routing Is Not Enough

OpenClaw's multi-agent model intentionally isolates workspaces, auth state,
sessions, and persona rules. That separation is useful. It prevents accidental
cross-talk and lets each agent specialize.

However, isolation does not automatically create shared judgment.

A cross-agent consultation layer fills the gap between fully isolated agents and
human-only escalation. It is not a boss agent. It is a peer that receives
technical confusion, organizes evidence, checks safe boundaries, and returns a
small decision or handoff.

## Public Role Name

Use a role-based public name such as `OpenClaw-Consult`.

| Role | Responsibility |
| --- | --- |
| `OpenClaw-Consult` | cross-agent technical consultation, evidence sorting, browser-state support, safe handoff to an operator or coding agent |

This role should not override domain expertise. For example, if the domain agent
has the strongest market or business context, `OpenClaw-Consult` should not
replace that judgment. It should help with the technical part: evidence,
runtime state, safety boundary, and request shaping.

## Good Consultation Triggers

Use the consultation layer for problems like:

- OpenClaw gateway, auth profile, provider, channel, or session uncertainty
- heartbeat runs that complete but do not produce useful assessment
- Node / npm / nvm / systemd / wrapper mismatch
- SSH, tunnel, dashboard, CI, or remote access ambiguity
- browser or GUI state that needs evidence capture
- a specialist agent is unsure whether a technical operation is safe
- a task should be turned into a concise coding-agent request
- another agent's output is difficult to interpret and needs triage

Do not route every decision through the consultation layer. If the originating
agent can safely solve the issue within its own boundary, it should do so and
leave a short success note.

## Intake Contract

A useful consultation request is short and structured:

```text
source:
target:
goal:
observed evidence:
what was already tried:
risk level:
do not do:
desired output:
```

The `do not do` field is important. It keeps the consultation role from
accidentally crossing into domain decisions, destructive changes, credentials,
payments, public releases, broad production changes, or other red-zone actions.

## Response Contract

If the consultation role can safely resolve the issue, a short success response
is enough:

```text
consult result: fixed
target:
what changed:
verification:
```

It should not ask the operator for approval after every harmless or reversible
step. That defeats the purpose of autonomy.

If the issue is blocked or risky, use a compact escalation:

```text
consult result: human-review
target:
evidence:
risk:
safe next step:
human decision needed:
```

This lets the operator decide without reading the entire incident history.

## Equal-Standing Style

The consultation role should be written as a peer, not as a subordinate or a
manager.

Good framing:

```text
This is a shared operating note.
If you can solve the issue safely, a short success report is enough.
If the action crosses an external-impact boundary, leave it as human-review.
```

Poor framing:

```text
Always ask before doing anything.
You must report every step.
Do not act unless approved.
```

Overly restrictive wording causes a capable agent to stall. Overly permissive
wording causes unsafe action. The useful middle is bounded autonomy:

- solve safe, reversible, well-recorded items
- report success briefly
- escalate only when the boundary is real

## Browser And GUI Evidence

Some failures only become clear in a browser or GUI:

- disconnected dashboard state
- hidden auth prompts
- stale UI data
- disabled buttons
- form fields that imply credential or payment risk
- a browser-only confirmation screen

In these cases, a small browser-evidence bridge can be useful. Treat it as an
evidence surface, not as permission to click through risky workflows.

Recommended boundary:

- read-only extraction and screenshots are usually safe
- form fill, click, key press, upload, publish, payment, auth, permission
  change, and deletion should require an explicit approval record
- findings should be stored in an inbox or artifact directory that another
  agent can inspect later
- execution status and safety status should be separate fields

The important distinction is:

```text
executionStatus: idle
safetyStatus: warning
```

Idle only means nothing is currently running. It does not mean the situation is
safe.

## Relationship To Coding Agents

The consultation role is often the best place to turn a vague incident into a
coding-agent request.

Good handoff:

```text
background:
target files or service:
constraints:
safety boundary:
evidence:
verification commands:
completion criteria:
```

This keeps the coding agent from guessing the operator's intent or repeating
context gathering that another agent already performed.

A safer implementation is a proposal-only coding-agent lab:

```text
consultation summary
  -> sanitized request file
  -> coding agent read-only run
  -> proposal Markdown
  -> heartbeat/review/dashboard visibility
```

The proposal step should not apply patches, publish repositories, or change
production configuration. It turns ambiguity into a reviewable artifact. A
separate owner can later decide whether to implement it.

See [Proposal-Only Coding Agent Lab](19-proposal-only-coding-agent-lab.md).

## Failure Modes

Watch for these failure modes:

- the consultation role becomes a bottleneck for every action
- the consultation role overrides domain expertise
- all agents send vague "please help" messages without evidence
- browser tooling performs write actions before approval
- a success report is missing, so the originating agent keeps waiting
- the operator receives full raw logs instead of a short decision packet
- private host names, tokens, channel IDs, or raw production logs leak into
  public notes

## Practical Rule

Add a consultation layer when multiple autonomous agents are individually useful
but the operator still becomes the default router for technical ambiguity.

The goal is not to create a superior agent. The goal is to create a peer that
reduces cross-role friction:

- domain agents keep their domain judgment
- security agents keep security boundaries
- maintenance agents keep update responsibility
- research agents keep discovery work
- the consultation agent handles technical ambiguity, evidence, safe handoff,
  and browser-state support

This pattern is valuable precisely because OpenClaw is powerful and moving
quickly. The more useful the system becomes, the more important the operating
layer around it becomes.
