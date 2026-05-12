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
- after updates, verify wrapper and systemd targets read-only
- show `expected` and `actual` separately on dashboards
- treat mismatch as `WARNING`

## Example Assessment

```text
observed: ok
assessment: WARNING
reason: interactive Node and gateway Node differ
expected_node: /home/operator/.nvm/versions/node/vXX/bin/node
actual_gateway_node: /usr/bin/node
next_action: align wrapper and restart service after test
```
