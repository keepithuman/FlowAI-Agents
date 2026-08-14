---
name: citrix-nat
description: How to configure NetScaler inbound (INAT) and reverse (RNAT) NAT rules. Vendor-neutral. Use when building, reviewing, or debugging NetScaler NAT automation, on Itential or otherwise.
---

# Citrix NetScaler — NAT (INAT/RNAT)

## When to use this skill

- Configuring inbound (INAT) or reverse (RNAT) NAT.
- Debugging NAT that isn't translating traffic as expected.

## Operational procedure

1. Decide the direction deliberately — inbound NAT (INAT) or reverse NAT (RNAT). They solve different directional problems and are easy to conflate.
2. Create the rule with the correct direction and scope.
3. Test from the actual originating network segment, not just from the appliance's own local perspective.

## Patterns

- **Direction is the first decision, not an afterthought** — INAT and RNAT solve opposite problems; confirm which one the request actually needs before configuring either.
- **Test from the real originating segment** — the appliance's own local perspective doesn't reveal what a real client on the actual network sees.

## Known limitations

- No offensive/destructive capability.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `listInat` | List inbound NAT (INAT) rules | NAT |
| `createInat` | Create a new inbound NAT rule | NAT |
| `updateInat` | Change an existing inbound NAT rule | NAT |
| `listRnat` | List reverse NAT (RNAT) rules | NAT |
| `createRnat` | Create a new reverse NAT rule | NAT |
| `updateRnat` | Change an existing reverse NAT rule | NAT |

## Verification checklist

- [ ] Direction (INAT vs RNAT) confirmed correct for the actual problem before configuring
- [ ] NAT rule tested from the actual originating network segment, not just the appliance's local perspective
- [ ] Confirmed the translated address/port matches what the destination system actually expects
