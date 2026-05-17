# 日本語概要

このディレクトリは、日本語の短い入口と要約を置く場所です。

正式な本体ドキュメントは [docs/en/](../en/) です。ここでは全文翻訳を維持しません。更新頻度が高いため、完全な日英同期よりも、発見を早く安全に残すことを優先します。

## 読む順番

1. [README.ja.md](../../README.ja.md) で全体像を見る
2. [docs/en/00-overview.md](../en/00-overview.md) で英語の本体に進む
3. 日々の運用メモは [daily/ja/](../../daily/ja/) に残す

作業中に出た知見をすぐ残す流れは [docs/en/11-work-as-you-go-knowledge-capture.md](../en/11-work-as-you-go-knowledge-capture.md) を参照してください。

## このリポジトリで扱うこと

- 複数 OpenClaw の責務分離
- heartbeat と安全判定の分離
- auth profile と gateway restart
- dashboard 自己DoS
- events.json の履歴エラーと未解決エラーの分離
- security agent の境界
- Node / wrapper / systemd のズレ
- Hermes / browser provider / Camofox adapter / Node runtime のズレ
- RustDesk / VNC / noVNC のような人間用 remote desktop と agent 用 browser provider の境界
- remote coding agent が host-control command を実行できる境界
- Hermes / review / learning note から coding agent に渡す proposal-only lab
- NotebookLM 向け source pack 作成と CLI upload / 人間GUI判断の分担
- 未解決相談を local ticket、risk、status、handoff、human-review に変換する duty desk ticket bridge
- 技術横断の相談役と browser 証拠、safe handoff
- Cloudflare Tunnel や private mesh を含む access layer
- 公開前の redaction

## 方針

英語を canonical docs にします。

日本語は、入口、要約、daily note を中心にします。安定した知見だけ、必要に応じて日本語でも短く整理します。

## 公開版で残すもの

公開版で価値があるのは、private script や実際の環境値ではなく、判断構造です。

- どの症状ならどの層を疑うか
- 何を evidence として見るか
- どこまで read-only で確認するか
- 何を変更する前に backup するか
- 何をもって復旧確認とするか
- 公開前に何を伏せるか

この形なら、実運用の知見を安全に残せます。
