---
name: vmware-vsphere
description: Operational procedures for VMware vSphere - VM operations, datacenter/cluster/host/network inventory, storage and datastore management, resource pools, content library and templates, guest customization, tagging, access control (RBAC), certificate management, VM encryption (KMS), cluster desired-state configuration, and performance metrics. Vendor-neutral - describes how to actually perform each operation on vSphere, regardless of what orchestrates it. Use when building, reviewing, or debugging any vSphere automation, on Itential or otherwise.
---

# VMware vSphere — Operational Skill Reference

This document teaches the real vSphere procedure for each capability area — the sequencing, decision points, and failure modes a domain expert would tell you about, independent of what tool or platform executes the API calls. It does not assume Itential — a ready-to-import reference implementation exists if you want it (see the end of this document), but everything above that is self-contained.

## When to use this skill

- Designing or reviewing any automation (agent, script, workflow) that touches VM operations, vSphere infrastructure inventory, storage, resource pools, content libraries, guest customization, tagging, access control, certificates, VM encryption, cluster configuration, or performance metrics.
- Debugging why a vSphere change didn't have the expected effect, even though the API call that made it reported success.
- Deciding what order to sequence a set of vSphere configuration changes in.

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

## Reference implementation

The procedures above are already built and running as a real, verified Itential FlowAI project — 13 agents — in [`projects/`](./projects/). If you're on Itential, that's a ready-to-import accelerator; if you're not, the procedures above are everything you need. See [`README.md`](./README.md) for the project index and [`registry.json`](../../registry.json) for the full machine-readable listing.

## Verification checklist

Confirm the real-world state actually changed — regardless of what platform executed the change:

- [ ] After any create/update/delete, re-read the object's own state via a `list`/`get` call — don't trust the create call's HTTP success alone
- [ ] For a cluster configuration change, confirm the draft's content matches what was actually approved before calling apply, and re-check compliance after applying
- [ ] For a KMS provider change, confirm no currently-encrypted VM depends on the provider being removed before removing it
- [ ] For a permission/role change, confirm who actually gains or loses access (including via inheritance) matches what was intended, not just that the API call succeeded
