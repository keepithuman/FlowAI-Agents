---
name: vmware-storage-datastore
description: How to check VMware vSphere datastore capacity and storage-policy (SPBM) compliance/compatibility. Vendor-neutral. Use when building, reviewing, or debugging vSphere storage automation, on Itential or otherwise.
---

# VMware vSphere — Storage & Datastore

## When to use this skill

- Checking datastore capacity before an operation that consumes storage.
- Checking storage-policy compliance or compatibility.

## Operational procedure

Datastore free space is the single most important number before proposing anything that consumes storage elsewhere (a new VM, a resized disk, a new content library item) — always re-check it fresh, don't reuse a number from earlier in the same conversation, since other activity on a shared datastore can change it in the meantime.

Storage-policy compatibility checking is a pre-flight check, not a permanent guarantee — a policy compatible with a datastore today can become non-compliant later if the datastore's underlying capabilities change. Worth re-checking periodically, not just once at VM-creation time.

Compliance reporting is inherently retrospective — it reports the last-known compliance state. If you need to know the compliance state *right now*, trigger a fresh compliance check rather than trusting a list that could be stale.

## Patterns

- **Re-check fresh, always** — datastore capacity and storage-policy compliance are both point-in-time facts on a shared resource; a number from earlier in the same session can already be stale.

## Known limitations

- No cross-object blast-radius reasoning — checking one datastore's free space doesn't account for other concurrent operations that might consume it at the same time.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Datastore_list` | List datastores | Datastore |
| `Vcenter.Datastore_get` | Get a single datastore's details, including free/used capacity | Datastore |
| `Vcenter.Datastore.DefaultPolicy_get` | Get a datastore's default storage policy | Storage Policy |
| `Vcenter.Storage.Policies_list` | List SPBM storage policies | Storage Policy |
| `Vcenter.Storage.Policies_checkCompatibility` | Check whether a storage policy is compatible with a given datastore | Storage Policy |
| `Vcenter.Storage.Policies.Compliance_list` | List storage-policy compliance status across entities | Storage Policy |
| `Vcenter.Storage.Policies.VM_list` | List which storage policy is assigned to which VM | Storage Policy |

## Verification checklist

- [ ] Datastore free space re-checked fresh immediately before proposing anything that consumes it — not reused from earlier in the conversation
- [ ] Storage-policy compatibility checked against the specific target datastore, not assumed from a prior check
- [ ] For a "right now" compliance question, triggered a fresh compliance check rather than trusting a potentially-stale list
