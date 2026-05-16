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

## 追加で残すべき知見

### proposal-only coding agent lab

Hermes review、daily note、learning note、project backlog から coding agent を呼ぶ場合、いきなり production を変更させるのではなく、まず Markdown proposal を作るだけの lab として扱うと安全だった。

この場合、重要なのは「AI同士が直接勝手に操作する」ことではない。request file、read-only run、proposal file、dashboard visibility という durable artifact の流れにすること。

安全な境界は次の形。

```text
allowed:
  sanitized notes を読む
  improvement proposal を書く
  OSS idea seed を書く
  dashboard で見える形にする

not in proposal step:
  production patch
  config change
  public release
  package publish
  domain / cloud / external service change
```

heartbeat には「気が向いたら」「よいsignalがあれば」で十分。quota や義務にすると自律ではなく作業命令になってしまう。

### AG-UI dashboard は新しい artifact も見せる

OpenClaw agent に proposal-only lab を追加した場合、dashboard 側も見える化しないと確認できない。

最低限、次を見るとよい。

- request count
- proposal count
- raw log count
- latest request
- latest proposal
- proposal content の短い表示

AG-UI は「画面が表示される」だけでは足りない。agent の現在の責務に合った artifact が読めているかを見る必要がある。

### AG-UI dependency update の罠

`npm audit fix --omit=dev` は、`package.json` に devDependencies が残っていても、`node_modules` から build-time dev dependency を外すことがある。

そのため、既存の standalone build は動いていても、次の `next build` で Tailwind / PostCSS / TypeScript 周辺が見つからず失敗することがある。

dependency update の順番は次が安全。

```text
npm outdated
low-risk patch/minor update
npm audit without force
npm run build
restart service
dashboard API check
```

`npm audit fix --force` が古い major への downgrade を提案したら止める。audit の赤表示を消すために framework を壊す方が危険。

### 古い nvm Node tree は最後に消す

Node / OpenClaw wrapper / gateway / systemd が揃ったあと、古い nvm Node tree が残っていると、doctor や heartbeat が inactive OpenClaw を見つけて混乱する。

ただし、削除前に次を確認する。

```text
current node
nvm default
systemd ExecStart / Environment
running process
OpenClaw doctor
AG-UI service
```

動作中が新しい Node だけなら、`rm -rf` ではなく `nvm uninstall` で古い version を消すのがよい。

削除後は Node doctor、OpenClaw gateway、AG-UI dashboard API を確認する。

## 追加で公開 docs へ昇格

- `docs/en/06-node-wrapper-alignment.md`
- `docs/en/13-ag-ui-dashboard-observability.md`
- `docs/en/16-cross-agent-consultation.md`
- `docs/en/19-proposal-only-coding-agent-lab.md`
