# Security Warning Notification

```text
[OpenClaw-Security/reverse-proxy] セキュリティ確認
assessment: SECURITY_WARNING
surface: reverse-proxy
observed: repeated private-network target probe candidates
evidence: sanitized access pattern, no raw source IP included
risk: SSRF-style probing against protected address ranges
next: run deny-rule wrapper in dry-run mode, then config test
secret_exposure: none observed
```
