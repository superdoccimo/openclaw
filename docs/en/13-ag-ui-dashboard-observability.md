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
- optional tool visibility, such as whether OpenHarness/OH is part of that
  agent's actual operating model

Inherited defaults are a common source of false negatives. For example, a dashboard can say there are no daily notes simply because it is still reading a security agent's `daily-log` directory while the research agent writes to `research/`.

The same problem can happen with review files. If the CLI or scheduler writes `weekly-review-YYYY-MM-DD.md` but the dashboard only searches for `review-YYYY-MM-DD.md`, the review panel will look empty even though the review mechanism is working.

Event stores need the same care. An append-only `events.json` can contain old
errors after a healthy recovery. A role-specific dashboard should show current
unresolved errors separately from historical errors, and should make the event
store path visible enough to catch copied deployment drift.

Optional tools should not become permanent red warnings. If an agent does not
use OpenHarness, a dashboard should be able to hide the OH panel and skip the OH
CLI probe instead of repeatedly reporting `oh` as missing. A missing optional
tool is not an incident if the role has a different approved workflow.

This should not be used to hide a real operating surface. If a tool, channel,
event store, note directory, or auth provider is part of the agent's actual
workflow, absence or stale evidence should remain visible. Role-specific
configuration is a way to remove irrelevant probes, not a way to make the
dashboard quiet.

Use configuration flags for this rather than local source forks, for example:

```text
AGUI_OPENHARNESS_ENABLED=false
```

The same pattern applies to any role-specific integration. Disable unsupported
evidence collectors explicitly, then keep the rest of the dashboard generic and
updateable.

## Updateable Deployment Shape

A copied dashboard working tree is easy to customize once and hard to update
later. Prefer a clean reusable-app checkout plus local-only configuration:

```text
origin: reusable dashboard repository
upstream: official AG-UI repository
local .env: agent label, event paths, optional integrations, command paths
systemd unit: port, host binding, service environment
```

Generic improvements should land in the reusable dashboard repository. Host
specific values should stay in ignored environment files and service units.
This lets official AG-UI changes be reviewed and pulled forward without mixing
private machine state into the app source.

## Useful Evidence

The operator should be able to answer these questions without SSHing into the host:

- Is the OpenClaw gateway reachable?
- Which channels are configured and running?
- Did the agent produce a visible Discord reply?
- Is there a non-ACK assistant response in the OpenClaw session log?
- What heartbeat tasks ran recently?
- Which durable notes were written most recently?
- Are recent events empty, queued, skipped, ok, or error?
- Are event errors current unresolved alerts or historical records already
  cleared by a recovery event?
- Is action authentication enabled before exposing the UI beyond loopback?

For a research agent, the most valuable dashboard row is often the durable note row. Chat notifications are pointers. Research notes, backlog entries, state files, and public docs are the reusable output.

For a security or maintenance agent, proposal artifacts can be just as
important. If a heartbeat or review can create proposal-only coding-agent
requests, the dashboard should show those artifacts too:

- request count
- proposal count
- raw run-log count, if retained
- latest request path
- latest proposal path
- last checked time
- a short rendering of the latest proposal

Otherwise the operator cannot distinguish "the lab is idle" from "the lab is
working but invisible."

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

## Dependency Update Boundary

AG-UI dashboards often sit on Next.js, CopilotKit, Tailwind, Hono, and other
fast-moving packages. Treat dependency updates as an operational change, not as
a blind cleanup.

Safe update sequence:

```text
check outdated packages
update low-risk patch/minor dependencies first
run audit without force
build
restart the user service
query the dashboard API
record remaining advisories separately
```

Avoid running forceful audit fixes that downgrade or cross major framework
versions. If an audit tool proposes a surprising framework downgrade, that is a
signal to stop and review the dependency tree, not permission to apply it.

Also watch for this trap:

```text
npm audit fix --omit=dev
```

It can remove build-time development dependencies from `node_modules` while
leaving `package.json` unchanged. The service may keep running from an existing
standalone build, but the next `next build` can fail because PostCSS, Tailwind,
TypeScript, or other build tools are missing.

After any audit fix, run a real build before restarting production-like
services.

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
- unresolved and historical event errors are distinguishable
- note count is non-zero when notes exist
- the latest note filename matches the agent's actual output directory
- the latest review filename matches the scheduler's real output convention

If the UI is healthy but the evidence is empty, suspect a role mismatch before suspecting the agent.

If the API returns only an auth error or a minimal error object, verify the
request first. A missing or malformed dashboard action token can look like a
server-side data failure. Check authentication before debugging the evidence
collectors.

## Publication Boundary

Do not publish concrete hostnames, private IPs, channel IDs, tokens, raw logs, or private wrapper code.

Publish the reusable pattern instead: what layer failed, what evidence was missing, how the dashboard was made role-specific, and what verification proved it.
