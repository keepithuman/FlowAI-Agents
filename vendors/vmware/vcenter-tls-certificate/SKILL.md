---
name: vmware-vcenter-tls-certificate
description: How to renew or replace vCenter's own TLS certificate. Vendor-neutral. Use when building, reviewing, or debugging vCenter TLS-certificate automation, on Itential or otherwise.
---

# VMware vSphere — vCenter TLS Certificate

## When to use this skill

- Renewing or replacing vCenter's own TLS certificate.
- Generating a CSR for external CA signing.

## Operational procedure

This is vCenter's *own* TLS identity (the certificate presented when connecting to vCenter itself) — not certificates on the VMs vCenter manages. Don't confuse a request about "a certificate on my web server VM" with this scope.

Renewing vCenter's own certificate can briefly interrupt every client connection to vCenter (API clients, the web UI, other integrations) while the new cert takes effect — this is not a zero-downtime operation. Time it deliberately.

## Patterns

- **Scope check first** — confirm a certificate request is actually about vCenter's own identity, not a certificate on a managed VM.
- **Time disruptive changes deliberately** — a vCenter cert renewal interrupts every connected client briefly; don't treat it as zero-downtime.

## Known limitations

- Does not cover certificates on VMs vCenter manages — only vCenter's own TLS identity.
- Trusted root chain management (which CAs vCenter trusts for other integrations) is a distinct concern — see the `trusted-root-chains` skill.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.CertificateManagement.Vcenter.Tls_get` | Get vCenter's current TLS certificate details | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.Tls_renew` | Renew vCenter's TLS certificate | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.Tls_set` | Replace vCenter's TLS certificate with a specific one | Certificate Management |
| `Vcenter.CertificateManagement.Vcenter.TlsCsr_create` | Generate a certificate signing request for external CA signing | Certificate Management |

## Verification checklist

- [ ] Confirmed the request is actually about vCenter's own TLS identity, not a VM-level certificate
- [ ] Certificate renewal timed deliberately, with stakeholders aware of the brief client-connection interruption
- [ ] After renewal, confirmed a real client connection presents the new certificate
