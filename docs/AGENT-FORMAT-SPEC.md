# Network and Infrastructure Agents — Vendor Package Format Spec

This document defines the format every vendor package in this marketplace must follow. Vendor content lives in `vendors/<slug>/`; this spec defines the *shape* that content must take.

This is a **pure skill library** — vendor-neutral, platform-neutral operational knowledge, meant to be loaded into a skill registry and called by whatever agent framework a consumer already uses. It does not prescribe agent architecture, tool-count ceilings, or approval-mechanism implementation details for any specific platform. Those are downstream decisions for whoever builds an agent on top of a skill — not something this repository bakes in.

The format reuses a convention that already has independent industry traction:

- **`SKILL.md`** — Anthropic's Claude Skills format: YAML frontmatter (`name`, `description`) followed by structured markdown, designed to be loaded on demand rather than kept in context permanently. Each skill folder is directly usable as a Claude Skill if a consumer drops it into `.claude/skills/`, or loaded into any other skill registry that understands the same shape.

Skills should be legible to any tool that already understands Claude Skills, with zero adapter code, and buildable by anyone with access to the vendor's own API documentation — **no dependency on any specific automation platform, and no dependency on live access to a running instance of the target product.**

## Directory shape

A vendor is a collection of individual, single-domain skills — not one monolithic document per vendor. Each skill maps to one functional domain someone would actually reach for on its own (e.g. "load balancing," "VM encryption," "DNS services") — think of it as the same granularity as a single-purpose agent scoped to one job, minus any agent-framework specifics.

```
vendors/<vendor-slug>/
├── README.md                    — human-facing vendor index: what skills exist, at a glance
├── <skill-slug>/
│   └── SKILL.md                 — one domain's real operational procedure + exhaustive tool reference
├── <skill-slug>/
│   └── SKILL.md
└── ...
```

`<vendor-slug>` and `<skill-slug>` are lowercase-kebab-case (`citrix`, `load-balancing`, `6connect`). One top-level folder per vendor. Split a vendor into as many skill folders as it has distinct functional domains — a vendor with a narrow product surface might have only 2-3 skills; one with a broad API surface (a NetScaler, a vSphere) might have a dozen or more. Don't force an artificial split if a vendor's real domain is genuinely one thing, and don't cram unrelated domains into one skill just to keep the count down.

## `README.md` — required sections (vendor level)

1. **One-paragraph summary** — what this vendor's product covers and who'd reach for it.
2. **Skills** — a table listing every skill folder and the one-line domain it covers, linking to each.
3. **Coverage summary** — total real operations documented across all skills, and how that's split (a short list, not the full table — that's what each skill's Tools section is for).
4. **Source** — what the tool references were built from (a specific OpenAPI spec + version, a specific API doc page, a CLI reference version). Cite it precisely enough that someone could go verify a given operation against the same source.
5. **Prerequisites** — what must exist on the real target product before any of this is usable (minimum version, required license/feature, auth model).

## `SKILL.md` — required sections (per skill)

Follows the Claude Skill format exactly: YAML frontmatter, then markdown.

```yaml
---
name: <vendor-slug>-<skill-slug>
description: <one line, specific enough to disambiguate this skill from others in a multi-skill index>
---
```

**Core principle: `SKILL.md` teaches one domain's procedure AND is that domain's exhaustive tool lookup — in one file.** Someone — or something — should be able to load just this one skill and correctly perform the operation on the real product, on any orchestration platform, with zero foreign-platform-specific knowledge required, and then find the exact real operation name for any step without leaving the document. It is not an agent-design document: it does not say how many tools to hand an LLM at once, does not name a specific approval-UI mechanism, and does not describe "agents" as objects. That's for whoever builds on top of this skill to decide, in their own framework, using their own judgment.

Required body sections, in this order:

1. **When to use this skill** — the trigger conditions: what a request looks like when it belongs to this specific domain. Scoped to one domain, this can be a short bullet list rather than a cross-domain routing table.
2. **Operational procedure** — the actual, real-world procedure for this domain, written the way a domain expert would write a runbook: what order things must happen in, what to check before acting, what decision points exist, and why. Reference the *product's own* API/CLI concepts by their real names — that's product knowledge. Weave load-bearing caveats (a resource that's schema-valid but non-functional in practice, an ordering constraint that isn't obvious) directly into the relevant step, in plain language — don't quarantine them in a separate "gotchas" list most readers will skip.
3. **Patterns** — reusable, product-level conventions for this domain that hold regardless of what executes them.
4. **Known limitations** — what the real product/API genuinely does not support in this domain (a documented product gap, not a bug list), and any scope boundary the procedure above deliberately doesn't cross.
5. **Tools** — the exhaustive, real-operation lookup for this domain, as one table or a small set of tables grouped by sub-category, with these columns:

   | Column | What goes in it |
   |---|---|
   | **Operation** | The real, actual name of the operation — an OpenAPI `operationId`, a documented REST endpoint (method + path), a real CLI command, or whatever the vendor's own documentation calls it. Never invented, never paraphrased. |
   | **Plain-English description** | What it actually does, in a sentence a non-specialist could follow. This is the "translate the vendor's jargon into English" value-add — don't just restate the operation name in title case. |
   | **Category** | Which sub-area within this domain this belongs to. |

   Add a **Method / Path** or **CLI syntax** column when the vendor's API is REST or CLI-shaped and that detail is genuinely useful at a glance; omit it if the operation name alone is unambiguous.

   **Every row must be traceable to a real source** — the OpenAPI spec, API doc, or CLI reference cited in that vendor's `README.md`. Do not add a row for an operation you inferred should exist but haven't confirmed in the actual source. If the source material doesn't clearly state what an operation does, say so in the description rather than guessing ("purpose unclear from the spec — verify before relying on this").
6. **Verification checklist** — what to check before considering this domain's procedure correctly implemented, at the level of "did the real-world state actually change as intended."

## Machine-readable index

Every vendor package, and every skill within it, must have a corresponding entry in the root [`registry.json`](../registry.json) — see that file's own header comment for its schema. The registry is what makes this a marketplace instead of a folder tree: it's the thing a catalog UI, search tool, or CLI would actually query. A skill without a registry entry is invisible to tooling even if the file is perfect.

## Anti-patterns (things a contribution will be rejected for)

- **Baking agent architecture into a skill.** No prescribing tool-count ceilings for a specific agent, no naming a specific approval-UI mechanism as *the* way to gate a write, no describing "agents" as objects this package defines. That's for whoever consumes the skill to decide.
- **A Tools row for an operation that wasn't confirmed in the vendor's own real documentation.** A reference table with invented rows is worse than no reference table — it looks authoritative and isn't.
- **No machine-readable index.** A marketplace that's only human-browsable markdown isn't discoverable by anything except a human with time to read every folder.
- **One monolithic `SKILL.md` covering every domain of a broad vendor.** Split by domain — the whole point of per-skill files is that a consumer loads only what a specific task needs, not an entire vendor's worth of context.
- **`SKILL.md` written as an exhaustive tool-mapping document with procedure as an afterthought.** The procedure is the durable, portable knowledge; the Tools table is a lookup aid. If a skill is 90% restated reference rows and 10% procedure, the ratio is backwards.
- **A dedicated "Gotchas" section that duplicates what's already stated inline in a procedure.** If a caveat matters, it belongs woven into the step it affects, in the reader's natural path through the document — not in a separate list most people skip past.
- **Self-referential language that asserts a property of the document instead of just having that property** (e.g., "note that this document is vendor-neutral" while also naming specific vendors as proof). State the rule; don't narrate that you're following it.
