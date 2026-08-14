---
name: vmware-tagging-categorization
description: How to create and manage VMware vSphere tag categories and tags for organizing and targeting inventory. Vendor-neutral. Use when building, reviewing, or debugging vSphere tagging automation, on Itential or otherwise.
---

# VMware vSphere — Tagging & Categorization

## When to use this skill

- Creating tag categories or tags.
- Using tags to organize or target inventory objects.

## Operational procedure

1. Decide whether the category allows multiple tags per object or exactly one — this is a category-level setting decided at creation time, not something to casually change later once tags are widely applied.
2. Create the tag category — it defines the *type* of tag (e.g. "Environment") and which object types it applies to.
3. Create tags within that category (e.g. "Production") — a tag can't exist outside a category.
4. Apply tags to objects. Tags are the mechanism most automation uses to *find* objects — a tagging mistake doesn't just look wrong, it can silently break whatever downstream process was filtering by that tag.

## Patterns

- **Category before tag, always** — a tag has no meaning or valid existence outside its category.
- **Decide cardinality (single vs. multiple tags per object) at category-creation time**, since changing it later is disruptive once tags are widely applied.

## Known limitations

- No cross-object blast-radius reasoning — this skill doesn't trace every downstream automation that filters by a given tag before that tag is changed or removed.

## Tools

Every operation below is a real, confirmed-active method on the VMware vSphere Automation REST API (source: the vSphere Automation adapter's live task catalog, dot-notation naming matching VMware's own API namespace).

| Operation | Description | Category |
|---|---|---|
| `Cis.Tagging.Category_list` | List tag categories (the type of tag, and what object types it applies to) | Tagging |
| `Cis.Tagging.Category_create` | Create a new tag category | Tagging |
| `Cis.Tagging.Category_get` | Get a tag category's details | Tagging |
| `Cis.Tagging.Tag_list` | List tags | Tagging |
| `Cis.Tagging.Tag_create` | Create a new tag within a category | Tagging |
| `Cis.Tagging.Tag_get` | Get a tag's details | Tagging |
| `Cis.Tagging.Tag_update` | Change an existing tag | Tagging |

## Verification checklist

- [ ] Tag category's applicable object types and cardinality (single/multiple) confirmed correct before tags are widely applied
- [ ] After tagging an object, confirmed the tag is visible via `Cis.Tagging.Tag_list`/`_get` on that object, not just that the create call succeeded
- [ ] Before changing or removing a widely-used tag, checked whether downstream automation filters by it
