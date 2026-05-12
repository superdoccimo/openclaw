# Auth Profile And Gateway Restart

OpenClaw の再認証は、login コマンドだけで終わりではありません。

複数 profile が残る環境では、有効な profile があるのに実運用経路だけ timeout することがあります。古い default profile が残り、gateway が古い認証状態を握ったまま動くためです。

## Common Failure Shape

```text
openclaw models auth list
  openai-codex:<account>  valid
  openai-codex:default    expired

gateway log
  Token refresh failed: refresh_token_reused
  Profile openai-codex:<redacted> timed out. Trying next account...
```

この状態では、auth list だけ見ると valid profile があるため正常に見えます。しかし Discord / Telegram / LINE 経由の実応答では gateway が古い状態を使って失敗することがあります。

## Read-Only Checks

```bash
openclaw doctor
openclaw status
openclaw models auth list
openclaw models auth order list --provider openai-codex
systemctl --user status openclaw-gateway.service --no-pager
journalctl --user -u openclaw-gateway.service -n 100 --no-pager
```

## Repair Pattern

実運用 profile を確認してから、優先順を明示します。

```bash
openclaw models auth order set --provider openai-codex openai-codex:<account>
systemctl --user restart openclaw-gateway.service
```

その後、次を確認します。

```bash
openclaw models auth order list --provider openai-codex
systemctl --user status openclaw-gateway.service --no-pager
journalctl --user -u openclaw-gateway.service -n 100 --no-pager
```

## Important Points

- `login` 成功と実運用経路の復旧は別
- valid profile があることと、優先順が正しいことは別
- auth order を直しただけでは gateway が古い状態を持ち続ける場合がある
- restart 後に channel 経由の応答まで確認する
- token や refresh token の値は絶対に通知や log に出さない

## Update Agent Risk

相互更新構成では、更新する側の profile も監視対象です。

たとえば `OpenClaw-A` が `OpenClaw-Control` を更新する場合、`OpenClaw-A` 側の auth order や gateway state が壊れていると、修正作業自体が不安定になります。
