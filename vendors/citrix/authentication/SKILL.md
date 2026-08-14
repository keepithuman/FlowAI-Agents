---
name: citrix-authentication
description: How to configure NetScaler LDAP/RADIUS authentication policies and authentication virtual servers. Vendor-neutral. Use when building, reviewing, or debugging NetScaler authentication automation, on Itential or otherwise.
---

# Citrix NetScaler — Authentication (LDAP/RADIUS)

## When to use this skill

- Setting up LDAP or RADIUS authentication.
- Debugging why users can't log in through an authentication or Gateway vserver.

## Operational procedure

1. Create the action first — the actual server connection (host, port, bind credentials, attribute mapping).
2. Test the raw LDAP/RADIUS connection independently, outside of any user-facing vserver.
3. Create the policy — the expression selecting when the action applies.
4. Bind the policy to an authentication or Gateway vserver. Skipping step 2 is what turns a misconfigured auth action into an outage — bound directly to production, it blocks every real login attempt at once.
5. Confirm a real login attempt succeeds after binding.

## Patterns

- **Action → policy → binding.**
- **Stage-then-attach for external connections** — validate an LDAP/RADIUS connection independently before wiring it into a user-facing vserver.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover the identity provider's own configuration (the actual LDAP/RADIUS server) — only the NetScaler-side action/policy/binding around it.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listAuthenticationldappolicy` | List LDAP authentication policies | Authentication |
| `createAuthenticationldappolicy` | Create a new LDAP authentication policy | Authentication |
| `listAuthenticationradiuspolicy` | List RADIUS authentication policies | Authentication |
| `createAuthenticationradiuspolicy` | Create a new RADIUS authentication policy | Authentication |
| `listAuthenticationvserver` | List authentication virtual servers | Authentication |
| `createAuthenticationvserver` | Create a new authentication vserver | Authentication |
| `createAuthenticationvserverAuthenticationldappolicyBinding` | Bind an LDAP policy to an authentication vserver | Authentication |
| `createAuthenticationvserverAuthenticationradiuspolicyBinding` | Bind a RADIUS policy to an authentication vserver | Authentication |

## Verification checklist

- [ ] LDAP/RADIUS connection tested independently before binding to a production-facing vserver
- [ ] A real login attempt tested after binding, not just the bind call's success
- [ ] Attribute mapping confirmed to produce the expected identity/group claims
