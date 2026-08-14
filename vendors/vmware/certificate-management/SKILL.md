---
name: vmware-certificate-management
description: How to manage vCenter's own TLS certificate (renewal, replacement) and trusted root certificate chains. Vendor-neutral. Use when building, reviewing, or debugging vCenter's own TLS certificate automation, on Itential or otherwise.
---

# VMware vSphere — Certificate Management (vCenter's Own TLS)

## When to use this skill

- Renewing or replacing vCenter's own TLS certificate.
- Managing trusted root certificate chains.

## Operational procedure

This is vCenter's *own* TLS identity (the certificate presented when connecting to vCenter itself) — not certificates on the VMs vCenter manages. Don't confuse a request about "a certificate on my web server VM" with this scope.

Renewing vCenter's own certificate can briefly interrupt every client connection to vCenter (API clients, the web UI, other integrations) while the new cert takes effect — this is not a zero-downtime operation. Time it deliberately.

Trusted root chains determine which external certificate authorities vCenter trusts for various integrations — removing a chain still actively relied on breaks whatever depended on that trust relationship, often invisibly until the next time that integration tries to connect.

## Patterns

- **Scope check first** — confirm a certificate request is actually about vCenter's own identity, not a certificate on a managed VM, before using this skill.
- **Time disruptive changes deliberately** — a vCenter cert renewal interrupts every connected client briefly; don't treat it as zero-downtime.

## Known limitations

- Does not cover certificates on VMs vCenter manages — only vCenter's own TLS identity and its trusted root chains.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.CertificateManagement.Vcenter.Tls_get` | Get vCenter's current TLS certificate details | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.Tls_renew` | Renew vCenter's TLS certificate | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.Tls_set` | Replace vCenter's TLS certificate with a specific one | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TlsCsr_create` | Generate a certificate signing request for external CA signing | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_list` | List trusted root certificate chains | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_create` | Add a new trusted root chain | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TrustedRootChains_delete` | Remove a trusted root chain | Certificate Management |

## Verification checklist

- [ ] Confirmed the request is actually about vCenter's own TLS identity, not a VM-level certificate
- [ ] Certificate renewal timed deliberately, with stakeholders aware of the brief client-connection interruption
- [ ] Before removing a trusted root chain, confirmed no active integration still depends on it
