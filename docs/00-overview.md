# Overview

このプロジェクトは、複数 OpenClaw 運用の実戦知見を公開用に整理するものです。

OpenClaw は単体で入れるだけなら比較的扱いやすくても、長期運用では問題の層が増えます。OpenClaw 本体、gateway、channel、OAuth、Hermes、OH、systemd、Node、dashboard、remote access、security monitoring が同時に動くためです。

「OpenClaw が返事しない」という見え方でも、原因は次のどれかかもしれません。

- gateway が古い認証状態を握っている
- auth profile の優先順が実運用と合っていない
- Discord / Telegram / LINE 側の権限や履歴取得が壊れている
- systemd service の PATH と人間の shell の PATH が違う
- dashboard が重い診断を連打している
- wrapper が古い Node / OpenClaw CLI を固定している
- tunnel / reverse proxy / private mesh が落ちている
- heartbeat は動いたが、安全確認まではできていない

## Design Position

このリポジトリは、OpenClaw を cron の代替として扱いません。

重要なのは、スクリプトの exit code を鵜呑みにせず、doctor、status、auth list、journal、dashboard、remote check、daily-log を組み合わせて、運用状態を判断することです。

## Target Reader

- OpenClaw を複数台、または複数 role で運用している人
- agent 同士の相互監視や更新補助を考えている人
- dashboard や heartbeat を実運用に入れたい人
- OAuth / gateway / systemd / Node のズレで壊れた経験がある人
- agent に安全な範囲で自律修正させたい人

## Non Goals

- OpenClaw の初心者向け installation guide
- 特定環境の完全な再現手順
- token、IP、domain、credential を含む実ログ集
- 攻撃手順や exploit の詳細化
- 実行可能な運用スクリプトの配布

## Knowledge Flow

```text
daily note
  -> incident pattern
  -> reusable runbook
  -> example schema or tool
  -> stable docs
```

発見は最初からきれいに分類しなくてよいです。まず daily note に残し、同じ型の問題が再発したら `docs/` に昇格します。

## Docs-Only Policy

このリポジトリは、当面は文字だけの公開ナレッジとして運用します。

実運用で使う script、wrapper、private tool は含めません。必要な場合は、何を確認するべきか、どの境界で自律修正を許すべきか、どの結果を dashboard や daily-log に残すべきかを Markdown で説明します。
