# 2026-05-18 Hermes back-office と gateway 境界

## 概要

- OpenClaw と Hermes を同じ環境で使う時、両方に同じ chat bot / channel を持たせると責務が混ざる。
- 安全な初期形は、OpenClaw が通知・chat gateway を担当し、Hermes は review、memory、kanban、proposal、weekly review material などの back-office を担当すること。
- Hermes が動いているかどうかは「gateway process があるか」だけでなく、「その agent 役割に対して有効な job と成果物があるか」で見る。

## 観測したこと

- Hermes 側の messaging token が未設定でも、それは直ちに故障とは限らない。
- OpenClaw が chat / notification を担当しているなら、Hermes の messaging platform は disabled by design でもよい。
- ただし Hermes gateway や scheduler が動いていても、active job が 0 なら back-office としてはまだ成果物が出ない。
- local review snapshot のような no-agent job を入れると、外部通知を増やさずに Hermes の有効活用を確認できる。

## 誤解しやすかったこと

- 「Hermes gateway がある」ことと「Hermes が OpenClaw と連携して役に立っている」ことは別。
- 「Hermes の chat token がない」ことと「Hermes が壊れている」ことも別。
- restricted sandbox 内の CLI status は、host の process / systemd / local database が見えず、gateway stopped のような false negative を出すことがある。
- sandbox で止まって見えたからといって、すぐ service restart に進むのは危ない。

## 実際に分けるべき層

```text
OpenClaw:
  chat intake
  notification
  visible operator reply

Hermes:
  cron review
  memory / kanban
  weekly review material
  proposal and handoff draft

Dashboard:
  scheduler state
  active job count
  latest review artifact
  messaging enabled/disabled by design

Operator:
  approval
  risky execution decision
```

## sanitized evidence

実値は載せない。構造だけ残す。

```text
safe state:
  OpenClaw chat channels: active
  Hermes messaging platforms: disabled or not configured
  Hermes scheduler: host-level check required
  Hermes back-office jobs: at least one local review job needed
  Dashboard: latest review artifact should be visible

unsafe state:
  same bot identity used by two gateways
  same token used by two pollers
  sandbox-only status used as restart reason
  gateway process alive but no job creates useful artifacts
```

## 次に見る場所

- OpenClaw channel / gateway status
- Hermes cron job list
- Hermes scheduler or gateway status
- user service status
- dashboard API の Hermes section
- latest review artifact timestamp
- local review output directory

## 安全境界

```text
allowed first:
  read-only status checks
  local no-agent review snapshot
  dashboard display of latest artifact
  proposal / handoff draft

requires approval:
  shared bot token design
  new messaging poller
  service restart
  package update
  external WebSocket or callback
  DNS / firewall / tunnel / reverse proxy change
  credential / token / auth file operation
  push / merge / release
```

## 公開 docs へ昇格

- `docs/en/23-hermes-backoffice-gateway-boundary.md` として stable docs に昇格した。
