---
name: vmware-content-library
description: How to manage VMware vSphere content libraries, including the local-vs-subscribed distinction. Vendor-neutral. Use when building, reviewing, or debugging vSphere content-library automation, on Itential or otherwise.
---

# VMware vSphere — Content Library

## When to use this skill

- Creating, updating, or deleting a content library.
- Determining whether a library is safe to edit directly.

## Operational procedure

A content library is either local (owned and directly editable) or subscribed (a read-only mirror of someone else's library, kept in sync automatically). Never edit an item in a subscribed library directly — it'll fail or get silently overwritten on the next sync; edit the source library instead.

## Patterns

- **Source-of-truth awareness** — know whether a library is local or subscribed before editing anything in it; subscribed libraries have a different (upstream) source of truth.

## Known limitations

- VM template capture/deploy is a distinct workflow that happens inside a library — see the `vm-templates` skill.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Plain-English description | Category |
|---|---|---|
| `Content.Library_list` | List content libraries (local and subscribed) | Content Library |
| `Content.Library_get` | Get a single content library's details | Content Library |
| `Content.LocalLibrary_create` | Create a new local (directly editable) content library | Content Library |
| `Content.LocalLibrary_update` | Change an existing local library's settings | Content Library |
| `Content.LocalLibrary_delete` | Delete a local content library | Content Library |

## Verification checklist

- [ ] Confirmed whether the target library is local or subscribed before attempting any edit
- [ ] For a subscribed library, confirmed the edit was made on the source library instead
