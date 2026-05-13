# Autonomy Enablement

When an OpenClaw agent does not act autonomously, the problem is not always the
model or the product capability. In field operations, "the agent is not
autonomous" often means the operating loop is underspecified.

Autonomy usually needs several pieces to line up:

- a heartbeat that gives the agent permission to think and act
- a workspace with durable notes and role boundaries
- a lightweight context mode that avoids dragging old chat state into every run
- an isolated heartbeat session when the main session is large or stale
- a clear rule for when to stay silent and when to notify
- access to public sources when research is part of the role
- a backlog or daily note where weak ideas can be parked without pretending they
  are ready
- comparison with working peer agents

If those pieces are missing, the agent may still be alive. It may pass status
checks, keep the gateway running, and send heartbeat acknowledgements. That does
not mean it has enough structure to produce useful autonomous work.

## Misleading Symptom

```text
heartbeat is running, but the agent does not do anything useful
```

This can be misread as:

- the model is not capable
- the channel is broken
- the product does not support autonomy
- the user must write a more exact command every time

In practice, it may be an operating design issue.

## Useful Comparison

Compare the passive agent with a working peer:

- Does the working peer have a dedicated heartbeat session?
- Does its heartbeat prompt tell it what kind of progress is useful?
- Does it have `lightContext` or equivalent lightweight context behavior?
- Does it skip when another agent run is busy?
- Does it have durable files for memory, backlog, research notes, or daily logs?
- Does it know which sources are allowed?
- Does it know what not to notify about?

The comparison often reveals that the passive agent is not missing intelligence.
It is missing an operating loop.

## First Useful Signal

The first good sign is not a loud status report. It is a small, grounded action:

```text
the agent finds public sources, extracts a reusable idea, writes a research
note, updates a backlog, and sends a short notification only because the finding
is useful
```

That is materially different from:

```text
HEARTBEAT_OK
```

Both can be healthy in different situations. The difference is whether the
heartbeat had enough structure to decide and advance a standing goal.

## Durable Notes Are The Output

A chat notification is not the durable artifact. It is only an interrupt or a
pointer.

For autonomous operations, the useful output should land in files that survive
the current conversation:

- research notes for what was observed
- backlog entries for ideas that are not ready yet
- state files for dedupe and last-seen timestamps
- daily notes for operational lessons
- stable docs only after a pattern becomes reusable

If a finding appears only in chat, it is easy to lose. If it is written to a
research note, backlog, and state file, later agents, reviews, and operators can
build on it.

This is especially important for research-oriented agents. Their value is often
not immediate action. It is accumulating small, well-grounded observations until
a pattern becomes strong enough to prototype or document.

## Field Pattern

A research-oriented agent became useful after the following changes were aligned:

- explicit heartbeat prompt for observation, learning, and note taking
- lightweight heartbeat context
- isolated heartbeat session
- busy-run skip behavior
- durable research and backlog files
- field notes that describe what kind of knowledge is worth preserving
- public-source boundaries and redaction rules

After that, the agent produced a concise discovery notification with a target,
observation, proposed backlog idea, note path, and public references.

This is a field-discovered operating pattern. Official documentation may cover
individual settings, but the reusable lesson is the combination: autonomy is
enabled by a loop, not by a single switch.

## Practical Rule

When an agent feels passive, do not start by adding more tasks.

First check whether it has:

- permission to act during heartbeat
- a small enough context to think clearly
- an isolated place for heartbeat work
- a durable place to write discoveries
- a clear notification threshold
- a comparison target that already works

Then run a small smoke test and look for a real note or backlog update, not just
an OK token.
