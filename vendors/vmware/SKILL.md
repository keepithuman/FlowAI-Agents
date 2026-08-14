---
name: vmware-vsphere
description: Operational procedures for VMware vSphere - VM operations, datacenter/cluster/host/network inventory, storage and datastore management, resource pools, content library and templates, guest customization, tagging, access control (RBAC), certificate management, VM encryption (KMS), cluster desired-state configuration, and performance metrics. Vendor-neutral - describes how to actually perform each operation on vSphere, regardless of what orchestrates it. Use when building, reviewing, or debugging any vSphere automation, on Itential or otherwise.
---

# VMware vSphere — Operational Skill Reference

This document teaches the real vSphere procedure for each capability area — the sequencing, decision points, and failure modes a domain expert would tell you about, independent of what tool or platform executes the API calls. It doesn't assume any particular orchestration platform — the exhaustive operation lookup lives in this same file's Tools section.

## When to use this skill

- Designing or reviewing any automation (agent, script, workflow) that touches VM operations, vSphere infrastructure inventory, storage, resource pools, content libraries, guest customization, tagging, access control, certificates, VM encryption, cluster configuration, or performance metrics.
- Debugging why a vSphere change didn't have the expected effect, even though the API call that made it reported success.
- Deciding what order to sequence a set of vSphere configuration changes in.

**Capability index** — jump straight to the procedure that answers a specific request:

| A request is about... | Procedure |
|---|---|
| Cloning or relocating a VM, console access | [VM operations](#vm-operations-clone-relocate-console-access) |
| Datacenter/cluster/host/network inventory, host connect/disconnect | [Datacenter, cluster, host & network inventory](#datacenter-cluster-host--network-inventory) |
| Datastore capacity, storage-policy compliance/compatibility | [Storage & datastore management](#storage--datastore-management) |
| Content library management, VM template capture/deploy | [Content library & templates](#content-library--templates) |
| Tag categories and tags for organizing/targeting inventory | [Tagging & categorization](#tagging--categorization) |
| vCenter's own TLS certificate renewal, trusted root chains | [Certificate management](#certificate-management-vcenters-own-tls) |
| "Is VCHA healthy?", appliance uptime/version/load | [vCenter & appliance diagnostics](#vcenter--appliance-diagnostics) |
| Creating/resizing/deleting a resource pool | [Resource pool management](#resource-pool-management) |
| Roles, privileges, who-can-do-what | [Access control (RBAC)](#access-control-rbac) |
| Guest OS customization specs for deployment | [Guest customization](#guest-customization) |
| Cluster desired-state config (DRS/HA), compliance checks | [Cluster configuration & compliance](#cluster-configuration--compliance-drsha-desired-state) |
| KMS provider setup backing VM encryption | [VM encryption & KMS](#vm-encryption--kms) |
| "What's the CPU usage on VM X?", new metric collection | [Performance metrics](#performance-metrics) |

## Operational procedures

### VM operations (clone, relocate, console access)

Decide whether you need a full clone (independent copy, more storage, safe to power on immediately) or an instant clone (shares the parent's disk via a running-memory fork, near-instant, but the source VM must be running) — these solve different problems, and picking instant-clone for a VM you intend to keep long-term as an independent asset is the wrong tool.

A relocate can move a VM's compute (to a different host/cluster), storage (to a different datastore), or both. Confirm the target host/cluster and datastore have the resources and network connectivity the VM needs before proposing a relocate — a VM relocated onto a host with no path to its required network segment ends up unreachable, not failed-loudly.

A console ticket grants time-limited remote access to a VM's console — treat the ticket itself as a credential; don't log it or leave it somewhere it could be captured long after the session should have expired.

### Datacenter, cluster, host & network inventory

Datacenters are the top-level organizational container — everything else (clusters, hosts, VMs, networks) lives inside one. Deleting a datacenter cascades to everything organized under it; there's no "delete but keep the contents" option.

Disconnecting a host doesn't power off or migrate the VMs running on it — they keep running, but vCenter loses management visibility and control until the host reconnects. Never disconnect a host as a first response to "it's misbehaving" without checking what's currently running on it and whether those VMs can tolerate a management gap.

Clusters and networks are typically inventory-lookup targets, not direct configuration targets, at this layer — cluster formation happens through host aggregation and networks are usually defined at the host/vSwitch layer, neither of which this kind of REST surface exposes directly for mutation. Treat questions here as "what exists and where" rather than "create/modify this."

### Storage & datastore management

Datastore free space is the single most important number before proposing anything that consumes storage elsewhere (a new VM, a resized disk, a new content library item) — always re-check it fresh, don't reuse a number from earlier in the same conversation, since other activity on a shared datastore can change it in the meantime.

Storage-policy compatibility checking is a pre-flight check, not a permanent guarantee — a policy compatible with a datastore today can become non-compliant later if the datastore's underlying capabilities change. Worth re-checking periodically, not just once at VM-creation time.

Compliance reporting is inherently retrospective — it reports the last-known compliance state. If you need to know the compliance state *right now*, trigger a fresh compliance check rather than trusting a list that could be stale.

### Content library & templates

A content library is either local (owned and directly editable) or subscribed (a read-only mirror of someone else's library, kept in sync automatically). Never edit an item in a subscribed library directly — it'll fail or get silently overwritten on the next sync; edit the source library instead.

Capturing a VM as a template freezes its current disk state as the template's base image — if the source VM keeps running and changing after capture, the template does not reflect those later changes.

Deploying from a template creates a new VM from that frozen image — always confirm placement (folder, resource pool, datastore, network) explicitly rather than accepting whatever defaults the template happened to specify, since a template captured in one environment can carry placement defaults that don't exist in another.

### Tagging & categorization

A tag category defines the *type* of tag (e.g. "Environment") and which object types it applies to; a tag is a specific value within that category (e.g. "Production"). Create the category before the tag — a tag can't exist outside a category.

Decide up front whether a category allows multiple tags per object or exactly one — this is a category-level setting decided at creation time, not something to casually change later once tags are widely applied.

Tags are the mechanism most automation uses to *find* objects — a tagging mistake doesn't just look wrong, it can silently break whatever downstream process was filtering by that tag.

### Certificate management (vCenter's own TLS)

This is vCenter's *own* TLS identity (the certificate presented when connecting to vCenter itself) — not certificates on the VMs vCenter manages. Don't confuse a request about "a certificate on my web server VM" with this scope.

Renewing vCenter's own certificate can briefly interrupt every client connection to vCenter (API clients, the web UI, other integrations) while the new cert takes effect — this is not a zero-downtime operation. Time it deliberately.

Trusted root chains determine which external certificate authorities vCenter trusts for various integrations — removing a chain still actively relied on breaks whatever depended on that trust relationship, often invisibly until the next time that integration tries to connect.

### vCenter & appliance diagnostics

Read-only, but the order of checks matters: check VCHA's active-node status first (is there a healthy primary right now), then general appliance health (load, uptime), before concluding "vCenter itself is fine, the problem is elsewhere."

Appliance uptime resets on any vCenter service restart, not just a full appliance reboot — a low uptime number doesn't necessarily mean the whole appliance just came up.

### Resource pool management

A resource pool's shares/reservations/limits only matter during genuine contention for the underlying host/cluster resources — on an underutilized cluster, a tightly-configured pool and a loosely-configured one behave identically. The real test of a pool's settings is what happens during actual contention, not whether anything looks wrong today.

Deleting a resource pool doesn't delete the VMs inside it — they get reparented to the pool's parent. Confirm what the parent actually is, and whether its settings suit those VMs, before deleting.

Nested resource pools inherit constraints from their parent — a child's limit can never effectively exceed what its parent allows, regardless of what the child's own setting says.

### Access control (RBAC)

A role is a named set of privileges; a permission is the actual assignment of a role to a specific principal on a specific object, with an inheritable flag. Creating a role changes nothing by itself — nobody gains access until a permission assignment references it.

Permissions are typically inheritable down the inventory hierarchy by default — a permission granted at the datacenter level usually applies to everything inside it unless explicitly set non-propagating. Granting broad access "for convenience" at a high level is a common way to over-grant without realizing it.

Before creating a new role, check whether an existing one already covers the need — proliferating near-duplicate roles makes the access model harder to audit later, which is its own security cost even when each individual role is scoped correctly.

### Guest customization

A customization spec is applied *during* VM deployment (from a template or clone), not to an already-running VM — if a VM is already up, this isn't the tool to change its hostname or network config.

Specs commonly reference identity-sensitive values (domain-join credentials, product keys) — treat spec content with the same care as any other credential material; don't echo it verbatim into a record anyone else can read.

A spec that works for one guest OS version doesn't necessarily work for another — network adapter naming and sysprep/cloud-init syntax vary by OS family and version; verify against the actual guest OS being deployed.

### Cluster configuration & compliance (DRS/HA desired state)

The draft → apply model is deliberate: a draft captures a proposed desired-state configuration without touching the live cluster, and only `apply` actually pushes it. Always create and review a draft before ever applying, even for a change that seems obviously safe.

A compliance check reports drift between actual and declared configuration — it's a detection tool, not a remediation tool. Finding non-compliance doesn't fix it; you still have to decide whether to apply a corrected draft or accept the drift.

Applying a cluster configuration change affects every host and VM in that cluster simultaneously (DRS/HA behavior is cluster-wide by definition) — there's no way to stage this to "just one host first."

### VM encryption & KMS

A KMS provider is the external key server vCenter delegates encryption-key management to — vCenter doesn't generate or store the actual keys long-term, the KMS does. Removing a provider that's still backing live encrypted VMs can make their data permanently inaccessible if the keys aren't recoverable elsewhere.

Before removing or replacing a KMS provider, confirm nothing currently depends on it — there's no automatic warning; the failure shows up later, when someone tries to access an encrypted VM and the key can't be retrieved.

FIPS module status is relevant context for regulated environments — surface it alongside any KMS change proposed in a compliance-driven request, since the two are often evaluated together in an audit.

### Performance metrics

An acquisition spec defines what gets collected and how often — creating one has a real, ongoing resource cost, so don't create broad, high-frequency specs "just in case" when a narrower one answers the actual question.

Querying already-collected data only returns data for the time range and objects an acquisition spec was actually configured to collect — if the data isn't there, the answer is "no spec was collecting that," not "the query is broken."

Counter availability varies by object type and sometimes by vSphere version — a counter that exists for VMs doesn't necessarily exist for hosts or datastores; check the real counter list for the specific object type rather than assuming a counter name transfers.

## Patterns

- **Read current state first, universally.** Every procedure above starts with looking up real current state before composing any proposed change — that's what makes a "current vs. proposed" comparison meaningful instead of a guess.
- **Draft/propose-then-apply is the vSphere-native version of a human-approval gate.** Where the underlying platform already models a two-step propose/apply flow (cluster configuration), that IS the approval mechanism — don't build a second one on top of it.
- **Deletion has different blast radius depending on what "contains" the deleted object.** Deleting a datacenter cascades to everything inside it. Deleting a resource pool reparents (doesn't delete) the VMs inside it. Deleting a KMS provider doesn't touch the VMs it encrypted, but can make their data unreachable. Know which kind of deletion you're proposing, and say so explicitly wherever the proposal is reviewed.
- **Two real capability gaps exist in the modern REST-based vSphere Automation API that don't exist in the older SOAP-based vSphere API**: there is no VM snapshot management (create/revert/delete) and no standard distributed-switch/port-group management (only Kubernetes/Tanzu-scoped switch methods exist). If a request needs either, it needs a different API surface than the one this document assumes — that's a real product limitation, not a search failure.

## Known limitations

- **No cross-object blast-radius reasoning.** These procedures describe how to look up the state directly relevant to one action, not how to trace a full dependency graph — e.g., checking every VM currently drawing from a resource pool before proposing a change to it. Whoever reviews a proposed change is still the backstop for catching that.
- **Ticket-driven / change-management-integrated variants of VM provisioning, resizing, or decommissioning are a different design** (structured intake, audit trail in an external system of record) and are out of scope for the procedures above, which assume a direct, free-text request. Building that integration is a legitimate follow-on, not something these procedures already handle.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

### VM Operations

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.VM_list` | List virtual machines | VM Operations |
| `Vcenter.VM_get` | Get a single VM's full configuration | VM Operations |
| `Vcenter.VM_clone` | Create a full independent copy of a VM | VM Operations |
| `Vcenter.VM_relocate` | Move a VM's compute (host/cluster), storage (datastore), or both | VM Operations |
| `Vcenter.Vm.Console.Tickets_create` | Generate a time-limited remote console access ticket for a VM | VM Operations |

### Datacenter, Cluster, Host & Network

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Datacenter_list` | List datacenters (top-level inventory containers) | Datacenter |
| `Vcenter.Datacenter_create` | Create a new datacenter | Datacenter |
| `Vcenter.Datacenter_delete` | Delete a datacenter — cascades to everything organized inside it | Datacenter |
| `Vcenter.Cluster_list` | List clusters | Cluster |
| `Vcenter.Host_list` | List ESXi hosts | Host |
| `Vcenter.Host_connect` | Connect a host to vCenter's management | Host |
| `Vcenter.Host_disconnect` | Disconnect a host from vCenter's management (VMs on it keep running, unmanaged) | Host |
| `Vcenter.Network_list` | List networks/port groups | Network |

### Storage & Datastore

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Datastore_list` | List datastores | Datastore |
| `Vcenter.Datastore_get` | Get a single datastore's details, including free/used capacity | Datastore |
| `Vcenter.Datastore.DefaultPolicy_get` | Get a datastore's default storage policy | Storage Policy |
| `Vcenter.Storage.Policies_list` | List SPBM storage policies | Storage Policy |
| `Vcenter.Storage.Policies_checkCompatibility` | Check whether a storage policy is compatible with a given datastore | Storage Policy |
| `Vcenter.Storage.Policies.Compliance_list` | List storage-policy compliance status across entities | Storage Policy |
| `Vcenter.Storage.Policies.VM_list` | List which storage policy is assigned to which VM | Storage Policy |

### Content Library & Templates

| Operation | Plain-English description | Category |
|---|---|---|
| `Content.Library_list` | List content libraries (local and subscribed) | Content Library |
| `Content.Library_get` | Get a single content library's details | Content Library |
| `Content.LocalLibrary_create` | Create a new local (directly editable) content library | Content Library |
| `Content.LocalLibrary_update` | Change an existing local library's settings | Content Library |
| `Content.LocalLibrary_delete` | Delete a local content library | Content Library |
| `Vcenter.VmTemplate.LibraryItems_create` | Capture a VM as a template library item (freezes current disk state as the base image) | VM Templates |
| `Vcenter.VmTemplate.LibraryItems_deploy` | Deploy a new VM from a template library item | VM Templates |
| `Vcenter.VmTemplate.LibraryItems_get` | Get a template library item's details | VM Templates |

### Tagging & Categorization

| Operation | Plain-English description | Category |
|---|---|---|
| `Cis.Tagging.Category_list` | List tag categories (the type of tag, and what object types it applies to) | Tagging |
| `Cis.Tagging.Category_create` | Create a new tag category | Tagging |
| `Cis.Tagging.Category_get` | Get a tag category's details | Tagging |
| `Cis.Tagging.Tag_list` | List tags | Tagging |
| `Cis.Tagging.Tag_create` | Create a new tag within a category | Tagging |
| `Cis.Tagging.Tag_get` | Get a tag's details | Tagging |
| `Cis.Tagging.Tag_update` | Change an existing tag | Tagging |

### Certificate Management (vCenter's own TLS)

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.CertificateManagement.Vcenter.Tls_get` | Get vCenter's current TLS certificate details | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.Tls_renew` | Renew vCenter's TLS certificate | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.Tls_set` | Replace vCenter's TLS certificate with a specific one | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TlsCsr_create` | Generate a certificate signing request for external CA signing | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_list` | List trusted root certificate chains | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_create` | Add a new trusted root chain | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_delete` | Remove a trusted root chain | Certificate Management |

### vCenter & Appliance Diagnostics (read-only)

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Vcha.Cluster_get` | Get vCenter High Availability (VCHA) cluster configuration | VCHA Status |
| `Vcenter.Vcha.Cluster.Active_get` | Get the currently active (primary) VCHA node | VCHA Status |
| `Vcenter.Vcha.Operations_get` | Get available/pending VCHA operations | VCHA Status |
| `Appliance.Health.System_get` | Get overall appliance system health | Appliance Health |
| `Appliance.Health.Load_get` | Get the appliance's current load | Appliance Health |
| `Appliance.System.Version_get` | Get the appliance's software version | Appliance Health |
| `Appliance.System.Uptime_get` | Get the appliance's uptime (resets on any vCenter service restart, not just a full reboot) | Appliance Health |

### Resource Pool Management

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.ResourcePool_list` | List resource pools | Resource Pool |
| `Vcenter.ResourcePool_get` | Get a single resource pool's configuration | Resource Pool |
| `Vcenter.ResourcePool_create` | Create a new resource pool | Resource Pool |
| `Vcenter.ResourcePool_update` | Change an existing resource pool's CPU/memory shares, reservations, or limits | Resource Pool |
| `Vcenter.ResourcePool_delete` | Delete a resource pool — VMs inside get reparented to the pool's parent, not deleted | Resource Pool |

### Access Control (RBAC)

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Authorization.Roles_list` | List roles (named privilege sets) | RBAC |
| `Vcenter.Authorization.Roles_create` | Create a new role | RBAC |
| `Vcenter.Authorization.Roles_update` | Change an existing role's privileges | RBAC |
| `Vcenter.Authorization.Roles_delete` | Delete a role | RBAC |
| `Vcenter.Authorization.Permissions_list` | List permissions (role assignments on specific objects) | RBAC |
| `Vcenter.Authorization.Permissions_create` | Assign a role to a principal on a specific object | RBAC |
| `Vcenter.Authorization.Permissions_delete` | Remove a permission assignment | RBAC |

### Guest Customization

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.Guest.CustomizationSpecs_list` | List guest OS customization specs | Guest Customization |
| `Vcenter.Guest.CustomizationSpecs_get` | Get a single customization spec's details | Guest Customization |
| `Vcenter.Guest.CustomizationSpecs_create` | Create a new customization spec (hostname, network config, domain join, etc.) | Guest Customization |
| `Vcenter.Guest.CustomizationSpecs_set` | Replace an existing customization spec's content | Guest Customization |
| `Vcenter.Guest.CustomizationSpecs_delete` | Delete a customization spec | Guest Customization |

### Cluster Configuration & Compliance (DRS/HA desired state)

| Operation | Plain-English description | Category |
|---|---|---|
| `Esx.Settings.Clusters.Configuration_get` | Get a cluster's current desired-state configuration | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration_checkCompliance$Task` | Check whether a cluster's actual configuration matches its declared desired state | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration.Drafts_create` | Create a draft of a proposed configuration change (doesn't touch the live cluster) | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration.Drafts_get` | Get a draft's contents | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration.Drafts_list` | List pending configuration drafts | Cluster Configuration |
| `Esx.Settings.Clusters.Configuration_apply$Task` | Apply a draft's configuration to the live cluster (affects every host/VM in it) | Cluster Configuration |

### VM Encryption & KMS

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.CryptoManager.Kms.Providers_list` | List configured Key Management Server (KMS) providers | VM Encryption |
| `Vcenter.CryptoManager.Kms.Providers_get` | Get a single KMS provider's configuration | VM Encryption |
| `Vcenter.CryptoManager.Kms.Providers_create` | Add a new KMS provider | VM Encryption |
| `Vcenter.CryptoManager.Kms.Providers_update` | Change an existing KMS provider's connection details | VM Encryption |
| `Vcenter.CryptoManager.Kms.Providers_delete` | Remove a KMS provider — can make VMs encrypted with its keys inaccessible if not recoverable elsewhere | VM Encryption |
| `Vcenter.Crypto.Fips.Modules_list` | List FIPS cryptographic modules in use (relevant for regulated/compliance environments) | VM Encryption |

### Performance Metrics

| Operation | Plain-English description | Category |
|---|---|---|
| `Vstats.AcqSpecs_list` | List metric-acquisition specs (what's being collected and how often) | Performance Metrics |
| `Vstats.AcqSpecs_create` | Create a new acquisition spec | Performance Metrics |
| `Vstats.AcqSpecs_update` | Change an existing acquisition spec | Performance Metrics |
| `Vstats.AcqSpecs_delete` | Delete an acquisition spec (stops that collection) | Performance Metrics |
| `Vstats.Counters_list` | List available performance counters | Performance Metrics |
| `Vstats.Data_queryDataPoints` | Query already-collected performance data | Performance Metrics |
| `Vstats.Metrics_list` | List available metrics | Performance Metrics |

Two common VMware operations are **not** present anywhere in this REST API's operation set (confirmed by exhaustive search, not a naming miss):

- **VM snapshots** (create/revert/delete) — this capability exists in the older SOAP-based vSphere API and has not been ported to the modern REST-based vSphere Automation API as of this writing.
- **Standard distributed-switch/port-group management** — the only switch-adjacent operations that exist are scoped to Kubernetes/Tanzu supervisor networking (`Vcenter.NamespaceManagement.Networks.Nsx.*`), not general-purpose vSphere networking.

## Verification checklist

Confirm the real-world state actually changed — regardless of what platform executed the change:

- [ ] After any create/update/delete, re-read the object's own state via a `list`/`get` call — don't trust the create call's HTTP success alone
- [ ] For a cluster configuration change, confirm the draft's content matches what was actually approved before calling apply, and re-check compliance after applying
- [ ] For a KMS provider change, confirm no currently-encrypted VM depends on the provider being removed before removing it
- [ ] For a permission/role change, confirm who actually gains or loses access (including via inheritance) matches what was intended, not just that the API call succeeded
