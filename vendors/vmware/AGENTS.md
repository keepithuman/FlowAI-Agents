# VMware vSphere — Agent Domain Guide

Read this before building, modifying, or invoking any agent in this vendor package. It orients; it does not enumerate — for exact tool names and payload shapes, load `SKILL.md` instead.

## Domain overview

vSphere (vCenter + ESXi) is the virtualization layer most enterprise infrastructure runs on top of, and the operations against it span several genuinely distinct teams: platform/infra teams provisioning and placing workloads, storage teams managing datastores and storage policies, security teams handling access control and encryption, automation teams building repeatable deployment specs, and monitoring teams watching performance. This package's agents are organized along those real team boundaries rather than by raw API namespace, because that's how the requests actually arrive in practice — "who can access this cluster" and "how much CPU headroom is on this datastore" are different people's problems, even though both live under the same vCenter.

## Design principles

**1. Every mutating agent proposes, then waits, then acts.**
Every agent capable of creating, updating, deleting, or reconfiguring a vSphere object follows the same sequence: look up current state → compose the *exact* payload it intends to send → present it via `view:WorkFlowEngine:ViewData` (current state vs. proposed state, explicit Approve/Reject) → only call the mutating method if the human clicked Approve. The one full exception is the vCenter & Appliance Diagnostics agent, which has no mutating tools at all — read-only by construction. Two other agents (Storage & Datastore, Performance Metrics) are hybrids: read-only questions are answered directly with no gate, but a genuine configuration change (a new storage-policy assignment decision, a new metric-collection spec) still goes through the same propose-then-approve sequence as everything else.

**2. Agents are split by team/domain, not by raw API namespace.**
The underlying API groups things by object type (`Vcenter.VM_*`, `Vcenter.Authorization.*`, `Cis.Tagging.*`), but a single agent holding every VM-related or every authorization-related tool degrades fast — past roughly 10 tools, an LLM tool-caller starts missing genuinely relevant tools in a long list or picking a plausible-but-wrong one. Every agent in this package tops out at 9 tools. Where a real team's job spans multiple raw namespaces (e.g., cluster configuration draws from `Esx.Settings.Clusters.*`), that's still one agent, because it's one coherent decision for one team — the split follows the operator's mental model, not the API's file structure.

**3. Some capabilities that exist in "VMware" broadly don't exist in this specific API.**
VMware's REST-based vSphere Automation API is the modern successor to the older SOAP-based vSphere API, and it hasn't ported every capability yet. Two gaps worth knowing before you go looking for them: there is no VM snapshot API (create/revert/delete snapshots) in this catalog, and there is no standard distributed-switch/port-group management (the only switch-adjacent methods that exist are scoped to Kubernetes/Tanzu supervisor networking, not general-purpose vSphere networking). Both are real, common VMware operations — they're just not reachable through this particular API surface, on this VMware version, as of when this package was built.

## Capability index

| A user asks for... | Agent |
|---|---|
| Clone or relocate a VM; get a console session | VM Operations Agent |
| Datacenter, cluster, host, or network inventory; connect/disconnect a host | Datacenter, Cluster, Host & Network Agent |
| Datastore capacity, storage-policy compliance/compatibility | Storage & Datastore Agent |
| Content library management, capture/deploy a VM template | Content Library & Templates Agent |
| Tag categories and tags for organizing/targeting inventory objects | Tagging & Categorization Agent |
| vCenter's own TLS certificate renewal, trusted root chains | Certificate Management Agent |
| "Is VCHA healthy?", appliance uptime/version/load | vCenter & Appliance Diagnostics Agent (read-only) |
| Create/resize/delete a resource pool | Resource Pool Management Agent |
| Roles, privileges, who-can-do-what (RBAC) | Access Control (RBAC) Agent |
| Guest OS customization specs (sysprep/cloud-init-style, used during deployment) | Guest Customization Agent |
| Cluster desired-state configuration (DRS/HA settings) and compliance checks | Cluster Configuration & Compliance Agent |
| KMS provider setup backing VM encryption | VM Encryption & KMS Agent |
| "What's the CPU usage on VM X?", setting up new metric collection | Performance Metrics Agent |

## Known limitations

- **No VM snapshot management.** Not present in the underlying API on this VMware version — see Design Principle 3.
- **No standard distributed-switch/port-group management.** Same underlying cause — see Design Principle 3.
- **No agent reasons about cross-object blast radius.** Each agent looks up the state directly relevant to its own action, not a full dependency graph (e.g., the Resource Pool agent doesn't automatically check every VM currently drawing from a pool before proposing a change to it). A human approver is still the backstop for catching that.
- **VM lifecycle actions that depend on an external system of record (ticketing, change management) are intentionally out of scope for this package.** These agents take a free-text `request` as input and act directly — they don't assume or require a specific ticketing system. A ticket-driven variant of VM provisioning/resize/decommission is a legitimately different design (structured intake, audit trail in an external system) and is out of scope here by design, not an oversight.
