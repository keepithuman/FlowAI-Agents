---
name: citrix-clustering
description: How to set up NetScaler multi-box clustering, including why base configuration must be correct before adding nodes. Vendor-neutral. Use when building, reviewing, or debugging NetScaler clustering automation, on Itential or otherwise.
---

# Citrix NetScaler — Clustering

## When to use this skill

- Setting up multi-box clustering.
- Reviewing cluster configuration before adding a new node.

## Operational procedure

1. Establish the cluster instance and its config-sync approach.
2. Get the base configuration correct on one node first.
3. Add nodes — they inherit the cluster's existing configuration, which is why step 2 has to come first, not after nodes have already joined.
4. Confirm each new node actually inherited the intended configuration, via list, not assumed from the add-node call's success.

## Patterns

- **Base configuration before membership** — get one node's configuration correct before adding others, since new nodes inherit it.

## Known limitations

- No offensive/destructive capability.
- HA (active/passive) pairing is a distinct mechanism from clustering — see the `ha-pairing` skill; don't conflate the two.

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

## Verification checklist

- [ ] Base cluster configuration verified correct before any additional node joins
- [ ] After a node joins, confirmed it actually inherited the intended configuration, not a partial/stale sync
- [ ] Cluster instance's node membership reviewed via list, not assumed from individual add-node calls
