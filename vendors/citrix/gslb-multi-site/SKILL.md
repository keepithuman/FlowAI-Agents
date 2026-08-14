---
name: citrix-gslb-multi-site
description: How to configure NetScaler GSLB (Global Server Load Balancing) for multi-site disaster recovery, geo-routing, and site failover. Vendor-neutral. Use when building, reviewing, or debugging NetScaler GSLB automation, on Itential or otherwise.
---

# Citrix NetScaler — GSLB / Multi-Site

## When to use this skill

- Setting up multi-site DR, geo-routing, or GSLB failover.
- Reviewing or debugging why traffic isn't routing to the expected site.

## Operational procedure

1. Create a GSLB site object for each physical/logical location first — this is what lets sites exchange health and metric information with each other; nothing else in GSLB works meaningfully without it.
2. Create the GSLB service(s) representing each site's local application endpoint, and bind a monitor to each.
3. Create the GSLB vserver (the global entry point, typically what a DNS name ultimately resolves through), bind the per-site services to it, and bind the domain.
4. **Failover**: disable (don't delete) the GSLB service at the site you're failing away from. Disabling is reversible and near-instant; deleting requires full recreation to fail back, which is the wrong tool for what's usually a temporary, urgent action.

## Patterns

- **Disable/re-enable beats delete/recreate** for anything you might need to reverse under time pressure, like a failover. Deletion is for permanent decommissioning, not for reversible operational actions.

## Known limitations

- No offensive/destructive capability.
- Failover procedures here cover configuration, not live incident execution — treat this as setup guidance, not an incident-response runbook for an active outage.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `listGslbvserver` | List GSLB (Global Server Load Balancing) virtual servers — the global entry points for multi-site routing | GSLB VServer |
| `createGslbvserver` | Create a new GSLB vserver | GSLB VServer |
| `updateGslbvserver` | Change an existing GSLB vserver's configuration | GSLB VServer |
| `listGslbservice` | List GSLB services (per-site local endpoints) | GSLB Service |
| `createGslbservice` | Create a new GSLB service representing one site's endpoint | GSLB Service |
| `updateGslbservice` | Change an existing GSLB service's configuration — commonly used to disable a service for failover | GSLB Service |
| `createGslbsite` | Create a GSLB site — required before sites can exchange health/metric data with each other | GSLB Site |
| `listGslbsite` | List configured GSLB sites | GSLB Site |
| `createGslbvserverServiceBinding` | Attach a per-site GSLB service to a GSLB vserver | GSLB Binding |
| `listGslbvserverServiceBinding` | List which GSLB services are bound to a GSLB vserver | GSLB Binding |
| `createGslbvserverDomainBinding` | Bind a domain name to a GSLB vserver so DNS queries resolve through it | GSLB Binding |
| `listGslbvserverDomainBinding` | List which domains are bound to a GSLB vserver | GSLB Binding |

## Verification checklist

- [ ] After binding a GSLB service to a vserver, confirm via list that the binding is present and the service's monitor is reporting healthy
- [ ] After a failover (disabling a service), confirm traffic is actually resolving to the remaining active site(s), not just that the disable call succeeded
- [ ] Domain binding confirmed present on the GSLB vserver clients are expected to resolve through
