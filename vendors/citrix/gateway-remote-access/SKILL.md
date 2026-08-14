---
name: citrix-gateway-remote-access
description: How to configure NetScaler Gateway (SSL VPN) for remote access and ICA proxy for Citrix Virtual Apps/Desktops access. Vendor-neutral. Use when building, reviewing, or debugging NetScaler Gateway/remote-access automation, on Itential or otherwise.
---

# Citrix NetScaler — Gateway & Remote Access

## When to use this skill

- Setting up SSL VPN / remote access through NetScaler Gateway.
- Setting up ICA proxy access to Citrix Virtual Apps/Desktops.
- Reviewing or debugging why remote users can authenticate but can't reach an internal resource.

## Operational procedure

**SSL VPN (NetScaler Gateway)**: create the Gateway vserver, bind a trusted SSL certificate to it (remote users will not connect through an untrusted cert without a manual override), configure and bind the authentication policy so users can actually log in, then bind the intranet applications/resources users are allowed to reach. Access is deny-by-default — a Gateway vserver with no resources explicitly bound authenticates users into nothing.

**ICA proxy** (access to Citrix Virtual Apps/Desktops through Gateway): requires the Gateway vserver to already exist. Create the ICA action/policy pointing at the actual StoreFront/broker address, bind it to the vserver. Verify with a real client launch, not just a successful API bind — most ICA proxy failures happen on the StoreFront handshake side, which is invisible from the NetScaler configuration alone.

## Patterns

- **Stage-then-attach for external dependencies** — confirm the SSL certificate and the StoreFront/broker address are valid and reachable on their own before wiring them into a live Gateway vserver.
- **Deny-by-default** — a Gateway vserver grants no access until resources are explicitly bound; absence of a binding means absence of access, not an error.

## Known limitations

- No offensive/destructive capability.
- ICA proxy verification requires an actual client launch to be conclusive — a successful API bind alone does not confirm StoreFront/broker connectivity.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

### SSL VPN

| Operation | Plain-English description | Category |
|---|---|---|
| `listVpnvserver` | List NetScaler Gateway (SSL VPN) virtual servers | Gateway |
| `createVpnvserver` | Create a new Gateway vserver | Gateway |
| `updateVpnvserver` | Change an existing Gateway vserver's configuration | Gateway |
| `createVpnglobalAuthenticationldappolicyBinding` | Bind an LDAP auth policy globally to the Gateway | Gateway |
| `createVpnglobalAuthenticationradiuspolicyBinding` | Bind a RADIUS auth policy globally to the Gateway | Gateway |
| `createVpnglobalIntranetapplicationBinding` | Grant Gateway users access to an internal application/resource | Gateway |

### ICA Proxy

| Operation | Plain-English description | Category |
|---|---|---|
| `listIcapolicy` | List ICA policies (govern access to Citrix Virtual Apps/Desktops via Gateway) | ICA Proxy |
| `createIcapolicy` | Create a new ICA policy | ICA Proxy |
| `updateIcapolicy` | Change an existing ICA policy | ICA Proxy |
| `createIcaaction` | Create an ICA action (points at the StoreFront/broker address) | ICA Proxy |
| `updateIcaaction` | Change an existing ICA action | ICA Proxy |
| `createIcapolicyVpnvserverBinding` | Bind an ICA policy to a Gateway vserver | ICA Proxy |
| `listIcapolicyVpnvserverBinding` | List ICA policies bound to a Gateway vserver | ICA Proxy |

## Verification checklist

- [ ] Gateway vserver's bound SSL certificate confirmed valid/trusted, not just present
- [ ] Authentication policy tested with a real login attempt
- [ ] At least one intranet application/resource explicitly bound before considering access "configured"
- [ ] ICA proxy verified with a real client launch, not just the API bind response
