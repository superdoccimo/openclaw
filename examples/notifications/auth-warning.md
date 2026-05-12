# Auth Warning Notification

```text
[OpenClaw-Control/auth] 認証確認
- OpenClaw-Control: OpenClaw openai-codex production profile ok
- OpenClaw-A: OpenClaw openai-codex OAuth expired
- OpenClaw-Security: ok
- OpenClaw-Research: Hermes openai-codex OAuth exhausted

assessment: SECURITY_WARNING
evidence:
- valid profile exists, but expired fallback profile is still present
- gateway restart has not been confirmed after auth repair

next:
- set auth order to production profile only
- restart openclaw-gateway.service
- verify channel response path

secret_exposure: none reported
```
