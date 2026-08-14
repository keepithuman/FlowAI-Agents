---
name: vmware-cluster-configuration-compliance
description: How to use VMware vSphere's draft/apply model for cluster desired-state configuration (DRS/HA) and compliance checking. Vendor-neutral. Use when building, reviewing, or debugging vSphere cluster desired-state automation, on Itential or otherwise.
---

# VMware vSphere — Cluster Configuration & Compliance (DRS/HA Desired State)

## When to use this skill

- Changing cluster desired-state configuration (DRS/HA).
- Checking configuration-drift compliance.

## Operational procedure

The draft → apply model is deliberate: a draft captures a proposed desired-state configuration without touching the live cluster, and only `apply` actually pushes it. Always create and review a draft before ever applying, even for a change that seems obviously safe.

A compliance check reports drift between actual and declared configuration — it's a detection tool, not a remediation tool. Finding non-compliance doesn't fix it; you still have to decide whether to apply a corrected draft or accept the drift.

Applying a cluster configuration change affects every host and VM in that cluster simultaneously (DRS/HA behavior is cluster-wide by definition) — there's no way to stage this to "just one host first."

## Patterns

- **Draft/propose-then-apply is the vSphere-native version of a human-approval gate.** Where the platform already models this two-step flow, that IS the approval mechanism — don't build a second one on top of it.
- **Compliance checks detect, they don't fix** — a non-compliance finding always requires a separate, explicit decision about whether to correct it.

## Known limitations

- No way to stage a cluster configuration change to a subset of hosts — apply is cluster-wide by definition; there's no partial-rollout option at this layer.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Esx.Settings.Clusters.Configuration_get` | Get a cluster's current desired-state configuration | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration_checkCompliance$Task` | Check whether a cluster's actual configuration matches its declared desired state | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration.Drafts_create` | Create a draft of a proposed configuration change (doesn't touch the live cluster) | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration.Drafts_get` | Get a draft's contents | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration.Drafts_list` | List pending configuration drafts | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration_apply$Task` | Apply a draft's configuration to the live cluster (affects every host/VM in it) | Cluster Configuration |

## Verification checklist

- [ ] Draft content reviewed and confirmed to match what was actually approved before calling apply
- [ ] Compliance re-checked after applying, not just before
- [ ] Everyone affected by the change is aware it's cluster-wide (every host/VM in the cluster), not scoped to one host
