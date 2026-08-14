---
name: citrix-load-balancing
description: How to stand up, scale, and verify a NetScaler (Citrix ADC) load-balanced application — virtual servers, backend services/service groups, and health monitors. Vendor-neutral: describes the real NetScaler procedure regardless of what orchestrates it. Use when building, reviewing, or debugging NetScaler load balancing, on Itential or otherwise.
---

# Citrix NetScaler — Load Balancing

Real NetScaler procedure for standing up and verifying a load-balanced application — the sequencing, decision points, and failure modes a domain expert would flag, independent of what tool or platform executes the API calls.

## When to use this skill

- Standing up a new load-balanced application or scaling an existing one.
- Reviewing or debugging a load-balancing configuration that reports success but isn't behaving as expected.
- Deciding what order to create LB objects in.

## Operational procedure

1. Decide the service type (HTTP, SSL, TCP, UDP, etc.) and persistence strategy (none, cookie, source-IP) up front. Service type is not an in-place edit on most NetScaler versions — changing it later typically means delete-and-recreate, not update.
2. Create the backend service(s) or service group **before** the virtual server. A vserver with nothing bound to it is valid but serves nothing — building it last avoids a window where the vserver exists and looks "up" with no real backend behind it.
3. Create the LB virtual server: VIP, port, service type, and load-balancing method (round robin, least connections, etc.).
4. Bind the service or service group to the vserver.
5. Attach a health monitor to the service (or service group). An unmonitored service defaults to a state NetScaler treats as reachable — without a real monitor, a backend can be fully down and the vserver won't know until a client request actually fails.
6. Verify by checking the vserver's own aggregate state, **and** each bound service's individual state and monitor result — not just that the create/bind calls returned success. A vserver can show a healthy aggregate state while one specific bound service is down, depending on threshold configuration.

## Patterns

- **Backend before frontend.** Services/service groups exist before the vserver that fronts them, and monitors exist before you trust a service's state — building in the reverse order creates a window where things look up but aren't actually verified.

## Known limitations

- No offensive/destructive capability — this skill covers configuration and delivery only.
- No cross-object blast-radius reasoning — verifying one vserver's health doesn't trace every downstream consumer of it (a GSLB service pointing at it, a CS vserver behind it, etc.); check those separately if relevant.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `getLbvserverByName` | Look up a single LB virtual server by name | LB VServer |
| `listLbvserver` | List all LB virtual servers | LB VServer |
| `createLbvserver` | Create a new LB virtual server (VIP, port, service type, LB method) | LB VServer |
| `updateLbvserver` | Change an existing LB virtual server's configuration | LB VServer |
| `listService` | List backend services (individual servers behind a vserver) | LB Service |
| `createService` | Register a new backend service | LB Service |
| `updateService` | Change an existing backend service's configuration | LB Service |
| `listServicegroup` | List service groups (pools of backend servers) | LB Service |
| `createServicegroup` | Create a new service group | LB Service |
| `updateServicegroup` | Change an existing service group's configuration | LB Service |
| `createLbvserverServiceBinding` | Attach an individual backend service to an LB vserver | LB Binding |
| `createLbvserverServicegroupBinding` | Attach a service group to an LB vserver | LB Binding |
| `createLbmonitor` | Create a health monitor (HTTP/TCP/ping check, etc.) | LB Monitor |
| `updateLbmonitor` | Change an existing health monitor's configuration | LB Monitor |
| `listLbmonitor` | List health monitors | LB Monitor |
| `createServiceLbmonitorBinding` | Attach a health monitor to a backend service | LB Monitor |

## Verification checklist

- [ ] Vserver's own aggregate state confirmed via `getLbvserverByName`/`listLbvserver` — not just that `createLbvserver` returned success
- [ ] Each bound service's individual state checked, not just the vserver's aggregate state
- [ ] A real health monitor is bound and its last-probe result checked, not just that the service exists
