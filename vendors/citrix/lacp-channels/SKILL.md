---
name: citrix-lacp-channels
description: How to configure NetScaler LACP link-aggregation channels, including the requirement that the upstream switch configuration must agree. Vendor-neutral. Use when building, reviewing, or debugging NetScaler LACP automation, on Itential or otherwise.
---

# Citrix NetScaler — LACP Channels

## When to use this skill

- Configuring LACP link aggregation.
- Debugging a channel that's configured but never forms an active bundle.

## Operational procedure

The NetScaler-side channel configuration and the upstream switch's LACP configuration must agree (same member ports, same negotiation mode) — a channel configured only on the NetScaler side will never form an active bundle if the switch side doesn't match it.

## Patterns

- **Container before member** — create the channel before adding interfaces into it.
- **Two-sided agreement is mandatory** — a NetScaler-side-only LACP configuration is inert; always confirm the upstream switch side matches.

## Known limitations

- No offensive/destructive capability.
- This skill only covers the NetScaler-side configuration — confirming the switch-side LACP configuration requires that vendor's own tooling, outside this skill's scope.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `createChannel` | Create a new LACP link-aggregation channel | LACP Channel |
| `updateChannel` | Change an existing channel's settings | LACP Channel |
| `createChannelInterfaceBinding` | Add a physical interface into a channel | LACP Channel |

## Verification checklist

- [ ] Channel confirmed to have actually formed an active bundle (not just configured) by checking NetScaler-side state
- [ ] Upstream switch-side LACP configuration confirmed to match (same member ports, same negotiation mode) via that vendor's own tooling
- [ ] Traffic tested across the channel to confirm real throughput/failover behavior, not just link state
