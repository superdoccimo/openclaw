# Publication And Redaction

Remove personal, secret, and environment-specific details before publishing.

The safest public material usually preserves structure, not values. Keep the role, failure mode, decision rule, rollback expectation, and verification method. Remove the original names, tokens, domains, private IPs, channel IDs, paths, and raw logs.

This matters because operational notes often include exactly the evidence that made the incident understandable. That evidence is useful, but the public version should describe the type of evidence rather than copy the private artifact.

## Replace Personal Agent Names

| Private concept | Public replacement |
| --- | --- |
| control agent name | `OpenClaw-Control` |
| domain-specific agent name | `OpenClaw-A` |
| security agent name | `OpenClaw-Security` |
| research agent name | `OpenClaw-Research` |
| human operator name | `Operator` |

## Replace Environment Details

| Private detail | Public replacement |
| --- | --- |
| actual domain | `example.com` or `redacted-domain` |
| actual host | `host-a`, `app-host-a`, `proxy-host-a` |
| actual private IP | placeholder only when needed |
| actual path | `/opt/openclaw-ops/...` |
| actual account | `<account>` |
| actual profile | `openai-codex:<account>` |
| actual channel ID | `<channel-id>` |

## Do Not Publish

- token
- refresh token
- API key
- cookie
- credential file content
- private key
- real email address
- real channel ID
- exact attack source IP
- raw access log with identifying data
- process argv that contains secret values
- executable operational scripts
- raw command output copied from production hosts

## Publish The Pattern

Prefer this shape:

```text
Symptom: dashboard reports OpenClaw unhealthy while CLI status is healthy
Likely layer: gateway service, wrapper path, auth profile, or dashboard cache
Evidence to collect: service status, configured entrypoint, CLI version, gateway status, recent journal
Safe action: read-only diagnosis first; backup before changing wrapper or service files
Verification: CLI and gateway report the same version and the service remains active after restart
```

Avoid this shape:

```text
Full private command transcript with real hostnames, tokens, channel IDs, absolute private paths, and raw logs
```

If a script is important, publish the diagnostic contract instead of the script:

- what it checks
- what it never changes
- what status labels it returns
- what a warning means
- how to verify the warning manually
- how to roll back any separate remediation step

## Safer Search Before Commit

Run these checks before adding files to git.

```bash
rg -n -i "token|refresh_token|api key|apikey|secret|cookie|credential|private key|password|bearer|authorization" .
rg -n -i "discord|telegram|line|channel|webhook|email|account" .
rg -n "([0-9]{1,3}\.){3}[0-9]{1,3}" .
rg -n "https?://[^ )]+" .
```

These commands find candidates. They do not prove the repository is safe.

## Writing Security Notes

Write defensive patterns, not exploit instructions.

Prefer:

```text
metadata service / private network target should be treated as deny candidate
```

Avoid:

```text
step-by-step exploit payloads against a real app
```

## Source Notes

Private source notes may contain rough wording, personal names, host hints, or implementation details. Keep them outside git or ignored by `.gitignore`.
