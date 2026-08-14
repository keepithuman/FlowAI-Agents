# VMware — vSphere

Operational knowledge for day-to-day VMware vSphere automation: VM operations, datacenter/cluster/host/network inventory, storage and datastore management, resource pools, content library and templates, guest OS customization, tagging, access control (RBAC), certificate management, VM encryption (KMS), cluster desired-state configuration, and performance metrics. Every write operation described here follows the same rule — propose the exact change, get human approval, only then apply it — regardless of what orchestrates the automation.

Start with [`SKILL.md`](./SKILL.md) — it covers both the real operational procedures and the exhaustive tool reference (every real operation name, plain-English description, and category) in one place.

## Coverage summary

**85 real vSphere Automation REST API operations**, across 13 categories: VM Operations, Datacenter/Cluster/Host/Network, Storage & Datastore, Content Library & Templates, Tagging & Categorization, Certificate Management, vCenter & Appliance Diagnostics, Resource Pool Management, Access Control (RBAC), Guest Customization, Cluster Configuration & Compliance, VM Encryption & KMS, and Performance Metrics.

## Source

Every operation in `SKILL.md`'s Tools section was confirmed against a live `VMware vSphere Automation` REST adapter's registered task catalog (1,363 total methods available on that catalog; 85 selected here as the common-operations subset — see `SKILL.md`'s Known Limitations for what's deliberately excluded). Naming follows VMware's own dot-notation API namespace (`Vcenter.VM_create`, `Cis.Tagging.Category_create`, etc.), not a paraphrase.

## Prerequisites

- A vCenter Server instance (the vSphere Automation REST API is served directly by vCenter, typically at `https://<vcenter>/api/`).
- Session-token auth obtained via the vCenter session-creation endpoint, using a vCenter account with appropriate privileges for the operations in question.
- Some operations (VM Encryption/KMS, Cluster Configuration desired-state, Tagging) require the corresponding vSphere feature/license tier — check the specific product edition before assuming availability.
- **Two real gaps to know about before reaching for this API**: no VM snapshot management, and no standard distributed-switch/port-group management — see `SKILL.md`'s Tools section closing note for detail.
