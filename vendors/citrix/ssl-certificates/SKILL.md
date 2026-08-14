---
name: citrix-ssl-certificates
description: How to install, bind, and renew SSL certificates on NetScaler (Citrix ADC), including zero-downtime renewal and expiry monitoring. Vendor-neutral. Use when building, reviewing, or debugging NetScaler SSL certificate automation, on Itential or otherwise.
---

# Citrix NetScaler — SSL Certificates

## When to use this skill

- Installing or renewing an SSL certificate on an SSL-enabled vserver.
- Checking certificate expiry.
- Reviewing or debugging a certificate rollout that reports success but isn't presenting the expected cert.

## Operational procedure

1. Stage the certificate and key files on the appliance (via whatever out-of-band file transfer the environment supports) before creating anything through the API — the cert-key object references files that must already exist.
2. Create the cert-key pair object pointing at the staged files.
3. Bind it to the target SSL vserver.
4. **Renewal**, specifically: create the *new* cert-key object under a distinct name first, bind it to the vserver, and confirm the live TLS handshake is actually presenting the new certificate (probe it directly — check the serial number and expiry a client actually receives, not just what the API's stored metadata says) before unbinding or deleting the old one. Deleting the old cert before confirming the new one is live risks an outage window with no valid cert bound at all.
5. **Expiry monitoring**: check expiry on a recurring schedule and treat an approaching date as a lead-time trigger to start the renewal procedure above — not a same-day emergency response.

## Patterns

- **Stage-then-attach, not attach-then-configure.** Confirm the certificate/key files are valid and present on the appliance before referencing them in a cert-key object.
- **Renew-then-cut-over-then-clean-up.** Never delete the old certificate until the new one is confirmed live via an actual TLS handshake probe.

## Known limitations

- No offensive/destructive capability.
- Doesn't cover certificate issuance/CA interaction — assumes cert/key material is already obtained and staged; only the NetScaler-side install/bind/renew procedure is in scope.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `listSslcertkey` | List installed SSL certificate-key pairs | SSL Certificate |
| `getSslcertkeyByName` | Look up a single certificate-key pair by name (includes expiry) | SSL Certificate |
| `createSslcertkey` | Install a new certificate-key pair (from files already staged on the appliance) | SSL Certificate |
| `updateSslcertkey` | Update an existing certificate-key pair's settings | SSL Certificate |
| `createSslvserverSslcertkeyBinding` | Bind a certificate to an SSL-enabled vserver | SSL Certificate |
| `listSslvserverSslcertkeyBinding` | List which certificate is bound to an SSL vserver | SSL Certificate |

## Verification checklist

- [ ] After binding, probe the live TLS handshake directly and confirm the serial number/expiry a real client receives — not just the API's stored metadata
- [ ] Old certificate not deleted/unbound until the new one is confirmed live
- [ ] Expiry re-checked on a recurring schedule, not just at initial install
