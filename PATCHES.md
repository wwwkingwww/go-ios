# oss27 fork patches (branch: oss27-multi-tun)

This branch is `danielpaulus/go-ios` at commit `77873fe353fc` (the commit pinned by
avatarclaw's oss27 iOS execution agent) plus the patches listed below. The fork's
`main` branch stays a mirror of upstream; keep every patch on a named branch on top
of the exact upstream commit we depend on.

## Patches

### 1. Per-instance Windows tunnel adapter name (`ios/tunnel/tunnel_windows.go`)

- Upstream hardcodes the wintun adapter name `tun0`, and then runs
  `netsh interface ipv6 set address <name> ...`. Two tunnel agents on one Windows
  host therefore fight over the same adapter name/address (field-verified collision:
  a second agent ends up operating on `tun0 1` while `netsh` targets `tun0`).
- Patch: read `GO_IOS_TUN_NAME` (default stays `tun0` when unset/empty). The agent
  launcher assigns a unique name per instance (e.g. `oss27tun3`).

## Maintenance

- Upstream sync: rebase this branch onto the next upstream commit we pin, resolve,
  push, then re-pin avatarclaw's `tools/oss27-ios-agent/go.mod` `replace` directive
  to the new branch commit (pseudo-version) and rebuild.
- License: upstream MIT (`LICENSE` untouched); keep this fork public.
