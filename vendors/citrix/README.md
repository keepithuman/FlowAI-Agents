# Citrix — NetScaler ADC

Operational knowledge for day-to-day NetScaler (Citrix ADC) automation: load balancing, traffic routing, SSL, GSLB, security, remote access, system administration, core networking, clustering/HA, DNS, bot management, and traffic optimization/analytics. Every write operation described here follows the same rule — propose the exact change, get human approval, only then apply it — regardless of what orchestrates the automation.

Start with [`SKILL.md`](./SKILL.md) — it covers both the real operational procedures and the exhaustive tool reference (every real operation name, plain-English description, and category) in one place.

## Coverage summary

**211 real NetScaler NITRO REST API operations**, across 13 categories: Load Balancing, Traffic Routing (Content Switching, Responder, Rewrite, Policy Building Blocks), SSL Certificates, GSLB & Multi-Site, Security & Access (WAF, Authentication, Rate Limiting), Monitoring & Diagnostics, Gateway & Remote Access (SSL VPN, ICA Proxy), System Administration (RBAC, Feature/Licensing, Config Backup), Networking Fundamentals (IP/VLAN, Routing/Interfaces, LACP, NAT), Clustering & High Availability, DNS Services, Bot Management, and Traffic Optimization & Analytics (Caching/Compression, Performance Profiles/Spillover, AppFlow).

## Source

Every operation in `SKILL.md`'s Tools section was confirmed against a live NetScaler NITRO REST API adapter's registered task catalog, cross-checked against the official **Citrix NetScaler NITRO 14.1 OpenAPI spec**. One operation (`createLbvserverResponderpolicyBinding` vs. `createResponderpolicyLbvserverBinding`) was specifically verified against the real spec to resolve a schema-valid-but-non-functional ambiguity — see `SKILL.md`'s Traffic Routing procedure.

## Prerequisites

- A NetScaler appliance (or VPX/CPX instance) with the NITRO REST API enabled — this is on by default on most builds, reachable at `https://<nsip>/nitro/v1/config/`.
- HTTP Basic auth (NetScaler admin credentials) or a session token obtained via the NITRO login endpoint.
- Some operations (WAF/AppFirewall, Bot Management, AppFlow, GSLB) require the corresponding NetScaler feature to be licensed and enabled — check `listNsfeature` before assuming a capability is available on a given appliance.
