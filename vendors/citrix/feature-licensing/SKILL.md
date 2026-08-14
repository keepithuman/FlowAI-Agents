---
name: citrix-feature-licensing
description: How to enable/disable NetScaler features and operating modes, and check licensing. Vendor-neutral. Use when building, reviewing, or debugging NetScaler feature/licensing automation, on Itential or otherwise.
---

# Citrix NetScaler — Feature & Licensing

## When to use this skill

- Enabling/disabling a feature or operating mode.
- Checking whether a capability is licensed before assuming it's available.

## Operational procedure

1. Check the feature's license status.
2. Check the feature/mode's current enabled state alongside it — check both together, not as two independent steps; some combinations fail in confusing, firmware-version-dependent ways when checked separately.
3. Enable or disable the feature/mode.
4. Confirm the feature is actually active via a real functional check, not just "enabled" in config.

## Patterns

- **License and feature status checked together**, never independently — some combinations fail in confusing, firmware-version-dependent ways if checked separately.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover config backup/restore around a feature change — see the `config-backup` skill; take a backup immediately before any feature change.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listNsfeature` | List which NetScaler features are enabled/disabled | Feature & Licensing |
| `updateNsfeature` | Enable or disable a feature | Feature & Licensing |
| `listNsmode` | List appliance operating modes | Feature & Licensing |
| `updateNsmode` | Change an appliance operating mode | Feature & Licensing |
| `listNslicense` | List installed licenses | Feature & Licensing |
| `createNslicense` | Install a new license | Feature & Licensing |

## Verification checklist

- [ ] License and feature status checked together before assuming a feature can be enabled
- [ ] A fresh config backup exists immediately before any feature/mode change is applied
- [ ] After enabling, confirmed the feature is actually active (not just "enabled" in config) via a real functional check
