# VMware — vSphere

Operational knowledge for day-to-day VMware vSphere automation, split into 13 individual skills — one per functional domain. Every write operation described in these skills follows the same rule — propose the exact change, get human approval, only then apply it — regardless of what orchestrates the automation.

Each skill below is self-contained: real operational procedure plus the exhaustive real-operation tool reference for that domain, in one `SKILL.md`. Load only the ones a given task needs.

## Skills

| Skill | Domain |
|---|---|
| [`vm-operations`](./vm-operations/) | VM clone, relocate, console access |
| [`datacenter-cluster-host-network`](./datacenter-cluster-host-network/) | Datacenter/cluster/host/network inventory, host connect/disconnect |
| [`storage-datastore`](./storage-datastore/) | Datastore capacity, SPBM storage-policy compliance/compatibility |
| [`content-library-templates`](./content-library-templates/) | Content library management, VM template capture/deploy |
| [`tagging-categorization`](./tagging-categorization/) | Tag categories and tags for organizing/targeting inventory |
| [`certificate-management`](./certificate-management/) | vCenter's own TLS certificate renewal, trusted root chains |
| [`vcenter-appliance-diagnostics`](./vcenter-appliance-diagnostics/) | VCHA health, appliance uptime/version/load |
| [`resource-pool-management`](./resource-pool-management/) | Resource pool create/resize/delete, shares/reservations/limits |
| [`access-control-rbac`](./access-control-rbac/) | Roles, privileges, permissions, who-can-do-what |
| [`guest-customization`](./guest-customization/) | Guest OS customization specs for deployment |
| [`cluster-configuration-compliance`](./cluster-configuration-compliance/) | Cluster desired-state config (DRS/HA), compliance checks |
| [`vm-encryption-kms`](./vm-encryption-kms/) | KMS provider setup backing VM encryption |
| [`performance-metrics`](./performance-metrics/) | Metric acquisition specs, querying collected performance data |

## Coverage summary

**85 real vSphere Automation REST API operations** across these 13 skills.

## Source

Every operation in each skill's Tools section was confirmed against a live `VMware vSphere Automation` REST adapter's registered task catalog (1,363 total methods available on that catalog; 85 selected here as the common-operations subset — see each skill's Known Limitations for what's deliberately excluded). Naming follows VMware's own dot-notation API namespace (`Vcenter.VM_create`, `Cis.Tagging.Category_create`, etc.), not a paraphrase.

## Prerequisites

- A vCenter Server instance (the vSphere Automation REST API is served directly by vCenter, typically at `https://<vcenter>/api/`).
- Session-token auth obtained via the vCenter session-creation endpoint, using a vCenter account with appropriate privileges for the operations in question.
- Some operations (VM Encryption/KMS, Cluster Configuration desired-state, Tagging) require the corresponding vSphere feature/license tier — check the specific product edition before assuming availability.
- **Two real gaps to know about before reaching for this API**: no VM snapshot management (see `vm-operations`'s Known Limitations), and no standard distributed-switch/port-group management (see `datacenter-cluster-host-network`'s Known Limitations).
