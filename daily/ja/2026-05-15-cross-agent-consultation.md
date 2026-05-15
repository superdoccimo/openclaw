# 2026-05-15

## 概要

- 複数 OpenClaw が自律し始めると、個別 agent の能力よりも「誰に相談するか」が問題になる。
- domain、security、maintenance、research の責務が分かれていても、技術的な障害は層をまたぐ。
- そのため、公開版では `OpenClaw-Consult` という役割名で、横断相談・証拠整理・安全境界・coding agent への handoff を整理する。

## 観測したこと

- domain agent は自分の専門判断には強いが、gateway、auth、systemd、Node wrapper、dashboard、SSH、browser state まで一人で抱えると重い。
- security agent は異常検知に強いが、すべての技術運用を security 判断に吸い込むと役割が広がりすぎる。
- maintenance agent は更新や peer health に強いが、domain 判断や browser 状態の解釈までは本来の責務ではない。
- research agent は upstream や OSS の変化を追えるが、本番操作の判断者にはしない方がよい。
- 人間がいても技術的には困難なことがあり、人間が不在ならなおさら相談先が必要になる。

## 誤解しやすかったこと

- 相談役は上司ではない。
- 相談役を作ることは、他の agent の専門性を否定することではない。
- すべてを相談役に通すと autonomy が落ちる。
- 毎回許可を求める運用は、能力のある agent にとって負担になる。
- 逆に、危険な操作を相談なしで進めると安全境界が壊れる。

## 実際に大事だった層

- 技術的な曖昧さを受ける層。
- browser / GUI 証拠を扱う層。
- `executionStatus` と `safetyStatus` を分ける層。
- 成功した時は短く成功報告だけで済ませる層。
- blocked / warning / critical / human-review の時だけ、人間が判断しやすい形へまとめる層。
- vague な相談を coding agent が実行しやすい依頼文へ変換する層。

## sanitized pattern

```text
source agent:
target:
goal:
observed evidence:
what was already tried:
risk level:
do not do:
desired output:
```

解決できた場合:

```text
consult result: fixed
target:
what changed:
verification:
```

人間判断が必要な場合:

```text
consult result: human-review
target:
evidence:
risk:
safe next step:
human decision needed:
```

## 公開 docs へ昇格

- `docs/en/16-cross-agent-consultation.md`

## 次に見る場所

- `docs/en/01-multi-agent-architecture.md`
- `docs/en/02-heartbeat-and-safety.md`
- `docs/en/07-safe-autonomous-remediation.md`
- `docs/en/12-autonomy-enablement.md`
- `docs/en/16-cross-agent-consultation.md`
