---
name: citrix-bot-management
description: How to configure NetScaler bot detection and mitigation profiles/policies. Vendor-neutral. Use when building, reviewing, or debugging NetScaler bot-management automation, on Itential or otherwise.
---

# Citrix NetScaler — Bot Management

## When to use this skill

- Configuring bot detection/mitigation.
- Reviewing a bot policy before enabling blocking mode.

## Operational procedure

1. Create the profile first — the actual detection technique (signature, fingerprint, rate-based).
2. Create the policy — the traffic-match expression.
3. Bind the policy in log-only/inspect mode.
4. Observe for false positives before switching to block mode — placed straight into blocking mode, a bot profile turns any false positive directly into a real outage for a real user, with no observation window to catch it first.

## Patterns

- **Profile → policy → binding**, the same action-then-match-then-attach shape as other NetScaler policy-driven features.
- **Log-only before blocking** — universal for anything that can reject real user traffic.

## Known limitations

- No offensive/destructive capability — this skill covers legitimate bot detection/mitigation configuration, never techniques for evading detection.
- No cross-object blast-radius reasoning — a bot policy's full traffic scope (which vservers it actually affects) isn't traced automatically once bound.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listBotpolicy` | List bot-management policies | Bot Management |
| `createBotpolicy` | Create a new bot policy | Bot Management |
| `updateBotpolicy` | Change an existing bot policy | Bot Management |
| `listBotprofile` | List bot-detection profiles (signature/fingerprint/rate-based) | Bot Management |
| `createBotprofile` | Create a new bot detection profile | Bot Management |
| `updateBotprofile` | Change an existing bot profile | Bot Management |
| `createBotpolicyCsvserverBinding` | Bind a bot policy to a content-switching vserver | Bot Management |
| `createBotpolicyLbvserverBinding` | Bind a bot policy to an LB vserver | Bot Management |

## Verification checklist

- [ ] New bot profile ran in log-only/inspect mode against real traffic before switching to blocking
- [ ] Bot policy binding confirmed present on the intended vserver(s), and only those
- [ ] After enabling blocking, confirmed no legitimate traffic pattern is being falsely flagged
