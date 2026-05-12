# Remote Access Tunnels

Remote access is part of the operational surface for dashboards, webhooks, review UIs, and local apps.

Cloudflare Tunnel is a practical option because it can expose dashboard or webhook entrypoints without opening direct inbound ports. Tailscale and other private mesh tools can also work well for operator-only access.

This project treats access as an operational layer, not as a single required product.

## Design Goal

- do not expose management ports directly to the internet
- monitor tunnel / mesh / reverse proxy state
- separate OpenClaw failures from access-layer failures
- make dashboard exposure explicit
- do not place tokens or tunnel credentials in process arguments or logs

## Cloudflare Tunnel Pattern

Useful for:

- limited dashboard exposure
- stable webhook entrypoints
- avoiding extra inbound firewall rules
- publishing apps behind a reverse proxy

Watch for:

- do not use patterns like `cloudflared tunnel run --token ...` if the token appears in argv
- do not print token, credential, or tunnel ID values
- separate tunnel health from upstream app health
- a healthy tunnel can still point to an upstream 404 or 500
- dashboard auth still matters

## Tailscale Or Private Mesh Pattern

Useful for:

- operator-only private access
- host-to-host peer checks
- SSH and internal dashboards that should not be public
- device identity and ACL-based access

Watch for:

- healthy mesh access does not prove the app service is healthy
- ACL changes should be runbooked
- agents should not receive broader network visibility than they need

## Health Check Layers

```text
user/browser
  -> tunnel or mesh
  -> reverse proxy
  -> dashboard/app
  -> OpenClaw gateway
  -> channel/provider
```

Observe and report the failing layer directly.

## Minimum Dashboard Fields

- access method: `cloudflare-tunnel` / `private-mesh` / `local-only`
- tunnel status
- upstream app status
- auth requirement
- last successful request time
- last error class
- credential exposure check

## Recommendation

Use Cloudflare Tunnel as a convenient default for public entrypoints. Use private mesh tools for operator-only access and peer checks when they fit better.

In either case, do not collapse access-layer failures into "OpenClaw is broken." Record access layer and application layer separately.
