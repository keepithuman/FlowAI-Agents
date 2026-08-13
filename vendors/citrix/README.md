# Citrix — NetScaler ADC

FlowAI agents for day-to-day NetScaler (Citrix ADC) operations: load balancing, traffic routing, SSL, GSLB, security, remote access, system administration, core networking, clustering/HA, DNS, bot management, and traffic optimization/analytics. Every mutating agent proposes the exact change and waits for human approval before touching the appliance — nothing here applies a change unattended.

13 projects, 33 agents, built and verified against a live NetScaler NITRO API integration (~6,600 registered adapter methods). Every project file in `projects/` is a real export — created via the platform's Agent Project Service, then `GET`-verified (tool references resolve, provider is set, zero broken bindings) before being committed here.

Start with [`AGENTS.md`](./AGENTS.md) for the domain overview and design principles, then load [`SKILL.md`](./SKILL.md) when you need the exact tool list for a specific agent.

## Project index

| Project file | Covers | Agents |
|---|---|---|
| [`load-balancing.project.json`](./projects/load-balancing.project.json) | LB virtual servers, backend services/service groups, bindings, health monitors | 3 |
| [`traffic-routing.project.json`](./projects/traffic-routing.project.json) | Content switching, responder policies, rewrite policies, policy building blocks (pattern sets/datasets/string maps) | 5 |
| [`ssl-certificates.project.json`](./projects/ssl-certificates.project.json) | Cert install/renewal/binding, expiry monitoring | 1 |
| [`gslb-multi-site.project.json`](./projects/gslb-multi-site.project.json) | GSLB vservers/services, sites, domain bindings, failover | 2 |
| [`security-access.project.json`](./projects/security-access.project.json) | WAF (AppFirewall) policies, LDAP/RADIUS authentication, rate limiting | 3 |
| [`monitoring-diagnostics.project.json`](./projects/monitoring-diagnostics.project.json) | Read-only appliance/vserver/service health, cert-expiry reporting — no write tools | 2 |
| [`gateway-remote-access.project.json`](./projects/gateway-remote-access.project.json) | NetScaler Gateway (SSL VPN), ICA proxy for Citrix Virtual Apps/Desktops | 2 |
| [`system-administration.project.json`](./projects/system-administration.project.json) | Admin RBAC, feature/mode/license management, config backup | 3 |
| [`networking-fundamentals.project.json`](./projects/networking-fundamentals.project.json) | IP addresses, VLANs, static routes, interfaces, LACP channels, NAT | 4 |
| [`clustering-ha.project.json`](./projects/clustering-ha.project.json) | Multi-box cluster setup, HA node pairing | 2 |
| [`dns-services.project.json`](./projects/dns-services.project.json) | DNS record management (A/CNAME/NS/PTR/SOA) | 2 |
| [`bot-management.project.json`](./projects/bot-management.project.json) | Bot-detection/mitigation policies and profiles | 1 |
| [`traffic-optimization-analytics.project.json`](./projects/traffic-optimization-analytics.project.json) | Caching, compression, TCP/HTTP performance profiles, spillover, AppFlow analytics export | 3 |

**Total: 33 agents**, each with 4–10 tools (no agent exceeds 10 — see `AGENTS.md` for why that ceiling matters).

## Prerequisites

- An Itential Platform instance with the **Citrix NetScaler NITRO API** integration installed, with its adapter methods registered as discoverable, active tools. This platform's adapter has previously carried a stale, inactive catalog variant alongside the current one — confirm any given method resolves to the active integration before relying on it in an agent.
- A configured LLM provider profile + model reachable by the Agent Project Service (`provider.profile` / `provider.model` UUIDs — these are environment-specific and must be re-resolved on import; the values baked into the exported project files reflect the environment they were built on, not a portable default).
- `view:WorkFlowEngine:ViewData` available as a tool — every mutating agent in this package uses it as the human-approval gate.
