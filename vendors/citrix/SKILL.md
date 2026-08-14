---
name: citrix-netscaler
description: Operational procedures for Citrix NetScaler (ADC) - load balancing, traffic routing, SSL, GSLB, security/access, remote access, system administration, core networking, clustering/HA, DNS, bot management, and traffic optimization/analytics. Vendor-neutral - describes how to actually perform each operation on NetScaler, regardless of what orchestrates it. Use when building, reviewing, or debugging any NetScaler automation, on Itential or otherwise.
---

# Citrix NetScaler — Operational Skill Reference

This document teaches the real NetScaler procedure for each capability area — the sequencing, decision points, and failure modes a domain expert would tell you about, independent of what tool or platform executes the API calls. It doesn't assume any particular orchestration platform — the exhaustive operation lookup lives in `API-REFERENCE.md` alongside this file.

## When to use this skill

- Designing or reviewing any automation (agent, script, workflow) that touches NetScaler load balancing, traffic routing, SSL, GSLB, security, remote access, system administration, networking, clustering/HA, DNS, bot management, or traffic optimization.
- Debugging why a NetScaler change didn't have the expected effect, even though the API call that made it reported success.
- Deciding what order to sequence a set of NetScaler configuration changes in.

**Capability index** — jump straight to the procedure that answers a specific request:

| A request is about... | Procedure |
|---|---|
| Standing up / scaling a load-balanced app | [Load balancing](#load-balancing) |
| Routing by URL/host to different backends | [Traffic routing — content switching](#traffic-routing--content-switching) |
| Redirecting, blocking, or custom-responding | [Traffic routing — responder policies](#traffic-routing--responder-policies) |
| Modifying request/response headers or content | [Traffic routing — rewrite policies](#traffic-routing--rewrite-policies) |
| Installing/renewing an SSL cert, checking expiry | [SSL certificates](#ssl-certificates) |
| Multi-site DR, geo-routing, GSLB failover | [GSLB / multi-site](#gslb--multi-site) |
| WAF, LDAP/RADIUS auth, rate limiting | [Security & access](#security--access--waf-authentication-rate-limiting) |
| "Is X up?", HA status, cert expiry reporting | [Monitoring & diagnostics](#monitoring--diagnostics) |
| SSL VPN / remote access, VDI access | [Gateway & remote access](#gateway--remote-access) |
| Admin RBAC, feature/licensing, config backup | [System administration](#system-administration) |
| IP/VLAN, routing/interfaces, LACP, NAT | [Networking fundamentals](#networking-fundamentals) |
| Multi-box clustering, HA pairing | [Clustering & high availability](#clustering--high-availability) |
| DNS record management | [DNS services](#dns-services) |
| Bot detection/mitigation | [Bot management](#bot-management) |
| Caching, compression, performance tuning, AppFlow | [Traffic optimization & analytics](#traffic-optimization--analytics) |

## Operational procedures

### Load balancing

Setting up a load-balanced application:

1. Decide the service type (HTTP, SSL, TCP, UDP, etc.) and persistence strategy (none, cookie, source-IP) up front. Service type is not an in-place edit on most NetScaler versions — changing it later typically means delete-and-recreate, not update.
2. Create the backend service(s) or service group **before** the virtual server. A vserver with nothing bound to it is valid but serves nothing — building it last avoids a window where the vserver exists and looks "up" with no real backend behind it.
3. Create the LB virtual server: VIP, port, service type, and load-balancing method (round robin, least connections, etc.).
4. Bind the service or service group to the vserver.
5. Attach a health monitor to the service (or service group). An unmonitored service defaults to a state NetScaler treats as reachable — without a real monitor, a backend can be fully down and the vserver won't know until a client request actually fails.
6. Verify by checking the vserver's own aggregate state, **and** each bound service's individual state and monitor result — not just that the create/bind calls returned success. A vserver can show a healthy aggregate state while one specific bound service is down, depending on threshold configuration.

### Traffic routing — content switching, responder, rewrite

**Content switching** (route by URL/host to different backends): create the CS vserver first — it's the entry point everything else attaches to. Then create CS actions (each names a target LB vserver or content group) and CS policies (the match expression), then bind policies to the CS vserver with an explicit priority. Lower priority numbers evaluate first; get this ordering wrong and a broad catch-all rule can silently shadow a more specific one that never fires.

**Responder policies** (redirect, block, custom response): decide the bind point deliberately — global, a specific CS vserver, or a specific LB vserver — because the *practical effect* of the same policy changes entirely depending on where it's bound. A globally-bound responder policy affects all traffic through the appliance, not just one application. Build the action (what to actually do) before the policy (the match expression), and the policy before any binding. When binding a responder policy to an LB vserver specifically, bind from the vserver's side (the vserver-owned resource) — the API also exposes the same relationship from the policy's side, but only the vserver-owned direction has a real command behind it on the appliance; the other passes validation and fails at runtime. Confirm which direction actually works with a minimal test bind before relying on it in automation.

**Rewrite policies** (modify request/response headers or content): same action-then-policy-then-bind sequence. Rewrite changes are traffic-visible immediately and apply in-line to real requests — always validate against a non-production vserver first; a bad rewrite expression doesn't fail loudly, it just serves subtly wrong content.

### SSL certificates

1. Stage the certificate and key files on the appliance (via whatever out-of-band file transfer the environment supports) before creating anything through the API — the cert-key object references files that must already exist.
2. Create the cert-key pair object pointing at the staged files.
3. Bind it to the target SSL vserver.
4. **Renewal**, specifically: create the *new* cert-key object under a distinct name first, bind it to the vserver, and confirm the live TLS handshake is actually presenting the new certificate (probe it directly — check the serial number and expiry a client actually receives, not just what the API's stored metadata says) before unbinding or deleting the old one. Deleting the old cert before confirming the new one is live risks an outage window with no valid cert bound at all.
5. **Expiry monitoring**: check expiry on a recurring schedule and treat an approaching date as a lead-time trigger to start the renewal procedure above — not a same-day emergency response.

### GSLB / multi-site

1. Create a GSLB site object for each physical/logical location first — this is what lets sites exchange health and metric information with each other; nothing else in GSLB works meaningfully without it.
2. Create the GSLB service(s) representing each site's local application endpoint, and bind a monitor to each.
3. Create the GSLB vserver (the global entry point, typically what a DNS name ultimately resolves through), bind the per-site services to it, and bind the domain.
4. **Failover**: disable (don't delete) the GSLB service at the site you're failing away from. Disabling is reversible and near-instant; deleting requires full recreation to fail back, which is the wrong tool for what's usually a temporary, urgent action.

### Security & access — WAF, authentication, rate limiting

**WAF (AppFirewall)**: create the profile (the actual ruleset/relaxations) before the policy (the match expression selecting which traffic the profile applies to). A profile with no policy bound to any vserver protects nothing. Start a new profile in log-only/non-blocking mode and run it against real traffic before switching to blocking — this is the only reliable way to catch false positives before they turn into a real outage for real users.

**Authentication (LDAP/RADIUS)**: create the action (the actual server connection: host, port, bind credentials, attribute mapping) before the policy (the expression selecting when it applies), before binding to an authentication or Gateway vserver. Test the raw LDAP/RADIUS connection independently — outside of any user-facing vserver — before wiring it into one; a misconfigured auth action bound directly to production blocks every real login attempt at once.

**Rate limiting**: define what's actually being counted first — the selector (e.g., per source IP, per URL) — before the limit identifier (the threshold and time window). A selector scoped incorrectly (for example, counting per unique client behind a NAT gateway serving many real users as one IP) throttles far more or less traffic than intended; validate the selector's real-world grouping behavior before trusting the threshold.

### Monitoring & diagnostics

This is inherently read-only, but the *order* of what to check matters:

1. Check the vserver's own aggregate state first.
2. Then check each individually bound service's state — a vserver can report a healthy aggregate state even when one specific critical service is down, depending on the vserver's configured down-service threshold.
3. Then check the monitor's actual last-probe result and reason string for that service, not just an up/down flag — the reason string is usually what tells you *why* (timeout vs. explicit failure response vs. certificate error), which determines what to fix.
4. Only escalate past the appliance once all three of the above have been checked — most "the app is down" reports trace back to one specific service or monitor, not the vserver or the appliance as a whole.

### Gateway & remote access

**SSL VPN (NetScaler Gateway)**: create the Gateway vserver, bind a trusted SSL certificate to it (remote users will not connect through an untrusted cert without a manual override), configure and bind the authentication policy so users can actually log in, then bind the intranet applications/resources users are allowed to reach. Access is deny-by-default — a Gateway vserver with no resources explicitly bound authenticates users into nothing.

**ICA proxy** (access to Citrix Virtual Apps/Desktops through Gateway): requires the Gateway vserver to already exist. Create the ICA action/policy pointing at the actual StoreFront/broker address, bind it to the vserver. Verify with a real client launch, not just a successful API bind — most ICA proxy failures happen on the StoreFront handshake side, which is invisible from the NetScaler configuration alone.

### System administration

**RBAC**: create the command policy (the actual permission boundary — what's allowed/denied) before the group, the group before the user, then bind user-to-group and group-to-policy. A user created but never bound to a group/policy is left with the appliance's default access level, which may be broader or narrower than intended depending on the appliance's own default policy — don't assume "unbound" means "no access."

**Feature/mode & licensing**: confirm a feature is actually licensed before attempting to enable it. Enabling an unlicensed feature usually fails cleanly, but some feature/license combinations fail in confusing, firmware-version-dependent ways — check license and feature status together, not as two independent steps.

**Config backup**: take a backup *immediately before* any RBAC or feature change, not after. A broken RBAC or feature change can itself lock out the access needed to restore from a backup taken too late to help.

### Networking fundamentals

**IP addressing & VLANs**: NetScaler distinguishes IP types (management, VIP, subnet-mapping) that behave differently — assign the correct *type* for the purpose, not just any free address. Create the VLAN before binding interfaces or IPs to it.

**Routing & interfaces**: verify an interface's actual physical/link state before relying on it in a route. A route bound to a down interface doesn't fail at creation time — it fails silently until traffic actually needs that path.

**LACP channels**: the NetScaler-side channel configuration and the upstream switch's LACP configuration must agree (same member ports, same negotiation mode) — a channel configured only on the NetScaler side will never form an active bundle if the switch side doesn't match it.

**NAT**: inbound NAT (INAT) and reverse NAT (RNAT) solve different directional problems and are easy to conflate — decide the direction and scope deliberately, and test from the actual originating network segment, not just from the appliance's own local perspective.

### Clustering & high availability

**Clustering**: establish the cluster instance and its config-sync approach before adding nodes — nodes joining a cluster inherit the cluster's existing configuration, so get the base configuration correct on one node first, then add nodes, rather than configuring after nodes have already joined.

**HA pairing**: verify the HA heartbeat has an independent network path before pairing two nodes. Pairing nodes whose heartbeat traverses the same single link as production traffic risks exactly the split-brain or simultaneous-failure scenario HA exists to prevent, under precisely the conditions (a link failure) HA is meant to survive.

### DNS services

**Forward records (A/CNAME)**: confirm the authoritative zone/SOA already exists and is correctly configured before adding records into it. A record added into a misconfigured or non-existent zone creates successfully but silently never resolves for anyone querying it externally.

**Zone & reverse records (NS/PTR/SOA)**: a PTR record must be added into the zone that actually corresponds to the IP (the correct `in-addr.arpa` or `ip6.arpa` zone) — a PTR record that "creates successfully" in the wrong zone will never be found by a real reverse lookup, and there's no error at creation time to signal the mismatch.

### Bot management

Create the profile (the actual detection technique — signature, fingerprint, rate-based) before the policy (the traffic-match expression). Start any new profile in log-only/inspect mode before switching it to block — a bot profile placed straight into blocking mode turns any false positive directly into a real outage for a real user, with no observation window to catch it first.

### Traffic optimization & analytics

**Caching**: verify a response is genuinely cacheable (correct cache-control headers, no session-specific or personalized content) before creating a cache policy for it. Caching a personalized or session-bound response is a data-leak risk between users, not just a wasted optimization.

**Compression**: validate the CPU-for-bandwidth tradeoff on the actual appliance tier in use. Compression is real CPU cost traded for bandwidth savings, and lower-tier or virtual appliances can bottleneck on CPU before the bandwidth savings materialize — measure, don't assume the tradeoff is free.

**Spillover**: set the overflow threshold meaningfully below the backend's actual saturation point, not at it. Spillover configured to trigger only once the primary is already failing defeats the purpose of having it.

**AppFlow analytics**: verify the collector's own network reachability (it's a real UDP/TCP export target) before wiring an action/policy to it. A policy bound to an unreachable collector drops analytics silently — there's no application-visible error when this happens, so it's easy to believe analytics are flowing when they aren't.

## Patterns

- **Action → policy → binding, almost universally.** Nearly every NetScaler policy-driven feature (responder, rewrite, content switching, WAF, bot management, AppFlow) follows the same three-step shape: build the thing that *acts* first, the thing that *matches* second, and the *attachment point* last. Once you recognize this shape, most "what order do I do this in" questions answer themselves.
- **Binding-resource naming reflects the API's schema generation, not necessarily what the appliance actually implements.** NITRO auto-generates a resource for each direction of many relationships. Verify the real, working direction with an actual test bind rather than trusting the resource name or schema validity alone — see the content-switching/responder procedure above for the specific example that's been hit in practice.
- **Disable/re-enable beats delete/recreate for anything you might need to reverse under time pressure** (GSLB failover, maintenance windows, temporarily pulling a bad backend). Deletion is for permanent decommissioning, not for reversible operational actions.
- **Stage-then-attach, not attach-then-configure**, for anything involving external files or connections (SSL certs, LDAP/RADIUS servers, AppFlow collectors, StoreFront/broker addresses for ICA). Confirm the external dependency is reachable/valid on its own before wiring it into a live-traffic-serving object.

## Known limitations

- **No offensive/destructive capability.** This skill covers configuration and delivery — never traffic manipulation for denial-of-service, credential harvesting, or evasion of the appliance's own security controls.
- **Failover/HA procedures cover configuration, not live incident execution.** Forcing a failover during an actual outage is a higher-stakes action than a routine configuration change; treat the clustering/HA procedures above as setup guidance, not an incident-response runbook.
- **No cross-object blast-radius reasoning.** These procedures describe how to look up the state directly relevant to one action, not how to trace a full dependency graph (e.g., everything a VLAN change might affect). Whoever reviews a proposed change is still the backstop for catching that.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

### Load Balancing

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

### Traffic Routing — Content Switching

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

### Traffic Routing — Responder Policies

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

### Traffic Routing — Rewrite Policies

| Operation | Plain-English description | Category |
|---|---|---|
| `createRewriteaction` | Create a rewrite action (how to modify a request/response — header or body) | Rewrite |
| `updateRewriteaction` | Change an existing rewrite action | Rewrite |
| `createRewritepolicy` | Create a rewrite policy (the match expression that triggers the rewrite) | Rewrite |
| `updateRewritepolicy` | Change an existing rewrite policy's rule | Rewrite |
| `listRewritepolicy` | List rewrite policies | Rewrite |

### Traffic Routing — Policy Building Blocks

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

### SSL Certificates

| Operation | Plain-English description | Category |
|---|---|---|
| `listSslcertkey` | List installed SSL certificate-key pairs | SSL Certificate |
| `getSslcertkeyByName` | Look up a single certificate-key pair by name (includes expiry) | SSL Certificate |
| `createSslcertkey` | Install a new certificate-key pair (from files already staged on the appliance) | SSL Certificate |
| `updateSslcertkey` | Update an existing certificate-key pair's settings | SSL Certificate |
| `createSslvserverSslcertkeyBinding` | Bind a certificate to an SSL-enabled vserver | SSL Certificate |
| `listSslvserverSslcertkeyBinding` | List which certificate is bound to an SSL vserver | SSL Certificate |

### GSLB & Multi-Site

| Operation | Plain-English description | Category |
|---|---|---|
| `listGslbvserver` | List GSLB (Global Server Load Balancing) virtual servers — the global entry points for multi-site routing | GSLB VServer |
| `createGslbvserver` | Create a new GSLB vserver | GSLB VServer |
| `updateGslbvserver` | Change an existing GSLB vserver's configuration | GSLB VServer |
| `listGslbservice` | List GSLB services (per-site local endpoints) | GSLB Service |
| `createGslbservice` | Create a new GSLB service representing one site's endpoint | GSLB Service |
| `updateGslbservice` | Change an existing GSLB service's configuration — commonly used to disable a service for failover | GSLB Service |
| `createGslbsite` | Create a GSLB site — required before sites can exchange health/metric data with each other | GSLB Site |
| `listGslbsite` | List configured GSLB sites | GSLB Site |
| `createGslbvserverServiceBinding` | Attach a per-site GSLB service to a GSLB vserver | GSLB Binding |
| `listGslbvserverServiceBinding` | List which GSLB services are bound to a GSLB vserver | GSLB Binding |
| `createGslbvserverDomainBinding` | Bind a domain name to a GSLB vserver so DNS queries resolve through it | GSLB Binding |
| `listGslbvserverDomainBinding` | List which domains are bound to a GSLB vserver | GSLB Binding |

### Security & Access — WAF (AppFirewall)

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

### Security & Access — Authentication

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

### Security & Access — Rate Limiting

| Operation | Plain-English description | Category |
|---|---|---|
| `listNslimitidentifier` | List rate-limit identifiers (the threshold/time-window definitions) | Rate Limiting |
| `createNslimitidentifier` | Create a new rate-limit identifier | Rate Limiting |
| `listNslimitselector` | List rate-limit selectors (what's being counted — e.g. per source IP) | Rate Limiting |
| `createNslimitselector` | Create a new rate-limit selector | Rate Limiting |

### Monitoring & Diagnostics (read-only)

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

### Gateway & Remote Access — SSL VPN

| Operation | Plain-English description | Category |
|---|---|---|
| `listVpnvserver` | List NetScaler Gateway (SSL VPN) virtual servers | Gateway |
| `createVpnvserver` | Create a new Gateway vserver | Gateway |
| `updateVpnvserver` | Change an existing Gateway vserver's configuration | Gateway |
| `createVpnglobalAuthenticationldappolicyBinding` | Bind an LDAP auth policy globally to the Gateway | Gateway |
| `createVpnglobalAuthenticationradiuspolicyBinding` | Bind a RADIUS auth policy globally to the Gateway | Gateway |
| `createVpnglobalIntranetapplicationBinding` | Grant Gateway users access to an internal application/resource | Gateway |

### Gateway & Remote Access — ICA Proxy

| Operation | Plain-English description | Category |
|---|---|---|
| `listIcapolicy` | List ICA policies (govern access to Citrix Virtual Apps/Desktops via Gateway) | ICA Proxy |
| `createIcapolicy` | Create a new ICA policy | ICA Proxy |
| `updateIcapolicy` | Change an existing ICA policy | ICA Proxy |
| `createIcaaction` | Create an ICA action (points at the StoreFront/broker address) | ICA Proxy |
| `updateIcaaction` | Change an existing ICA action | ICA Proxy |
| `createIcapolicyVpnvserverBinding` | Bind an ICA policy to a Gateway vserver | ICA Proxy |
| `listIcapolicyVpnvserverBinding` | List ICA policies bound to a Gateway vserver | ICA Proxy |

### System Administration — RBAC

| Operation | Plain-English description | Category |
|---|---|---|
| `listSystemuser` | List NetScaler admin user accounts | RBAC |
| `createSystemuser` | Create a new admin user account | RBAC |
| `updateSystemuser` | Change an existing admin user's settings | RBAC |
| `listSystemgroup` | List admin groups | RBAC |
| `createSystemgroup` | Create a new admin group | RBAC |
| `createSystemuserSystemgroupBinding` | Add a user to an admin group | RBAC |
| `listSystemcmdpolicy` | List command policies (the actual permission boundary) | RBAC |
| `createSystemcmdpolicy` | Create a new command policy | RBAC |
| `createSystemuserSystemcmdpolicyBinding` | Attach a command policy directly to a user | RBAC |

### System Administration — Feature & Licensing

| Operation | Plain-English description | Category |
|---|---|---|
| `listNsfeature` | List which NetScaler features are enabled/disabled | Feature & Licensing |
| `updateNsfeature` | Enable or disable a feature | Feature & Licensing |
| `listNsmode` | List appliance operating modes | Feature & Licensing |
| `updateNsmode` | Change an appliance operating mode | Feature & Licensing |
| `listNslicense` | List installed licenses | Feature & Licensing |
| `createNslicense` | Install a new license | Feature & Licensing |

### System Administration — Config Backup

| Operation | Plain-English description | Category |
|---|---|---|
| `listSystembackup` | List existing configuration backups | Config Backup |
| `createSystembackup` | Trigger a new configuration backup | Config Backup |
| `getSystembackupByName` | Get details of a specific backup | Config Backup |

### Networking Fundamentals — IP & VLAN

| Operation | Plain-English description | Category |
|---|---|---|
| `listNsip` | List IP addresses configured on the appliance | IP & VLAN |
| `createNsip` | Add a new IP address | IP & VLAN |
| `updateNsip` | Change an existing IP address's settings | IP & VLAN |
| `listVlan` | List VLANs | IP & VLAN |
| `createVlan` | Create a new VLAN | IP & VLAN |
| `updateVlan` | Change an existing VLAN's settings | IP & VLAN |
| `createVlanInterfaceBinding` | Bind a physical/logical interface to a VLAN | IP & VLAN |
| `createVlanNsipBinding` | Bind an IP address to a VLAN | IP & VLAN |

### Networking Fundamentals — Routing & Interfaces

| Operation | Plain-English description | Category |
|---|---|---|
| `listRoute` | List static routes | Routing |
| `createRoute` | Create a new static route | Routing |
| `updateRoute` | Change an existing route | Routing |
| `deleteRoute` | Remove a static route | Routing |
| `listInterface` | List physical/logical network interfaces | Interfaces |
| `updateInterface` | Change an interface's settings | Interfaces |
| `createInterfacepair` | Create an interface pair (used for certain HA/forwarding configurations) | Interfaces |

### Networking Fundamentals — LACP Channels

| Operation | Plain-English description | Category |
|---|---|---|
| `createChannel` | Create a new LACP link-aggregation channel | LACP Channel |
| `updateChannel` | Change an existing channel's settings | LACP Channel |
| `createChannelInterfaceBinding` | Add a physical interface into a channel | LACP Channel |

### Networking Fundamentals — NAT

| Operation | Plain-English description | Category |
|---|---|---|
| `listInat` | List inbound NAT (INAT) rules | NAT |
| `createInat` | Create a new inbound NAT rule | NAT |
| `updateInat` | Change an existing inbound NAT rule | NAT |
| `listRnat` | List reverse NAT (RNAT) rules | NAT |
| `createRnat` | Create a new reverse NAT rule | NAT |
| `updateRnat` | Change an existing reverse NAT rule | NAT |

### Clustering & High Availability

| Operation | Plain-English description | Category |
|---|---|---|
| `listCluster` | List cluster configurations | Clustering |
| `createCluster` | Create a new cluster instance | Clustering |
| `listClusterinstance` | List cluster instances | Clustering |
| `createClusterinstance` | Create a new cluster instance record | Clustering |
| `listClusternode` | List nodes in a cluster | Clustering |
| `createClusternode` | Add a new node to a cluster | Clustering |
| `createClusterinstanceClusternodeBinding` | Attach a node to a specific cluster instance | Clustering |
| `listHanode` | List HA (active/passive pair) nodes | HA Pairing |
| `createHanode` | Configure a new HA node pairing | HA Pairing |
| `updateHanode` | Change an existing HA node's configuration | HA Pairing |

### DNS Services — Forward Records

| Operation | Plain-English description | Category |
|---|---|---|
| `listDnsaddrec` | List DNS A (address) records | DNS Forward Records |
| `createDnsaddrec` | Create a new A record | DNS Forward Records |
| `updateDnsaddrec` | Change an existing A record | DNS Forward Records |
| `listDnscnamerec` | List DNS CNAME (alias) records | DNS Forward Records |
| `createDnscnamerec` | Create a new CNAME record | DNS Forward Records |
| `updateDnscnamerec` | Change an existing CNAME record | DNS Forward Records |

### DNS Services — Zone & Reverse Records

| Operation | Plain-English description | Category |
|---|---|---|
| `listDnsnsrec` | List DNS NS (nameserver) records | DNS Zone Records |
| `createDnsnsrec` | Create a new NS record | DNS Zone Records |
| `listDnsptrrec` | List DNS PTR (reverse-lookup) records | DNS Reverse Records |
| `createDnsptrrec` | Create a new PTR record | DNS Reverse Records |
| `listDnssoarec` | List DNS SOA (zone authority) records | DNS Zone Records |
| `createDnssoarec` | Create a new SOA record | DNS Zone Records |

### Bot Management

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

### Traffic Optimization & Analytics — Caching & Compression

| Operation | Plain-English description | Category |
|---|---|---|
| `listCachepolicy` | List integrated-caching policies | Caching |
| `createCachepolicy` | Create a new caching policy | Caching |
| `updateCachepolicy` | Change an existing caching policy | Caching |
| `createCachecontentgroup` | Create a cache content group (defines what's cacheable and for how long) | Caching |
| `createCachepolicyLbvserverBinding` | Bind a caching policy to an LB vserver | Caching |
| `listCmppolicy` | List compression policies | Compression |
| `createCmppolicy` | Create a new compression policy | Compression |
| `updateCmppolicy` | Change an existing compression policy | Compression |
| `createCmppolicyLbvserverBinding` | Bind a compression policy to an LB vserver | Compression |

### Traffic Optimization & Analytics — Performance Profiles & Spillover

| Operation | Plain-English description | Category |
|---|---|---|
| `listNstcpprofile` | List TCP performance-tuning profiles | Performance Profiles |
| `createNstcpprofile` | Create a new TCP profile | Performance Profiles |
| `updateNstcpprofile` | Change an existing TCP profile | Performance Profiles |
| `listNshttpprofile` | List HTTP performance-tuning profiles | Performance Profiles |
| `createNshttpprofile` | Create a new HTTP profile | Performance Profiles |
| `updateNshttpprofile` | Change an existing HTTP profile | Performance Profiles |
| `listSpilloverpolicy` | List spillover (overflow) policies | Spillover |
| `createSpilloverpolicy` | Create a new spillover policy | Spillover |
| `createSpilloverpolicyLbvserverBinding` | Bind a spillover policy to an LB vserver | Spillover |

### Traffic Optimization & Analytics — AppFlow

| Operation | Plain-English description | Category |
|---|---|---|
| `listAppflowpolicy` | List AppFlow analytics-export policies | AppFlow |
| `createAppflowpolicy` | Create a new AppFlow policy | AppFlow |
| `updateAppflowpolicy` | Change an existing AppFlow policy | AppFlow |
| `createAppflowaction` | Create an AppFlow action (what to export and where) | AppFlow |
| `updateAppflowaction` | Change an existing AppFlow action | AppFlow |
| `createAppflowcollector` | Create an AppFlow collector (the real network target analytics get sent to) | AppFlow |
| `createAppflowpolicyCsvserverBinding` | Bind an AppFlow policy to a content-switching vserver | AppFlow |

## Verification checklist

Confirm the real-world state actually changed — regardless of what platform executed the change:

- [ ] After any create/bind, re-read the object's own state via a `list`/`get` call — don't trust the create call's HTTP success alone
- [ ] For anything bound to a vserver, confirm the vserver's *aggregate* state as well as the individual bound object's state
- [ ] For SSL cert changes, confirm the live TLS handshake presents the expected certificate, not just what the config object claims
- [ ] For anything with an "enable blocking/mode" step (WAF, bot management), confirm it ran in observe/log-only mode against real traffic first
