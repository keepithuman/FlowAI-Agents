---
name: citrix-policy-building-blocks
description: How to build NetScaler datasets, pattern sets, and string maps referenced by policy expressions across content switching, responder, rewrite, and other NetScaler policy types. Vendor-neutral. Use when building, reviewing, or debugging NetScaler policy-expression building blocks, on Itential or otherwise.
---

# Citrix NetScaler — Policy Building Blocks

## When to use this skill

- Building a dataset, pattern set, or string map that a policy expression will reference.
- Debugging a policy that fails to create because it references a building block that doesn't exist yet.

## Operational procedure

1. Identify which policy types will reference this building block (content switching, responder, rewrite, WAF, etc.).
2. Create the dataset, pattern set, or string map first.
3. Add values, patterns, or key/value mappings into it.
4. Only then create the policy that references it — a policy expression referencing a not-yet-created dataset/patset/stringmap fails validation at policy-create time.
5. Confirm the referencing policy behaves correctly against an entry that's actually in the dataset/patset/stringmap.

## Patterns

- **Building block before policy, always** — this is a hard dependency, not just a best practice; policy creation fails validation if the referenced dataset/patset/stringmap doesn't already exist.

## Known limitations

- No offensive/destructive capability.
- This skill only covers building the datasets/pattern sets/string maps themselves — the policies that reference them belong to the relevant policy skill (`content-switching`, `responder-policies`, `rewrite-policies`, `waf`, etc.).

## Tools

Every operation below is a real, confirmed-active method on the Citrix NetScaler NITRO REST API (source: the NetScaler adapter's live task catalog, cross-checked against the official Citrix NetScaler NITRO 14.1 OpenAPI spec).

| Operation | Description | Category |
|---|---|---|
| `listPolicydataset` | List datasets (typed value collections referenced by policy expressions) | Policy Building Blocks |
| `createPolicydataset` | Create a new dataset | Policy Building Blocks |
| `createPolicydatasetValueBinding` | Add a value into an existing dataset | Policy Building Blocks |
| `listPolicypatset` | List pattern sets (string-pattern collections referenced by policy expressions) | Policy Building Blocks |
| `createPolicypatset` | Create a new pattern set | Policy Building Blocks |
| `createPolicypatsetPatternBinding` | Add a pattern into an existing pattern set | Policy Building Blocks |
| `listPolicystringmap` | List string maps (key→value lookup tables referenced by policy expressions) | Policy Building Blocks |
| `createPolicystringmap` | Create a new string map | Policy Building Blocks |
| `createPolicystringmapPatternBinding` | Add a key/value pair into an existing string map | Policy Building Blocks |

## Verification checklist

- [ ] Building block confirmed to exist (via list/get) before any dependent policy is created
- [ ] Values/patterns/mappings added confirmed present via a list call, not just the add call's success
- [ ] Referencing policy tested with input matching an actual entry in the dataset/patset/stringmap
