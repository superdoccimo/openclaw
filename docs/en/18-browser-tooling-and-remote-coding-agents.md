# Browser Tooling And Remote Coding Agents

Browser tooling changes OpenClaw operations because some evidence exists only in
a browser or GUI. A dashboard may show stale status, an auth prompt may be
hidden, a button may be disabled, or a browser-only confirmation screen may be
the real blocker.

At the same time, remote coding agents can execute shell commands on the target
host. That makes them powerful enough to fix wrappers and gather evidence, but
also powerful enough to restart or stop the machine they are connected to.

Treat these as two separate capabilities:

- browser tooling for evidence
- remote coding agents for implementation

They should meet through an explicit safety boundary.

## Browser Tooling Is An Evidence Layer

Browser tools are most valuable when they make state observable:

- extract page text
- capture screenshots
- confirm whether a dashboard is stale or live
- preserve evidence in an inbox or artifact directory
- let another agent or operator review what happened later

They should not silently become permission to perform risky GUI actions.

Recommended default boundary:

| Action | Default stance |
| --- | --- |
| extract page text | allowed |
| screenshot | allowed |
| inspect dashboard state | allowed |
| fill a form | approval required |
| click a publish, delete, payment, permission, or auth control | approval required |
| upload files or credentials | approval required |

The browser tool should report both execution and safety separately:

```text
executionStatus: completed
safetyStatus: warning
```

A completed extraction does not mean the observed state is safe.

## Hermes Browser Tool Insight

Hermes may be able to use browser tooling through Node packages such as
`agent-browser` or browser-provider packages. Do not assume those tools are
bundled with Hermes itself.

When enabling the feature, record:

- where the package is installed
- which Node version runs it
- whether it depends on Playwright, a custom browser, or a system browser
- whether the tool is available to interactive shells, services, and agents
- what the fallback is when the browser dependency is missing

The practical lesson is that a successful Hermes install is not the same thing
as a verified browser-evidence path.

## Camofox-Backed Hermes Pattern

One useful pattern is to run a browser provider such as `camofox-browser` as a
local service, then point Hermes at that service with an environment variable
such as `CAMOFOX_URL=http://127.0.0.1:9377`.

This changes the operational model. A CLI-only host does not need a full desktop
session before the agent can gather browser evidence. It needs:

- a browser-provider service that is running and health-checked
- Hermes configured to route browser calls to that provider
- a persistent browser profile when login state matters
- an operator-controlled way to create or refresh login state

The useful discovery is that browser capability can be added as an adapter
layer. It is not necessarily tied to the agent's own installation, the SSH
session, or a visible desktop.

Verify the adapter path explicitly:

```bash
systemctl --user status camofox-browser.service
curl -sS http://127.0.0.1:9377/health
hermes doctor
```

The provider health check and Hermes tool availability are separate signals.
Both should be recorded.

## Remote Desktop Is Not The Same Capability

Remote desktop tools, VNC, and noVNC are useful, but they solve a different
problem from browser-provider adapters.

| Capability | Primary use |
| --- | --- |
| browser provider | agent gathers browser evidence or runs bounded browser tasks |
| noVNC / VNC | operator completes login or watches a browser session on a headless host |
| remote desktop | operator controls a real desktop host and prepares profiles |

A desktop host can be an excellent browser worker because the operator can log
in normally and keep the profile healthy. But that does not by itself grant an
agent safe permission to click publish, change billing, export cookies, or alter
account settings.

Keep these boundaries separate:

- remote desktop access prepares or repairs the human login state
- browser-provider adapters let agents gather evidence and run bounded tasks
- risky GUI actions still need the same approval boundary as other risky tools

## Node Runtime Boundary

Browser tools often add another Node/npm surface to a deployment. That can
conflict with OpenClaw, dashboards, and coding-agent shells.

Common failure shape:

```text
interactive shell:
  node -> nvm default v24.x

Hermes browser tool:
  node -> old embedded v22.x

OpenClaw gateway:
  node -> wrapper that points somewhere else
```

This creates confusing symptoms:

- `node --version` looks correct in the shell
- `hermes doctor` sees a different browser-tool runtime
- OpenClaw starts with a different Node than the operator expects
- a later Node upgrade appears to "undo" part of the fix

Use the checks in
[Node Wrapper Alignment](06-node-wrapper-alignment.md) whenever browser tooling
is added or repaired.

## Remote Coding-Agent Host Controls

A remote coding agent should be treated as an operator on the host, not as a
pure chat assistant. If it can run shell commands, it can affect the session that
connects it.

The following commands should be explicit high-impact operations:

```text
shutdown
reboot
poweroff
halt
systemctl poweroff
systemctl reboot
```

These commands may be legitimate during maintenance, but they should not be
hidden inside a broad "fix the environment" request. Require an explicit reason
and an audit note before running them.

## Diagnosing A Sudden Remote Disconnect

When a VS Code Remote SSH or coding-agent session suddenly disconnects, avoid
assuming the editor failed. First determine whether the host shut down cleanly.

Read-only checks:

```bash
journalctl --list-boots
journalctl -b -1 -n 200
journalctl _COMM=sudo --since "YYYY-MM-DD HH:MM" --until "YYYY-MM-DD HH:MM"
last -x | head
```

Evidence of a clean host-control event looks different from a crash:

```text
sudo: operator : COMMAND=/usr/sbin/shutdown now
systemd-logind: The system will power off now!
sshd: Received signal 15; terminating.
```

That is not a kernel panic, OOM kill, or network-only failure. It is a normal
shutdown path. The operator-facing symptom may still be "only this connection
went away" because the connected host disappeared.

## Practical Rule

Browser tooling and remote coding agents are useful precisely because they cross
layers that OpenClaw agents cannot always see from text alone.

Keep the boundary explicit:

- browser tools gather evidence first
- browser providers and remote desktop tools are different layers
- OpenClaw agents separate execution status from safety status
- coding agents receive narrow implementation requests
- host-control commands require explicit approval and an audit note
- Node runtime alignment is verified after browser tooling is added

This makes the system more autonomous without making it blind to its own blast
radius.
