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

### 2. Raw attribute query + full element dict passthrough (`ios/accessibility/accessibility_control.go`)

- Purpose: protocol reconnaissance for on-screen rect retrieval by element
  handle (`deviceElement:valueForAttribute:`). Adds:
  - `QueryAttributeValueRaw` — returns the raw response value, unwrapping
    archived objects (`[]byte` via `nskeyedarchiver`) instead of dropping
    non-string values on the floor; `QueryAttributeValue` (string) now
    delegates to it with identical behavior for existing callers.
  - `QueryElementRect` — probes candidate attribute names, feeds each raw
    value through `tryExtractRect` (ax_rect.go).
  - `AXElementData.Attributes` — the full element dict from Move events
    (upstream kept only SpokenDescription + handle); `AXElementData.Rect()`
    extracts a frame from the known `*Value_v1` key family if present.
- **Negative result, verified on device (iOS 27.0.1, 2026-10-21): this
  protocol has NO rect attribute.** The Move-event dict carries only four
  keys (SpokenDescriptionValue_v1 / ElementValue_v1 = handle / 
  CaptionTextValue_v1 / InspectorSectionsValue_v1 = attribute *metadata*),
  and the valueForAttribute attribute universe is exactly
  `Identifier / Label / Traits / TraitsHumanReadable / Value` — no frame.
  Device-native rects exist only in the audit stream, attached to
  accessibility-*issue* elements (a small subset), not the full tree.
- Status: compiled + device-verified; **not consumed** by avatarclaw's
  agent, go.mod not re-pinned (no consumer yet). Kept so the protocol
  question stays answered and the tooling is one re-pin away.
  Context: avatarclaw repo `docs/superpowers/plans/2026-10-21-oss27-smart-precision.md`.

## Maintenance

- Upstream sync: rebase this branch onto the next upstream commit we pin, resolve,
  push, then re-pin avatarclaw's `tools/oss27-ios-agent/go.mod` `replace` directive
  to the new branch commit (pseudo-version) and rebuild.
- License: upstream MIT (`LICENSE` untouched); keep this fork public.
