---
name: citrix-dns-services
description: How to manage NetScaler DNS records — forward (A/CNAME) and zone/reverse (NS/PTR/SOA). Vendor-neutral. Use when building, reviewing, or debugging NetScaler DNS automation, on Itential or otherwise.
---

# Citrix NetScaler — DNS Services

## When to use this skill

- Creating or updating A/CNAME records.
- Creating or updating NS/PTR/SOA records.
- Debugging a DNS record that "created successfully" but doesn't resolve.

## Operational procedure

**Forward records (A/CNAME)**: confirm the authoritative zone/SOA already exists and is correctly configured before adding records into it. A record added into a misconfigured or non-existent zone creates successfully but silently never resolves for anyone querying it externally.

**Zone & reverse records (NS/PTR/SOA)**: a PTR record must be added into the zone that actually corresponds to the IP (the correct `in-addr.arpa` or `ip6.arpa` zone) — a PTR record that "creates successfully" in the wrong zone will never be found by a real reverse lookup, and there's no error at creation time to signal the mismatch.

## Patterns

- **Zone-first** — confirm the authoritative zone/SOA is correct before trusting any record created inside it; a record's create-success doesn't validate the zone it lives in.

## Known limitations

- No offensive/destructive capability.
- DNSSEC and zone-transfer configuration are not covered by these two procedures — verify separately if in scope.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

### Forward Records

| Operation | Plain-English description | Category |
|---|---|---|
| `listDnsaddrec` | List DNS A (address) records | DNS Forward Records |
| `createDnsaddrec` | Create a new A record | DNS Forward Records |
| `updateDnsaddrec` | Change an existing A record | DNS Forward Records |
| `listDnscnamerec` | List DNS CNAME (alias) records | DNS Forward Records |
| `createDnscnamerec` | Create a new CNAME record | DNS Forward Records |
| `updateDnscnamerec` | Change an existing CNAME record | DNS Forward Records |

### Zone & Reverse Records

| Operation | Plain-English description | Category |
|---|---|---|
| `listDnsnsrec` | List DNS NS (nameserver) records | DNS Zone Records |
| `createDnsnsrec` | Create a new NS record | DNS Zone Records |
| `listDnsptrrec` | List DNS PTR (reverse-lookup) records | DNS Reverse Records |
| `createDnsptrrec` | Create a new PTR record | DNS Reverse Records |
| `listDnssoarec` | List DNS SOA (zone authority) records | DNS Zone Records |
| `createDnssoarec` | Create a new SOA record | DNS Zone Records |

## Verification checklist

- [ ] Authoritative zone/SOA confirmed correct before trusting any record added into it
- [ ] A/CNAME record confirmed to actually resolve via a real external query, not just the create call's success
- [ ] PTR record confirmed added into the correct `in-addr.arpa`/`ip6.arpa` zone matching the IP, and confirmed to resolve via a real reverse lookup
