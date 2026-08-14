---
name: citrix-routing-interfaces
description: How to configure NetScaler static routes and review interface state, including the link-state-before-routing dependency. Vendor-neutral. Use when building, reviewing, or debugging NetScaler routing/interface automation, on Itential or otherwise.
---

# Citrix NetScaler — Routing & Interfaces

## When to use this skill

- Configuring static routes or reviewing interface state.
- Debugging traffic that silently fails to reach its destination.

## Operational procedure

1. Check the interface's actual physical/link state first.
2. Only then create or update a route that relies on that interface — a route bound to a down interface doesn't fail at creation time, it fails silently until traffic actually needs that path.
3. Test the route with real traffic from the actual originating segment, not just a create-success check.

## Patterns

- **Verify link-layer state before trusting a layer-3 configuration built on top of it.** A route's create-success doesn't validate the interface underneath it is actually up.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover LACP channel state specifically — see the `lacp-channels` skill for member-port/channel-formation checks.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listRoute` | List static routes | Routing |
| `createRoute` | Create a new static route | Routing |
| `updateRoute` | Change an existing route | Routing |
| `deleteRoute` | Remove a static route | Routing |
| `listInterface` | List physical/logical network interfaces | Interfaces |
| `updateInterface` | Change an interface's settings | Interfaces |
| `createInterfacepair` | Create an interface pair (used for certain HA/forwarding configurations) | Interfaces |

## Verification checklist

- [ ] Interface physical/link state confirmed up before relying on a route bound to it
- [ ] Route tested with real traffic from the actual originating segment, not assumed correct from create-success alone
- [ ] Interface pair (if created) confirmed to actually be in use by the HA/forwarding configuration it was intended for
