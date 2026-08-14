---
name: vmware-vm-encryption-kms
description: How to configure and manage VMware vSphere Key Management Server (KMS) providers backing VM encryption. Vendor-neutral. Use when building, reviewing, or debugging vSphere VM-encryption/KMS automation, on Itential or otherwise.
---

# VMware vSphere — VM Encryption & KMS

## When to use this skill

- Adding, updating, or removing a KMS provider.
- Reviewing VM encryption / FIPS compliance posture.

## Operational procedure

1. Confirm the KMS provider's connection details before adding it — vCenter delegates key management to it and doesn't generate or store the actual keys long-term.
2. Add the KMS provider.
3. Before removing or replacing a provider, confirm nothing currently depends on it — there's no automatic warning; the failure shows up later, when someone tries to access an encrypted VM and the key can't be retrieved.
4. Surface FIPS module status alongside any KMS change in a compliance-driven request, since the two are often evaluated together in an audit.

## Patterns

- **Confirm no active dependency before removing a KMS provider** — there's no automatic warning, and the failure mode (an inaccessible encrypted VM) surfaces much later and is often unrecoverable.

## Known limitations

- Removing a KMS provider is a high-blast-radius, potentially irrecoverable action with no platform-level safety net — this skill's caution here is load-bearing, not boilerplate.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vcenter.CryptoManager.Kms.Providers_list` | List configured Key Management Server (KMS) providers | VM Encryption |
| `Vcenter.CryptoManager.Kms.Providers_get` | Get a single KMS provider's configuration | VM Encryption |
| `Vcenter.CryptoManager.Kms.Providers_create` | Add a new KMS provider | VM Encryption |
| `Vcenter.CryptoManager.Kms.Providers_update` | Change an existing KMS provider's connection details | VM Encryption |
| `Vcenter.CryptoManager.Kms.Providers_delete` | Remove a KMS provider — can make VMs encrypted with its keys inaccessible if not recoverable elsewhere | VM Encryption |
| `Vcenter.Crypto.Fips.Modules_list` | List FIPS cryptographic modules in use (relevant for regulated/compliance environments) | VM Encryption |

## Verification checklist

- [ ] Confirmed no currently-encrypted VM depends on a KMS provider before removing or replacing it
- [ ] FIPS module status surfaced alongside any KMS change in a compliance-driven request
- [ ] After a KMS provider change, confirmed at least one dependent encrypted VM can still successfully retrieve its key
