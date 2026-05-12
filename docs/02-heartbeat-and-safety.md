# Heartbeat And Safety

heartbeat は「実行されたか」を示すだけでは足りません。

`exit 0` は、チェック処理が最後まで走ったことを意味します。環境が安全であることや、実運用の応答経路が正常であることまでは保証しません。

## Separate Execution From Assessment

悪い例:

```text
heartbeat finished
status: ok
```

よい例:

```text
observed: ok
assessment: SECURITY_WARNING
reason: gateway reachable, but auth profile has expired fallback
next_action: repair auth order and restart gateway
```

## Suggested State Model

| Field | Meaning |
| --- | --- |
| `observed` | チェックが観測できたか |
| `assessment` | 安全性や運用上の判断 |
| `evidence` | 何を見て判断したか |
| `next_action` | 人間または agent が次にやること |
| `last_checked_at` | 最終確認時刻 |

`observed: ok` と `assessment: SECURITY_OK` は別物です。

## Recommended Assessment Values

- `SECURITY_OK`
- `SECURITY_WARNING`
- `SECURITY_UNKNOWN`
- `CHECK_FAILED`
- `STALE`

## What Heartbeat Should Check

- OpenClaw CLI が expected path から起動しているか
- gateway service が active か
- auth profile が実運用 profile を優先しているか
- channel の送受信経路が生きているか
- dashboard API が重すぎないか
- tunnel / reverse proxy / private mesh が到達可能か
- recent journal に timeout や credential error がないか

## What Heartbeat Should Avoid

- deep status を短間隔で連打する
- token や credential の値を出す
- 修正まで自動実行して evidence を残さない
- `exit 0` だけで dashboard を green にする

## Notification Format

通知は短く、人間が判断できる形にします。

```text
[OpenClaw-Control/auth] 認証確認
- OpenClaw-Control: OpenClaw openai-codex production profile ok
- OpenClaw-A: OpenClaw openai-codex OAuth expired
- OpenClaw-Research: Hermes openai-codex OAuth exhausted
- OpenClaw-Security: ok
assessment: SECURITY_WARNING
next: repair auth order, then restart gateway
```
