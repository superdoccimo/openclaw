# 2026-05-14

## 概要

- watch script の error を、通知と dashboard 表示だけで終わらせない方針を整理した。
- `events.json` の直近 error は、人間向けの赤表示であると同時に、次の heartbeat が一次切り分けする入口になる。
- 公開 docs では、実 host 名や private path ではなく、watch event -> heartbeat triage -> safe boundary という判断構造だけを残す。

## 観測したこと

- dashboard に watch error の件数だけが出ると、何が起きたか分かりにくい。
- error summary を3件ほど出すと、同じ watch が繰り返し失敗していることが見える。
- ただし dashboard は read-only viewer なので、表示しただけでは agent が自律修正するわけではない。
- watch が `warn` / `fail` / `recovered` を検出したとき、system event として次の heartbeat に渡す必要がある。

## 誤解しやすかったこと

- 通知できたことは、原因を切り分けたことではない。
- `events.json` に残ったことは、修正に進んだことではない。
- watch script が check-only であることと、agent が heartbeat で一次切り分けすることは両立する。

## 実際に大事だった層

- dashboard UI ではなく、watch result を heartbeat に渡して agent が考える層。
- watch script は remote service を勝手に変更せず、原因分類と safe boundary の材料を渡す。
- 実マシン変更、credential、service restart、update 実行は別の approval boundary で扱う。

## sanitized evidence

- recent events contained repeated remote watch errors.
- adding top summaries to the dashboard made the repeated pattern visible.
- the useful public pattern is not the private remote target, but the handoff from check-only watch to heartbeat triage.

## 次に見る場所

- `docs/en/15-watch-event-follow-up.md`
- `docs/en/02-heartbeat-and-safety.md`
- `docs/en/07-safe-autonomous-remediation.md`
- `docs/en/13-ag-ui-dashboard-observability.md`
