---
name: citrix-compression
description: How to configure NetScaler compression policies, including the CPU-for-bandwidth tradeoff on lower-tier appliances. Vendor-neutral. Use when building, reviewing, or debugging NetScaler compression automation, on Itential or otherwise.
---

# Citrix NetScaler — Compression

## When to use this skill

- Configuring compression for backend responses.
- Evaluating whether compression is worth the CPU cost on a given appliance.

## Operational procedure

1. Identify the actual appliance tier in use.
2. Create the compression policy.
3. Bind it to the LB vserver.
4. Measure the CPU-for-bandwidth tradeoff on that specific appliance tier — lower-tier or virtual appliances can bottleneck on CPU before the bandwidth savings materialize; don't assume the tradeoff is free.

## Patterns

- **Measure the tradeoff on the real appliance tier** — compression's CPU cost is not uniform across appliance models; don't assume it's free.

## Known limitations

- No offensive/destructive capability.
- Caching is a distinct feature with its own correctness concerns — see the `caching` skill.

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Plain-English description | Category |
|---|---|---|
| `listCmppolicy` | List compression policies | Compression |
| `createCmppolicy` | Create a new compression policy | Compression |
| `updateCmppolicy` | Change an existing compression policy | Compression |
| `createCmppolicyLbvserverBinding` | Bind a compression policy to an LB vserver | Compression |

## Verification checklist

- [ ] CPU impact measured on the actual appliance tier in use, not assumed free
- [ ] Real client response confirmed compressed as expected after binding, not just the bind call's success
- [ ] Appliance CPU utilization monitored after enabling, especially on lower-tier/virtual appliances
