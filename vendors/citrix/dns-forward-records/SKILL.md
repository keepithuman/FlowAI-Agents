---
name: citrix-dns-forward-records
description: How to manage NetScaler DNS forward records (A/CNAME), including the zone-must-exist-first dependency. Vendor-neutral. Use when building, reviewing, or debugging NetScaler DNS forward-record automation, on Itential or otherwise.
---

# Citrix NetScaler — DNS Forward Records (A/CNAME)

## When to use this skill

- Creating or updating A/CNAME records.
- Debugging a record that "created successfully" but doesn't resolve.

## Operational procedure

1. Confirm the authoritative zone/SOA already exists and is correctly configured.
2. Create or update the A/CNAME record inside that zone.
3. Confirm the record actually resolves via a real external query — a record added into a misconfigured or non-existent zone creates successfully but silently never resolves for anyone querying it externally.

## Patterns

- **Zone-first** — confirm the authoritative zone/SOA is correct before trusting any record created inside it; a record's create-success doesn't validate the zone it lives in.

## Known limitations

- No offensive/destructive capability.
- Zone/SOA record management itself is covered by the `dns-zone-reverse-records` skill — this skill assumes the zone already exists.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listDnsaddrec` | List DNS A (address) records | DNS Forward Records |
| `createDnsaddrec` | Create a new A record | DNS Forward Records |
| `updateDnsaddrec` | Change an existing A record | DNS Forward Records |
| `listDnscnamerec` | List DNS CNAME (alias) records | DNS Forward Records |
| `createDnscnamerec` | Create a new CNAME record | DNS Forward Records |
| `updateDnscnamerec` | Change an existing CNAME record | DNS Forward Records |

## Verification checklist

- [ ] Authoritative zone/SOA confirmed correct before trusting any record added into it
- [ ] A/CNAME record confirmed to actually resolve via a real external query, not just the create call's success
