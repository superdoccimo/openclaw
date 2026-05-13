# Work-As-You-Go Knowledge Capture

Useful OpenClaw field knowledge is often discovered while fixing something else.
If it is not captured immediately, the reusable part is usually lost.

The goal is not to make every note polished on the first pass. The goal is to
leave enough sanitized structure that another operator can later understand the
failure mode, promote the pattern, or turn it into a safer runbook.

## Capture While Working

When an agent or operator is debugging, updating, or comparing agents, capture
small notes as soon as a reusable pattern appears.

Good candidates:

- a misleading status signal
- a layer boundary that was easy to confuse
- a check that should be read-only first
- a safe wrapper boundary
- a rollback or verification step
- a prompt or heartbeat instruction that changed behavior
- an auth, gateway, dashboard, Node, wrapper, or systemd drift pattern

Do not wait until the incident is fully solved. Rough notes are acceptable when
they clearly separate observation from assessment.

## Minimum Note Shape

A rough daily note should answer these questions:

- What was observed?
- What looked misleading?
- Which layer actually mattered?
- What evidence was useful?
- What should be checked next time?
- Is this a one-off note or a reusable pattern?

This structure is more important than perfect prose.

## Layer First, Tool Second

OpenClaw failures often look like one symptom:

```text
the agent did not reply
```

That symptom can come from many layers:

- channel permission
- gateway service state
- provider authentication
- systemd path or wrapper drift
- dashboard cache or self-load
- heartbeat delivery target
- model context or isolated heartbeat session behavior
- remote access or tunnel failure

Capture the layer before the command. Commands change faster than the diagnostic
shape.

## From Private Operations To Public Notes

Private operations may include real host names, private paths, domains, channel
IDs, tokens, raw logs, source IPs, and internal agent names. Public notes should
preserve the decision structure without those values.

Before moving a note into this repository:

- replace private agent names with public role names
- replace real hosts and addresses with role-based labels
- summarize logs instead of copying them
- describe private scripts as patterns, not source code
- keep token, cookie, credential, and private key values out completely
- describe the safety boundary, dry-run behavior, rollback expectation, and
  verification method

## Promotion Path

Use this path for fast-moving operations:

```text
daily note
  -> repeated pattern
  -> stable docs page
  -> example notification or state format
  -> README link if it becomes a major topic
```

Daily notes are allowed to be incomplete. Stable docs should be reusable.

## Agent Responsibilities

`OpenClaw-Research` is a good place to capture upstream and operational learning:

- watch official release notes and public repositories
- compare behavior across agents
- record reusable OpenClaw operation patterns
- turn rough notes into sanitized daily notes
- propose stable docs when a pattern repeats

It should not silently publish private operational details. Public updates should
be deliberate, sanitized, and reviewable.

## Useful Output

A useful note is often short:

```text
Observed: heartbeat completed quickly with an OK result, but no isolated
heartbeat session appeared.
Misleading signal: heartbeat liveness looked healthy.
Layer: heartbeat prompt and session configuration, not channel connectivity.
Evidence: status showed channel OK; session list lacked heartbeat session.
Next: compare with a working peer and add explicit heartbeat prompt,
lightContext, isolatedSession, and skipWhenBusy if appropriate.
Promote later: yes, if repeated across another agent.
```

The note does not need real host names, tokens, or raw command output to be
useful.

## Capture First Good Signals

When a previously passive agent starts producing useful autonomous output, record
the first good signal. This helps distinguish real improvement from simple
heartbeat liveness.

Useful signs:

- the agent found a public source without being prompted for that exact source
- the message included a clear target, observation, and proposed next step
- the agent wrote or updated a local research note
- the agent separated a promising idea from immediate implementation
- the message was short enough to be useful in a chat channel

Avoid recording private channel details. Preserve the behavior pattern instead.

Example:

```text
Observed: OpenClaw-Research produced its first useful autonomous discovery
notification.
Signal: it linked a public repository and paper, proposed a small tool idea,
and wrote the finding into research notes.
Layer: heartbeat prompt and knowledge-capture workflow, not channel delivery.
Next: keep the idea in backlog until repeated evidence shows it is worth a
prototype.
```
