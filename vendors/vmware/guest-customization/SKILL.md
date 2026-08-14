---
name: vmware-guest-customization
description: How to create and apply VMware vSphere guest OS customization specs during VM deployment (hostname, network config, domain join). Vendor-neutral. Use when building, reviewing, or debugging vSphere guest-customization automation, on Itential or otherwise.
---

# VMware vSphere — Guest Customization

## When to use this skill

- Creating a guest OS customization spec.
- Applying one during VM deployment from a template or clone.

## Operational procedure

1. Confirm the target is a new deployment (from a template or clone), not an already-running VM — a customization spec is applied *during* deployment; if a VM is already up, this isn't the tool to change its hostname or network config.
2. Create the customization spec (hostname, network config, domain join, etc.), verified against the actual guest OS family/version being deployed — network adapter naming and sysprep/cloud-init syntax vary by OS family and version.
3. Handle any credentials/product keys in the spec with the same care as other secret material — don't echo them verbatim into a record anyone else can read.
4. Apply the spec during deployment.

## Patterns

- **Deployment-time only** — a customization spec has no effect on an already-running VM; confirm the request is about deployment before reaching for this skill.
- **Treat spec content as credential material** — domain-join credentials and product keys inside a spec need the same handling discipline as any other secret.

## Known limitations

- Does not cover post-deployment configuration changes to a running VM's guest OS — only what's applied at deployment time via the spec.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Description | Category |
|---|---|---|
| `Vcenter.Guest.CustomizationSpecs_list` | List guest OS customization specs | Guest Customization |
| `Vcenter.Guest.CustomizationSpecs_get` | Get a single customization spec's details | Guest Customization |
| `Vcenter.Guest.CustomizationSpecs_create` | Create a new customization spec (hostname, network config, domain join, etc.) | Guest Customization |
| `Vcenter.Guest.CustomizationSpecs_set` | Replace an existing customization spec's content | Guest Customization |
| `Vcenter.Guest.CustomizationSpecs_delete` | Delete a customization spec | Guest Customization |

## Verification checklist

- [ ] Confirmed the target is a new deployment (from template/clone), not an already-running VM
- [ ] Spec content containing credentials/product keys handled with the same care as other secret material — not echoed into a plain record
- [ ] Spec verified against the actual guest OS family/version being deployed, not assumed compatible
