---
name: vmware-vm-templates
description: How to capture a VMware vSphere VM as a template and deploy new VMs from one. Vendor-neutral. Use when building, reviewing, or debugging vSphere VM-template automation, on Itential or otherwise.
---

# VMware vSphere — VM Templates

## When to use this skill

- Capturing a VM as a template.
- Deploying a new VM from an existing template.

## Operational procedure

Capturing a VM as a template freezes its current disk state as the template's base image — if the source VM keeps running and changing after capture, the template does not reflect those later changes.

Deploying from a template creates a new VM from that frozen image — always confirm placement (folder, resource pool, datastore, network) explicitly rather than accepting whatever defaults the template happened to specify, since a template captured in one environment can carry placement defaults that don't exist in another.

## Patterns

- **Explicit placement, always** — never accept a template's carried-over placement defaults without confirming they apply in the target environment.

## Known limitations

- Requires a content library to already exist — see the `content-library` skill for creating one.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.VmTemplate.LibraryItems_create` | Capture a VM as a template library item (freezes current disk state as the base image) | VM Templates |
| `Vcenter.VmTemplate.LibraryItems_deploy` | Deploy a new VM from a template library item | VM Templates |
| `Vcenter.VmTemplate.LibraryItems_get` | Get a template library item's details | VM Templates |

## Verification checklist

- [ ] Template capture confirmed to reflect the intended VM state (not a stale/mid-change snapshot of the source VM)
- [ ] Deployment placement (folder, resource pool, datastore, network) explicitly confirmed, not left at template defaults
