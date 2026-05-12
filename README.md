# openclaw

Unofficial field notes and operational patterns for running multiple OpenClaw agents.

このリポジトリは OpenClaw 本体や公式リポジトリではありません。複数の OpenClaw を長期運用する中で得られた、非公式の運用ナレッジ、runbook、失敗パターン、安全境界を整理するためのプロジェクトです。

初心者向けのインストール手順ではありません。対象は、gateway、認証 profile、heartbeat、dashboard 負荷、systemd、SSH 越しの監視、Hermes / OH 連携、安全な自律修正、daily-log、review loop です。

## What This Covers

- 複数 agent の責務分離と相互監視
- `exit 0` と `SECURITY_OK` を分ける heartbeat 設計
- OAuth / auth profile / gateway restart の運用
- dashboard が自分で OpenClaw を重くする問題
- security agent の権限境界と wrapper 経由の修正
- Node / npm / nvm / systemd / wrapper のズレ
- Cloudflare Tunnel などを使った安全なアクセス経路
- 人間が読める通知文、daily-log、evidence の残し方

## Repository Layout

```text
docs/
  00-overview.md
  01-multi-agent-architecture.md
  02-heartbeat-and-safety.md
  03-auth-profile-and-gateway-restart.md
  04-dashboard-self-dos.md
  05-security-agent-boundaries.md
  06-node-wrapper-alignment.md
  07-safe-autonomous-remediation.md
  08-remote-access-tunnels.md
  09-publication-and-redaction.md
  10-maintenance-workflow.md
examples/
  notifications/
  state-formats/
daily/
  TEMPLATE.md
.github/
  ISSUE_TEMPLATE/
  pull_request_template.md
```

## Agent Names Used In Public Docs

公開版では、個人用の OpenClaw 名を使わず、役割ベースの名前にします。

| Public name | Role |
| --- | --- |
| `OpenClaw-Control` | 管制塔、生活運用、相互監視、dashboard |
| `OpenClaw-A` | domain-specific agent |
| `OpenClaw-Security` | security monitoring agent |
| `OpenClaw-Research` | OSS、調査、仕様変更追跡 |
| `Operator` | 人間の運用者 |

## Remote Access Policy

このプロジェクトでは、特定のネットワーク製品を必須にしません。Cloudflare Tunnel は dashboard や webhook の公開入口として扱いやすく、実運用で便利でした。一方で、Tailscale などの private mesh も用途によっては有効です。

重要なのは、管理用 port を直接公開しないこと、access layer を OpenClaw の問題と混ぜないこと、そして tunnel / mesh / reverse proxy の状態も監視対象に含めることです。

## Security And Redaction

公開版には以下を入れません。

- token、refresh token、API key、cookie、credential
- 実 IP、実ドメイン、個人名、実メールアドレス
- Discord / Telegram / LINE の channel ID
- 攻撃元 IP の生ログ
- credential の存在や場所を推測しやすいログ
- 個人用 host 名、private path、SSH alias

公開前チェックは [docs/09-publication-and-redaction.md](docs/09-publication-and-redaction.md) を使います。

## Update Style

このリポジトリは高頻度に更新される前提です。新しい発見は、まず daily note に短く残し、再利用できる知見になった時点で `docs/` や `examples/` に昇格します。

## Status

Living project. Docs-only for now. Executable scripts and private operational tooling are intentionally kept out of this repository.

## License

Documentation is licensed under CC BY 4.0. See [LICENSE.md](LICENSE.md).
