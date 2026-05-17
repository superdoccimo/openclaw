# 2026-05-17

## 概要

- 低重要度の全体向けメッセージに対して、agent がテキスト返信せずリアクションだけ返すケースを確認した。
- これは無視ではなく、受信確認として成立する場合がある。
- ただし、直接相談、障害、更新失敗、handoff まで「リアクションだけでよい」と広げてはいけない。
- equal-standing な heartbeat では、命令口調を避けることと、必要な本文共有を避けることを混同しない。
- AG-UI dashboard では、役割にないツールを「未導入エラー」として出し続けない設計が必要。

## 観測したこと

- `@here` のような全体連絡に対して、agent は Discord reaction を付けたが、本文返信は空だった。
- セッションログ上はメッセージ受信、reaction tool call、reaction 成功が残っていた。
- 人間の画面では「返事がない」ように見えたが、実際には reaction による受信確認があった。
- heartbeat に「リアクションだけで終わらず必ず短く返す」と書くと、実質的にコメントを強制する方向へ寄る。
- その修正は、低重要度の全体連絡では相手の自然な反応を狭めるため、equal-standing policy と衝突する。

## 誤解しやすかったこと

- `no text reply` は `no response` ではない。
- 全体連絡に毎回テキスト返信を求めると、通知ノイズが増える。
- 「見える返事が欲しい」という人間側の都合だけで agent の振る舞いを固定すると、人格・判断・口調の強制になる。
- reaction は軽い応答であり、作業依頼・相談・警告とは別の扱いにする。
- 「対等」は沈黙の許可ではない。短い本文が必要な場面では、命令口調を避けながら事実を共有する。

## sanitized pattern

Heartbeat や相談役の運用メモでは、次のように書く。

```text
General announcements may be acknowledged with a reaction only.
If a short text reply is useful, the agent may send one.
Do not force a text comment when acknowledgement is already clear.
For direct questions, incidents, failed updates, handoffs, or unresolved risks,
prefer a short text reply or durable note.
```

避ける書き方:

```text
Always reply in text when mentioned.
Do not only react.
Reaction-only is always enough.
```

## Dashboard scope

- AG-UI dashboard は agent の役割に合わせて証拠面を変える。
- 役割にない OpenHarness / OH などを未導入エラーとして出し続けると、実際には問題でないものが警告になる。
- その場合は `AGUI_OPENHARNESS_ENABLED=false` のような環境変数で、UI表示とCLI probeを無効化する。
- host 固有設定は `.env` や systemd unit に置き、dashboard source へ直接混ぜない。

## 公開 docs へ昇格

- `docs/en/02-heartbeat-and-safety.md`
- `docs/en/13-ag-ui-dashboard-observability.md`

## 次に見る場所

- `docs/en/14-heartbeat-insight-policy.md`
- `docs/en/16-cross-agent-consultation.md`
- `docs/en/17-event-store-recovery-semantics.md`
