---
name: citrix-ha-pairing
description: How to configure NetScaler active/passive HA node pairing, including the independent-heartbeat-path requirement. Vendor-neutral. Use when building, reviewing, or debugging NetScaler HA pairing automation, on Itential or otherwise.
---

# Citrix NetScaler — HA Pairing

## When to use this skill

- Setting up HA (active/passive) node pairing.
- Reviewing HA configuration before relying on it for failover.

## Operational procedure

1. Verify the HA heartbeat has an independent network path, separate from production traffic — pairing nodes whose heartbeat traverses the same single link risks exactly the split-brain or simultaneous-failure scenario HA exists to prevent.
2. Pair the two nodes.
3. Confirm sync/failover state on both nodes, not just that the pairing call succeeded.
4. Test a real failover in a maintenance window before trusting HA for an actual incident.

## Patterns

- **Heartbeat path independence is load-bearing** — HA configured on top of a shared link with production traffic defeats its own purpose under the exact failure mode it exists to survive.

## Known limitations

- No offensive/destructive capability.
- These procedures cover configuration, not live incident execution. Forcing a failover during an actual outage is a higher-stakes action than a routine configuration change — treat this skill as setup guidance, not an incident-response runbook.
- Multi-box clustering is a distinct mechanism from HA pairing — see the `clustering` skill; don't conflate the two.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listHanode` | List HA (active/passive pair) nodes | HA Pairing |
| `createHanode` | Configure a new HA node pairing | HA Pairing |
| `updateHanode` | Change an existing HA node's configuration | HA Pairing |

## Verification checklist

- [ ] HA heartbeat confirmed to traverse a network path independent of production traffic, not the same link
- [ ] After pairing, confirmed sync/failover state on both nodes, not just that the pairing call succeeded
- [ ] A real failover tested in a maintenance window before trusting HA for an actual incident
