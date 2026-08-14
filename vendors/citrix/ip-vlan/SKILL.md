---
name: citrix-ip-vlan
description: How to configure NetScaler IP addresses and VLANs, including the different IP types NetScaler distinguishes. Vendor-neutral. Use when building, reviewing, or debugging NetScaler IP/VLAN automation, on Itential or otherwise.
---

# Citrix NetScaler — IP Addressing & VLANs

## When to use this skill

- Configuring IP addresses or VLANs.
- Debugging an IP/VLAN misconfiguration.

## Operational procedure

NetScaler distinguishes IP types (management, VIP, subnet-mapping) that behave differently — assign the correct *type* for the purpose, not just any free address. Create the VLAN before binding interfaces or IPs to it.

## Patterns

- **Container before member** — create the VLAN before binding interfaces or IPs into it.
- **IP type is a functional choice, not just an address** — management/VIP/subnet-mapping behave differently; picking the wrong type for the purpose causes subtle failures, not obvious errors.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover routing or interface link-state — see the `routing-interfaces` skill for what happens on top of an IP/VLAN once it exists.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

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

## Verification checklist

- [ ] IP address type (management/VIP/subnet-mapping) confirmed correct for its intended purpose, not just "a free address"
- [ ] VLAN confirmed created before any interface/IP binding was attempted
- [ ] After binding, confirmed via list that the interface/IP is actually associated with the intended VLAN
