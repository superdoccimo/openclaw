# Node Wrapper Alignment

OpenClaw, Hermes, OH, and dashboards are strongly affected by Node, npm, nvm, systemd, and wrappers.

The interactive shell may use a new Node version while a systemd service or wrapper still pins an old binary. That creates contradictory doctor, gateway, dashboard, and heartbeat output.

## Common Failure Shape

```text
interactive shell:
  node -> /home/operator/.nvm/versions/node/vXX/bin/node

systemd service:
  node -> /usr/bin/node

wrapper:
  openclaw -> /opt/old-openclaw/bin/openclaw
```

In this state, the operator may believe OpenClaw was updated, while production still starts the old CLI.

## Read-Only Checks

```bash
command -v node
node --version
command -v npm
npm --version
command -v openclaw
openclaw --version
systemctl --user cat openclaw-gateway.service
systemctl --user show openclaw-gateway.service -p Environment -p ExecStart
```

## What To Compare

- interactive shell `node`
- systemd user service `ExecStart`
- wrapper-pinned paths
- `openclaw --version`
- `openclaw doctor`
- gateway journal
- actual dashboard process command

## Recommended Practice

- if a wrapper is used, make the Node and OpenClaw path explicit
- prefer a wrapper that resolves the current `nvm` default at execution time
  when the operator intentionally upgrades Node often
- after updates, verify wrapper and systemd targets read-only
- show `expected` and `actual` separately on dashboards
- treat mismatch as `WARNING`
- after the new runtime is verified, remove stale inactive Node trees so old
  OpenClaw installs do not keep appearing as confusing evidence

## Version-Floating Wrapper Pattern

Hard-coding a full Node path is simple, but it can become stale:

```text
/home/operator/.nvm/versions/node/v24.15.0/bin/node
```

That is acceptable for a pinned production service. It is weaker for an
operator workstation where Node may move from one patch release to the next.

In that case, use a small wrapper that:

- loads `nvm`
- resolves `nvm version default`
- executes the matching `node`, `npm`, `npx`, or CLI command
- fails loudly if the default version is missing

The important point is not the exact wrapper implementation. The important
point is that every entrypoint agrees:

- interactive shell
- user `systemd` service
- Hermes-installed browser tools
- OpenClaw CLI
- coding-agent shell

If one of those paths quietly keeps an old embedded Node tree, an update can
appear successful in the shell while Hermes, OpenClaw, or a dashboard continues
to run the old runtime.

## Stale Node Tree Cleanup

After aligning a runtime, old nvm Node installs can become misleading evidence.

This is especially true when an old Node tree still contains a previous global
OpenClaw install:

```text
active:
  /home/operator/.nvm/versions/node/v24.current/lib/node_modules/openclaw

inactive but still present:
  /home/operator/.nvm/versions/node/v24.old/lib/node_modules/openclaw
```

A doctor script may correctly report that the active wrapper and gateway are
aligned, while also listing the old inactive install. That is useful during
repair, but it becomes noise after the repair is complete.

Before removing old Node trees, check:

```bash
nvm current
nvm alias
command -v node
command -v openclaw
systemctl --user show openclaw-gateway.service -p ExecStart -p Environment
ps aux | grep '/.nvm/versions/node/'
```

If only the expected Node is active, prefer:

```bash
nvm uninstall <old-version>
```

Do not remove the current runtime with `rm -rf`. Use `nvm uninstall` for stale
versions and keep the active version explicit.

After cleanup, verify:

```bash
node --version
npm --version
nvm current
nvm alias default
openclaw --version
systemctl --user show openclaw-gateway.service -p ActiveState -p SubState
```

If a dashboard or heartbeat has a Node doctor, it should stop reporting old
inactive OpenClaw installs after the cleanup. That reduces false concern and
makes future drift easier to see.

## Hermes And Browser Tool Checks

Hermes-style browser tooling may introduce another Node surface. Do not assume
that a browser tool is built into Hermes just because `hermes doctor` reports it.
Identify where it comes from:

- package installed in the Hermes workspace
- package installed globally
- executable found through `PATH`
- wrapper or shim that points to a specific Node tree

Useful read-only checks:

```bash
hermes doctor
command -v agent-browser
agent-browser --version
command -v node
node --version
npm config get prefix
```

When browser tooling is used by autonomous agents, runtime drift is not only an
installation issue. It changes whether screenshots, extraction, and browser
evidence are available during incidents.

## Example Assessment

```text
observed: ok
assessment: WARNING
reason: interactive Node and gateway Node differ
expected_node: /home/operator/.nvm/versions/node/vXX/bin/node
actual_gateway_node: /usr/bin/node
next_action: align wrapper and restart service after test
```
