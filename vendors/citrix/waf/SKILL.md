---
name: citrix-waf
description: How to configure NetScaler Web Application Firewall (AppFirewall) profiles and policies, including safe log-only rollout. Vendor-neutral. Use when building, reviewing, or debugging NetScaler WAF automation, on Itential or otherwise.
---

# Citrix NetScaler — Web Application Firewall (AppFirewall)

## When to use this skill

- Configuring or reviewing a WAF profile/policy.
- Rolling out WAF protection safely to a vserver.

## Operational procedure

1. Create the profile first — the actual ruleset/relaxations.
2. Create the policy — the match expression selecting which traffic the profile applies to.
3. Bind the policy to the target vserver in log-only/non-blocking mode. A profile with no policy bound to any vserver protects nothing.
4. Run it against real traffic and watch for false positives — this is the only reliable way to catch them before they turn into a real outage.
5. Switch to blocking mode only after that observation window.

## Patterns

- **Profile → policy → binding.**
- **Log-only before blocking, always** — the only reliable way to catch false positives before they become a real outage.

## Known limitations

- No offensive/destructive capability — this skill covers legitimate WAF configuration, never techniques for crafting requests that evade it.
- No cross-object blast-radius reasoning — a WAF policy's full traffic scope isn't traced automatically once bound.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `listAppfwprofile` | List Web Application Firewall profiles (rulesets/relaxations) | WAF |
| `createAppfwprofile` | Create a new WAF profile | WAF |
| `updateAppfwprofile` | Change an existing WAF profile's settings | WAF |
| `listAppfwpolicy` | List WAF policies (the match expression selecting which traffic a profile applies to) | WAF |
| `createAppfwpolicy` | Create a new WAF policy | WAF |
| `updateAppfwpolicy` | Change an existing WAF policy's match rule | WAF |
| `createLbvserverAppfwpolicyBinding` | Bind a WAF policy to an LB vserver | WAF |
| `createCsvserverAppfwpolicyBinding` | Bind a WAF policy to a content-switching vserver | WAF |

## Verification checklist

- [ ] New profile ran in log-only mode against real traffic before switching to blocking
- [ ] Policy binding confirmed present on the intended vserver(s), and only those
- [ ] After enabling blocking, confirmed no legitimate traffic pattern is being falsely flagged
