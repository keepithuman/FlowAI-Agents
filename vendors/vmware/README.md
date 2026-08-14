# VMware — vSphere

FlowAI agents for day-to-day VMware vSphere operations: VM lifecycle actions, datacenter/cluster/host/network inventory, storage and datastore management, resource pools, content library and templates, guest OS customization, tagging, access control (RBAC), certificate management, VM encryption (KMS), cluster desired-state configuration, performance metrics, and read-only vCenter/appliance diagnostics. Every mutating agent proposes the exact change and waits for human approval before touching the environment — nothing here applies a change unattended.

13 agents, built and verified against a live `VMware vSphere Automation` REST integration (1,363 registered adapter methods). The project file in `projects/` is a real export — created via the platform's Agent Project Service, then `GET`-verified (tool references resolve, provider is set) before being committed here.

Start with [`AGENTS.md`](./AGENTS.md) for the domain overview and design principles, then load [`SKILL.md`](./SKILL.md) when you need the real operational procedure for a specific domain.

## Project index

| Project file | Covers | Agents |
|---|---|---|
| [`vmware-vsphere-operations.project.json`](./projects/vmware-vsphere-operations.project.json) | VM operations, infrastructure inventory, storage, resource pools, content library, guest customization, tagging, RBAC, certificates, VM encryption, cluster configuration, performance metrics, diagnostics | 13 |

## Prerequisites

- An Itential Platform instance with the **VMware vSphere Automation** integration installed and its adapter methods registered as discoverable, active tools (`GET /tools?name=<method>` should return results under a `vSphere Automation`-titled, `active: true` integration).
- A configured LLM provider profile + model reachable by the Agent Project Service — the values baked into the exported project file reflect the environment they were built on, not a portable default; resolve `provider` to your own environment on import.
- `view:WorkFlowEngine:ViewData` available as a tool — every mutating agent in this package uses it as the human-approval gate. The two read-only diagnostics-style agents (vCenter & Appliance Diagnostics, and the read-only portions of Storage & Datastore / Performance Metrics) don't need it.
