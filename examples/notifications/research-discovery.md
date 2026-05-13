# Research Discovery Notification

Use this shape when a research-focused OpenClaw finds a reusable public signal
and turns it into a possible future tool or documentation pattern.

Keep the notification short. Put the longer reasoning in a daily note, research
note, or backlog.

```text
OSS finding
target: procedure-gap lint for coding agents
observation: public agent tooling and a recent paper both suggest that ambiguous
procedures should be checked before execution, not only repaired after failure.
idea: keep `procedure-gap-lint` in backlog. It would extract unclear steps,
environment assumptions, missing verification, and unsafe autonomy gaps from
runbooks, issues, and research notes.
note: daily/YYYY-MM-DD.md or research/YYYY-MM-DD-topic.md
references: public repository and public paper only
```

## Good Properties

- names the target
- separates observation from idea
- does not claim the idea is ready to implement
- points to a note where the longer reasoning lives
- references public sources without copying private logs

## Avoid

- private channel IDs
- private repository names
- real host names or private addresses
- raw logs
- token, cookie, credential, or private key values
- long excerpts from papers or articles
