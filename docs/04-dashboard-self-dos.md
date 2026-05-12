# Dashboard Self-DoS

dashboard は便利ですが、設計を間違えると OpenClaw を自分で重くします。

特に、短い interval で `openclaw status` や `channels status --deep` のような重い診断を繰り返すと、dashboard が監視対象を圧迫します。

## Failure Pattern

```text
dashboard polling every few seconds
  -> openclaw status
  -> channel deep checks
  -> remote SSH checks
  -> journal reads
  -> gateway timeout increases
  -> dashboard shows stale or wrong status
```

この状態では、dashboard があるから異常を見つけられる一方で、dashboard 自身が遅延や timeout の原因にもなります。

## Safer Pattern

- heavy check は interval を長くする
- API response を cache する
- stale 表示を明示する
- timeout を短くしすぎない
- dashboard は read-only を基本にする
- deep check は manual trigger または low frequency にする
- OpenClaw 本体と dashboard API の journal を分けて見る

## Dashboard Should Show Assessment, Not Only Execution

悪い表示:

```text
heartbeat: done
status: green
```

よい表示:

```text
heartbeat: done
observed: ok
assessment: SECURITY_WARNING
evidence_age: 8m
dashboard_cache: stale
```

## What To Monitor About Dashboard Itself

- service が active か
- port が想定通りか
- API response が timeout していないか
- cache が古すぎないか
- auth が有効か
- dashboard check が OpenClaw CLI を過剰に呼んでいないか

dashboard は表示装置であると同時に、運用対象でもあります。
