# Remote Access Tunnels

OpenClaw の dashboard、webhook、review UI、local app を扱うとき、remote access layer は重要です。

Cloudflare Tunnel は、public inbound port を開けずに dashboard や webhook entrypoint を出せるため、実運用で扱いやすい選択肢です。一方で、Tailscale のような private mesh も operator-only access には向いています。

このプロジェクトでは、特定製品を前提にせず、access layer を運用パターンとして扱います。

## Design Goal

- 管理用 port を直接 internet に出さない
- tunnel / mesh / reverse proxy の状態を監視対象にする
- OpenClaw 本体の障害と access layer の障害を分ける
- dashboard の公開範囲を明示する
- token や tunnel credential を process argv や log に出さない

## Cloudflare Tunnel Pattern

向いている用途:

- dashboard を限定公開したい
- webhook endpoint を安定させたい
- inbound firewall rule を増やしたくない
- reverse proxy と組み合わせて app を公開したい

注意点:

- `cloudflared tunnel run --token ...` のように token を argv に置かない
- token / credential / tunnel ID を log に出さない
- tunnel health と app health を分けて見る
- tunnel が healthy でも upstream app が 404 / 500 の場合がある
- dashboard 側に auth を置く

## Tailscale Or Private Mesh Pattern

向いている用途:

- operator-only の private access
- host 間の相互監視
- SSH や internal dashboard を internet に出したくない
- device identity を使いたい

注意点:

- mesh が正常でも app service が正常とは限らない
- ACL の変更を runbook 化する
- agent に広すぎる network visibility を与えない

## Health Check Layers

```text
user/browser
  -> tunnel or mesh
  -> reverse proxy
  -> dashboard/app
  -> OpenClaw gateway
  -> channel/provider
```

どの層で壊れているかを分けて確認します。

## Minimum Dashboard Fields

- access method: `cloudflare-tunnel` / `private-mesh` / `local-only`
- tunnel status
- upstream app status
- auth requirement
- last successful request time
- last error class
- credential exposure check

## Recommendation

公開入口には Cloudflare Tunnel を使いやすい default として扱い、operator-only の private access や host 間監視には Tailscale などの mesh を選べる形にします。

どちらを使う場合も、OpenClaw の問題として丸めず、access layer と application layer を分けて記録します。
