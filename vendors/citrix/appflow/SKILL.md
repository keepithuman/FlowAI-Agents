---
name: citrix-appflow
description: How to configure NetScaler AppFlow analytics export — actions, policies, and collectors. Vendor-neutral. Use when building, reviewing, or debugging NetScaler AppFlow automation, on Itential or otherwise.
---

# Citrix NetScaler — AppFlow Analytics

## When to use this skill

- Configuring AppFlow analytics export.
- Debugging why analytics data appears to have stopped flowing.

## Operational procedure

Verify the collector's own network reachability (it's a real UDP/TCP export target) before wiring an action/policy to it. A policy bound to an unreachable collector drops analytics silently — there's no application-visible error when this happens, so it's easy to believe analytics are flowing when they aren't.

## Patterns

- **Stage-then-attach for external targets** — confirm an AppFlow collector is reachable before wiring a policy to it.
- **Silent failure mode** — a policy bound to an unreachable collector produces no application-visible error; verify data is actually arriving at the collector, not just that the NetScaler-side config is correct.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover analysis of AppFlow-exported data itself — only the NetScaler-side configuration to export it.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listAppflowpolicy` | List AppFlow analytics-export policies | AppFlow |
| `createAppflowpolicy` | Create a new AppFlow policy | AppFlow |
| `updateAppflowpolicy` | Change an existing AppFlow policy | AppFlow |
| `createAppflowaction` | Create an AppFlow action (what to export and where) | AppFlow |
| `updateAppflowaction` | Change an existing AppFlow action | AppFlow |
| `createAppflowcollector` | Create an AppFlow collector (the real network target analytics get sent to) | AppFlow |
| `createAppflowpolicyCsvserverBinding` | Bind an AppFlow policy to a content-switching vserver | AppFlow |

## Verification checklist

- [ ] Collector's network reachability confirmed directly before wiring a policy to it
- [ ] After binding, confirmed data is actually arriving at the collector — not just that the NetScaler-side config succeeded
- [ ] Export scope (what's bound where) reviewed to confirm it matches the intended traffic, not broader
