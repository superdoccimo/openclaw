# Dashboard Self-DoS

Dashboards are useful, but a poorly designed dashboard can make OpenClaw heavier.

The common failure is repeatedly running expensive diagnostics such as `openclaw status` or deep channel checks at short intervals.

## Failure Pattern

```text
dashboard polling every few seconds
  -> openclaw status
  -> channel deep checks
  -> remote SSH checks
  -> journal reads
  -> gateway timeout increases
  -> dashboard shows stale or wrong status
```

The dashboard helps detect problems, but it can also become part of the problem.

## Safer Pattern

- run heavy checks less frequently
- cache API responses
- show stale state explicitly
- avoid overly aggressive timeouts
- keep dashboard actions read-only by default
- make deep checks manual or low-frequency
- inspect OpenClaw journals and dashboard API journals separately

## Dashboard Should Show Assessment, Not Only Execution

Weak display:

```text
heartbeat: done
status: green
```

Better display:

```text
heartbeat: done
observed: ok
assessment: SECURITY_WARNING
evidence_age: 8m
dashboard_cache: stale
```

## What To Monitor About Dashboard Itself

- service state
- expected port
- API timeouts
- cache age
- auth requirement
- whether dashboard checks call OpenClaw CLI too often

A dashboard is both an observation surface and an operational component.
