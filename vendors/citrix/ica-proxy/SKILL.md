---
name: citrix-ica-proxy
description: How to configure NetScaler ICA proxy for access to Citrix Virtual Apps/Desktops through Gateway. Vendor-neutral. Use when building, reviewing, or debugging NetScaler ICA-proxy automation, on Itential or otherwise.
---

# Citrix NetScaler — ICA Proxy

## When to use this skill

- Setting up ICA proxy access to Citrix Virtual Apps/Desktops through Gateway.
- Debugging an ICA proxy failure after a successful Gateway login.

## Operational procedure

1. Confirm the Gateway vserver already exists (see the `ssl-vpn` skill).
2. Create the ICA action pointing at the actual StoreFront/broker address.
3. Create the ICA policy.
4. Bind the policy to the Gateway vserver.
5. Verify with a real client launch, not just a successful API bind — most ICA proxy failures happen on the StoreFront handshake side, which is invisible from the NetScaler configuration alone.

## Patterns

- **Stage-then-attach** — confirm the StoreFront/broker address is valid and reachable on its own before wiring it into a live Gateway vserver.

## Known limitations

- No offensive/destructive capability.
- ICA proxy verification requires an actual client launch to be conclusive — a successful API bind alone does not confirm StoreFront/broker connectivity.
- Depends on a Gateway vserver already existing — see the `ssl-vpn` skill for that setup.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listIcapolicy` | List ICA policies (govern access to Citrix Virtual Apps/Desktops via Gateway) | ICA Proxy |
| `createIcapolicy` | Create a new ICA policy | ICA Proxy |
| `updateIcapolicy` | Change an existing ICA policy | ICA Proxy |
| `createIcaaction` | Create an ICA action (points at the StoreFront/broker address) | ICA Proxy |
| `updateIcaaction` | Change an existing ICA action | ICA Proxy |
| `createIcapolicyVpnvserverBinding` | Bind an ICA policy to a Gateway vserver | ICA Proxy |
| `listIcapolicyVpnvserverBinding` | List ICA policies bound to a Gateway vserver | ICA Proxy |

## Verification checklist

- [ ] StoreFront/broker address confirmed reachable independently before wiring into the ICA action
- [ ] ICA proxy verified with a real client launch, not just the API bind response
- [ ] Confirmed the Gateway vserver's authentication succeeds before troubleshooting ICA proxy specifically
