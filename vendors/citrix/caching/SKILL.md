---
name: citrix-caching
description: How to configure NetScaler integrated caching policies, including the data-leak risk of caching personalized responses. Vendor-neutral. Use when building, reviewing, or debugging NetScaler caching automation, on Itential or otherwise.
---

# Citrix NetScaler — Caching

## When to use this skill

- Configuring integrated caching for a backend response.
- Reviewing a caching policy for correctness/safety before enabling it.

## Operational procedure

Verify a response is genuinely cacheable (correct cache-control headers, no session-specific or personalized content) before creating a cache policy for it. Caching a personalized or session-bound response is a data-leak risk between users, not just a wasted optimization.

## Patterns

- **Cacheability is a correctness question, not just a performance one** — caching session-bound or personalized content risks leaking one user's data to another.

## Known limitations

- No offensive/destructive capability.
- Compression is a distinct feature with its own tradeoffs — see the `compression` skill.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listCachepolicy` | List integrated-caching policies | Caching |
| `createCachepolicy` | Create a new caching policy | Caching |
| `updateCachepolicy` | Change an existing caching policy | Caching |
| `createCachecontentgroup` | Create a cache content group (defines what's cacheable and for how long) | Caching |
| `createCachepolicyLbvserverBinding` | Bind a caching policy to an LB vserver | Caching |

## Verification checklist

- [ ] Response confirmed genuinely cacheable (no session/personalized content) before enabling the policy
- [ ] Cache content group's TTL/scope reviewed for correctness, not left at a default that's wrong for this content
- [ ] After enabling, confirmed via a real request that a second user doesn't receive the first user's cached, personalized response
