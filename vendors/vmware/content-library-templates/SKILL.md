---
name: vmware-content-library-templates
description: How to manage VMware vSphere content libraries and VM templates — capture, deploy, local vs. subscribed libraries. Vendor-neutral. Use when building, reviewing, or debugging vSphere content-library/template automation, on Itential or otherwise.
---

# VMware vSphere — Content Library & Templates

## When to use this skill

- Managing a content library.
- Capturing a VM as a template or deploying a new VM from one.

## Operational procedure

A content library is either local (owned and directly editable) or subscribed (a read-only mirror of someone else's library, kept in sync automatically). Never edit an item in a subscribed library directly — it'll fail or get silently overwritten on the next sync; edit the source library instead.

Capturing a VM as a template freezes its current disk state as the template's base image — if the source VM keeps running and changing after capture, the template does not reflect those later changes.

Deploying from a template creates a new VM from that frozen image — always confirm placement (folder, resource pool, datastore, network) explicitly rather than accepting whatever defaults the template happened to specify, since a template captured in one environment can carry placement defaults that don't exist in another.

## Patterns

- **Source-of-truth awareness** — know whether a library is local or subscribed before editing anything in it; subscribed libraries have a different (upstream) source of truth.
- **Explicit placement, always** — never accept a template's carried-over placement defaults without confirming they apply in the target environment.

## Known limitations

- No cross-object blast-radius reasoning — deploying many VMs from one template doesn't automatically check aggregate resource impact on the target placement.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Content.Library_list` | List content libraries (local and subscribed) | Content Library |
| `Content.Library_get` | Get a single content library's details | Content Library |
| `Content.LocalLibrary_create` | Create a new local (directly editable) content library | Content Library |
| `Content.LocalLibrary_update` | Change an existing local library's settings | Content Library |
| `Content.LocalLibrary_delete` | Delete a local content library | Content Library |
| `Vcenter.VmTemplate.LibraryItems_create` | Capture a VM as a template library item (freezes current disk state as the base image) | VM Templates |
| `Vcenter.VmTemplate.LibraryItems_deploy` | Deploy a new VM from a template library item | VM Templates |
| `Vcenter.VmTemplate.LibraryItems_get` | Get a template library item's details | VM Templates |

## Verification checklist

- [ ] Confirmed whether the target library is local or subscribed before attempting any edit
- [ ] Template capture confirmed to reflect the intended VM state (not a stale/mid-change snapshot of the source VM)
- [ ] Deployment placement (folder, resource pool, datastore, network) explicitly confirmed, not left at template defaults
