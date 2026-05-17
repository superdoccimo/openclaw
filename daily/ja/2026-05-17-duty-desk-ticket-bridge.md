# 2026-05-17 Duty Desk Ticket Bridge

## 概要

- 複数 OpenClaw の相談を、その場の会話だけで処理せず、local ticket に落とす pattern を整理した。
- OpenClaw-Consult は「全部を解決する agent」ではなく、未解決相談を受け取り、分類し、保留し、handoff する duty desk として扱う。
- heartbeat には長い手順を書かず、短い参照だけを置く方が安全。

## 観測したこと

- 人間が別のAIの意見を仲介して coding agent に渡す流れは、実質的に specifier / builder / validator の手作業版だった。
- ただし、実行 pipeline を増やす前に、相談・保留・承認・記録が残る intake surface が必要だった。
- local ticket bridge は、未解決相談を `ticket / risk / status` に変換するだけでも価値がある。

## 誤解しやすかったこと

- 「もっと強い agent が必要」と見えるが、実際には intake と safety gate が不足していることがある。
- browser automation や coding-agent execution を先に増やすと、結果をどこへ戻すか、誰が承認するかが曖昧になる。
- heartbeat に全手順を書くと、bootstrap context が重くなり、重要な境界が埋もれる。

## 安全境界

```text
allowed:
  create local tickets
  classify risk
  create proposal or handoff drafts
  record hold/reject/approval decisions after operator decision

requires approval:
  service restart
  package install or update
  push / merge / release / public visibility change
  DNS / firewall / tunnel / external exposure
  credential / token / cookie / auth file operations
  payment / billing / trading / contract action
```

## 次回見る場所

- `docs/en/21-duty-desk-ticket-bridge.md`
- `docs/en/19-proposal-only-coding-agent-lab.md`
- `docs/en/20-notebooklm-source-pack-handoff.md`

## 公開 docs へ昇格

- `docs/en/21-duty-desk-ticket-bridge.md` として stable docs に昇格した。
