---
name: citrix-networking-fundamentals
description: How to configure NetScaler IP addressing, VLANs, static routes, interfaces, LACP channels, and NAT (INAT/RNAT). Vendor-neutral. Use when building, reviewing, or debugging core NetScaler networking automation, on Itential or otherwise.
---

# Citrix NetScaler — Networking Fundamentals

## When to use this skill

- Configuring IP addresses or VLANs.
- Configuring static routes or reviewing interface state.
- Configuring LACP link aggregation.
- Configuring inbound (INAT) or reverse (RNAT) NAT.

## Operational procedure

**IP addressing & VLANs**: NetScaler distinguishes IP types (management, VIP, subnet-mapping) that behave differently — assign the correct *type* for the purpose, not just any free address. Create the VLAN before binding interfaces or IPs to it.

**Routing & interfaces**: verify an interface's actual physical/link state before relying on it in a route. A route bound to a down interface doesn't fail at creation time — it fails silently until traffic actually needs that path.

**LACP channels**: the NetScaler-side channel configuration and the upstream switch's LACP configuration must agree (same member ports, same negotiation mode) — a channel configured only on the NetScaler side will never form an active bundle if the switch side doesn't match it.

**NAT**: inbound NAT (INAT) and reverse NAT (RNAT) solve different directional problems and are easy to conflate — decide the direction and scope deliberately, and test from the actual originating network segment, not just from the appliance's own local perspective.

## Patterns

- **Container before member** — create the VLAN/channel before binding interfaces or IPs into it.
- **Verify link-layer state before trusting a layer-3 configuration built on top of it** (routes on interfaces, channels on member ports).

## Known limitations

- No offensive/destructive capability.
- No cross-object blast-radius reasoning — a VLAN or routing change's full downstream impact (every vserver relying on that path) isn't traced automatically.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

### IP & VLAN

| Operation | Plain-English description | Category |
|---|---|---|
| `listNsip` | List IP addresses configured on the appliance | IP & VLAN |
| `createNsip` | Add a new IP address | IP & VLAN |
| `updateNsip` | Change an existing IP address's settings | IP & VLAN |
| `listVlan` | List VLANs | IP & VLAN |
| `createVlan` | Create a new VLAN | IP & VLAN |
| `updateVlan` | Change an existing VLAN's settings | IP & VLAN |
| `createVlanInterfaceBinding` | Bind a physical/logical interface to a VLAN | IP & VLAN |
| `createVlanNsipBinding` | Bind an IP address to a VLAN | IP & VLAN |

### Routing & Interfaces

| Operation | Plain-English description | Category |
|---|---|---|
| `listRoute` | List static routes | Routing |
| `createRoute` | Create a new static route | Routing |
| `updateRoute` | Change an existing route | Routing |
| `deleteRoute` | Remove a static route | Routing |
| `listInterface` | List physical/logical network interfaces | Interfaces |
| `updateInterface` | Change an interface's settings | Interfaces |
| `createInterfacepair` | Create an interface pair (used for certain HA/forwarding configurations) | Interfaces |

### LACP Channels

| Operation | Plain-English description | Category |
|---|---|---|
| `createChannel` | Create a new LACP link-aggregation channel | LACP Channel |
| `updateChannel` | Change an existing channel's settings | LACP Channel |
| `createChannelInterfaceBinding` | Add a physical interface into a channel | LACP Channel |

### NAT

| Operation | Plain-English description | Category |
|---|---|---|
| `listInat` | List inbound NAT (INAT) rules | NAT |
| `createInat` | Create a new inbound NAT rule | NAT |
| `updateInat` | Change an existing inbound NAT rule | NAT |
| `listRnat` | List reverse NAT (RNAT) rules | NAT |
| `createRnat` | Create a new reverse NAT rule | NAT |
| `updateRnat` | Change an existing reverse NAT rule | NAT |

## Verification checklist

- [ ] IP address type (management/VIP/subnet-mapping) confirmed correct for its intended purpose, not just "a free address"
- [ ] Interface physical/link state confirmed up before relying on a route bound to it
- [ ] LACP channel confirmed to have actually formed an active bundle (not just configured) by checking both NetScaler and switch-side state
- [ ] NAT rule tested from the actual originating network segment, not just the appliance's local perspective
