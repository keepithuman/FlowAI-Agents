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

Required body sections:

1. **When to use this skill** — the trigger conditions (what a request looks like when it belongs here).
2. **Agent-to-tool map** — for every agent in every project under this vendor: its exact tool list (by real, callable method name — not paraphrased), and which project file it lives in.
3. **Patterns** — the concrete, reusable request/response shapes an agent-builder needs (payload wrapper conventions, ID formats, binding/dependency ordering between create calls).
4. **Gotchas** — real, previously-hit failure modes and their fixes, each with: what broke, why, and the fix. Not hypothetical — only include gotchas that were actually discovered building or running an agent in this package.
5. **Verification checklist** — what to check after building or modifying an agent in this package before considering it done (tool resolution correctness, provider resolution, whatever is specific to this vendor's platform quirks).

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
