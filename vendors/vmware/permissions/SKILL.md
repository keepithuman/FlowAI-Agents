---
name: vmware-permissions
description: How to assign and remove VMware vSphere permissions — role assignments on specific objects — including inheritance behavior. Vendor-neutral. Use when building, reviewing, or debugging vSphere permission automation, on Itential or otherwise.
---

# VMware vSphere — Permissions

## When to use this skill

- Assigning or removing a permission (a role assignment on a specific object).
- "Who can do what" questions.

## Operational procedure

A permission is the actual assignment of a role to a specific principal on a specific object, with an inheritable flag. Permissions are typically inheritable down the inventory hierarchy by default — a permission granted at the datacenter level usually applies to everything inside it unless explicitly set non-propagating. Granting broad access "for convenience" at a high level is a common way to over-grant without realizing it.

## Patterns

- **Check inheritance scope explicitly** — a permission granted high in the hierarchy propagates down by default; that's often not what was actually intended.

## Known limitations

- No cross-object blast-radius reasoning — this skill doesn't automatically enumerate everything a broad, inheritable permission grant will actually affect; that's the reviewer's job.
- Requires the role already exist — see the `roles` skill.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Authorization.Permissions_list` | List permissions (role assignments on specific objects) | RBAC |
| `Vcenter.Authorization.Permissions_create` | Assign a role to a principal on a specific object | RBAC |
| `Vcenter.Authorization.Permissions_delete` | Remove a permission assignment | RBAC |

## Verification checklist

- [ ] Permission's inheritable/propagating flag confirmed intentional, not left at a default that over-grants
- [ ] After assigning a permission, confirmed who actually gains access (including via inheritance), not just that the API call succeeded
