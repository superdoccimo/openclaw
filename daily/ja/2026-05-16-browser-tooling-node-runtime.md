# 2026-05-16

## 概要

- Hermes に browser tool を追加できることは、OpenClaw 運用上かなり重要な発見だった。
- ただし browser tool は Hermes 本体の機能と決めつけず、npm package、PATH、Node runtime、browser dependency を分けて確認する必要がある。
- remote coding agent は便利だが、接続先ホストで shell command を実行できるため、host-control command を別の安全境界として扱う。

## 観測したこと

- `hermes doctor` が browser tool を見る場合でも、その実体が Hermes 同梱か、Hermes workspace の npm dependency か、global package か、PATH 上の wrapper かを切り分ける必要がある。
- browser tool が動くかどうかは、OpenClaw の自律運用における browser evidence の可否に直結する。
- Hermes は `camofox-browser` のような browser provider を後付けし、`CAMOFOX_URL` のような接続先設定で browser call を逃がせる場合がある。
- CLI だけの host でも、browser provider が local service として起動していれば、agent が browser evidence を作る土台になる。
- RustDesk、VNC、noVNC は人間がログイン状態を作る入口であり、browser provider は agent が bounded task を実行する入口。ここを混同しない方がよい。
- Node は interactive shell、Hermes、OpenClaw gateway、coding-agent shell、systemd service で別々の実体を向きやすい。
- 古い embedded Node が PATH や wrapper で復活すると、shell では新しい Node に見えても、実際の agent tool は古い Node で動くことがある。

## 誤解しやすかったこと

- `node -v` が正しいだけでは十分ではない。
- `npm config get prefix` が正しいだけでも十分ではない。
- Hermes が browser tool を検出したことと、OpenClaw から安定して browser evidence を取れることは同じではない。
- RustDesk で desktop に入れることと、agent が safe に GUI 自動操作できることは同じではない。
- browser provider の health が OK でも、ログイン済み profile、2FA、追加認証、publish / billing / account setting の承認境界は別に見る。
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

Hermes + Camofox のような構成では、最低限この3つを別々に確認する。

```text
provider health:
  local browser provider is running

Hermes availability:
  Hermes can see the browser tool

operator login state:
  required accounts are logged in through a human-controlled path
```

この3つを分けると、「ブラウザが起動した」「Hermes が tool を認識した」「実サービスにログインして作業できる」を混同しにくい。

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
