# Network and Infrastructure Agents — Vendor Package Format Spec

This document defines the format every vendor package in this marketplace must follow. Vendor content lives in `vendors/<slug>/`; this spec defines the *shape* that content must take.

The format deliberately reuses two conventions that already have independent industry traction, rather than inventing a third:

- **`AGENTS.md`** — the open, tool-agnostic convention (see [agents.md](https://agents.md)) for a single markdown file that orients an AI agent (or a human) to a domain before it does any work. Widely supported by coding agents (Claude Code, Codex, Cursor, and others) as a root-of-context file. Here, one exists per vendor instead of per repo.
- **`SKILL.md`** — Anthropic's Claude Skills format: YAML frontmatter (`name`, `description`) followed by structured markdown, designed to be loaded on demand rather than kept in context permanently. A vendor's `SKILL.md` is directly usable as a Claude Skill if a consumer drops the vendor folder into `.claude/skills/`.

Neither format is a marketplace invention — that's the point. A vendor package should be legible to any tool that already understands `AGENTS.md` or Claude Skills, with zero adapter code.

## Directory shape

```
vendors/<vendor-slug>/
├── README.md          — human-facing vendor index (what's here, at a glance)
├── AGENTS.md           — orientation: how to work in this vendor's domain
├── SKILL.md             — the real operational procedure, vendor-neutral
└── projects/
    └── <project-slug>.project.json   — real, exported FlowAI project bundles (Itential accelerator)
```

`<vendor-slug>` is lowercase-kebab-case (`citrix`, `cisco-ios`, `service-now`). One folder per vendor, not per product line, unless a vendor's product lines are unrelated enough to need separate agent design patterns (judgment call — document the reasoning in that vendor's README.md if you split).

## `README.md` — required sections

1. **One-paragraph summary** — what this vendor integration covers and who'd reach for it.
2. **Project index** — a table: project name, what it covers, agent count, link to the `.project.json` file.
3. **Prerequisites** — what must exist on the target platform before any of these agents will work (adapter/integration installed, minimum firmware/API version, required feature flags).

## `AGENTS.md` — required sections

This is the file an agent (human or AI) reads *first*, before touching any tool. It orients, it doesn't enumerate. Required sections, in order:

1. **Domain overview** — what real-world problem this vendor's agents solve, in plain language.
2. **Design principles** — the non-negotiable patterns every agent in this vendor package follows. At minimum, state:
   - The approval mechanism used. Every agent capable of mutating a target system's state must gate on explicit human approval before acting — this spec does not permit an unattended-write agent, regardless of the target system's apparent risk level.
   - The modularity rule actually applied. No agent's tool list may exceed 10 entries; an agent covering more than one concern must be split into multiple agents or projects.
   - Any vendor-specific footguns that a generic LLM tool-caller would not know about (e.g., a schema-valid API call that has no real backing implementation on some deployments)
3. **Capability index** — a table mapping "thing a user might ask for" → which project/agent handles it. This is the router. It must be scannable in under a minute.
4. **Known limitations** — what this vendor's agents deliberately do NOT cover yet, and why (scope boundary, not a bug list).

`AGENTS.md` must **not** contain full tool signatures, exact request/response payloads, or step-by-step API sequences — that's `SKILL.md`'s job. If `AGENTS.md` is doing `SKILL.md`'s job, it will bloat past the point of being read.

## `SKILL.md` — required sections

Follows the Claude Skill format exactly: YAML frontmatter, then markdown.

```yaml
---
name: <vendor-slug>
description: <one line, specific enough to disambiguate this skill from others in a multi-skill index>
---
```

**Core principle: `SKILL.md` teaches the domain procedure, not the Itential wiring.** "Vendor" in this spec means the product being automated, not Itential. A `SKILL.md` must be implementable on a completely different orchestration platform with zero Itential-specific knowledge required. The `.project.json` files are an accelerator for readers who happen to be on Itential — not the point of the document.

Required body sections, in this order:

1. **When to use this skill** — the trigger conditions (what a request looks like when it belongs here).
2. **Operational procedures** — the actual, real-world procedure for each capability area, written the way a domain expert would write a runbook: what order things must happen in, what to check before acting, what decision points exist, and why. Reference the *product's own* API/CLI concepts by their real names (e.g., a NITRO resource name, a CLI verb) — that's product knowledge, not platform wiring. Weave in load-bearing caveats (a resource that's schema-valid but non-functional in practice, an ordering constraint that isn't obvious) directly into the relevant step, in plain language — don't quarantine them in a separate "gotchas" list that most readers will skip. Do **not** reference Itential tool names, `referenceId` strings, agent names, or project files in this section.
3. **Patterns** — reusable, product-level conventions that hold regardless of what executes them.
4. **Reference implementation** *(one short paragraph, not a table)* — a pointer to `projects/` and that vendor's `README.md`/`registry.json` entry, framed as "already built and verified on Itential if you want the accelerator." Do not enumerate tool names, `referenceId` strings, or agent-by-agent tool lists here — that level of detail belongs in the project files themselves, which are already the source of truth for it. If a reader needs the exact tool list for a specific agent, the project file is one click away; restating it in `SKILL.md` just gives it two places to go stale.
5. **Verification checklist** — what to check before considering a procedure correctly implemented, at the level of "did the real-world state actually change as intended" — not Itential-implementation minutiae (tool resolution, provider fields). That level of detail, if worth keeping at all, belongs with whoever maintains the reference implementation, not in the portable procedure doc.

## `projects/*.project.json` — required shape

Each file is a **real, exported FlowAI project bundle** (`agentProjectBundleVersion: 1` shape — see the platform's Agent Project Service `project-bundles/{projId}/export` response). Never hand-author these from memory. A project file must be either:

- a direct export of a project that was actually created and verified on a live Itential Platform instance, or
- clearly marked `"status": "draft"` in an accompanying note in the vendor's README if it hasn't been verified live yet.

Do not publish a project file whose agents have never been created and checked (`GET` back, tool references confirmed resolvable) on an actual platform. A marketplace of agent definitions that don't actually work when imported is worse than no marketplace.

## Machine-readable index

Every vendor package must have a corresponding block in the root [`registry.json`](../registry.json) — see that file's own header comment for its schema. The registry is what makes this a marketplace instead of a folder tree: it's the thing a catalog UI, search tool, or CLI would actually query. A vendor package without a registry entry is invisible to tooling even if the files are perfect.
