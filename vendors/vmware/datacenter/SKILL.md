---
name: vmware-datacenter
description: How to manage VMware vSphere datacenters — the top-level inventory container. Vendor-neutral. Use when building, reviewing, or debugging vSphere datacenter automation, on Itential or otherwise.
---

# VMware vSphere — Datacenter

## When to use this skill

- Creating or deleting a datacenter.
- Understanding what a datacenter deletion actually cascades to.

## Operational procedure

Datacenters are the top-level organizational container — everything else (clusters, hosts, VMs, networks) lives inside one. Deleting a datacenter cascades to everything organized under it; there's no "delete but keep the contents" option.

## Patterns

- **No partial deletion** — deleting a datacenter is all-or-nothing for everything inside it; there's no way to preserve contents while removing the container.

## Known limitations

- No cross-object blast-radius reasoning — this skill doesn't enumerate everything a datacenter contains before deletion; that's the reviewer's job.
- Host, cluster, and network management within a datacenter are covered by other skills (`host`, `cluster-network-inventory`).

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Datacenter_list` | List datacenters (top-level inventory containers) | Datacenter |
| `Vcenter.Datacenter_create` | Create a new datacenter | Datacenter |
| `Vcenter.Datacenter_delete` | Delete a datacenter — cascades to everything organized inside it | Datacenter |

## Verification checklist

- [ ] Before deleting a datacenter, confirmed everything organized under it was actually intended to be removed
- [ ] After creation, confirmed the datacenter is visible via `Vcenter.Datacenter_list` before organizing anything inside it
