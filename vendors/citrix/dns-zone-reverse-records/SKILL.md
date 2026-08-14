---
name: citrix-dns-zone-reverse-records
description: How to manage NetScaler DNS zone authority (NS/SOA) and reverse-lookup (PTR) records. Vendor-neutral. Use when building, reviewing, or debugging NetScaler DNS zone/reverse-record automation, on Itential or otherwise.
---

# Citrix NetScaler — DNS Zone & Reverse Records (NS/PTR/SOA)

## When to use this skill

- Setting up or reviewing zone authority (NS/SOA) records.
- Creating or debugging PTR (reverse-lookup) records.

## Operational procedure

A PTR record must be added into the zone that actually corresponds to the IP (the correct `in-addr.arpa` or `ip6.arpa` zone) — a PTR record that "creates successfully" in the wrong zone will never be found by a real reverse lookup, and there's no error at creation time to signal the mismatch.

## Patterns

- **Zone-first, especially for PTR** — the correct `in-addr.arpa`/`ip6.arpa` zone is not optional context, it's a hard correctness requirement with no create-time error if wrong.

## Known limitations

- No offensive/destructive capability.
- DNSSEC and zone-transfer configuration are not covered by this procedure — verify separately if in scope.
- Forward records (A/CNAME) are covered by the `dns-forward-records` skill.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listDnsnsrec` | List DNS NS (nameserver) records | DNS Zone Records |
| `createDnsnsrec` | Create a new NS record | DNS Zone Records |
| `listDnsptrrec` | List DNS PTR (reverse-lookup) records | DNS Reverse Records |
| `createDnsptrrec` | Create a new PTR record | DNS Reverse Records |
| `listDnssoarec` | List DNS SOA (zone authority) records | DNS Zone Records |
| `createDnssoarec` | Create a new SOA record | DNS Zone Records |

## Verification checklist

- [ ] PTR record confirmed added into the correct `in-addr.arpa`/`ip6.arpa` zone matching the IP
- [ ] PTR record confirmed to resolve via a real reverse lookup, not just the create call's success
- [ ] Zone's NS/SOA records confirmed consistent before relying on records inside it
