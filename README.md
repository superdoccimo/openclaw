# openclaw

Unofficial field notes and operational patterns for running multiple OpenClaw agents.

This repository is not the official OpenClaw project. It collects field-tested notes, runbooks, failure patterns, and safety boundaries from long-lived multi-agent operations.

This is not a beginner installation guide. It focuses on operational reality: gateway state, auth profiles, heartbeat design, dashboard load, systemd services, SSH-based checks, Hermes / OH handoffs, safe autonomous remediation, daily logs, and review loops.

## Languages

The canonical documentation is written in English.

Japanese summaries and operation notes are available in [README.ja.md](README.ja.md), [docs/ja/](docs/ja/), and [daily/ja/](daily/ja/).

## What This Covers

- Multi-agent responsibility boundaries and peer monitoring
- Heartbeat design that separates `exit 0` from `SECURITY_OK`
- OAuth / auth profile / gateway restart operations
- Dashboard self-DoS and stale status problems
- Security agent boundaries and wrapper-based remediation
- Node / npm / nvm / systemd / wrapper alignment
- Remote access patterns such as Cloudflare Tunnel and private mesh networks
- Human-readable notifications, daily logs, and sanitized evidence

## Repository Layout

```text
README.md
README.ja.md
docs/
  en/
    00-overview.md
    01-multi-agent-architecture.md
    02-heartbeat-and-safety.md
    03-auth-profile-and-gateway-restart.md
    04-dashboard-self-dos.md
    05-security-agent-boundaries.md
    06-node-wrapper-alignment.md
    07-safe-autonomous-remediation.md
    08-remote-access-tunnels.md
    09-publication-and-redaction.md
    10-maintenance-workflow.md
  ja/
    README.md
examples/
  notifications/
  state-formats/
daily/
  TEMPLATE.md
  ja/
    TEMPLATE.md
.github/
  ISSUE_TEMPLATE/
  pull_request_template.md
```

## Public Agent Names

Private agent names are not used in public docs. Public docs use role-based names.

| Public name | Role |
| --- | --- |
| `OpenClaw-Control` | control tower, daily operations, peer monitoring, dashboard |
| `OpenClaw-A` | domain-specific agent |
| `OpenClaw-Security` | security monitoring agent |
| `OpenClaw-Research` | OSS research, discovery, upstream change tracking |
| `Operator` | human operator |

## Remote Access Policy

This project does not require a specific networking product.

Cloudflare Tunnel is a practical default for exposing dashboards or webhook entrypoints without opening direct inbound ports. Tailscale and other private mesh tools can also be useful for operator-only access and host-to-host checks.

The important boundary is to keep the access layer separate from the OpenClaw layer. Tunnel, mesh, reverse proxy, dashboard, gateway, and provider failures should be observed and reported as separate layers.

## Docs-Only Policy

This repository is docs-only for now. Executable scripts, private wrappers, raw command output, and production logs are intentionally kept out of the public repository.

When a tool or script becomes relevant, describe the operational pattern, safety boundary, dry-run expectation, rollback expectation, and verification method in Markdown.

## Security And Redaction

Do not publish:

- tokens, refresh tokens, API keys, cookies, credentials, or private keys
- real IP addresses, domains, email addresses, host names, or SSH aliases
- Discord / Telegram / LINE channel IDs
- raw production logs or attack source IPs
- process arguments that contain secrets
- private executable scripts or wrappers

Use [docs/en/09-publication-and-redaction.md](docs/en/09-publication-and-redaction.md) before publishing new material.

## Update Style

This is a living project. New discoveries should start as daily notes. Promote them to stable docs only when the pattern becomes reusable.

## License

Documentation is licensed under CC BY 4.0. See [LICENSE.md](LICENSE.md).
