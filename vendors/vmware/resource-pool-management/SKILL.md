---
name: vmware-resource-pool-management
description: How to create, resize, and delete VMware vSphere resource pools, including shares/reservations/limits and nested-pool inheritance. Vendor-neutral. Use when building, reviewing, or debugging vSphere resource-pool automation, on Itential or otherwise.
---

# VMware vSphere — Resource Pool Management

## When to use this skill

- Creating, resizing, or deleting a resource pool.
- Reviewing resource pool shares/reservations/limits.

## Operational procedure

1. Identify the parent pool/cluster and its available resources — a nested pool's effective limit can never exceed what its parent allows, regardless of what the child's own setting says.
2. Create or resize the pool with shares/reservations/limits.
3. Validate settings against actual or simulated contention — on an underutilized cluster, a tightly-configured pool and a loosely-configured one behave identically; the real test is what happens during actual contention.
4. Before deleting a pool, confirm the parent's suitability for the VMs that will be reparented into it — deleting a resource pool doesn't delete the VMs inside it, it reparents them.

## Patterns

- **Contention is the real test** — a resource pool's settings look fine on an idle cluster regardless of whether they're actually correct; validate against real or simulated contention when it matters.
- **Deletion reparents, doesn't delete, for resource pools** — always confirm the parent's suitability for the VMs before deleting.

## Known limitations

- No cross-object blast-radius reasoning — this skill doesn't automatically enumerate every VM currently drawing from a pool before a change to it; that check is the reviewer's job.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.ResourcePool_list` | List resource pools | Resource Pool |
| `Vcenter.ResourcePool_get` | Get a single resource pool's configuration | Resource Pool |
| `Vcenter.ResourcePool_create` | Create a new resource pool | Resource Pool |
| `Vcenter.ResourcePool_update` | Change an existing resource pool's CPU/memory shares, reservations, or limits | Resource Pool |
| `Vcenter.ResourcePool_delete` | Delete a resource pool — VMs inside get reparented to the pool's parent, not deleted | Resource Pool |

## Verification checklist

- [ ] Parent pool's settings confirmed suitable for reparented VMs before deleting a child pool
- [ ] Nested pool limits checked against parent constraints — a child's effective limit can't exceed the parent's, regardless of its own setting
- [ ] Shares/reservations/limits validated against actual or simulated contention, not just "looks fine on an idle cluster"
