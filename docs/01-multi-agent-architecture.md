# Multi-Agent Architecture

複数 OpenClaw 運用では、agent ごとに責務を分けます。役割を分けないと、market decision、security monitoring、daily ops、research、update work が混ざり、agent が何を守るべきか分からなくなります。

## Public Role Names

| Agent | Responsibility |
| --- | --- |
| `OpenClaw-Control` | 管制塔、生活運用、dashboard、相互監視 |
| `OpenClaw-A` | domain-specific work、特定領域の判断や更新補助 |
| `OpenClaw-Security` | 異常検知、防御判断、security finding |
| `OpenClaw-Research` | OSS 調査、仕様変更追跡、発見の整理 |

## Example Relationship

```text
OpenClaw-Control
  monitors OpenClaw-A
  monitors OpenClaw-Security
  monitors OpenClaw-Research
  owns dashboard and global status

OpenClaw-A
  assists updates for OpenClaw-Control
  owns domain-specific workflows

OpenClaw-Security
  monitors exposed surfaces
  reports SECURITY_WARNING / SECURITY_OK
  proposes or applies only bounded remediation

OpenClaw-Research
  tracks upstream changes
  summarizes new operational findings
```

## Separation Rules

- `OpenClaw-Control` は全体状態を見るが、個別領域の判断を奪わない
- `OpenClaw-A` は domain-specific work を扱うが、全体監視の責任者にしない
- `OpenClaw-Security` は security に集中し、market や生活運用を扱わない
- `OpenClaw-Research` は調査と発見を主にし、本番変更は提案止まりにする

## Why Multiple Agents Help

1台だけでは、「その host 固有の問題」なのか「OpenClaw 全体の問題」なのかが見えにくいです。

複数 agent があると、比較できます。

- A では OAuth が通るが B では通らない
- Control から見ると Security は止まっているが、Security の local status は healthy
- dashboard は green だが journal には timeout がある
- updater は成功したが gateway が古い状態を握っている

比較できること自体が、運用上の強い観測点になります。

## Update Path Risk

相互更新は便利ですが、更新担当側の認証や wrapper が壊れていると、修正したつもりで別の host に古い状態を持ち込むことがあります。

更新 path も監視対象です。

```text
OpenClaw-A updates OpenClaw-Control
OpenClaw-Control updates OpenClaw-A / OpenClaw-Security / OpenClaw-Research
```

片側だけ auth order や Node wrapper を直しても、もう片側の updater が古い状態なら再発します。
