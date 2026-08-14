---
name: citrix-performance-profiles
description: How to configure NetScaler TCP and HTTP performance-tuning profiles. Vendor-neutral. Use when building, reviewing, or debugging NetScaler TCP/HTTP profile automation, on Itential or otherwise.
---

# Citrix NetScaler — Performance Profiles (TCP/HTTP)

## When to use this skill

- Tuning TCP or HTTP performance-related behavior for a vserver.
- Reviewing an existing performance profile before changing it.

## Operational procedure

1. Identify whether the tuning need is connection-level (TCP profile: window sizing, congestion control, timeouts) or protocol-level (HTTP profile: header limits, keep-alive, HTTP version handling).
2. Change one setting at a time.
3. Measure against real traffic before changing another setting — profile settings interact with the underlying network path in ways that are hard to predict from the setting names alone, and an overly aggressive change can degrade the exact metric it was meant to improve.

## Patterns

- **One variable at a time** — profile settings interact with the real network path in ways that aren't obvious from the setting name; changing several at once makes it hard to attribute an outcome to a specific change.

## Known limitations

- No offensive/destructive capability.
- Spillover (overflow protection) is a distinct concern from performance tuning — see the `spillover` skill.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `listNstcpprofile` | List TCP performance-tuning profiles | Performance Profiles |
| `createNstcpprofile` | Create a new TCP profile | Performance Profiles |
| `updateNstcpprofile` | Change an existing TCP profile | Performance Profiles |
| `listNshttpprofile` | List HTTP performance-tuning profiles | Performance Profiles |
| `createNshttpprofile` | Create a new HTTP profile | Performance Profiles |
| `updateNshttpprofile` | Change an existing HTTP profile | Performance Profiles |

## Verification checklist

- [ ] Changed one profile setting at a time and measured against real traffic before changing another
- [ ] Confirmed which vserver(s) actually use the profile before assuming a change has taken effect
- [ ] Real-world metric (latency, connection reuse, throughput) checked after the change, not just that the update call succeeded
