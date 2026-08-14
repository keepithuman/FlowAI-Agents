---
name: citrix-config-backup
description: How to take and review NetScaler configuration backups, and when to take one relative to a risky change. Vendor-neutral. Use when building, reviewing, or debugging NetScaler config-backup automation, on Itential or otherwise.
---

# Citrix NetScaler — Config Backup

## When to use this skill

- Taking or reviewing a configuration backup.
- Deciding when a backup needs to happen relative to another change.

## Operational procedure

1. Identify the risky change about to happen — an RBAC or feature/licensing change.
2. Trigger a configuration backup *immediately before* that change, not after.
3. Confirm the backup exists and is retrievable (`getSystembackupByName`) before proceeding — a broken RBAC or feature change can itself lock out the access needed to restore from a backup taken too late to help.
4. Proceed with the risky change.

## Patterns

- **Backup-before-risky-change**, specifically for RBAC and feature/licensing changes, since those are the two categories of change that can lock out the access needed to recover.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover restoring from a backup — only taking one; treat restore as a separate, higher-stakes procedure requiring its own explicit review.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `listSystembackup` | List existing configuration backups | Config Backup |
| `createSystembackup` | Trigger a new configuration backup | Config Backup |
| `getSystembackupByName` | Get details of a specific backup | Config Backup |

## Verification checklist

- [ ] Backup confirmed present via `listSystembackup`/`getSystembackupByName` immediately before the risky change proceeds — not just that the create call returned success
- [ ] Backup taken *before*, not after, any RBAC or feature/licensing change
