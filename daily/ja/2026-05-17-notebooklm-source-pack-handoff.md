# 2026-05-17

## 概要

- NotebookLM は GUI を全部自動操作する対象ではなく、source pack を作って CLI で投入する対象として扱う方がよい。
- ChatGPT のような厳格な site は、別 profile の自動 browser だと human verification で止まりやすい。
- 既に人間が開いている入力欄への paste 補助は使えるが、NotebookLM の source 追加や生成 artifact 操作は多段 GUI なので別に考える。

## 観測したこと

- 単純な page なら browser bridge で open / click / fill / screenshot が成立する。
- 厳格な site では、新規の自動操作 profile が human verification に止まりやすい。
- 既に人間が開いている通常 browser に対しては、clipboard + paste の形で prompt 投入を補助できる。
- NotebookLM は prompt 入力だけでなく、source 追加、file 選択、URL 追加、Drive 連携、artifact 生成 button、生成結果確認があるため、完全自動 GUI 操作は brittle。
- `notebooklm-tui` のような CLI が使えるなら、source 追加や backup / restore を terminal 側へ逃がせる。

## 誤解しやすかったこと

- 「browser が開ける」ことと「NotebookLM を最後まで操作できる」ことは同じではない。
- 「ログイン済み page が人間には見えている」ことと「agent の browser provider が同じ session を掴める」ことも同じではない。
- `ydotool type` のような仮想 keyboard 入力は、短文でも layout や space の扱いで崩れることがある。
- 長文 prompt は type ではなく clipboard + paste の方が安定しやすい。
- NotebookLM の CLI は便利だが、公式安定 API ではなく内部 API adapter として扱う必要がある。

## 使える分担

OpenClaw agent がやること:

```text
public repo / video / article / sanitized run log を集める
source-pack/ を作る
manifest を書く
NotebookLM CLI で source を upload する
notebook id と投入結果を記録する
次に作るべき artifact を提案する
```

Operator がやること:

```text
NotebookLM の login / auth refresh
生成 artifact の button を押す
生成結果を見て採用判断する
公開前の最終確認をする
```

この分担なら、agent は価値の高い準備作業を担い、Operator は壊れやすい GUI と判断部分を担当できる。

## 健康観察

NotebookLM 側の仕様変更に備えて、Media Factory 実行前に軽く見る。

```text
auth check
notebook list
small Markdown upload
upload result / error classification
```

エラーは次のように分類する。

```text
auth:
  cookie missing / expired

csrf:
  top page token extraction failed

rpc:
  internal API method or payload changed

upload:
  file type / size / notebook id / source limit

network:
  local connectivity / tunnel / DNS
```

auth は人間へ再ログインをお願いする。csrf / rpc は tool 側の仕様追従メモを残す。upload だけなら source pack の形式や対象 notebook を見る。

## 公開 docs へ昇格

- `docs/en/20-notebooklm-source-pack-handoff.md`
