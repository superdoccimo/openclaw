# Node Wrapper Alignment

OpenClaw / Hermes / OH / dashboard は、Node、npm、nvm、systemd、wrapper の影響を強く受けます。

人間の shell では新しい Node を使っていても、systemd service や wrapper が古い binary を固定していることがあります。その結果、doctor、gateway、dashboard、heartbeat の表示が矛盾します。

## Common Failure Shape

```text
interactive shell:
  node -> /home/operator/.nvm/versions/node/vXX/bin/node

systemd service:
  node -> /usr/bin/node

wrapper:
  openclaw -> /opt/old-openclaw/bin/openclaw
```

この場合、更新したつもりでも実運用では古い CLI が起動します。

## Read-Only Checks

```bash
command -v node
node --version
command -v npm
npm --version
command -v openclaw
openclaw --version
systemctl --user cat openclaw-gateway.service
systemctl --user show openclaw-gateway.service -p Environment -p ExecStart
```

## What To Compare

- interactive shell の `node`
- systemd user service の `ExecStart`
- wrapper 内で固定された path
- `openclaw --version`
- `openclaw doctor`
- gateway journal
- dashboard process の actual command

## Recommended Practice

- wrapper を使うなら、どの Node / OpenClaw を呼ぶか明示する
- 更新後に wrapper と systemd の実体を read-only で確認する
- dashboard には `expected` と `actual` を分けて表示する
- mismatch は `WARNING` として扱う

## Example Assessment

```text
observed: ok
assessment: WARNING
reason: interactive Node and gateway Node differ
expected_node: /home/operator/.nvm/versions/node/vXX/bin/node
actual_gateway_node: /usr/bin/node
next_action: align wrapper and restart service after test
```
