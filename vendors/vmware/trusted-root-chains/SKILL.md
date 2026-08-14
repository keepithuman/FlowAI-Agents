---
name: vmware-trusted-root-chains
description: How to manage which certificate authorities vCenter trusts for its integrations. Vendor-neutral. Use when building, reviewing, or debugging vCenter trusted-root-chain automation, on Itential or otherwise.
---

# VMware vSphere — Trusted Root Chains

## When to use this skill

- Adding or removing a trusted root certificate chain.
- Reviewing which CAs vCenter currently trusts.

## Operational procedure

Trusted root chains determine which external certificate authorities vCenter trusts for various integrations — removing a chain still actively relied on breaks whatever depended on that trust relationship, often invisibly until the next time that integration tries to connect.

## Patterns

- **Confirm no active dependency before removing a trusted root chain** — the failure surfaces later, at the next connection attempt from a dependent integration, not immediately.

## Known limitations

- Does not cover vCenter's own TLS certificate — see the `vcenter-tls-certificate` skill for that.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_list` | List trusted root certificate chains | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_create` | Add a new trusted root chain | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_delete` | Remove a trusted root chain | Certificate Management |

## Verification checklist

- [ ] Before removing a trusted root chain, confirmed no active integration still depends on it
- [ ] After removal, confirmed the intended integration(s) are unaffected and unintended ones actually lost trust as expected
