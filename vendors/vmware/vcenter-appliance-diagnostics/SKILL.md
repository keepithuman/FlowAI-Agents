---
name: vmware-vcenter-appliance-diagnostics
description: How to check VMware vCenter High Availability (VCHA) status and appliance health/uptime/version/load. Vendor-neutral, read-only. Use when debugging whether vCenter itself is healthy, on Itential or otherwise.
---

# VMware vSphere — vCenter & Appliance Diagnostics

## When to use this skill

- "Is VCHA healthy?" questions.
- Appliance uptime, version, or load checks.
- Ruling vCenter itself in or out as the cause of a broader issue.

## Operational procedure

1. Check VCHA's active-node status first — is there a healthy primary right now.
2. Check general appliance health next (load, uptime). Remember uptime resets on any vCenter service restart, not just a full appliance reboot — a low uptime number doesn't necessarily mean the whole appliance just came up.
3. Only conclude "vCenter itself is fine, the problem is elsewhere" after both of the above are checked.

## Patterns

- **VCHA active-node status first, then general appliance health** — in that order, for any "is vCenter itself okay" question.

## Known limitations

- Read-only by design — this skill diagnoses, it doesn't remediate. A VCHA or appliance-health issue found here requires vCenter's own HA/appliance-management procedures to fix, which are outside this skill's scope.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Description | Category |
|---|---|---|
| `Vcenter.Vcha.Cluster_get` | Get vCenter High Availability (VCHA) cluster configuration | VCHA Status |
| `Vcenter.Vcha.Cluster.Active_get` | Get the currently active (primary) VCHA node | VCHA Status |
| `Vcenter.Vcha.Operations_get` | Get available/pending VCHA operations | VCHA Status |
| `Appliance.Health.System_get` | Get overall appliance system health | Appliance Health |
| `Appliance.Health.Load_get` | Get the appliance's current load | Appliance Health |
| `Appliance.System.Version_get` | Get the appliance's software version | Appliance Health |
| `Appliance.System.Uptime_get` | Get the appliance's uptime (resets on any vCenter service restart, not just a full reboot) | Appliance Health |

## Verification checklist

- [ ] VCHA active-node status checked first
- [ ] Appliance uptime interpreted correctly — a low value means a recent service restart, not necessarily a full reboot
- [ ] "vCenter itself is fine" conclusion only reached after both VCHA and appliance health were checked, not assumed
