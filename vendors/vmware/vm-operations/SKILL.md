---
name: vmware-vm-operations
description: How to clone, relocate, and get console access to VMware vSphere VMs. Vendor-neutral. Use when building, reviewing, or debugging vSphere VM clone/relocate/console automation, on Itential or otherwise.
---

# VMware vSphere — VM Operations

## When to use this skill

- Cloning or relocating a VM.
- Granting remote console access to a VM.

## Operational procedure

Decide whether you need a full clone (independent copy, more storage, safe to power on immediately) or an instant clone (shares the parent's disk via a running-memory fork, near-instant, but the source VM must be running) — these solve different problems, and picking instant-clone for a VM you intend to keep long-term as an independent asset is the wrong tool.

A relocate can move a VM's compute (to a different host/cluster), storage (to a different datastore), or both. Confirm the target host/cluster and datastore have the resources and network connectivity the VM needs before proposing a relocate — a VM relocated onto a host with no path to its required network segment ends up unreachable, not failed-loudly.

A console ticket grants time-limited remote access to a VM's console — treat the ticket itself as a credential; don't log it or leave it somewhere it could be captured long after the session should have expired.

## Patterns

- **Read current state first** — confirm target host/cluster/datastore resources and connectivity before proposing a clone or relocate, not after.

## Known limitations

- No cross-object blast-radius reasoning — relocating a VM doesn't automatically check every dependent (load balancer pool membership, monitoring config) that assumes its current location.
- No VM snapshot management (create/revert/delete) exists in this API surface at all. This capability exists in the older SOAP-based vSphere API and has not been ported to the modern REST-based vSphere Automation API as of this writing — if a request needs a snapshot, it needs a different API surface.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.VM_list` | List virtual machines | VM Operations |
| `Vcenter.VM_get` | Get a single VM's full configuration | VM Operations |
| `Vcenter.VM_clone` | Create a full independent copy of a VM | VM Operations |
| `Vcenter.VM_relocate` | Move a VM's compute (host/cluster), storage (datastore), or both | VM Operations |
| `Vcenter.Vm.Console.Tickets_create` | Generate a time-limited remote console access ticket for a VM | VM Operations |

## Verification checklist

- [ ] Target host/cluster and datastore confirmed to have sufficient resources and required network connectivity before relocating
- [ ] After clone/relocate, re-read the VM's state via `Vcenter.VM_get` — don't trust the operation call's success alone
- [ ] Console tickets not logged or persisted anywhere they could be captured after the session should expire
