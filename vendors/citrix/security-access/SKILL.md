---
name: citrix-security-access
description: How to configure NetScaler Web Application Firewall (AppFirewall), LDAP/RADIUS authentication, and rate limiting. Vendor-neutral. Use when building, reviewing, or debugging NetScaler security and access-control automation, on Itential or otherwise.
---

# Citrix NetScaler — Security & Access (WAF, Authentication, Rate Limiting)

## When to use this skill

- Configuring or reviewing a Web Application Firewall (AppFirewall) profile/policy.
- Setting up LDAP or RADIUS authentication.
- Configuring rate limiting / throttling.

## Operational procedure

**WAF (AppFirewall)**: create the profile (the actual ruleset/relaxations) before the policy (the match expression selecting which traffic the profile applies to). A profile with no policy bound to any vserver protects nothing. Start a new profile in log-only/non-blocking mode and run it against real traffic before switching to blocking — this is the only reliable way to catch false positives before they turn into a real outage for real users.

**Authentication (LDAP/RADIUS)**: create the action (the actual server connection: host, port, bind credentials, attribute mapping) before the policy (the expression selecting when it applies), before binding to an authentication or Gateway vserver. Test the raw LDAP/RADIUS connection independently — outside of any user-facing vserver — before wiring it into one; a misconfigured auth action bound directly to production blocks every real login attempt at once.

**Rate limiting**: define what's actually being counted first — the selector (e.g., per source IP, per URL) — before the limit identifier (the threshold and time window). A selector scoped incorrectly (for example, counting per unique client behind a NAT gateway serving many real users as one IP) throttles far more or less traffic than intended; validate the selector's real-world grouping behavior before trusting the threshold.

## Patterns

- **Action/profile → policy → binding**, the same shape as every other NetScaler policy-driven feature.
- **Stage-then-attach for external connections** — validate an LDAP/RADIUS connection independently before wiring it into a user-facing vserver.
- **Log-only before blocking, universally**, for anything that can reject real user traffic (WAF, rate limiting) — an observation window catches false positives before they become outages.

## Known limitations

- No offensive/destructive capability — this skill never covers using these features to attack or evade detection, only to configure legitimate protection.
- No cross-object blast-radius reasoning — e.g., binding an auth policy globally to the Gateway isn't automatically checked against every vserver it will affect.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

### WAF (AppFirewall)

| Operation | Plain-English description | Category |
|---|---|---|
| `listAppfwprofile` | List Web Application Firewall profiles (rulesets/relaxations) | WAF |
| `createAppfwprofile` | Create a new WAF profile | WAF |
| `updateAppfwprofile` | Change an existing WAF profile's settings | WAF |
| `listAppfwpolicy` | List WAF policies (the match expression selecting which traffic a profile applies to) | WAF |
| `createAppfwpolicy` | Create a new WAF policy | WAF |
| `updateAppfwpolicy` | Change an existing WAF policy's match rule | WAF |
| `createLbvserverAppfwpolicyBinding` | Bind a WAF policy to an LB vserver | WAF |
| `createCsvserverAppfwpolicyBinding` | Bind a WAF policy to a content-switching vserver | WAF |

### Authentication

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

### Rate Limiting

| Operation | Plain-English description | Category |
|---|---|---|
| `listNslimitidentifier` | List rate-limit identifiers (the threshold/time-window definitions) | Rate Limiting |
| `createNslimitidentifier` | Create a new rate-limit identifier | Rate Limiting |
| `listNslimitselector` | List rate-limit selectors (what's being counted — e.g. per source IP) | Rate Limiting |
| `createNslimitselector` | Create a new rate-limit selector | Rate Limiting |

## Verification checklist

- [ ] New WAF profile ran in log-only mode against real traffic before switching to blocking
- [ ] LDAP/RADIUS connection tested independently before binding to a production-facing vserver
- [ ] Rate-limit selector's real-world grouping behavior validated (e.g., NAT'd clients) before trusting the configured threshold
