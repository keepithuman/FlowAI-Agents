---
name: citrix-ssl-vpn
description: How to configure NetScaler Gateway (SSL VPN) for remote access — vserver, certificate, authentication, and intranet resource binding. Vendor-neutral. Use when building, reviewing, or debugging NetScaler Gateway/SSL VPN automation, on Itential or otherwise.
---

# Citrix NetScaler — SSL VPN (NetScaler Gateway)

## When to use this skill

- Setting up SSL VPN / remote access through NetScaler Gateway.
- Debugging why remote users can authenticate but can't reach an internal resource.

## Operational procedure

Create the Gateway vserver, bind a trusted SSL certificate to it (remote users will not connect through an untrusted cert without a manual override), configure and bind the authentication policy so users can actually log in, then bind the intranet applications/resources users are allowed to reach. Access is deny-by-default — a Gateway vserver with no resources explicitly bound authenticates users into nothing.

## Patterns

- **Stage-then-attach** — confirm the SSL certificate is valid/trusted before binding it to a live Gateway vserver.
- **Deny-by-default** — absence of a resource binding means absence of access, not an error.

## Known limitations

- No offensive/destructive capability.
- Authentication policy configuration itself (LDAP/RADIUS action/policy) belongs to the `authentication` skill — this skill covers the Gateway vserver and its bindings, not the identity backend.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listVpnvserver` | List NetScaler Gateway (SSL VPN) virtual servers | Gateway |
| `createVpnvserver` | Create a new Gateway vserver | Gateway |
| `updateVpnvserver` | Change an existing Gateway vserver's configuration | Gateway |
| `createVpnglobalAuthenticationldappolicyBinding` | Bind an LDAP auth policy globally to the Gateway | Gateway |
| `createVpnglobalAuthenticationradiuspolicyBinding` | Bind a RADIUS auth policy globally to the Gateway | Gateway |
| `createVpnglobalIntranetapplicationBinding` | Grant Gateway users access to an internal application/resource | Gateway |

## Verification checklist

- [ ] Gateway vserver's bound SSL certificate confirmed valid/trusted, not just present
- [ ] Authentication tested with a real login attempt
- [ ] At least one intranet resource explicitly bound before considering access "configured"
