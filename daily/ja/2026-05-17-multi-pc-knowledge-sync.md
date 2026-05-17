# 2026-05-17

## 概要

- 複数PCや複数エージェントが同じ `openclaw` field-note repo に知見を書く場合、知見そのものよりも「混在を防ぐ同期手順」が重要になる。
- 対策は、各PCで直接記憶から書き足すのではなく、まずローカル clone で `fetch` し、`main..origin/main` の差分を見てから追記すること。
- そのPC固有の `.codex/skills` は実行用のローカル知見として扱い、公開repoには判断構造だけを残す。

## 観測したこと

- 別PCからすでに複数の知見コミットが入っていた。
- ローカル作業ツリーは clean でも、`origin/main` が先に進んでいることがある。
- 先に `fetch` して差分を見ると、同じ日に別PCが追加した note や stable docs を確認してから、重複しない追記ができる。
- `.codex/skills` はPCごとに差があり、あるPCで見える skill が別PCでは見えないことがある。

## 誤解しやすかったこと

- `git status` が clean でも、リモートとの差分がないとは限らない。
- 「知見を残す」は、必ずしも skill をそのまま公開することではない。
- private path、PC名、実アカウント、token、raw log を消したあとでも、判断構造だけなら十分に再利用できる。
- 別PC由来の知見と同じ題材でも、症状、層、対策が違うなら別 daily note に分けた方がよい。

## 実際に壊れていた層

- OpenClaw runtime ではなく、knowledge capture workflow。
- 複数PCで同じ公開repoを使う場合、push 前の `fetch` / diff / fast-forward 確認が運用境界になる。

## sanitized evidence

実値は載せない。構造だけ残す。

```text
local clone:
  status: clean
  relation: behind origin/main

remote:
  new daily notes
  new stable docs
  same-day field knowledge from another PC

safe action:
  fetch
  inspect main..origin/main
  fast-forward
  add one focused note
  commit and push one topic
```

## 次に見る場所

- `git status --short --branch`
- `git fetch origin`
- `git diff --stat main..origin/main`
- `git diff --name-status main..origin/main`
- `daily/ja/` の同日 note
- `docs/en/` の既存 stable docs

## 昇格候補

- [x] docs/en
- [ ] examples
