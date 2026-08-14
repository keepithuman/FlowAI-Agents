---
name: vmware-roles
description: How to create and manage VMware vSphere roles — named privilege sets — including why role creation alone changes nothing. Vendor-neutral. Use when building, reviewing, or debugging vSphere role automation, on Itential or otherwise.
---

# VMware vSphere — Roles

## When to use this skill

- Creating or reviewing roles and privileges.
- Deciding whether an existing role already covers a need before creating a new one.

## Operational procedure

1. Check whether an existing role already covers the need — proliferating near-duplicate roles makes the access model harder to audit later, which is its own security cost even when each individual role is scoped correctly.
2. If not, create the role — a named set of privileges.
3. Remember that role creation alone changes nothing; nobody gains access until a permission assignment (see the `permissions` skill) references it.

## Patterns

- **Role creation is inert by itself** — nothing changes until a permission assignment actually references the role.
- **Check for an existing role first** — near-duplicate roles are an audit cost even when each one is individually correct.

## Known limitations

- Assigning a role to a principal on an object is a distinct action — see the `permissions` skill; a role alone grants nothing.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Description | Category |
|---|---|---|
| `Vcenter.Authorization.Roles_list` | List roles (named privilege sets) | RBAC |
| `Vcenter.Authorization.Roles_create` | Create a new role | RBAC |
| `Vcenter.Authorization.Roles_update` | Change an existing role's privileges | RBAC |
| `Vcenter.Authorization.Roles_delete` | Delete a role | RBAC |

## Verification checklist

- [ ] Confirmed an existing role doesn't already cover the need before creating a new one
- [ ] Before deleting a role, confirmed no active permission assignment still references it
