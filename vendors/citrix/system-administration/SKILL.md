---
name: citrix-system-administration
description: How to manage NetScaler RBAC (admin users/groups/command policies), feature and license enablement, and configuration backups. Vendor-neutral. Use when building, reviewing, or debugging NetScaler system-administration automation, on Itential or otherwise.
---

# Citrix NetScaler — System Administration

## When to use this skill

- Managing admin RBAC (users, groups, command policies).
- Enabling/disabling a feature or checking licensing.
- Taking or reviewing configuration backups.

## Operational procedure

**RBAC**: create the command policy (the actual permission boundary — what's allowed/denied) before the group, the group before the user, then bind user-to-group and group-to-policy. A user created but never bound to a group/policy is left with the appliance's default access level, which may be broader or narrower than intended depending on the appliance's own default policy — don't assume "unbound" means "no access."

**Feature/mode & licensing**: confirm a feature is actually licensed before attempting to enable it. Enabling an unlicensed feature usually fails cleanly, but some feature/license combinations fail in confusing, firmware-version-dependent ways — check license and feature status together, not as two independent steps.

**Config backup**: take a backup *immediately before* any RBAC or feature change, not after. A broken RBAC or feature change can itself lock out the access needed to restore from a backup taken too late to help.

## Patterns

- **Backup-before-risky-change**, specifically for RBAC and feature/licensing changes, since those are the two categories of change that can lock out the access needed to recover.
- **Policy → group → user → binding**, for RBAC specifically — build the permission boundary first, then the containers, then attach.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover restoring from a backup — only taking one; treat restore as a separate, higher-stakes procedure requiring its own explicit review.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

### RBAC

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

### Feature & Licensing

| Operation | Plain-English description | Category |
|---|---|---|
| `listNsfeature` | List which NetScaler features are enabled/disabled | Feature & Licensing |
| `updateNsfeature` | Enable or disable a feature | Feature & Licensing |
| `listNsmode` | List appliance operating modes | Feature & Licensing |
| `updateNsmode` | Change an appliance operating mode | Feature & Licensing |
| `listNslicense` | List installed licenses | Feature & Licensing |
| `createNslicense` | Install a new license | Feature & Licensing |

### Config Backup

| Operation | Plain-English description | Category |
|---|---|---|
| `listSystembackup` | List existing configuration backups | Config Backup |
| `createSystembackup` | Trigger a new configuration backup | Config Backup |
| `getSystembackupByName` | Get details of a specific backup | Config Backup |

## Verification checklist

- [ ] Command policy's actual permission boundary reviewed before attaching it to any group
- [ ] New admin user confirmed bound to a group with the intended policy — not left at appliance default
- [ ] Feature/license status checked together before assuming a feature can be enabled
- [ ] A fresh config backup exists immediately before any RBAC or feature change is applied
