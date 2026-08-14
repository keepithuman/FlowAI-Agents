---
name: vmware-access-control-rbac
description: How to manage VMware vSphere roles and permissions (RBAC) — creating roles, assigning permissions, and reasoning about inheritance. Vendor-neutral. Use when building, reviewing, or debugging vSphere access-control automation, on Itential or otherwise.
---

# VMware vSphere — Access Control (RBAC)

## When to use this skill

- Creating or reviewing roles and privileges.
- Assigning or removing permissions.
- "Who can do what" questions.

## Operational procedure

A role is a named set of privileges; a permission is the actual assignment of a role to a specific principal on a specific object, with an inheritable flag. Creating a role changes nothing by itself — nobody gains access until a permission assignment references it.

Permissions are typically inheritable down the inventory hierarchy by default — a permission granted at the datacenter level usually applies to everything inside it unless explicitly set non-propagating. Granting broad access "for convenience" at a high level is a common way to over-grant without realizing it.

Before creating a new role, check whether an existing one already covers the need — proliferating near-duplicate roles makes the access model harder to audit later, which is its own security cost even when each individual role is scoped correctly.

## Patterns

- **Role creation is inert by itself** — nothing changes until a permission assignment actually references the role.
- **Check inheritance scope explicitly** — a permission granted high in the hierarchy propagates down by default; that's often not what was actually intended.

## Known limitations

- No cross-object blast-radius reasoning — this skill doesn't automatically enumerate everything a broad, inheritable permission grant will actually affect; that's the reviewer's job.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Authorization.Roles_list` | List roles (named privilege sets) | RBAC |
| `Vcenter.Authorization.Roles_create` | Create a new role | RBAC |
| `Vcenter.Authorization.Roles_update` | Change an existing role's privileges | RBAC |
| `Vcenter.Authorization.Roles_delete` | Delete a role | RBAC |
| `Vcenter.Authorization.Permissions_list` | List permissions (role assignments on specific objects) | RBAC |
| `Vcenter.Authorization.Permissions_create` | Assign a role to a principal on a specific object | RBAC |
| `Vcenter.Authorization.Permissions_delete` | Remove a permission assignment | RBAC |

## Verification checklist

- [ ] Confirmed an existing role doesn't already cover the need before creating a new one
- [ ] Permission's inheritable/propagating flag confirmed intentional, not left at a default that over-grants
- [ ] After assigning a permission, confirmed who actually gains access (including via inheritance), not just that the API call succeeded
