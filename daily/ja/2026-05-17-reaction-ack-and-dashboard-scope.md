# 2026-05-17

## 概要

- 全体向けメッセージに対して、agent がテキスト返信せずリアクションだけ返すケースを確認した。
- これは無視ではなく、受信確認として成立する場合がある。
- equal-standing な heartbeat では、テキスト返信を強制すると人格や自然な判断を狭める。
- AG-UI dashboard では、役割にないツールを「未導入エラー」として出し続けない設計が必要。

## 観測したこと

- `@here` のような全体連絡に対して、agent は Discord reaction を付けたが、本文返信は空だった。
- セッションログ上はメッセージ受信、reaction tool call、reaction 成功が残っていた。
- 人間の画面では「返事がない」ように見えたが、実際には reaction による受信確認があった。
- heartbeat に「リアクションだけで終わらず短く返してよい」と書くと、実質的にコメントを促す方向へ寄る。
- その修正は、相手の自然な反応を狭めるため、equal-standing policy と衝突する。

## 誤解しやすかったこと

- `no text reply` は `no response` ではない。
- 全体連絡に毎回テキスト返信を求めると、通知ノイズが増える。
- 「見える返事が欲しい」という人間側の都合だけで agent の振る舞いを固定すると、人格・判断・口調の強制になる。
- reaction は軽い応答であり、作業依頼・相談・警告とは別の扱いにする。

## sanitized pattern

Heartbeat や相談役の運用メモでは、次のように書く。

```text
General announcements may be acknowledged with a reaction only.
If a short text reply is useful, the agent may send one.
Do not force a text comment when acknowledgement is already clear.
```

避ける書き方:

```text
Always reply in text when mentioned.
Do not only react.
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
