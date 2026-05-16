# 2026-05-16

## 概要

- Hermes に browser tool を追加できることは、OpenClaw 運用上かなり重要な発見だった。
- ただし browser tool は Hermes 本体の機能と決めつけず、npm package、PATH、Node runtime、browser dependency を分けて確認する必要がある。
- remote coding agent は便利だが、接続先ホストで shell command を実行できるため、host-control command を別の安全境界として扱う。

## 観測したこと

- `hermes doctor` が browser tool を見る場合でも、その実体が Hermes 同梱か、Hermes workspace の npm dependency か、global package か、PATH 上の wrapper かを切り分ける必要がある。
- browser tool が動くかどうかは、OpenClaw の自律運用における browser evidence の可否に直結する。
- Node は interactive shell、Hermes、OpenClaw gateway、coding-agent shell、systemd service で別々の実体を向きやすい。
- 古い embedded Node が PATH や wrapper で復活すると、shell では新しい Node に見えても、実際の agent tool は古い Node で動くことがある。

## 誤解しやすかったこと

- `node -v` が正しいだけでは十分ではない。
- `npm config get prefix` が正しいだけでも十分ではない。
- Hermes が browser tool を検出したことと、OpenClaw から安定して browser evidence を取れることは同じではない。
- VS Code Remote SSH の切断は、editor 側の問題とは限らない。接続先ホストで正常な shutdown が実行された場合も同じように見える。

## 安全境界

browser tool はまず evidence layer として扱う。

```text
allowed by default:
  extract
  screenshot
  inspect dashboard state

requires explicit approval:
  form fill
  click publish/delete/payment/auth/permission controls
  upload
  credential handling
```

remote coding agent では、次のような command を通常の環境修正と分ける。

```text
shutdown
reboot
poweroff
halt
systemctl poweroff
systemctl reboot
```

## 次回見る場所

- `docs/en/06-node-wrapper-alignment.md`
- `docs/en/16-cross-agent-consultation.md`
- `docs/en/18-browser-tooling-and-remote-coding-agents.md`

## 公開 docs へ昇格

- `docs/en/18-browser-tooling-and-remote-coding-agents.md`
