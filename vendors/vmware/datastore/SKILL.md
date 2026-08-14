---
name: vmware-datastore
description: How to check VMware vSphere datastore free/used capacity before an operation that consumes storage. Vendor-neutral, read-only. Use when checking datastore capacity, on Itential or otherwise.
---

# VMware vSphere — Datastore Capacity

## When to use this skill

- Checking datastore capacity before an operation that consumes storage (new VM, resized disk, new content library item).

## Operational procedure

Datastore free space is the single most important number before proposing anything that consumes storage elsewhere — always re-check it fresh, don't reuse a number from earlier in the same conversation, since other activity on a shared datastore can change it in the meantime.

## Patterns

- **Re-check fresh, always** — datastore capacity is a point-in-time fact on a shared resource; a number from earlier in the same session can already be stale.

## Known limitations

- Storage-policy compliance/compatibility is a distinct concern — see the `storage-policy` skill.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Datastore_list` | List datastores | Datastore |
| `Vcenter.Datastore_get` | Get a single datastore's details, including free/used capacity | Datastore |

## Verification checklist

- [ ] Datastore free space re-checked fresh immediately before proposing anything that consumes it — not reused from earlier in the conversation
