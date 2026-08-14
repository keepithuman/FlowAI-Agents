# Citrix — NetScaler ADC

Operational knowledge for day-to-day NetScaler (Citrix ADC) automation, split into 13 individual skills — one per functional domain. Every write operation described in these skills follows the same rule — propose the exact change, get human approval, only then apply it — regardless of what orchestrates the automation.

Each skill below is self-contained: real operational procedure plus the exhaustive real-operation tool reference for that domain, in one `SKILL.md`. Load only the ones a given task needs.

## Skills

| Skill | Domain |
|---|---|
| [`load-balancing`](./load-balancing/) | Virtual servers, backend services/service groups, health monitors |
| [`traffic-routing`](./traffic-routing/) | Content switching, responder policies, rewrite policies, policy building blocks |
| [`ssl-certificates`](./ssl-certificates/) | Certificate install/bind/renewal, expiry monitoring |
| [`gslb-multi-site`](./gslb-multi-site/) | Global Server Load Balancing, multi-site DR, geo-routing, failover |
| [`security-access`](./security-access/) | WAF (AppFirewall), LDAP/RADIUS authentication, rate limiting |
| [`monitoring-diagnostics`](./monitoring-diagnostics/) | Read-only health/status checks — vserver, service, monitor, HA, certificate |
| [`gateway-remote-access`](./gateway-remote-access/) | NetScaler Gateway (SSL VPN), ICA proxy for Virtual Apps/Desktops |
| [`system-administration`](./system-administration/) | Admin RBAC, feature/licensing, configuration backup |
| [`networking-fundamentals`](./networking-fundamentals/) | IP addressing, VLANs, routing, interfaces, LACP, NAT |
| [`clustering-ha`](./clustering-ha/) | Multi-box clustering, HA node pairing |
| [`dns-services`](./dns-services/) | Forward (A/CNAME) and zone/reverse (NS/PTR/SOA) DNS records |
| [`bot-management`](./bot-management/) | Bot detection and mitigation |
| [`traffic-optimization-analytics`](./traffic-optimization-analytics/) | Caching, compression, performance profiles, spillover, AppFlow |

## Coverage summary

**211 real NetScaler NITRO REST API operations** across these 13 skills.

## Source

Every operation in each skill's Tools section was confirmed against a live NetScaler NITRO REST API adapter's registered task catalog, cross-checked against the official **Citrix NetScaler NITRO 14.1 OpenAPI spec**. One operation (`createLbvserverResponderpolicyBinding` vs. `createResponderpolicyLbvserverBinding`) was specifically verified against the real spec to resolve a schema-valid-but-non-functional ambiguity — see the `traffic-routing` skill.

## Prerequisites

- A NetScaler appliance (or VPX/CPX instance) with the NITRO REST API enabled — this is on by default on most builds, reachable at `https://<nsip>/nitro/v1/config/`.
- HTTP Basic auth (NetScaler admin credentials) or a session token obtained via the NITRO login endpoint.
- Some operations (WAF/AppFirewall, Bot Management, AppFlow, GSLB) require the corresponding NetScaler feature to be licensed and enabled — check `listNsfeature` before assuming a capability is available on a given appliance.
