# Contributing

This repository collects operational patterns, not raw private logs.

Before contributing, redact:

- token, refresh token, API key, cookie, credential
- real IP, real domain, real email address
- personal agent names
- Discord / Telegram / LINE channel IDs
- private host names and SSH aliases
- raw attack logs with identifying data

Prefer this structure for new findings:

```text
problem
cause
misleading signal
better check
safe remediation boundary
example notification
```

If the note is still rough, add it as a daily note first. Promote it to stable docs after the pattern repeats or becomes reusable.

Executable scripts, private operational tools, and raw run output should not be added to this repository. Describe the pattern, boundary, and verification method in Markdown instead.
