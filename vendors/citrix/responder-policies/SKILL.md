---
name: citrix-responder-policies
description: How to configure NetScaler responder policies to redirect, block, or custom-respond to requests, including the vserver-owned vs. policy-owned LB-binding gotcha. Vendor-neutral. Use when building, reviewing, or debugging NetScaler responder-policy automation, on Itential or otherwise.
---

# Citrix NetScaler — Responder Policies

## When to use this skill

- Redirecting, blocking, or custom-responding to requests.
- Debugging why a responder policy isn't firing, or bound in the wrong scope.

## Operational procedure

1. Decide the bind point deliberately — global, a specific CS vserver, or a specific LB vserver. The practical effect of the same policy changes entirely depending on where it's bound; a globally-bound policy affects all traffic through the appliance, not just one application.
2. Build the responder action first — what to actually do (redirect, respond-with, drop).
3. Build the responder policy next — the match expression that triggers the action.
4. Bind the policy to the chosen scope. When binding to an LB vserver specifically, bind from the vserver's side (the vserver-owned resource) — the API also exposes the same relationship from the policy's side, but only the vserver-owned direction has a real command behind it on the appliance; the other passes validation and fails at runtime.
5. Confirm which direction actually works with a minimal test bind before relying on it in automation.

## Patterns

- **Action → policy → binding.**
- **Binding-resource naming reflects the API's schema generation, not necessarily what the appliance actually implements.** NITRO auto-generates a resource for each direction of many relationships — verify the real, working direction with an actual test bind rather than trusting the resource name or schema validity alone.

## Known limitations

- No offensive/destructive capability.
- No cross-object blast-radius reasoning — a globally-bound responder policy's full traffic impact isn't traced automatically; review that explicitly before binding globally.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `createResponderaction` | Create a responder action (what to actually do: redirect, respond-with, drop) | Responder |
| `updateResponderaction` | Change an existing responder action | Responder |
| `createResponderpolicy` | Create a responder policy (the match expression that triggers the action) | Responder |
| `updateResponderpolicy` | Change an existing responder policy's rule | Responder |
| `listResponderpolicy` | List responder policies | Responder |
| `createResponderpolicyCsvserverBinding` | Bind a responder policy to a content-switching vserver | Responder Binding |
| `createLbvserverResponderpolicyBinding` | Bind a responder policy to an LB vserver — **this is the vserver-owned direction that actually has a real backing command**; the policy-owned direction (`createResponderpolicyLbvserverBinding`) is schema-valid but returns NetScaler error 1088 at runtime and should not be used | Responder Binding |
| `listLbvserverResponderpolicyBinding` | List responder policies bound to an LB vserver | Responder Binding |

## Verification checklist

- [ ] Bind point (global/CS vserver/LB vserver) confirmed to be the intended scope, not a broader default
- [ ] For any responder policy bound to an LB vserver, confirmed via the vserver-owned binding direction (`createLbvserverResponderpolicyBinding`), not the policy-owned one
- [ ] Tested with a real request matching the policy's expression to confirm the action actually fires
