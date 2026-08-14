# Citrix — NetScaler ADC

Operational knowledge for day-to-day NetScaler (Citrix ADC) automation, split into 30 focused skills — each one answers exactly one job someone would actually ask for, not a bundle of related-but-separate concerns. Every write operation described in these skills follows the same rule — propose the exact change, get human approval, only then apply it — regardless of what orchestrates the automation.

Each skill below is self-contained: real operational procedure plus the exhaustive real-operation tool reference for that one job, in one `SKILL.md`. Load only the ones a given task needs.

## Skills

| Skill | Job |
|---|---|
| [`load-balancing`](./load-balancing/) | Stand up/scale a load-balanced app — vservers, services, monitors |
| [`content-switching`](./content-switching/) | Route by URL/host to different backends |
| [`responder-policies`](./responder-policies/) | Redirect, block, or custom-respond to requests |
| [`rewrite-policies`](./rewrite-policies/) | Modify request/response headers or content |
| [`policy-building-blocks`](./policy-building-blocks/) | Datasets, pattern sets, string maps referenced by policy expressions |
| [`ssl-certificates`](./ssl-certificates/) | Install, bind, renew SSL certificates |
| [`gslb-multi-site`](./gslb-multi-site/) | Multi-site DR, geo-routing, GSLB failover |
| [`waf`](./waf/) | Web Application Firewall (AppFirewall) profiles/policies |
| [`authentication`](./authentication/) | LDAP/RADIUS authentication policies |
| [`rate-limiting`](./rate-limiting/) | Throttling / rate-limit selectors and identifiers |
| [`monitoring-diagnostics`](./monitoring-diagnostics/) | Read-only health/status checks, in the right order |
| [`ssl-vpn`](./ssl-vpn/) | NetScaler Gateway — SSL VPN remote access |
| [`ica-proxy`](./ica-proxy/) | ICA proxy access to Citrix Virtual Apps/Desktops |
| [`rbac`](./rbac/) | Admin RBAC — users, groups, command policies |
| [`feature-licensing`](./feature-licensing/) | Feature/mode enablement and licensing checks |
| [`config-backup`](./config-backup/) | Configuration backups and when to take one |
| [`ip-vlan`](./ip-vlan/) | IP addressing and VLANs |
| [`routing-interfaces`](./routing-interfaces/) | Static routes and interface state |
| [`lacp-channels`](./lacp-channels/) | LACP link aggregation |
| [`nat`](./nat/) | Inbound (INAT) and reverse (RNAT) NAT |
| [`clustering`](./clustering/) | Multi-box clustering |
| [`ha-pairing`](./ha-pairing/) | Active/passive HA node pairing |
| [`dns-forward-records`](./dns-forward-records/) | DNS A/CNAME records |
| [`dns-zone-reverse-records`](./dns-zone-reverse-records/) | DNS NS/SOA/PTR records |
| [`bot-management`](./bot-management/) | Bot detection and mitigation |
| [`caching`](./caching/) | Integrated caching policies |
| [`compression`](./compression/) | Compression policies |
| [`performance-profiles`](./performance-profiles/) | TCP/HTTP performance-tuning profiles |
| [`spillover`](./spillover/) | Overflow protection for a backend |
| [`appflow`](./appflow/) | AppFlow analytics export |

## Coverage summary

**211 real NetScaler NITRO REST API operations** across these 30 skills.

## Source

Every operation in each skill's Tools section was confirmed against a live NetScaler NITRO REST API adapter's registered task catalog, cross-checked against the official **Citrix NetScaler NITRO 14.1 OpenAPI spec**. One operation (`createLbvserverResponderpolicyBinding` vs. `createResponderpolicyLbvserverBinding`) was specifically verified against the real spec to resolve a schema-valid-but-non-functional ambiguity — see the `responder-policies` skill.

## Prerequisites

- A NetScaler appliance (or VPX/CPX instance) with the NITRO REST API enabled — this is on by default on most builds, reachable at `https://<nsip>/nitro/v1/config/`.
- HTTP Basic auth (NetScaler admin credentials) or a session token obtained via the NITRO login endpoint.
- Some operations (WAF/AppFirewall, Bot Management, AppFlow, GSLB) require the corresponding NetScaler feature to be licensed and enabled — check `listNsfeature` (the `feature-licensing` skill) before assuming a capability is available on a given appliance.
