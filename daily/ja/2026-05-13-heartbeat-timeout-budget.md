# 2026-05-13

## 概要

- heartbeat の timeout は、認証切れや channel 障害とは別の層として扱う必要がある。
- `Request timed out before a response was generated` は、まず gateway log と heartbeat timeout 設定を見て判断する。
- 全体 timeout を上げる前に、heartbeat 専用の timeout budget が足りているかを確認する。

## 観測したこと

- ある agent で Telegram への heartbeat 応答が timeout した。
- Codex OAuth profile と auth order は正常だった。
- gateway は起動しており、channel も configured だった。
- journal では embedded run が timeout しており、timeout 値は heartbeat 設定と一致していた。
- heartbeat は単なる ACK ではなく、イベント確認、peer update 結果、ログ確認、学習メモまで期待されるようになっていた。

## 誤解しやすかったこと

- timeout 表示だけを見ると、認証切れ、Telegram 障害、OpenClaw 本体故障に見える。
- `agents.defaults.timeoutSeconds` をすぐ上げると、通常会話と heartbeat の層が混ざる。
- gateway が active でも、heartbeat 用の作業量に対して timeout が短いと失敗する。

## 実際に壊れていた層

- 認証や channel ではなく、heartbeat 実行時間の budget。
- heartbeat が高度化したのに、timeoutSeconds が古い短めの値のままだった。

## sanitized evidence

- auth profile は production profile を優先していた。
- gateway log では `timeoutMs=<heartbeat timeout>` と一致する embedded run timeout が見えた。
- 設定変更は heartbeat timeout のみを対象にし、global timeout は変更しなかった。
- 実 host 名、private IP、account、channel ID、raw log、token は載せない。

## 次に見る場所

- `docs/en/02-heartbeat-and-safety.md`
- `docs/en/03-auth-profile-and-gateway-restart.md`
- `docs/en/10-maintenance-workflow.md`

## 昇格候補

- [x] docs/en
- [ ] examples
