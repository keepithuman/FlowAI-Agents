---
name: vmware-datacenter-cluster-host-network
description: How to manage VMware vSphere datacenter, cluster, host, and network inventory — including host connect/disconnect. Vendor-neutral. Use when building, reviewing, or debugging vSphere infrastructure-inventory automation, on Itential or otherwise.
---

# VMware vSphere — Datacenter, Cluster, Host & Network

## When to use this skill

- Datacenter/cluster/host/network inventory lookups.
- Connecting or disconnecting a host.

## Operational procedure

Datacenters are the top-level organizational container — everything else (clusters, hosts, VMs, networks) lives inside one. Deleting a datacenter cascades to everything organized under it; there's no "delete but keep the contents" option.

Disconnecting a host doesn't power off or migrate the VMs running on it — they keep running, but vCenter loses management visibility and control until the host reconnects. Never disconnect a host as a first response to "it's misbehaving" without checking what's currently running on it and whether those VMs can tolerate a management gap.

Clusters and networks are typically inventory-lookup targets, not direct configuration targets, at this layer — cluster formation happens through host aggregation and networks are usually defined at the host/vSwitch layer, neither of which this kind of REST surface exposes directly for mutation. Treat questions here as "what exists and where" rather than "create/modify this."

## Patterns

- **Read current state first** — always check what's actually running on a host before disconnecting it.

## Known limitations

- No standard distributed-switch/port-group management exists in this API surface — the only switch-adjacent operations are scoped to Kubernetes/Tanzu supervisor networking (`Vcenter.NamespaceManagement.Networks.Nsx.*`), not general-purpose vSphere networking.
- Cluster formation and network (vSwitch/port-group) configuration are largely out of scope for direct mutation at this layer — this skill covers lookup, not creation, for clusters and networks.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Datacenter_list` | List datacenters (top-level inventory containers) | Datacenter |
| `Vcenter.Datacenter_create` | Create a new datacenter | Datacenter |
| `Vcenter.Datacenter_delete` | Delete a datacenter — cascades to everything organized inside it | Datacenter |
| `Vcenter.Cluster_list` | List clusters | Cluster |
| `Vcenter.Host_list` | List ESXi hosts | Host |
| `Vcenter.Host_connect` | Connect a host to vCenter's management | Host |
| `Vcenter.Host_disconnect` | Disconnect a host from vCenter's management (VMs on it keep running, unmanaged) | Host |
| `Vcenter.Network_list` | List networks/port groups | Network |

## Verification checklist

- [ ] Before disconnecting a host, confirmed what VMs are currently running on it and whether they can tolerate a management gap
- [ ] After deleting a datacenter, confirmed everything organized under it was actually intended to be removed (no "keep the contents" option exists)
- [ ] Host reconnection confirmed via `Vcenter.Host_list`/`Vcenter.Host_connect` response, not assumed
