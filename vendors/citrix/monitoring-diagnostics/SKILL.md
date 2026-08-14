---
name: citrix-monitoring-diagnostics
description: How to correctly sequence NetScaler health/status checks — vserver aggregate state, bound service state, monitor probe results, HA and certificate status. Vendor-neutral, read-only. Use when debugging why a NetScaler-fronted application appears down or degraded, on Itential or otherwise.
---

# Citrix NetScaler — Monitoring & Diagnostics

## When to use this skill

- "Is X up?" questions about a load-balanced or GSLB application.
- HA status checks.
- Certificate expiry reporting.
- Debugging why a NetScaler change didn't have the expected effect even though the API reported success.

## Operational procedure

This is inherently read-only, but the *order* of what to check matters:

1. Check the vserver's own aggregate state first.
2. Then check each individually bound service's state — a vserver can report a healthy aggregate state even when one specific critical service is down, depending on the vserver's configured down-service threshold.
3. Then check the monitor's actual last-probe result and reason string for that service, not just an up/down flag — the reason string is usually what tells you *why* (timeout vs. explicit failure response vs. certificate error), which determines what to fix.
4. Only escalate past the appliance once all three of the above have been checked — most "the app is down" reports trace back to one specific service or monitor, not the vserver or the appliance as a whole.

## Patterns

- **Aggregate state → individual bound object state → underlying probe/reason**, in that order, for any "is it up" question on this platform.

## Known limitations

- Read-only by design — this skill diagnoses, it doesn't remediate. Any fix identified here belongs in the relevant configuration skill (load balancing, GSLB, SSL certificates, etc.).

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `getNsversionByName` | Get the NetScaler appliance's firmware version | Appliance Health |
| `getHanodeByName` | Get a single HA node's status | HA Status |
| `listHanode` | List all HA nodes and their sync/failover state | HA Status |
| `listLbvserver` | List all LB virtual servers (used here for health visibility) | VServer Health |
| `getLbvserverByName` | Get a single LB vserver's current state | VServer Health |
| `listService` | List backend services (used here for health visibility) | Service Health |
| `listServicegroup` | List service groups (used here for health visibility) | Service Health |
| `listCsvserver` | List content-switching vservers (used here for config visibility) | Config Visibility |
| `listResponderpolicy` | List responder policies (used here for config visibility) | Config Visibility |
| `listRewritepolicy` | List rewrite policies (used here for config visibility) | Config Visibility |
| `listGslbvserver` | List GSLB vservers (used here for config visibility) | Config Visibility |
| `listSslcertkey` | List SSL certificates (used here for expiry visibility) | Certificate Health |
| `getSslcertkeyByName` | Get a single certificate's expiry/status | Certificate Health |

## Verification checklist

- [ ] Vserver aggregate state checked first
- [ ] Each individually bound service's state checked, not inferred from the aggregate
- [ ] Monitor's last-probe reason string read, not just the up/down flag
- [ ] Escalation past the appliance only happens after all of the above are ruled out
