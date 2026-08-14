---
name: citrix-rbac
description: How to manage NetScaler admin RBAC — command policies, groups, users, and their bindings. Vendor-neutral. Use when building, reviewing, or debugging NetScaler admin RBAC automation, on Itential or otherwise.
---

# Citrix NetScaler — RBAC (Admin Users, Groups, Command Policies)

## When to use this skill

- Managing admin RBAC (users, groups, command policies).
- Reviewing what access an admin user actually has.

## Operational procedure

1. Create the command policy first — the actual permission boundary (what's allowed/denied).
2. Create the group.
3. Create the user.
4. Bind the user to the group, and the group to the policy. A user created but never bound to a group/policy is left with the appliance's default access level, which may be broader or narrower than intended — don't assume "unbound" means "no access."
5. Confirm the user's actual effective access matches intent.

## Patterns

- **Policy → group → user → binding.** Build the permission boundary first, then the containers, then attach.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover config backup/restore around an RBAC change — see the `config-backup` skill; take a backup immediately before any RBAC change.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listSystemuser` | List NetScaler admin user accounts | RBAC |
| `createSystemuser` | Create a new admin user account | RBAC |
| `updateSystemuser` | Change an existing admin user's settings | RBAC |
| `listSystemgroup` | List admin groups | RBAC |
| `createSystemgroup` | Create a new admin group | RBAC |
| `createSystemuserSystemgroupBinding` | Add a user to an admin group | RBAC |
| `listSystemcmdpolicy` | List command policies (the actual permission boundary) | RBAC |
| `createSystemcmdpolicy` | Create a new command policy | RBAC |
| `createSystemuserSystemcmdpolicyBinding` | Attach a command policy directly to a user | RBAC |

## Verification checklist

- [ ] Command policy's actual permission boundary reviewed before attaching it to any group
- [ ] New admin user confirmed bound to a group with the intended policy — not left at appliance default
- [ ] A fresh config backup exists immediately before any RBAC change is applied
