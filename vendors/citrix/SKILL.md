---
name: citrix-netscaler
description: FlowAI agents for Citrix NetScaler (ADC) operations - load balancing, traffic routing, SSL, GSLB, security/access, remote access, system administration, core networking, clustering/HA, DNS, bot management, and traffic optimization/analytics. Use when building, wiring, or debugging a NetScaler FlowAI agent, or when deciding which existing agent/tool handles a NetScaler operation.
---

# Citrix NetScaler — Agent Skill Reference

Load this when you need the exact tool list for a specific agent, the real payload conventions for NetScaler NITRO objects, or a documented gotcha. For orientation and design rationale, read `AGENTS.md` first.

## When to use this skill

- Building a new NetScaler agent and need to know which real, callable tool names exist for a given object type.
- Debugging why an existing agent's tool call failed against a live NetScaler appliance.
- Deciding which of the 13 projects/33 agents already covers a requested capability, at the tool-reference level (the capability index in `AGENTS.md` covers this at a coarser grain).
- Extending an existing agent's tool list and need to avoid re-discovering the duplicate-catalog gotcha from scratch.

## Agent-to-tool map

Every `referenceId` below is real — pulled directly from a live platform export, not paraphrased. Format: `integration:Citrix NetScaler NITRO API%3Alatest:NetScaler:<method>` (the `%3A` is a URL-encoded colon before `latest`). `view:WorkFlowEngine:ViewData` is the shared human-approval tool used by every mutating agent.

### `projects/load-balancing.project.json`

| Agent | Tools (method names only, `NetScaler:` prefix implied) |
|---|---|
| LB VServer Agent | `getLbvserverByName`, `listLbvserver`, `createLbvserver`, `updateLbvserver`, ViewData |
| LB Service & Service Group Agent | `listService`, `createService`, `updateService`, `listServicegroup`, `createServicegroup`, `updateServicegroup`, ViewData |
| LB Binding & Health Monitor Agent | `createLbvserverServiceBinding`, `createLbvserverServicegroupBinding`, `createLbmonitor`, `updateLbmonitor`, `listLbmonitor`, `createServiceLbmonitorBinding`, ViewData |

### `projects/traffic-routing.project.json`

| Agent | Tools |
|---|---|
| CS VServer & Policy Agent | `listCsvserver`, `createCsvserver`, `updateCsvserver`, `createCsaction`, `updateCsaction`, `createCspolicy`, `updateCspolicy`, `listCspolicy`, ViewData |
| CS Binding Agent | `createCsvserverCspolicyBinding`, `listCsvserverCspolicyBinding`, `createCsvserverLbvserverBinding`, `listCsvserverLbvserverBinding`, ViewData |
| Responder Policy Agent | `createResponderaction`, `updateResponderaction`, `createResponderpolicy`, `updateResponderpolicy`, `listResponderpolicy`, `createResponderpolicyCsvserverBinding`, `createLbvserverResponderpolicyBinding`, `listLbvserverResponderpolicyBinding`, ViewData |
| Rewrite Policy Agent | `createRewriteaction`, `updateRewriteaction`, `createRewritepolicy`, `updateRewritepolicy`, `listRewritepolicy`, ViewData |
| Policy Building Blocks Agent | `listPolicydataset`, `createPolicydataset`, `createPolicydatasetValueBinding`, `listPolicypatset`, `createPolicypatset`, `createPolicypatsetPatternBinding`, `listPolicystringmap`, `createPolicystringmap`, `createPolicystringmapPatternBinding`, ViewData |

### `projects/ssl-certificates.project.json`

| Agent | Tools |
|---|---|
| SSL Certificate Agent | `listSslcertkey`, `getSslcertkeyByName`, `createSslcertkey`, `updateSslcertkey`, `createSslvserverSslcertkeyBinding`, `listSslvserverSslcertkeyBinding`, ViewData |

### `projects/gslb-multi-site.project.json`

| Agent | Tools |
|---|---|
| GSLB VServer & Service Agent | `listGslbvserver`, `createGslbvserver`, `updateGslbvserver`, `listGslbservice`, `createGslbservice`, `updateGslbservice`, ViewData |
| GSLB Site & Domain Binding Agent | `createGslbsite`, `listGslbsite`, `createGslbvserverServiceBinding`, `listGslbvserverServiceBinding`, `createGslbvserverDomainBinding`, `listGslbvserverDomainBinding`, ViewData |

### `projects/security-access.project.json`

| Agent | Tools |
|---|---|
| WAF Policy Agent | `listAppfwprofile`, `createAppfwprofile`, `updateAppfwprofile`, `listAppfwpolicy`, `createAppfwpolicy`, `updateAppfwpolicy`, `createLbvserverAppfwpolicyBinding`, `createCsvserverAppfwpolicyBinding`, ViewData |
| Authentication Policy Agent | `listAuthenticationldappolicy`, `createAuthenticationldappolicy`, `listAuthenticationradiuspolicy`, `createAuthenticationradiuspolicy`, `listAuthenticationvserver`, `createAuthenticationvserver`, `createAuthenticationvserverAuthenticationldappolicyBinding`, `createAuthenticationvserverAuthenticationradiuspolicyBinding`, ViewData |
| Rate Limiting Agent | `listNslimitidentifier`, `createNslimitidentifier`, `listNslimitselector`, `createNslimitselector`, ViewData |

### `projects/monitoring-diagnostics.project.json` (read-only — no ViewData, no write tools)

| Agent | Tools |
|---|---|
| Vserver & Service Health Agent | `getNsversionByName`, `getHanodeByName`, `listHanode`, `listLbvserver`, `getLbvserverByName`, `listService`, `listServicegroup` |
| Config & Certificate Diagnostics Agent | `listCsvserver`, `listResponderpolicy`, `listRewritepolicy`, `listGslbvserver`, `listSslcertkey`, `getSslcertkeyByName` |

### `projects/gateway-remote-access.project.json`

| Agent | Tools |
|---|---|
| SSL VPN Gateway Agent | `listVpnvserver`, `createVpnvserver`, `updateVpnvserver`, `createVpnglobalAuthenticationldappolicyBinding`, `createVpnglobalAuthenticationradiuspolicyBinding`, `createVpnglobalIntranetapplicationBinding`, ViewData |
| ICA Proxy Agent | `listIcapolicy`, `createIcapolicy`, `updateIcapolicy`, `createIcaaction`, `updateIcaaction`, `createIcapolicyVpnvserverBinding`, `listIcapolicyVpnvserverBinding`, ViewData |

### `projects/system-administration.project.json`

| Agent | Tools |
|---|---|
| RBAC Agent | `listSystemuser`, `createSystemuser`, `updateSystemuser`, `listSystemgroup`, `createSystemgroup`, `createSystemuserSystemgroupBinding`, `listSystemcmdpolicy`, `createSystemcmdpolicy`, `createSystemuserSystemcmdpolicyBinding`, ViewData |
| Feature & Licensing Agent | `listNsfeature`, `updateNsfeature`, `listNsmode`, `updateNsmode`, `listNslicense`, `createNslicense`, ViewData |
| Config Backup Agent | `listSystembackup`, `createSystembackup`, `getSystembackupByName`, ViewData |

### `projects/networking-fundamentals.project.json`

| Agent | Tools |
|---|---|
| IP & VLAN Agent | `listNsip`, `createNsip`, `updateNsip`, `listVlan`, `createVlan`, `updateVlan`, `createVlanInterfaceBinding`, `createVlanNsipBinding`, ViewData |
| Routing & Interfaces Agent | `listRoute`, `createRoute`, `updateRoute`, `deleteRoute`, `listInterface`, `updateInterface`, `createInterfacepair`, ViewData |
| Channel (LACP) Agent | `createChannel`, `updateChannel`, `createChannelInterfaceBinding`, ViewData |
| NAT Agent | `listInat`, `createInat`, `updateInat`, `listRnat`, `createRnat`, `updateRnat`, ViewData |

### `projects/clustering-ha.project.json`

| Agent | Tools |
|---|---|
| Clustering Agent | `listCluster`, `createCluster`, `listClusterinstance`, `createClusterinstance`, `listClusternode`, `createClusternode`, `createClusterinstanceClusternodeBinding`, ViewData |
| HA Pair Agent | `listHanode`, `createHanode`, `updateHanode`, ViewData |

### `projects/dns-services.project.json`

| Agent | Tools |
|---|---|
| Forward Record Agent (A/CNAME) | `listDnsaddrec`, `createDnsaddrec`, `updateDnsaddrec`, `listDnscnamerec`, `createDnscnamerec`, `updateDnscnamerec`, ViewData |
| Zone & Reverse Record Agent (NS/PTR/SOA) | `listDnsnsrec`, `createDnsnsrec`, `listDnsptrrec`, `createDnsptrrec`, `listDnssoarec`, `createDnssoarec`, ViewData |

### `projects/bot-management.project.json`

| Agent | Tools |
|---|---|
| Bot Management Agent | `listBotpolicy`, `createBotpolicy`, `updateBotpolicy`, `listBotprofile`, `createBotprofile`, `updateBotprofile`, `createBotpolicyCsvserverBinding`, `createBotpolicyLbvserverBinding`, ViewData |

### `projects/traffic-optimization-analytics.project.json`

| Agent | Tools |
|---|---|
| Caching & Compression Agent | `listCachepolicy`, `createCachepolicy`, `updateCachepolicy`, `createCachecontentgroup`, `createCachepolicyLbvserverBinding`, `listCmppolicy`, `createCmppolicy`, `updateCmppolicy`, `createCmppolicyLbvserverBinding`, ViewData |
| Performance Profiles & Spillover Agent | `listNstcpprofile`, `createNstcpprofile`, `updateNstcpprofile`, `listNshttpprofile`, `createNshttpprofile`, `updateNshttpprofile`, `listSpilloverpolicy`, `createSpilloverpolicy`, `createSpilloverpolicyLbvserverBinding`, ViewData |
| AppFlow Analytics Agent | `listAppflowpolicy`, `createAppflowpolicy`, `updateAppflowpolicy`, `createAppflowaction`, `updateAppflowaction`, `createAppflowcollector`, `createAppflowpolicyCsvserverBinding`, ViewData |

## Patterns

- **Every agent's `inputSchema` is identical**: `{"type":"object","additionalProperties":false,"required":["request"],"properties":{"request":{"type":"string"}}}`. Agents take a free-text description of what's wanted, not a structured form — the LLM composes the exact NITRO payload from context, not from a rigid field mapping. `instructions` must reference `{{request}}` at least once or agent create/update fails validation.
- **Dependency order on create chains**: action before policy, policy before binding (`createResponderaction` → `createResponderpolicy` → `createLbvserverResponderpolicyBinding`); parent object before child binding (`createClusterinstance` before `createClusterinstanceClusternodeBinding`); profile before policy before vserver binding (`createAppfwprofile` → `createAppfwpolicy` → `createLbvserverAppfwpolicyBinding`).
- **NITRO binding-resource naming is `<owner>_<owned>_binding`**, and the owner is whichever side actually has NetScaler CLI support — not necessarily the side that reads most naturally. See the responder/lbvserver gotcha below; when adding a new binding type, verify which direction is real before wiring it into an agent, don't assume from the name alone.
- **List/get before propose, always.** Every agent's `instructions` require a read call before composing a proposed change — this isn't just good practice, it's what makes the ViewData "current vs. proposed" comparison meaningful instead of a guess.

## Gotchas

**1. Two app-name variants for the same integration — one dead, one live.**
This platform's NetScaler adapter registered two catalog variants: a stale `Nitro` one (`active: false`, ~150 methods, pre-adapter-refresh) and the current `NetScaler` one (`active: true`, ~6,600 methods). `GET /tools?name=<method>` can return both for a method that exists in both catalogs, and list order is not active-first — a naive `data[0]` pick can silently wire the dead variant onto an agent. The failure is silent at agent-create time: the call succeeds, but the tool entry lands as `unauthorizedReferenceId` instead of `referenceId` in the stored agent, and the tool will never actually resolve at run time.
*Fix:* always filter `GET /tools?name=<method>` results for the entry whose `referenceId` contains `:NetScaler:` (not `:Nitro:`) and `active == true`. After creating or updating any agent, `GET` it back and confirm `tools[]` has zero `unauthorizedReferenceId` entries.

**2. A schema-valid binding with no real backing command.**
NITRO auto-generates both `responderpolicy_lbvserver_binding` and `lbvserver_responderpolicy_binding` as REST resources (both accept POST per the OpenAPI schema), but only the vserver-owned direction — `lbvserver_responderpolicy_binding`, i.e. `createLbvserverResponderpolicyBinding` — has a real NetScaler CLI command behind it. The other, `createResponderpolicyLbvserverBinding`, fails at runtime with NetScaler error `1088` ("No such command") despite passing request validation. Confirmed by cross-referencing the real Citrix NetScaler NITRO 14.1 OpenAPI spec directly, not by guessing from the error message alone.
*Fix:* the Responder Policy Agent in `traffic-routing.project.json` only has `createLbvserverResponderpolicyBinding` wired — `createResponderpolicyLbvserverBinding` is deliberately excluded from its tool list, and its `instructions` explicitly warn against the broken alternative by name.

**3. Bundle-import `providerResolutions` (keyed by profile name) did not reliably resolve `provider`.**
When creating agents via `POST /agent-project-service/project-bundles/import` with a `providerResolutions` map keyed by profile name, agents have come back with `provider: null` even though the named profile/model existed on the target platform.
*Fix:* prefer creating agents individually via `POST /agent-project-service/projects/{projId}/agents` with `provider: {profile: <uuid>, model: <uuid>}` passed directly — this reliably resolves. If bundle import is used anyway, always `GET` every agent back afterward and `PATCH` in the direct UUIDs for any agent whose `provider` came back null.

## Verification checklist

Run this after creating or modifying any agent in this package, before considering it done:

- [ ] `GET /agent-project-service/agents/{agentId}` — every `tools[]` entry has a `referenceId` field (never `unauthorizedReferenceId`)
- [ ] Every `referenceId` for a NetScaler method contains `:NetScaler:`, never `:Nitro:`
- [ ] `provider` is a populated object with both `profile` and `model` UUIDs — never `null`
- [ ] Tool count for the agent is ≤ 10 — if not, it's covering more than one concern and should be split
- [ ] If the agent can mutate state, `view:WorkFlowEngine:ViewData` is in its `tools[]` and its `instructions` describe the propose → approve → act sequence explicitly
- [ ] If the agent binds a responder policy to an LB vserver, it uses `createLbvserverResponderpolicyBinding` and not `createResponderpolicyLbvserverBinding`
- [ ] The project's `GET /agent-project-service/projects/{projId}` component count matches the number of agents actually created
- [ ] No `POST /agent-session-manager/sessions` or `run-agent` call was made unless a live test run was actually intended — creation and verification are `GET`-only
