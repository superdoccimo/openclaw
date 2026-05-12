# Publication And Redaction

公開前に、個人情報、secret、実環境の情報を取り除きます。

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
| actual private IP | `10.0.0.10` style placeholder only when needed |
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
