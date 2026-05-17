# Heartbeat And Safety

Heartbeat should not only answer "did the check run?"

`exit 0` means the check completed. It does not prove the environment is safe or that the real production response path is working.

## Operator Policy: Equal-Standing Heartbeats

This is a field policy, not an OpenClaw product requirement.

In this operating style, heartbeat is not treated as a command queue where a subordinate bot executes orders. It is treated as a short shared observation window between the operator and the agent.

That changes how the heartbeat prompt is written. Prefer sharing purpose, boundaries, evidence sources, and desired durable artifacts over imperative task lists.

For example:

```text
The heartbeat is a short shared observation window.
Use the state file and recent notes as evidence.
If something useful or strange appears, leave a short research note.
If there is nothing useful to add, a quiet OK is enough.
```

Avoid turning the prompt into:

```text
Do A. Then do B. Then report C.
```

This matters because autonomy is partly shaped by the frame. A prompt that leaves room for judgment tends to produce better observations, especially for research agents. A prompt that only lists commands can collapse the agent into a mechanical checker.

## Acknowledgement Is Not Always Text

In chat-connected operations, a reaction can be a valid acknowledgement.

This rule is intentionally narrow. For an informational all-hands message, a
thumbs-up reaction may be the best response: it confirms receipt without
creating noise. Forcing a text reply such as "you must comment when mentioned"
can violate the equal-standing policy. It turns a shared observation window into
a behavioral constraint and can suppress the agent's natural judgment about when
speech is useful.

Prefer wording like:

```text
For general announcements, a reaction is enough.
If a short text reply feels useful, use one.
Do not force a comment when acknowledgement is already clear.
```

Avoid wording like:

```text
Always reply in text when mentioned.
Do not only react.
```

This distinction matters during incident review. "No text reply" does not
necessarily mean "no response." Check whether the agent reacted, recorded the
message, opened a session, or left another durable signal before treating the
turn as a failure.

Equal standing does not mean silence, and it does not remove the need to share
useful context. Prefer a short text reply or durable note when:

- the operator asks a direct question or asks for confirmation
- the message is an incident, failed update, auth problem, or unresolved risk
- another agent needs a handoff or consultation
- the response changes what a human should do next
- a reaction would be ambiguous or easy to miss

In those cases, avoid command tone, but still communicate the facts. The useful
boundary is "do not force speech for low-value acknowledgements," not "avoid
speech whenever a reaction is possible."

## Separate Execution From Assessment

Weak signal:

```text
heartbeat finished
status: ok
```

Better signal:

```text
observed: ok
assessment: SECURITY_WARNING
reason: gateway reachable, but auth profile has expired fallback
next_action: repair auth order and restart gateway
```

## Suggested State Model

| Field | Meaning |
| --- | --- |
| `observed` | whether the check produced usable observations |
| `assessment` | safety or operational judgment |
| `evidence` | what the judgment is based on |
| `next_action` | what a human or agent should do next |
| `last_checked_at` | last observation time |

`observed: ok` and `assessment: SECURITY_OK` are different claims.

## Recommended Assessment Values

- `SECURITY_OK`
- `SECURITY_WARNING`
- `SECURITY_UNKNOWN`
- `CHECK_FAILED`
- `STALE`

## What Heartbeat Should Check

- whether OpenClaw CLI is launched from the expected path
- whether the gateway service is active
- whether auth order prioritizes the production profile
- whether channel send/receive paths work
- whether heartbeat timeout budget matches the work it is expected to do
- whether dashboard APIs are too heavy or stale
- whether tunnel / reverse proxy / private mesh access is reachable
- whether recent journal entries show timeouts or credential errors

## Timeout Budget Is A Separate Layer

A timeout message does not automatically mean the channel, OAuth profile, or
model is broken.

Example symptom:

```text
Request timed out before a response was generated. Please try again, or
increase agents.defaults.timeoutSeconds in your config.
```

Before raising the global timeout, check which runtime produced the timeout.

Useful evidence:

```bash
openclaw models auth list
openclaw models auth order get --provider openai-codex
openclaw config get agents.defaults.heartbeat.timeoutSeconds
systemctl --user status openclaw-gateway.service --no-pager
journalctl --user -u openclaw-gateway.service --since "30 minutes ago" --no-pager
```

If the gateway log shows an embedded heartbeat run ending at a timeout such as
`timeoutMs=240000`, compare that with
`agents.defaults.heartbeat.timeoutSeconds`. A 240 second heartbeat budget can be
too small once the heartbeat is expected to read events, reason about peer
updates, inspect logs, and write a useful note.

Prefer the narrowest fix:

```bash
openclaw config get agents.defaults.heartbeat.timeoutSeconds
openclaw config set agents.defaults.heartbeat.timeoutSeconds 420 --strict-json --dry-run
openclaw config set agents.defaults.heartbeat.timeoutSeconds 420 --strict-json
openclaw config validate
systemctl --user restart openclaw-gateway.service
```

Do not increase `agents.defaults.timeoutSeconds` first unless normal interactive
chat turns are also timing out. Raising the global timeout can hide stuck runs,
make channel feedback slower, and blur the difference between heartbeat work and
human conversation.

After the change, verify:

- gateway reaches `ready`
- event loop health returns to normal
- the next heartbeat either completes or fails with a more specific cause
- no token, refresh token, channel ID, host name, or raw production log was
  copied into public notes

## What Heartbeat Should Avoid

- running deep status checks at short intervals
- printing token or credential values
- auto-applying fixes without evidence
- turning the dashboard green based only on `exit 0`

## Notification Format

Keep notifications short enough for a human to act on.

```text
[OpenClaw-Control/auth] auth check
- OpenClaw-Control: OpenClaw openai-codex production profile ok
- OpenClaw-A: OpenClaw openai-codex OAuth expired
- OpenClaw-Research: Hermes openai-codex OAuth exhausted
- OpenClaw-Security: ok
assessment: SECURITY_WARNING
next: repair auth order, then restart gateway
```
