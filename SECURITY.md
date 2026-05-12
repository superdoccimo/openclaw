# Security Policy

Do not open public issues or pull requests that contain secrets, raw credentials, real tokens, private keys, private host names, real IP addresses, or private logs.

When reporting a security-related operational pattern, include sanitized evidence only.

Good:

```text
Repeated private-network target probe candidates were observed.
No token values were present in logs.
```

Bad:

```text
Raw access logs, real source IPs, real domain names, tokens, or exploit payloads.
```

This repository is for defensive operations and safe runbooks.
