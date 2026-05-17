# 2026-05-17

## 概要

- `openai-codex` に期限切れ profile と有効 profile が混在していても、それだけでは異常と判断しない。
- 実運用で見るべきなのは、存在している profile 一覧ではなく、実際に選ばれている effective profile である。
- dashboard や watch は「期限切れ profile がある」ではなく「effective profile が期限切れ」の場合に警告する。

## 観測したこと

- 過去に別モデルや別ログイン経路を使った環境では、期限切れの default profile が残ることがある。
- TUI で再認証しても、古い profile が消えるとは限らない。
- `auth order` では有効 profile が優先されていたため、実際の送信経路は壊れていなかった。
- dashboard 側が provider 配下の全 profile を「使用中」とみなし、期限切れ profile を実運用問題として扱っていた。

## 誤解しやすかったこと

- `expired profile exists` は `expired profile is in use` ではない。
- `auth list` や provider summary だけでは、実際の優先順までは判断できない。
- JSON を直接編集する前に、OpenClaw の auth order と status JSON を確認する。
- 古い profile の掃除を急ぐより、実使用 profile の選択状態を先に見る。

## 実際に壊れていた層

- 認証そのものではなく、dashboard の観測ロジックが壊れていた。
- provider summary の stale count を、そのまま user-facing issue に昇格していた。
- `oauth.providers[].effectiveProfiles` を見ていなかったため、実使用中ではない期限切れ profile を警告対象にしていた。

## sanitized evidence

- 実アカウント名、token、refresh token、profile 実IDは載せない。
- `openai-codex:<account>` のような仮名だけを使う。
- 判断に必要な構造は次の通り。

```text
provider: openai-codex
profiles:
  - openai-codex:default        expired
  - openai-codex:<account>      ok
auth order:
  - openai-codex:<account>
effectiveProfiles:
  - openai-codex:<account>
```

この状態では stale profile は存在するが、effective profile は有効なので障害扱いしない。

## 次に見る場所

- `openclaw models auth order get --provider openai-codex`
- `openclaw models status --json`
- `oauth.providers[].effectiveProfiles`
- dashboard の issue 判定が provider summary ではなく effective profile を使っているか

## 昇格候補

- [ ] docs/en
- [ ] examples
