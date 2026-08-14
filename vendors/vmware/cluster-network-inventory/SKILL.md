---
name: vmware-cluster-network-inventory
description: How to look up VMware vSphere cluster and network/port-group inventory. Vendor-neutral, read-only. Use when you need to know what clusters or networks exist in a vSphere environment, on Itential or otherwise.
---

# VMware vSphere — Cluster & Network Inventory

## When to use this skill

- Looking up what clusters exist in a vCenter environment.
- Looking up what networks/port groups exist.

## Operational procedure

1. Confirm the request is a lookup ("what exists"), not an expectation of mutation — cluster formation happens through host aggregation and networks are usually defined at the host/vSwitch layer, neither of which this REST surface exposes directly for mutation.
2. List clusters or networks as needed.
3. For a genuine configuration change (DRS/HA desired state, vSwitch/port-group), redirect to the `cluster-configuration-compliance` skill or the vendor's other management surface — this API doesn't expose those for mutation.

## Patterns

- **Lookup, not mutation** — this skill answers "what exists," not "change this." Cluster formation and vSwitch/port-group configuration require a different management surface entirely.

## Known limitations

- No standard distributed-switch/port-group management exists on this API surface at all — the only switch-adjacent operations are scoped to Kubernetes/Tanzu supervisor networking (`Vcenter.NamespaceManagement.Networks.Nsx.*`), not general-purpose vSphere networking.
- Cluster desired-state configuration (DRS/HA) is a distinct concern — see the `cluster-configuration-compliance` skill.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Cluster_list` | List clusters | Cluster |
| `Vcenter.Network_list` | List networks/port groups | Network |

## Verification checklist

- [ ] Confirmed the request is genuinely a lookup, not an expectation that a cluster or network can be created/modified via this surface
- [ ] For a cluster desired-state change, redirected to the `cluster-configuration-compliance` skill instead
