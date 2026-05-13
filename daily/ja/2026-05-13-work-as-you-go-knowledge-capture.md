# 2026-05-13

## 概要

- 複数 OpenClaw 運用中の発見を、作業しながら public docs に育てる方針を整理した。
- docs-only でも、判断構造、失敗層、復旧確認、redaction 境界を残せば再利用価値が高い。
- chat 通知は成果物ではなくポインタ。成果物は research note、backlog、state、daily note に残る。

## 観測したこと

- ある research agent では、heartbeat は動いているように見えたが、自律的に調査・記録する動きが弱かった。
- working peer と比較すると、heartbeat prompt、light context、isolated heartbeat session、busy 時 skip などの差が見えた。
- 運用ナレッジは private な実値ではなく、層分けと判断構造として公開 repo に残せる。
- 整備後、research agent が初めて有用な chat 通知を出した。公開 GitHub repo と論文を参照し、coding agent の手順ギャップを checklist 化する小さな OSS 案として整理していた。
- 同じ発見が research note、idea backlog、state file にも残っていたため、次回以降の heartbeat や review が継続できる状態になっていた。

## 誤解しやすかったこと

- `heartbeat OK` は「自律的に十分考えた」ことを意味しない。
- channel が OK でも、agent の prompt や session 設計が弱いと、実質的には短い ACK だけで終わることがある。
- docs-only repo は軽く見えがちだが、実運用では private script より判断構造の方が再利用しやすいことがある。

## 実際に壊れていた層

- channel や gateway の完全停止ではなく、heartbeat の自律指示と session 設計の層。
- 知見が散らばっており、作業中に daily note と stable docs へ落とす流れがまだ弱かった。

## sanitized evidence

- working peer では heartbeat 専用 session があり、heartbeat 実行に十分な時間がかかっていた。
- 対象 agent では heartbeat が短時間で silent OK になり、専用 session が見えていなかった。
- 初回の有用通知では、対象、内容、案、保存先メモ、公開参考リンクが短く揃っていた。
- 実 host 名、private IP、channel ID、token、raw log は載せない。

## 次に見る場所

- `docs/en/11-work-as-you-go-knowledge-capture.md`
- `docs/en/10-maintenance-workflow.md`
- `docs/en/02-heartbeat-and-safety.md`
- `docs/en/09-publication-and-redaction.md`

## 昇格候補

- [x] docs/en
- [ ] examples
