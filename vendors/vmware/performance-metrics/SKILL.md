---
name: vmware-performance-metrics
description: How to configure VMware vSphere performance-metric acquisition specs and query collected data. Vendor-neutral. Use when building, reviewing, or debugging vSphere performance-metrics automation, on Itential or otherwise.
---

# VMware vSphere — Performance Metrics

## When to use this skill

- "What's the CPU/memory/etc. usage on X?" questions.
- Setting up new metric collection.

## Operational procedure

An acquisition spec defines what gets collected and how often — creating one has a real, ongoing resource cost, so don't create broad, high-frequency specs "just in case" when a narrower one answers the actual question.

Querying already-collected data only returns data for the time range and objects an acquisition spec was actually configured to collect — if the data isn't there, the answer is "no spec was collecting that," not "the query is broken."

Counter availability varies by object type and sometimes by vSphere version — a counter that exists for VMs doesn't necessarily exist for hosts or datastores; check the real counter list for the specific object type rather than assuming a counter name transfers.

## Patterns

- **Narrowest spec that answers the question** — acquisition specs have an ongoing resource cost; don't over-collect "just in case."
- **Empty query result means "wasn't being collected," not "broken"** — always check the acquisition spec's actual scope before assuming a query failure.

## Known limitations

- Doesn't cover historical data outside what an acquisition spec was actually configured to collect — there's no retroactive collection.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Vstats.AcqSpecs_list` | List metric-acquisition specs (what's being collected and how often) | Performance Metrics |
| `Vstats.AcqSpecs_create` | Create a new acquisition spec | Performance Metrics |
| `Vstats.AcqSpecs_update` | Change an existing acquisition spec | Performance Metrics |
| `Vstats.AcqSpecs_delete` | Delete an acquisition spec (stops that collection) | Performance Metrics |
| `Vstats.Counters_list` | List available performance counters | Performance Metrics |
| `Vstats.Data_queryDataPoints` | Query already-collected performance data | Performance Metrics |
| `Vstats.Metrics_list` | List available metrics | Performance Metrics |

## Verification checklist

- [ ] Acquisition spec's scope (objects, time range, frequency) confirmed to actually cover the question being asked, before assuming a query failure
- [ ] Counter existence confirmed for the specific object type in question, not assumed to transfer from another object type
- [ ] New acquisition specs scoped narrowly to the actual need, not created broad "just in case"
