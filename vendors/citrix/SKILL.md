---
name: citrix-netscaler
description: Operational procedures for Citrix NetScaler (ADC) - load balancing, traffic routing, SSL, GSLB, security/access, remote access, system administration, core networking, clustering/HA, DNS, bot management, and traffic optimization/analytics. Vendor-neutral - describes how to actually perform each operation on NetScaler, regardless of what orchestrates it. Use when building, reviewing, or debugging any NetScaler automation, on Itential or otherwise.
---

# Citrix NetScaler — Operational Skill Reference

This document teaches the real NetScaler procedure for each capability area — the sequencing, decision points, and failure modes a domain expert would tell you about, independent of what tool or platform executes the API calls. It does not assume Itential. Section 4 (Itential Reference Implementation) is a bonus for readers who happen to be on that platform — everything before it stands on its own.

## When to use this skill

- Designing or reviewing any automation (agent, script, workflow) that touches NetScaler load balancing, traffic routing, SSL, GSLB, security, remote access, system administration, networking, clustering/HA, DNS, bot management, or traffic optimization.
- Debugging why a NetScaler change didn't have the expected effect, even though the API call that made it reported success.
- Deciding what order to sequence a set of NetScaler configuration changes in.

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

**Responder policies** (redirect, block, custom response): decide the bind point deliberately — global, a specific CS vserver, or a specific LB vserver — because the *practical effect* of the same policy changes entirely depending on where it's bound. A globally-bound responder policy affects all traffic through the appliance, not just one application. Build the action (what to actually do) before the policy (the match expression), and the policy before any binding.

*Product-level gotcha:* NetScaler's NITRO API auto-generates binding resources in both directions for many relationships (e.g., both "responder policy → LB vserver" and "LB vserver → responder policy" exist as schema-valid, POST-able resources). In practice, only the vserver-owned direction has a real backing CLI command on the appliance — the policy-owned direction can pass request validation and still fail at runtime with a "no such command" class of error. Before wiring a new binding type into any automation, confirm which direction the appliance actually executes by testing a minimal real bind and checking the result — don't assume from the resource name or schema validity alone.

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

## 4. Itential reference implementation

If you're building on the Itential Platform, the procedures above are already implemented, built, and `GET`-verified — 13 projects, 33 agents, real exports in `projects/`. This section maps procedure → what's already done for you. Every write-capable agent listed here implements the propose-exact-change → human-approval → act-only-on-approval pattern via the `view:WorkFlowEngine:ViewData` tool; none of it executes unattended.

| Procedure area | Project file | Agent(s) | Tools (method names, `NetScaler:` prefix implied) |
|---|---|---|---|
| Load balancing | `load-balancing.project.json` | LB VServer Agent | `getLbvserverByName`, `listLbvserver`, `createLbvserver`, `updateLbvserver` |
| | | LB Service & Service Group Agent | `listService`, `createService`, `updateService`, `listServicegroup`, `createServicegroup`, `updateServicegroup` |
| | | LB Binding & Health Monitor Agent | `createLbvserverServiceBinding`, `createLbvserverServicegroupBinding`, `createLbmonitor`, `updateLbmonitor`, `listLbmonitor`, `createServiceLbmonitorBinding` |
| Content switching | `traffic-routing.project.json` | CS VServer & Policy Agent | `listCsvserver`, `createCsvserver`, `updateCsvserver`, `createCsaction`, `updateCsaction`, `createCspolicy`, `updateCspolicy`, `listCspolicy` |
| | | CS Binding Agent | `createCsvserverCspolicyBinding`, `listCsvserverCspolicyBinding`, `createCsvserverLbvserverBinding`, `listCsvserverLbvserverBinding` |
| Responder policies | `traffic-routing.project.json` | Responder Policy Agent | `createResponderaction`, `updateResponderaction`, `createResponderpolicy`, `updateResponderpolicy`, `listResponderpolicy`, `createResponderpolicyCsvserverBinding`, `createLbvserverResponderpolicyBinding` (the working direction — see the product-level gotcha above), `listLbvserverResponderpolicyBinding` |
| Rewrite policies | `traffic-routing.project.json` | Rewrite Policy Agent | `createRewriteaction`, `updateRewriteaction`, `createRewritepolicy`, `updateRewritepolicy`, `listRewritepolicy` |
| Policy building blocks (pattern sets/datasets/string maps) | `traffic-routing.project.json` | Policy Building Blocks Agent | `listPolicydataset`, `createPolicydataset`, `createPolicydatasetValueBinding`, `listPolicypatset`, `createPolicypatset`, `createPolicypatsetPatternBinding`, `listPolicystringmap`, `createPolicystringmap`, `createPolicystringmapPatternBinding` |
| SSL certificates | `ssl-certificates.project.json` | SSL Certificate Agent | `listSslcertkey`, `getSslcertkeyByName`, `createSslcertkey`, `updateSslcertkey`, `createSslvserverSslcertkeyBinding`, `listSslvserverSslcertkeyBinding` |
| GSLB / multi-site | `gslb-multi-site.project.json` | GSLB VServer & Service Agent | `listGslbvserver`, `createGslbvserver`, `updateGslbvserver`, `listGslbservice`, `createGslbservice`, `updateGslbservice` |
| | | GSLB Site & Domain Binding Agent | `createGslbsite`, `listGslbsite`, `createGslbvserverServiceBinding`, `listGslbvserverServiceBinding`, `createGslbvserverDomainBinding`, `listGslbvserverDomainBinding` |
| WAF | `security-access.project.json` | WAF Policy Agent | `listAppfwprofile`, `createAppfwprofile`, `updateAppfwprofile`, `listAppfwpolicy`, `createAppfwpolicy`, `updateAppfwpolicy`, `createLbvserverAppfwpolicyBinding`, `createCsvserverAppfwpolicyBinding` |
| Authentication | `security-access.project.json` | Authentication Policy Agent | `listAuthenticationldappolicy`, `createAuthenticationldappolicy`, `listAuthenticationradiuspolicy`, `createAuthenticationradiuspolicy`, `listAuthenticationvserver`, `createAuthenticationvserver`, `createAuthenticationvserverAuthenticationldappolicyBinding`, `createAuthenticationvserverAuthenticationradiuspolicyBinding` |
| Rate limiting | `security-access.project.json` | Rate Limiting Agent | `listNslimitidentifier`, `createNslimitidentifier`, `listNslimitselector`, `createNslimitselector` |
| Monitoring & diagnostics | `monitoring-diagnostics.project.json` | Vserver & Service Health Agent (read-only) | `getNsversionByName`, `getHanodeByName`, `listHanode`, `listLbvserver`, `getLbvserverByName`, `listService`, `listServicegroup` |
| | | Config & Certificate Diagnostics Agent (read-only) | `listCsvserver`, `listResponderpolicy`, `listRewritepolicy`, `listGslbvserver`, `listSslcertkey`, `getSslcertkeyByName` |
| SSL VPN Gateway | `gateway-remote-access.project.json` | SSL VPN Gateway Agent | `listVpnvserver`, `createVpnvserver`, `updateVpnvserver`, `createVpnglobalAuthenticationldappolicyBinding`, `createVpnglobalAuthenticationradiuspolicyBinding`, `createVpnglobalIntranetapplicationBinding` |
| ICA proxy | `gateway-remote-access.project.json` | ICA Proxy Agent | `listIcapolicy`, `createIcapolicy`, `updateIcapolicy`, `createIcaaction`, `updateIcaaction`, `createIcapolicyVpnvserverBinding`, `listIcapolicyVpnvserverBinding` |
| RBAC | `system-administration.project.json` | RBAC Agent | `listSystemuser`, `createSystemuser`, `updateSystemuser`, `listSystemgroup`, `createSystemgroup`, `createSystemuserSystemgroupBinding`, `listSystemcmdpolicy`, `createSystemcmdpolicy`, `createSystemuserSystemcmdpolicyBinding` |
| Feature & licensing | `system-administration.project.json` | Feature & Licensing Agent | `listNsfeature`, `updateNsfeature`, `listNsmode`, `updateNsmode`, `listNslicense`, `createNslicense` |
| Config backup | `system-administration.project.json` | Config Backup Agent | `listSystembackup`, `createSystembackup`, `getSystembackupByName` |
| IP & VLAN | `networking-fundamentals.project.json` | IP & VLAN Agent | `listNsip`, `createNsip`, `updateNsip`, `listVlan`, `createVlan`, `updateVlan`, `createVlanInterfaceBinding`, `createVlanNsipBinding` |
| Routing & interfaces | `networking-fundamentals.project.json` | Routing & Interfaces Agent | `listRoute`, `createRoute`, `updateRoute`, `deleteRoute`, `listInterface`, `updateInterface`, `createInterfacepair` |
| LACP channels | `networking-fundamentals.project.json` | Channel (LACP) Agent | `createChannel`, `updateChannel`, `createChannelInterfaceBinding` |
| NAT | `networking-fundamentals.project.json` | NAT Agent | `listInat`, `createInat`, `updateInat`, `listRnat`, `createRnat`, `updateRnat` |
| Clustering | `clustering-ha.project.json` | Clustering Agent | `listCluster`, `createCluster`, `listClusterinstance`, `createClusterinstance`, `listClusternode`, `createClusternode`, `createClusterinstanceClusternodeBinding` |
| HA pairing | `clustering-ha.project.json` | HA Pair Agent | `listHanode`, `createHanode`, `updateHanode` |
| DNS forward records | `dns-services.project.json` | Forward Record Agent (A/CNAME) | `listDnsaddrec`, `createDnsaddrec`, `updateDnsaddrec`, `listDnscnamerec`, `createDnscnamerec`, `updateDnscnamerec` |
| DNS zone/reverse records | `dns-services.project.json` | Zone & Reverse Record Agent (NS/PTR/SOA) | `listDnsnsrec`, `createDnsnsrec`, `listDnsptrrec`, `createDnsptrrec`, `listDnssoarec`, `createDnssoarec` |
| Bot management | `bot-management.project.json` | Bot Management Agent | `listBotpolicy`, `createBotpolicy`, `updateBotpolicy`, `listBotprofile`, `createBotprofile`, `updateBotprofile`, `createBotpolicyCsvserverBinding`, `createBotpolicyLbvserverBinding` |
| Caching & compression | `traffic-optimization-analytics.project.json` | Caching & Compression Agent | `listCachepolicy`, `createCachepolicy`, `updateCachepolicy`, `createCachecontentgroup`, `createCachepolicyLbvserverBinding`, `listCmppolicy`, `createCmppolicy`, `updateCmppolicy`, `createCmppolicyLbvserverBinding` |
| Performance profiles & spillover | `traffic-optimization-analytics.project.json` | Performance Profiles & Spillover Agent | `listNstcpprofile`, `createNstcpprofile`, `updateNstcpprofile`, `listNshttpprofile`, `createNshttpprofile`, `updateNshttpprofile`, `listSpilloverpolicy`, `createSpilloverpolicy`, `createSpilloverpolicyLbvserverBinding` |
| AppFlow analytics | `traffic-optimization-analytics.project.json` | AppFlow Analytics Agent | `listAppflowpolicy`, `createAppflowpolicy`, `updateAppflowpolicy`, `createAppflowaction`, `updateAppflowaction`, `createAppflowcollector`, `createAppflowpolicyCsvserverBinding` |

Every mutating agent above also has `view:WorkFlowEngine:ViewData` in its tool list (omitted from the table for brevity) — the two read-only Monitoring & Diagnostics agents do not.

## Gotchas

**Product-level (vendor-neutral — true on any NetScaler, regardless of what orchestrates it):**

- The responder-to-LB-vserver binding direction issue described in the traffic-routing procedure above: only `lbvserver_responderpolicy_binding` (vserver-owned) has a real backing command; `responderpolicy_lbvserver_binding` (policy-owned) is schema-valid and fails at runtime. Confirmed against the real Citrix NetScaler NITRO 14.1 OpenAPI spec, not just inferred from the runtime error.

**Itential-implementation-level (specific to how this was wired up on Itential's Agent Project Service / Tools Service — not a NetScaler behavior):**

- **Two app-name variants for the same integration — one dead, one live.** This platform's NetScaler adapter registered a stale `Nitro` catalog (`active: false`, ~150 methods, pre-refresh) alongside the current `NetScaler` catalog (`active: true`, ~6,600 methods). `GET /tools?name=<method>` can return both, and list order is not active-first — a naive first-result pick can silently wire the dead variant onto an agent. The failure is silent at create time: the tool entry lands as `unauthorizedReferenceId` instead of `referenceId`, and never resolves at run time. *Fix:* filter for `referenceId` containing `:NetScaler:` (not `:Nitro:`) and `active == true`; after creating/updating any agent, `GET` it back and confirm zero `unauthorizedReferenceId` entries.
- **Bundle-import `providerResolutions` (keyed by profile name) did not reliably resolve `provider`.** Agents came back with `provider: null` after `POST /agent-project-service/project-bundles/import` even with a correctly-named `providerResolutions` entry. *Fix:* create agents individually via `POST /agent-project-service/projects/{projId}/agents` with `provider: {profile: <uuid>, model: <uuid>}` passed directly, which resolves reliably; if using bundle import anyway, `GET` every agent afterward and `PATCH` in the direct UUIDs for any that came back null.

## Verification checklist

**Product-level (confirm the real-world state actually changed — on any platform):**

- [ ] After any create/bind, re-read the object's own state via a `list`/`get` call — don't trust the create call's HTTP success alone
- [ ] For anything bound to a vserver, confirm the vserver's *aggregate* state as well as the individual bound object's state
- [ ] For SSL cert changes, confirm the live TLS handshake presents the expected certificate, not just what the config object claims
- [ ] For anything with an "enable blocking/mode" step (WAF, bot management), confirm it ran in observe/log-only mode against real traffic first

**Itential-implementation-level (if using the reference implementation in section 4):**

- [ ] Every `tools[]` entry on the agent has a `referenceId` field, never `unauthorizedReferenceId`
- [ ] Every NetScaler `referenceId` contains `:NetScaler:`, never `:Nitro:`
- [ ] `provider` is a populated object with both `profile` and `model` UUIDs — never `null`
- [ ] Agent tool count is ≤ 10 — if not, it's covering more than one procedure and should be split
- [ ] If the agent can mutate state, `view:WorkFlowEngine:ViewData` is present and its `instructions` describe propose → approve → act explicitly
- [ ] No `POST /agent-session-manager/sessions` or `run-agent` call was made unless a live test run was actually intended
