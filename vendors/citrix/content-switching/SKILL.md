---
name: citrix-content-switching
description: How to route NetScaler traffic by URL/host to different backends using content-switching virtual servers, actions, and policies. Vendor-neutral. Use when building, reviewing, or debugging NetScaler content-switching automation, on Itential or otherwise.
---

# Citrix NetScaler — Content Switching

## When to use this skill

- Routing by URL/host to different backends.
- Reviewing or debugging why a content-switching rule isn't matching as expected.

## Operational procedure

1. Create the CS vserver first — it's the entry point everything else attaches to.
2. Create a CS action naming the target LB vserver or content group a policy should route to.
3. Create the CS policy — the match expression (e.g. "if URL starts with /api").
4. Bind the policy to the CS vserver with an explicit priority. Lower priority numbers evaluate first — get this ordering wrong and a broad catch-all rule can silently shadow a more specific one that never fires.
5. List the full set of bindings back and confirm the priority order matches intent.

## Patterns

- **Action → policy → binding.** Build the thing that acts first (the CS action), the thing that matches second (the CS policy), and the attachment point last (the binding with a priority).
- **Priority order is load-bearing.** A broad catch-all policy bound at a lower priority number than a specific one will always win — always review the full priority ordering after any bind, not just the individual bind's success.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover responder or rewrite policies, which use the same vserver but are configured and bound independently — see the `responder-policies` and `rewrite-policies` skills.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
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

## Verification checklist

- [ ] After binding a CS policy, listed the full set of bindings and confirmed priority order matches intent — not just that this one bind succeeded
- [ ] Confirmed the LB vserver target behind the CS vserver is correct and healthy
- [ ] Tested with a real request matching the specific policy's expression, not just a broad catch-all
