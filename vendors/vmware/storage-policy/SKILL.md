---
name: vmware-storage-policy
description: How to check VMware vSphere SPBM storage-policy compliance and compatibility. Vendor-neutral. Use when building, reviewing, or debugging vSphere storage-policy automation, on Itential or otherwise.
---

# VMware vSphere — Storage Policy (SPBM)

## When to use this skill

- Checking storage-policy compliance or compatibility.
- Confirming which storage policy is assigned to a given VM.

## Operational procedure

1. Identify the target datastore or VM.
2. Check storage-policy compatibility against that specific datastore — this is a pre-flight check, not a permanent guarantee; a policy compatible today can become non-compliant later if the datastore's underlying capabilities change.
3. For a compliance question, trigger a fresh compliance check rather than trusting a potentially-stale list — compliance reporting is inherently retrospective.
4. Re-check compatibility periodically, not just once at VM-creation time.

## Patterns

- **Compatibility is a snapshot, not a guarantee** — re-check periodically, since underlying datastore capabilities can change after the fact.
- **Compliance lists are retrospective** — trigger a fresh check for a "right now" answer.

## Known limitations

- Datastore capacity itself is a distinct concern — see the `datastore` skill.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Datastore.DefaultPolicy_get` | Get a datastore's default storage policy | Storage Policy |
| `Vcenter.Storage.Policies_list` | List SPBM storage policies | Storage Policy |
| `Vcenter.Storage.Policies_checkCompatibility` | Check whether a storage policy is compatible with a given datastore | Storage Policy |
| `Vcenter.Storage.Policies.Compliance_list` | List storage-policy compliance status across entities | Storage Policy |
| `Vcenter.Storage.Policies.VM_list` | List which storage policy is assigned to which VM | Storage Policy |

## Verification checklist

- [ ] Storage-policy compatibility checked against the specific target datastore, not assumed from a prior check
- [ ] For a "right now" compliance question, triggered a fresh compliance check rather than trusting a potentially-stale list
