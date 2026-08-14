---
name: citrix-rewrite-policies
description: How to configure NetScaler rewrite policies to modify request/response headers or content. Vendor-neutral. Use when building, reviewing, or debugging NetScaler rewrite-policy automation, on Itential or otherwise.
---

# Citrix NetScaler — Rewrite Policies

## When to use this skill

- Modifying request/response headers or content.
- Debugging traffic that's subtly wrong after a rewrite change.

## Operational procedure

1. Build the rewrite action first — how to modify a request/response (header or body).
2. Build the rewrite policy next — the match expression that triggers it.
3. Bind the policy to the target vserver.
4. Validate against a non-production vserver first, always. Rewrite changes are traffic-visible immediately and apply in-line to real requests — a bad rewrite expression doesn't fail loudly, it just serves subtly wrong content.
5. Inspect a real request/response after binding to confirm the actual output.

## Patterns

- **Action → policy → binding.**
- **Validate against non-production first, always** — a rewrite mistake is silent, not loud; there's no error state that flags "this rewrite produced malformed output," only real traffic quietly serving wrong content.

## Known limitations

- No offensive/destructive capability.
- No cross-object blast-radius reasoning — a rewrite policy's full traffic scope isn't traced automatically once bound.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `createRewriteaction` | Create a rewrite action (how to modify a request/response — header or body) | Rewrite |
| `updateRewriteaction` | Change an existing rewrite action | Rewrite |
| `createRewritepolicy` | Create a rewrite policy (the match expression that triggers the rewrite) | Rewrite |
| `updateRewritepolicy` | Change an existing rewrite policy's rule | Rewrite |
| `listRewritepolicy` | List rewrite policies | Rewrite |

## Verification checklist

- [ ] Rewrite validated against a non-production vserver before wide rollout
- [ ] Real request/response inspected after binding to confirm the actual output, not just that the bind call succeeded
- [ ] Rewrite expression reviewed for scope — confirmed it matches only the intended traffic, not a broader pattern
