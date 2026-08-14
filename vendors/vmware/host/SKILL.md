---
name: vmware-host
description: How to connect, disconnect, and list VMware vSphere ESXi hosts, and what disconnecting actually does (and doesn't do) to running VMs. Vendor-neutral. Use when building, reviewing, or debugging vSphere host-management automation, on Itential or otherwise.
---

# VMware vSphere — Host

## When to use this skill

- Connecting or disconnecting a host.
- Deciding whether disconnecting a host is a safe first response to a problem.

## Operational procedure

Disconnecting a host doesn't power off or migrate the VMs running on it — they keep running, but vCenter loses management visibility and control until the host reconnects. Never disconnect a host as a first response to "it's misbehaving" without checking what's currently running on it and whether those VMs can tolerate a management gap.

## Patterns

- **Read current state first** — always check what's actually running on a host before disconnecting it.
- **Disconnect ≠ shutdown** — VMs keep running; only vCenter's visibility and control are lost.

## Known limitations

- No cross-object blast-radius reasoning — this skill doesn't automatically enumerate every VM on a host and its tolerance for a management gap; that's the reviewer's job.
- Cluster/network inventory is covered by the `cluster-network-inventory` skill.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Host_list` | List ESXi hosts | Host |
| `Vcenter.Host_connect` | Connect a host to vCenter's management | Host |
| `Vcenter.Host_disconnect` | Disconnect a host from vCenter's management (VMs on it keep running, unmanaged) | Host |

## Verification checklist

- [ ] Before disconnecting a host, confirmed what VMs are currently running on it and whether they can tolerate a management gap
- [ ] Host reconnection confirmed via `Vcenter.Host_list`, not assumed
- [ ] After reconnecting, confirmed vCenter's management visibility is actually restored for VMs on that host
