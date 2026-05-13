# Auth Profile And Gateway Restart

Re-authentication is not finished when a login command succeeds.

In environments with multiple auth profiles, the production path can still time out even when a valid profile exists. A stale default profile may remain, and the gateway may continue holding old auth state.

## Common Failure Shape

```text
openclaw models auth list
  openai-codex:<account>  valid
  openai-codex:default    expired

gateway log
  Token refresh failed: refresh_token_reused
  Profile openai-codex:<redacted> timed out. Trying next account...
```

`auth list` looks acceptable because a valid profile exists. The real Discord / Telegram / LINE response path can still fail because the gateway is using stale state.

## Read-Only Checks

```bash
openclaw doctor
openclaw status
openclaw models auth list
openclaw models auth order get --provider openai-codex
systemctl --user status openclaw-gateway.service --no-pager
journalctl --user -u openclaw-gateway.service -n 100 --no-pager
```

## Repair Pattern

Confirm the production profile first, then set the order explicitly.

```bash
openclaw models auth order set --provider openai-codex openai-codex:<account>
systemctl --user restart openclaw-gateway.service
```

Then verify the order, service state, journal, and channel response path.

```bash
openclaw models auth order get --provider openai-codex
systemctl --user status openclaw-gateway.service --no-pager
journalctl --user -u openclaw-gateway.service -n 100 --no-pager
```

## Important Points

- successful login and production recovery are separate events
- having a valid profile and using the correct priority order are separate checks
- fixing auth order may not refresh gateway memory by itself
- verify channel responses after restart
- never print token or refresh token values in logs or notifications

## Update Agent Risk

In peer-update setups, the updater's profile is also production infrastructure.

If `OpenClaw-A` updates `OpenClaw-Control`, then `OpenClaw-A` auth order and gateway state can affect the update itself.
