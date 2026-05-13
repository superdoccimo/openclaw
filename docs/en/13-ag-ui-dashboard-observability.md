# AG-UI Dashboard Observability

AG-UI is useful in OpenClaw operations as an operator-facing observability layer, not as proof that the agent is autonomous.

The important question is not whether a dashboard renders. The important question is whether it shows the right evidence for that agent's role.

## Role-Specific Dashboard

Do not blindly copy one agent's dashboard to another agent.

A dashboard built for a security agent will usually emphasize incidents, auth, and remediation boundaries. A dashboard built for a research agent should emphasize research notes, idea backlog movement, durable memory, Discord visibility, and GitHub publication readiness.

At minimum, make these inputs role-specific:

- agent label, dashboard title, and short role tag
- agent data directory
- heartbeat state path
- event store path
- daily or research note directory
- review file naming convention
- note label shown in the UI
- command paths used by the service environment
- action authentication policy

Inherited defaults are a common source of false negatives. For example, a dashboard can say there are no daily notes simply because it is still reading a security agent's `daily-log` directory while the research agent writes to `research/`.

The same problem can happen with review files. If the CLI or scheduler writes `weekly-review-YYYY-MM-DD.md` but the dashboard only searches for `review-YYYY-MM-DD.md`, the review panel will look empty even though the review mechanism is working.

## Useful Evidence

The operator should be able to answer these questions without SSHing into the host:

- Is the OpenClaw gateway reachable?
- Which channels are configured and running?
- Did the agent produce a visible Discord reply?
- Is there a non-ACK assistant response in the OpenClaw session log?
- What heartbeat tasks ran recently?
- Which durable notes were written most recently?
- Are recent events empty, queued, skipped, ok, or error?
- Is action authentication enabled before exposing the UI beyond loopback?

For a research agent, the most valuable dashboard row is often the durable note row. Chat notifications are pointers. Research notes, backlog entries, state files, and public docs are the reusable output.

## Service Environment

When running a Next.js dashboard under user systemd, keep the service environment explicit.

Record these values in the service or environment file:

- `WorkingDirectory`
- `EnvironmentFile`
- `PORT`
- `HOSTNAME`
- `PATH`
- command paths such as `OPENCLAW_BIN`, `HERMES_BIN`, and `OH_BIN`

The `PATH` entry matters. A command may work in an interactive shell and fail under systemd if it relies on `env node`, nvm, or a user-local binary.

Public frontend variables such as `NEXT_PUBLIC_*` are baked into the built client. Rebuild and restart after changing labels, titles, or other public UI defaults.

## Access Boundary

Use loopback-only binding by default.

If the dashboard is exposed through a tunnel, reverse proxy, or private mesh, require a separate access control layer. Do not rely on "obscure URL" access. If the UI can trigger agent actions, also require an action token or an equivalent authenticated control path.

Avoid storing Discord bot tokens in the dashboard just to prove a message was sent. Reading OpenClaw session logs is often enough to verify visible replies without widening the secret surface.

## Verification

After deployment, verify the dashboard as an evidence source:

```text
GET /dashboard
GET /api/dashboard?force=1
```

The API response should confirm, at least:

- gateway health is available
- action authentication state is visible
- heartbeat state is readable
- note count is non-zero when notes exist
- the latest note filename matches the agent's actual output directory
- the latest review filename matches the scheduler's real output convention

If the UI is healthy but the evidence is empty, suspect a role mismatch before suspecting the agent.

## Publication Boundary

Do not publish concrete hostnames, private IPs, channel IDs, tokens, raw logs, or private wrapper code.

Publish the reusable pattern instead: what layer failed, what evidence was missing, how the dashboard was made role-specific, and what verification proved it.
