# openclaw

Unofficial field notes and operational patterns for running multiple OpenClaw agents.

This repository is not the official OpenClaw project. It collects field-tested notes, runbooks, failure patterns, and safety boundaries from long-lived multi-agent operations.

This is not a beginner installation guide. It focuses on operational reality: gateway state, auth profiles, heartbeat design, dashboard load, systemd services, SSH-based checks, Hermes / OH handoffs, safe autonomous remediation, daily logs, and review loops.

## Relationship To Official Docs

Use the official OpenClaw documentation as the source of truth for product behavior, installation, command syntax, and supported configuration.

Use this repository for the parts that only become obvious after operating multiple agents for a while: which layer failed, what evidence was useful, how to avoid repeating the incident, and how to document the recovery without publishing private environment details.

The goal is not to mirror official documentation. The goal is to preserve field-tested operating judgment.

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
- AG-UI dashboard observability and role-specific UI drift
- Heartbeat insight policy for notification and quiet-decision quality
- Watch event follow-up loops that turn repeated failures into safe triage
- Cross-agent consultation for technical ambiguity, browser evidence, and coding-agent handoffs
- Event store recovery semantics that separate historical errors from unresolved alerts
- Browser tooling and remote coding-agent boundaries for Hermes-style browser providers, Camofox-style adapters, Node alignment, and host-control safety
- Proposal-only coding-agent labs for turning reviews, learning notes, and
  repeated findings into bounded Markdown proposals
- NotebookLM source-pack handoffs for using CLI upload adapters while keeping
  brittle artifact-generation UI actions with the operator
- Duty desk ticket bridges for turning unresolved cross-agent consultations
  into local tickets, risk labels, handoff drafts, and human-review holds
- Multi-PC knowledge synchronization for keeping daily notes and stable docs
  coherent when several machines write into the same repository
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
    11-work-as-you-go-knowledge-capture.md
    12-autonomy-enablement.md
    13-ag-ui-dashboard-observability.md
    14-heartbeat-insight-policy.md
    15-watch-event-follow-up.md
    16-cross-agent-consultation.md
    17-event-store-recovery-semantics.md
    18-browser-tooling-and-remote-coding-agents.md
    19-proposal-only-coding-agent-lab.md
    20-notebooklm-source-pack-handoff.md
    21-duty-desk-ticket-bridge.md
    22-multi-pc-knowledge-sync.md
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
| `OpenClaw-Consult` | cross-agent technical consultation, browser evidence, safe handoff shaping |
| `Operator` | human operator |

## Remote Access Policy

This project does not require a specific networking product.

Cloudflare Tunnel is a practical default for exposing dashboards or webhook entrypoints without opening direct inbound ports. Tailscale and other private mesh tools can also be useful for operator-only access and host-to-host checks.

The important boundary is to keep the access layer separate from the OpenClaw layer. Tunnel, mesh, reverse proxy, dashboard, gateway, and provider failures should be observed and reported as separate layers.

## Docs-Only Policy

This repository is docs-only for now. Executable scripts, private wrappers, raw command output, and production logs are intentionally kept out of the public repository.

When a tool or script becomes relevant, describe the operational pattern, safety boundary, dry-run expectation, rollback expectation, and verification method in Markdown.

Documentation alone is a useful artifact. In multi-agent operations, the reusable value is often the decision structure rather than the private script that implemented it once.

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
