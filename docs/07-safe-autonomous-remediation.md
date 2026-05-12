# Safe Autonomous Remediation

OpenClaw に自律修正を任せる場合、境界を明確にします。

基本方針は、read-only は広く、write は狭く、apply は wrapper 経由、test 必須、rollback 必須、secret は出さない、です。

## Levels

| Level | Capability | Example |
| --- | --- | --- |
| L0 | observe only | status, journal, cache age |
| L1 | propose | runbook と patch plan を出す |
| L2 | bounded apply | allowlisted wrapper を実行する |
| L3 | operator-approved apply | 人間承認後に config 変更する |

最初から L2 や L3 にしません。再発パターンが十分に固まってから昇格します。

## Requirements Before Apply

- 対象が allowlist に入っている
- current state を記録した
- dry-run または config test が通る
- rollback command がある
- secret を表示しない
- result を daily-log に残す

## Bad Automation

```text
agent found suspicious request
agent edits nginx config directly
agent restarts service
dashboard turns green
```

この流れでは、何を根拠に変更したか、誤爆した時に戻せるか、config が正しいかが分かりません。

## Better Automation

```text
agent found repeated deny candidate
agent writes sanitized finding
agent runs wrapper --dry-run
agent runs nginx -t or equivalent test
agent applies allowlisted rule through wrapper
agent records result and rollback command
agent reports SECURITY_WARNING -> SECURITY_OK only after verification
```

## Wrapper Principle

agent に raw config edit を許すのではなく、運用者が設計した wrapper に閉じます。

wrapper は次を持つべきです。

- allowlist
- dry-run
- config test
- backup
- rollback
- structured output
- secret masking
