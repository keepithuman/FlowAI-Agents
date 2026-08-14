---
name: citrix-rate-limiting
description: How to configure NetScaler rate limiting — selectors and limit identifiers — for throttling traffic. Vendor-neutral. Use when building, reviewing, or debugging NetScaler rate-limiting automation, on Itential or otherwise.
---

# Citrix NetScaler — Rate Limiting

## When to use this skill

- Configuring rate limiting / throttling.
- Debugging why a rate limit is throttling far more or less traffic than intended.

## Operational procedure

1. Define the selector first — what's actually being counted (e.g., per source IP, per URL).
2. Validate the selector's real-world grouping behavior before trusting it — a selector scoped incorrectly (for example, counting per unique client behind a NAT gateway serving many real users as one IP) throttles far more or less traffic than intended.
3. Create the limit identifier — the threshold and time window.
4. Bind the identifier via an enforcing policy (see the `responder-policies`/`rewrite-policies`/`policy-building-blocks` skills) — a limit identifier alone doesn't throttle anything.
5. Test the threshold against real traffic volume before relying on it in production.

## Patterns

- **Selector before identifier** — define what's being counted before defining the threshold on it.
- **Validate real-world grouping behavior** — a selector that looks correct in isolation can group traffic very differently than intended once real, NAT'd, or proxied clients are involved.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover binding the limit identifier into a responder/rewrite policy expression that actually enforces it — see the `responder-policies`/`rewrite-policies` and `policy-building-blocks` skills for that half of the workflow.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `listNslimitidentifier` | List rate-limit identifiers (the threshold/time-window definitions) | Rate Limiting |
| `createNslimitidentifier` | Create a new rate-limit identifier | Rate Limiting |
| `listNslimitselector` | List rate-limit selectors (what's being counted — e.g. per source IP) | Rate Limiting |
| `createNslimitselector` | Create a new rate-limit selector | Rate Limiting |

## Verification checklist

- [ ] Selector's real-world grouping behavior validated (e.g., NAT'd clients) before trusting the configured threshold
- [ ] Threshold tested against real traffic volume before relying on it in production
- [ ] Confirmed the limit identifier is actually referenced by an enforcing policy — a limit identifier alone doesn't throttle anything
