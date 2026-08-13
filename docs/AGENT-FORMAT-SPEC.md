# FlowAI Agents — Vendor Package Format Spec

This document defines the format every vendor package in this marketplace must follow. It is vendor-agnostic — nothing below refers to Citrix, NetScaler, or any other specific integration. Vendor content lives in `vendors/<slug>/`; this spec defines the *shape* that content must take.

The format deliberately reuses two conventions that already have independent industry traction, rather than inventing a third:

- **`AGENTS.md`** — the open, tool-agnostic convention (see [agents.md](https://agents.md)) for a single markdown file that orients an AI agent (or a human) to a domain before it does any work. Widely supported by coding agents (Claude Code, Codex, Cursor, and others) as a root-of-context file. Here, one exists per vendor instead of per repo.
- **`SKILL.md`** — Anthropic's Claude Skills format: YAML frontmatter (`name`, `description`) followed by structured markdown, designed to be loaded on demand rather than kept in context permanently. A vendor's `SKILL.md` is directly usable as a Claude Skill if a consumer drops the vendor folder into `.claude/skills/`.

Neither format is a marketplace invention — that's the point. A vendor package should be legible to any tool that already understands `AGENTS.md` or Claude Skills, with zero adapter code.

## Directory shape

```
vendors/<vendor-slug>/
├── README.md          — human-facing vendor index (what's here, at a glance)
├── AGENTS.md           — orientation: how to work in this vendor's domain
├── SKILL.md             — on-demand reference: exact tools, patterns, gotchas
└── projects/
    └── <project-slug>.project.json   — real, exported FlowAI project bundles
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
   - The approval model (does every mutating action require human sign-off before it executes, and via what mechanism?)
   - The modularity rule actually applied (how tools are grouped into agents, and the reasoning — not just "here are the agents")
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

**Core principle: `SKILL.md` teaches the domain procedure, not the Itential wiring.** This marketplace is vendor-neutral by design — "vendor" here means the *product* (Citrix NetScaler, Cisco IOS, ServiceNow), not Itential. Someone should be able to read a `SKILL.md` and correctly implement the same agent behavior on a completely different orchestration platform, with zero Itential-specific knowledge required. The `.project.json` files are the accelerator *if* the reader happens to be on Itential — not the point of the document.

Required body sections, in this order:

1. **When to use this skill** — the trigger conditions (what a request looks like when it belongs here).
2. **Operational procedures** — the actual, real-world procedure for each capability area, written the way a domain expert would write a runbook: what order things must happen in, what to check before acting, what decision points exist, and why. Reference the *product's own* API/CLI concepts by their real names (e.g., a NITRO resource name, a CLI verb) — that's product knowledge, not platform wiring. Do **not** reference Itential tool names, `referenceId` strings, agent names, or project files in this section. If you can't describe a step without naming an Itential tool, you're describing the implementation, not the procedure — push it to section 4 instead.
3. **Patterns** — reusable, product-level conventions that hold regardless of what executes them (e.g., a vendor API's dependency ordering between object types, a naming convention, a resource that's schema-valid but non-functional in practice).
4. **Itential reference implementation** *(clearly separated, clearly optional)* — for each procedure in section 2, which project/agent in `projects/` already implements it, and its exact tool list (real, callable method names — not paraphrased). Framed explicitly as: "if you're building this on Itential, here's what's already done for you" — not as the primary content.
5. **Gotchas** — split into two clearly labeled groups:
   - **Product-level** (vendor-neutral — true regardless of orchestration platform, e.g. an API resource that's schema-valid but has no real backing command)
   - **Itential-implementation-level** (specific to how this was wired up on Itential's Agent Project Service / Tools Service — e.g. a tool-catalog resolution quirk)
6. **Verification checklist** — what to check before considering a procedure correctly implemented. Split the same way as gotchas: product-level checks (did the real-world state actually change as intended) vs. Itential-implementation checks (did the tool reference resolve, is provider populated) if the reader is using the reference implementation.

A `SKILL.md` that's 90% Itential tool-mapping tables and 10% procedure has the ratio backwards — fix it before it ships.

## `projects/*.project.json` — required shape

Each file is a **real, exported FlowAI project bundle** (`agentProjectBundleVersion: 1` shape — see the platform's Agent Project Service `project-bundles/{projId}/export` response). Never hand-author these from memory. A project file must be either:

- a direct export of a project that was actually created and verified on a live Itential Platform instance, or
- clearly marked `"status": "draft"` in an accompanying note in the vendor's README if it hasn't been verified live yet.

Do not publish a project file whose agents have never been created and checked (`GET` back, tool references confirmed resolvable) on an actual platform. A marketplace of agent definitions that don't actually work when imported is worse than no marketplace.

## Machine-readable index

Every vendor package must have a corresponding block in the root [`registry.json`](../registry.json) — see that file's own header comment for its schema. The registry is what makes this a marketplace instead of a folder tree: it's the thing a catalog UI, search tool, or CLI would actually query. A vendor package without a registry entry is invisible to tooling even if the files are perfect.

## Anti-patterns (things that have gone wrong in practice — don't repeat them)

- **One mega-agent per vendor with 20+ tools.** An LLM tool-caller degrades as its tool list grows — it starts picking plausible-but-wrong tools, or ignoring genuinely relevant ones buried in a long list. Split by sub-capability instead. As a rule of thumb, if an agent's tool count is pushing past ~10, it's covering more than one concern — split it.
- **Trusting a tool discovery result without checking `active`/duplicate-catalog issues.** Some platform integrations register multiple app-name variants for the same underlying API (an old inactive one alongside a current active one). Tool resolution must filter for the active, correct variant — never take the first result blindly. Document this per-vendor in that vendor's `SKILL.md` if it applies.
- **Skipping the human-approval gate on any mutating action "because the demo is low-risk."** The approval gate is the product, not a demo nicety — every agent that can change a real system's state proposes the exact change and waits for explicit sign-off before acting. This is non-negotiable across all vendors in this marketplace, not a per-vendor style choice.
- **Publishing a project file that was never created and verified on a real platform.** Hallucinated tool names and made-up payload shapes look identical to real ones until someone tries to import the bundle. Every published project must have been built and `GET`-verified for real.
- **No machine-readable index.** A marketplace that's only human-browsable markdown isn't discoverable by anything except a human with time to read every folder.
- **`SKILL.md` written as an Itential tool-mapping document with procedure as an afterthought.** This inverts the actual value: the procedure is the durable, portable knowledge; the tool mapping is a convenience for one specific platform. A reader on a different orchestration platform should get full value from a vendor's `SKILL.md` without ever seeing an Itential-specific string.
