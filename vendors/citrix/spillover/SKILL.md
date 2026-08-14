---
name: citrix-spillover
description: How to configure NetScaler spillover (overflow) policies to protect a backend from oversaturation. Vendor-neutral. Use when building, reviewing, or debugging NetScaler spillover automation, on Itential or otherwise.
---

# Citrix NetScaler — Spillover

## When to use this skill

- Configuring overflow protection for a backend.
- Reviewing whether a spillover threshold is actually protective.

## Operational procedure

1. Determine the backend's actual saturation point.
2. Set the overflow threshold meaningfully below that point, not at it — spillover configured to trigger only once the primary is already failing defeats the purpose of having it.
3. Create the spillover policy and bind it to the LB vserver.
4. Test spillover behavior under simulated load before relying on it in production.

## Patterns

- **Threshold below saturation, not at it** — spillover's entire value is triggering before failure, not at the moment of failure.

## Known limitations

- No offensive/destructive capability.
- TCP/HTTP performance tuning is a distinct concern from overflow protection — see the `performance-profiles` skill.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listSpilloverpolicy` | List spillover (overflow) policies | Spillover |
| `createSpilloverpolicy` | Create a new spillover policy | Spillover |
| `createSpilloverpolicyLbvserverBinding` | Bind a spillover policy to an LB vserver | Spillover |

## Verification checklist

- [ ] Threshold confirmed meaningfully below actual backend saturation point, not at it
- [ ] Spillover behavior tested under simulated load before relying on it in production
- [ ] Confirmed the LB vserver's real request volume relative to the configured threshold, not an assumed figure
