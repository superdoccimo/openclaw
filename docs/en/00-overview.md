# Overview

This repository organizes field notes from running multiple OpenClaw agents in long-lived environments.

OpenClaw can be manageable as a single installation, but real operations quickly become layered. OpenClaw itself, gateway state, channels, OAuth, Hermes, OH, systemd, Node, dashboards, remote access, and security monitoring can all fail independently.

When the visible symptom is "OpenClaw is not responding", the actual cause may be one of these:

- the gateway is holding stale auth state
- auth profile priority does not match the production profile
- Discord / Telegram / LINE permissions or history access are broken
- a systemd service uses a different PATH than the interactive shell
- the dashboard is repeatedly running expensive checks
- a wrapper is pinned to an old Node or OpenClaw CLI
- tunnel, reverse proxy, or private mesh access is down
- heartbeat executed, but safety was not actually assessed

## Design Position

This repository does not treat OpenClaw as a cron replacement.

Operational health should not be inferred from exit codes alone. A useful operator view combines doctor output, status, auth profile state, journal evidence, dashboard state, remote checks, daily logs, and human-readable assessment.

## Target Reader

- operators running multiple OpenClaw agents or roles
- people designing peer monitoring between agents
- people adding dashboards or heartbeat checks to production-like environments
- people who have seen OAuth, gateway, systemd, Node, or wrapper drift
- people who want bounded autonomous remediation without leaking secrets or losing rollback paths

## Non Goals

- beginner installation guide
- exact reproduction of a private environment
- raw logs containing tokens, IPs, domains, or credentials
- exploit instructions
- executable operational scripts

## Knowledge Flow

```text
daily note
  -> repeated incident pattern
  -> reusable runbook
  -> example notification or state format
  -> stable docs
```

Discoveries do not need to be perfectly classified on the first day. Capture them as daily notes first, then promote them when the pattern becomes reusable.

## Docs-Only Policy

This public repository is docs-only.

Private scripts, wrappers, and tools should stay outside the repository. When implementation detail matters, describe the check, safety boundary, dry-run behavior, rollback expectation, and verification method in Markdown.
