# openclaw 日本語概要

このリポジトリは、複数の OpenClaw を長期運用するための非公式フィールドノートです。

OpenClaw 本体のインストール手順ではありません。gateway、heartbeat、auth profile、dashboard、systemd、SSH 監視、Hermes / OH 連携、安全な自律修正、daily-log など、実運用で壊れやすい部分を整理します。

正式な本体ドキュメントは英語です。

- 英語の入口: [README.md](README.md)
- 英語の安定docs: [docs/en/](docs/en/)
- 日本語の概要: [docs/ja/](docs/ja/)
- 日々の日本語メモ: [daily/ja/](daily/ja/)

## 公式 docs との関係

OpenClaw 本体の仕様、install、command syntax、正式な config は公式 docs を正とします。

このリポジトリは、公式 docs の代替ではありません。
複数 OpenClaw を実際に動かした時に起きる、auth、gateway、dashboard、heartbeat、systemd、Node wrapper、remote access、redaction などの切り分け知見を残す場所です。

公開版で残すべきなのは、private script や実値ではなく、判断構造です。
「どの層を疑うか」「何を evidence とするか」「どこから rollback できるか」「何を公開してはいけないか」を残すだけでも十分価値があります。

## 方針

完全な日英二重管理はしません。

英語を canonical docs とし、日本語は入口、要約、日々の運用メモを中心にします。安定した知見だけ、必要に応じて日本語側にも要約します。

## 公開名

個人用の OpenClaw 名は公開版では使いません。

| 公開名 | 役割 |
| --- | --- |
| `OpenClaw-Control` | 管制塔、生活運用、相互監視、dashboard |
| `OpenClaw-A` | domain-specific agent |
| `OpenClaw-Security` | security monitoring agent |
| `OpenClaw-Research` | OSS 調査、仕様変更追跡 |
| `Operator` | 人間の運用者 |
