---
name: citrix-clustering-ha
description: How to configure NetScaler clustering and high-availability (HA) node pairing. Vendor-neutral. Use when building, reviewing, or debugging NetScaler clustering/HA automation, on Itential or otherwise.
---

# Citrix NetScaler — Clustering & High Availability

## When to use this skill

- Setting up multi-box clustering.
- Setting up HA (active/passive) node pairing.
- Reviewing clustering/HA configuration before relying on it for failover.

## Operational procedure

**Clustering**: establish the cluster instance and its config-sync approach before adding nodes — nodes joining a cluster inherit the cluster's existing configuration, so get the base configuration correct on one node first, then add nodes, rather than configuring after nodes have already joined.

**HA pairing**: verify the HA heartbeat has an independent network path before pairing two nodes. Pairing nodes whose heartbeat traverses the same single link as production traffic risks exactly the split-brain or simultaneous-failure scenario HA exists to prevent, under precisely the conditions (a link failure) HA is meant to survive.

## Patterns

- **Base configuration before membership** — get one node's configuration correct before adding others to a cluster, since new nodes inherit it.

## Known limitations

- No offensive/destructive capability.
- These procedures cover configuration, not live incident execution. Forcing a failover during an actual outage is a higher-stakes action than a routine configuration change — treat this skill as setup guidance, not an incident-response runbook.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listCluster` | List cluster configurations | Clustering |
| `createCluster` | Create a new cluster instance | Clustering |
| `listClusterinstance` | List cluster instances | Clustering |
| `createClusterinstance` | Create a new cluster instance record | Clustering |
| `listClusternode` | List nodes in a cluster | Clustering |
| `createClusternode` | Add a new node to a cluster | Clustering |
| `createClusterinstanceClusternodeBinding` | Attach a node to a specific cluster instance | Clustering |
| `listHanode` | List HA (active/passive pair) nodes | HA Pairing |
| `createHanode` | Configure a new HA node pairing | HA Pairing |
| `updateHanode` | Change an existing HA node's configuration | HA Pairing |

## Verification checklist

- [ ] Base cluster configuration verified correct before any additional node joins
- [ ] HA heartbeat confirmed to traverse a network path independent of production traffic, not the same link
- [ ] After pairing, confirmed sync/failover state on both nodes, not just that the pairing call succeeded
