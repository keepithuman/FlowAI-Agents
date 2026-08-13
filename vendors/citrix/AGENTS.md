# Citrix NetScaler — Agent Domain Guide

Read this before building, modifying, or invoking any agent in this vendor package. It orients; it does not enumerate — for exact tool names and payload shapes, load `SKILL.md` instead.

## Domain overview

NetScaler (Citrix ADC) is an application-delivery controller: load balancing, content switching, SSL offload, GSLB, WAF, SSL VPN/remote access, and the networking/system-administration layers underneath all of it. Real NetScaler estates are operated by a mix of network engineers and app teams, and the operations they perform repeatedly are almost entirely **configuration changes to a live, traffic-serving appliance** — which means the risk profile of "the agent got the tool call right but nobody meant to run it yet" is real and constant. This package's agents exist to remove the toil of composing the right API call correctly, not to remove the human decision of whether to apply it.

## Design principles

**1. Every mutating agent proposes, then waits, then acts — never all three in one breath.**
Every agent capable of creating, updating, deleting, or binding a NetScaler object follows the same sequence: look up current state → compose the *exact* payload it intends to send → present it via `view:WorkFlowEngine:ViewData` (current state vs. proposed state, explicit Approve/Reject) → only call the mutating method if the human clicked Approve. This is enforced in every agent's `instructions`, not left to model discretion. The only agents without this gate are the two in `monitoring-diagnostics.project.json`, which have no mutating tools at all — read-only by construction, not by prompt instruction alone.

**2. Agents are split by sub-capability, not by NetScaler object family.**
NetScaler's own object model groups tightly related things together (e.g., a responder policy and its LB-vserver binding are "the same feature" to a NetScaler admin), but a single agent holding every tool for a whole domain degrades fast — past roughly 10 tools, an LLM tool-caller starts missing genuinely relevant tools in a long list or picking a plausible-but-wrong one. Every agent in this package tops out at 10 tools. Where a domain naturally has more surface area than that (traffic routing, networking fundamentals, system administration), it's a **project** containing several small agents, not one large agent. See the capability index below for the actual split.

**3. Some schema-valid API calls have no real backing command.**
NetScaler's NITRO API auto-generates two REST resources for many bindings — one from each side of the relationship (e.g. `responderpolicy_lbvserver_binding` and `lbvserver_responderpolicy_binding`). Both are schema-valid and both accept POST. On real appliances, only the vserver-owned direction (`lbvserver_responderpolicy_binding`) has an actual NetScaler CLI command behind it; the other returns NetScaler error 1088 "No such command" at runtime despite passing schema validation. Every agent that could hit this ambiguity is instructed to use the vserver-owned direction explicitly and never the alternative — see `SKILL.md`'s gotchas section for the specific method names.

## Capability index

| A user asks for... | Project | Agent(s) |
|---|---|---|
| Stand up / scale a load-balanced app | `load-balancing` | LB VServer Agent, LB Service & Service Group Agent, LB Binding & Health Monitor Agent |
| Route by URL/host to different backends | `traffic-routing` | CS VServer & Policy Agent, CS Binding Agent |
| Redirect, block, or return a custom response | `traffic-routing` | Responder Policy Agent |
| Modify request/response headers or content | `traffic-routing` | Rewrite Policy Agent |
| Reusable value sets referenced by policy expressions | `traffic-routing` | Policy Building Blocks Agent |
| Install, renew, or bind an SSL certificate; check expiry | `ssl-certificates` | SSL Certificate Agent |
| Multi-site DR / geo-routing / GSLB failover | `gslb-multi-site` | GSLB VServer & Service Agent, GSLB Site & Domain Binding Agent |
| WAF / AppFirewall protection | `security-access` | WAF Policy Agent |
| LDAP/RADIUS authentication for a vserver | `security-access` | Authentication Policy Agent |
| Throttle abusive traffic | `security-access` | Rate Limiting Agent |
| "Is X up?" / HA status / cert expiry report | `monitoring-diagnostics` | Vserver & Service Health Agent, Config & Certificate Diagnostics Agent |
| SSL VPN / remote-access setup | `gateway-remote-access` | SSL VPN Gateway Agent |
| VDI access via NetScaler Gateway | `gateway-remote-access` | ICA Proxy Agent |
| Admin users/groups/command policies | `system-administration` | RBAC Agent |
| Enable/disable a NetScaler feature, licensing | `system-administration` | Feature & Licensing Agent |
| Config backup | `system-administration` | Config Backup Agent |
| IP addresses, VLANs | `networking-fundamentals` | IP & VLAN Agent |
| Static routes, interfaces | `networking-fundamentals` | Routing & Interfaces Agent |
| LACP channels | `networking-fundamentals` | Channel (LACP) Agent |
| Inbound/reverse NAT | `networking-fundamentals` | NAT Agent |
| Multi-box cluster setup | `clustering-ha` | Clustering Agent |
| HA pair configuration | `clustering-ha` | HA Pair Agent |
| A/CNAME DNS records | `dns-services` | Forward Record Agent |
| NS/PTR/SOA DNS records | `dns-services` | Zone & Reverse Record Agent |
| Bot detection/mitigation | `bot-management` | Bot Management Agent |
| Caching / compression | `traffic-optimization-analytics` | Caching & Compression Agent |
| TCP/HTTP performance tuning, overflow/spillover | `traffic-optimization-analytics` | Performance Profiles & Spillover Agent |
| Export traffic analytics (AppFlow) | `traffic-optimization-analytics` | AppFlow Analytics Agent |

## Known limitations

- **No offensive/destructive actions.** These agents cover configuration and delivery — they do not include, and never will include, anything resembling traffic manipulation for denial-of-service, credential harvesting, or evasion of the appliance's own security controls.
- **Clustering/HA agents cover setup, not live failover execution under incident pressure.** Forcing a failover during an actual outage is a higher-stakes action than this package's current approval-gate pattern was designed around; treat these as configuration agents, not incident-response agents, until that's explicitly reviewed.
- **No agent currently reasons about cross-object blast radius** (e.g., "this VLAN change will also affect these 4 other bound objects"). Each agent looks up the state directly relevant to its own action, not a full dependency graph. A human approver is still the backstop for catching that.
