---
name: citrix-traffic-routing
description: How to configure NetScaler content switching, responder policies, rewrite policies, and the shared policy building blocks (datasets, pattern sets, string maps) that back them. Vendor-neutral. Use when building, reviewing, or debugging NetScaler traffic-routing automation, on Itential or otherwise.
---

# Citrix NetScaler — Traffic Routing (Content Switching, Responder, Rewrite)

## When to use this skill

- Routing by URL/host to different backends (content switching).
- Redirecting, blocking, or custom-responding to requests (responder policies).
- Modifying request/response headers or content (rewrite policies).
- Building the datasets/pattern sets/string maps that policy expressions reference.

## Operational procedure

**Content switching**: create the CS vserver first — it's the entry point everything else attaches to. Then create CS actions (each names a target LB vserver or content group) and CS policies (the match expression), then bind policies to the CS vserver with an explicit priority. Lower priority numbers evaluate first; get this ordering wrong and a broad catch-all rule can silently shadow a more specific one that never fires.

**Responder policies**: decide the bind point deliberately — global, a specific CS vserver, or a specific LB vserver — because the practical effect of the same policy changes entirely depending on where it's bound. A globally-bound responder policy affects all traffic through the appliance, not just one application. Build the action (what to actually do) before the policy (the match expression), and the policy before any binding. When binding a responder policy to an LB vserver specifically, bind from the vserver's side (the vserver-owned resource) — the API also exposes the same relationship from the policy's side, but only the vserver-owned direction has a real command behind it on the appliance; the other passes validation and fails at runtime. Confirm which direction actually works with a minimal test bind before relying on it in automation.

**Rewrite policies**: same action-then-policy-then-bind sequence. Rewrite changes are traffic-visible immediately and apply in-line to real requests — always validate against a non-production vserver first; a bad rewrite expression doesn't fail loudly, it just serves subtly wrong content.

**Policy building blocks**: datasets, pattern sets, and string maps are referenced by policy expressions across content switching, responder, rewrite, and other policy types — create the building block before any policy that references it, since a policy expression referencing a not-yet-created dataset/patset/stringmap fails validation at policy-create time.

## Patterns

- **Action → policy → binding, almost universally.** Build the thing that acts first, the thing that matches second, and the attachment point last.
- **Binding-resource naming reflects the API's schema generation, not necessarily what the appliance actually implements.** NITRO auto-generates a resource for each direction of many relationships — verify the real, working direction with an actual test bind rather than trusting the resource name or schema validity alone (see the responder-binding gotcha above).

## Known limitations

- No offensive/destructive capability.
- No cross-object blast-radius reasoning — e.g., a globally-bound responder policy's full traffic impact isn't traced automatically; review that explicitly before binding globally.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

### Content Switching

| Operation | Plain-English description | Category |
|---|---|---|
| `listCsvserver` | List content-switching virtual servers | CS VServer |
| `createCsvserver` | Create a new content-switching vserver (the entry point that routes to different backends by URL/host) | CS VServer |
| `updateCsvserver` | Change an existing CS vserver's configuration | CS VServer |
| `createCsaction` | Create a CS action (names the target LB vserver or content group a policy routes to) | CS Policy |
| `updateCsaction` | Change an existing CS action | CS Policy |
| `createCspolicy` | Create a CS policy (the match expression — e.g. "if URL starts with /api") | CS Policy |
| `updateCspolicy` | Change an existing CS policy's match rule | CS Policy |
| `listCspolicy` | List CS policies | CS Policy |
| `createCsvserverCspolicyBinding` | Attach a CS policy to a CS vserver, with a priority | CS Binding |
| `listCsvserverCspolicyBinding` | List which CS policies are bound to a CS vserver | CS Binding |
| `createCsvserverLbvserverBinding` | Attach an LB vserver as a target behind a CS vserver | CS Binding |
| `listCsvserverLbvserverBinding` | List which LB vservers are bound behind a CS vserver | CS Binding |

### Responder Policies

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

### Rewrite Policies

| Operation | Plain-English description | Category |
|---|---|---|
| `createRewriteaction` | Create a rewrite action (how to modify a request/response — header or body) | Rewrite |
| `updateRewriteaction` | Change an existing rewrite action | Rewrite |
| `createRewritepolicy` | Create a rewrite policy (the match expression that triggers the rewrite) | Rewrite |
| `updateRewritepolicy` | Change an existing rewrite policy's rule | Rewrite |
| `listRewritepolicy` | List rewrite policies | Rewrite |

### Policy Building Blocks

| Operation | Plain-English description | Category |
|---|---|---|
| `listPolicydataset` | List datasets (typed value collections referenced by policy expressions) | Policy Building Blocks |
| `createPolicydataset` | Create a new dataset | Policy Building Blocks |
| `createPolicydatasetValueBinding` | Add a value into an existing dataset | Policy Building Blocks |
| `listPolicypatset` | List pattern sets (string-pattern collections referenced by policy expressions) | Policy Building Blocks |
| `createPolicypatset` | Create a new pattern set | Policy Building Blocks |
| `createPolicypatsetPatternBinding` | Add a pattern into an existing pattern set | Policy Building Blocks |
| `listPolicystringmap` | List string maps (key→value lookup tables referenced by policy expressions) | Policy Building Blocks |
| `createPolicystringmap` | Create a new string map | Policy Building Blocks |
| `createPolicystringmapPatternBinding` | Add a key/value pair into an existing string map | Policy Building Blocks |

## Verification checklist

- [ ] After binding a CS/responder/rewrite policy, list the binding back and confirm priority/order, not just that the bind call succeeded
- [ ] For any responder policy bound to an LB vserver, confirmed via the vserver-owned binding direction (`createLbvserverResponderpolicyBinding`), not the policy-owned one
- [ ] For a globally-bound responder/rewrite policy, confirmed the full traffic scope affected matches intent
- [ ] Rewrite/responder changes validated against a non-production vserver before wide rollout
