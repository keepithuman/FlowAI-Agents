# Network and Infrastructure Agents

[![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)

A vendor-organized marketplace of network and infrastructure automation agents — organized by the real operational procedure for each domain (vendor-neutral, no platform assumed), with a ready-to-import [Itential FlowAI](https://www.itential.com/) reference implementation for every procedure so you can accelerate the build if you happen to be on that platform.

## Why this exists

Most agent examples floating around are either a single hero demo or a wall of undocumented JSON, and most are tied to one specific orchestration platform whether they need to be or not. This repo aims to be neither: the procedure knowledge in each vendor's `SKILL.md` stands on its own regardless of what executes it, and every accompanying `.project.json` reference implementation was actually created on a live Itential Platform instance and `GET`-verified (tool references resolve, provider is set, every mutating agent has its approval gate wired) before being committed — not hand-authored from memory. The documentation format is deliberately built on conventions that already exist elsewhere — `AGENTS.md` and Claude's `SKILL.md` — so a vendor package here is legible to tools that already understand either one, with no adapter layer.

## How it's organized

```
vendors/<vendor-slug>/
├── README.md      — what's here, at a glance
├── AGENTS.md       — orientation: domain overview, design principles, capability index
├── SKILL.md          — the real operational procedure, vendor-neutral
└── projects/
    └── *.project.json   — real, exported FlowAI project bundles (Itential accelerator, optional)
```

The exact contract every vendor package must satisfy is defined in [`docs/AGENT-FORMAT-SPEC.md`](./docs/AGENT-FORMAT-SPEC.md) — read that before adding a new vendor or judging whether an existing one is "done."

[`registry.json`](./registry.json) is the machine-readable index of every vendor, project, and agent in this repo — the thing a catalog UI or search tool would actually query, rather than crawling markdown.

## Using an agent from this marketplace

1. Find the vendor and project you need via [`registry.json`](./registry.json) or by browsing `vendors/`.
2. Read that vendor's `AGENTS.md` for the design principles and capability map, and `SKILL.md` if you need exact tool names or known gotchas.
3. Import the project file: `POST /agent-project-service/project-bundles/import` on your Itential Platform instance, with the `.project.json` contents as the `bundle` field. You'll need to resolve `provider` (LLM profile/model) to something that exists in *your* environment — the values in each export reflect the platform they were built on, not a portable default.
4. Verify before trusting: `GET` each imported agent back, confirm every tool resolved to a real `referenceId` (not `unauthorizedReferenceId`), and confirm `provider` is populated. See the vendor's `SKILL.md` for a fuller verification checklist.

## Vendors currently in the marketplace

| Vendor | Domain | Projects | Agents |
|---|---|---|---|
| [Citrix](./vendors/citrix/) | NetScaler ADC — load balancing, traffic routing, SSL, GSLB, security, remote access, system administration, networking, clustering/HA, DNS, bot management, traffic optimization | 13 | 33 |

## Design principles that apply across every vendor, not just one

- **Propose, then wait, then act.** Every agent capable of changing a real system's state proposes the exact change and requires explicit human approval before it executes. This is not a per-vendor style choice — it's the baseline every vendor package must meet.
- **Small, single-purpose agents over mega-agents.** An agent with 20+ tools degrades as an LLM tool-caller — it starts missing relevant tools or picking plausible-but-wrong ones. Every agent in this marketplace tops out around 10 tools; broader domains become multiple agents in one project, not one large agent.
- **Nothing published that wasn't actually built and verified.** Every `.project.json` here is a real export from a platform where the project was created and its agents `GET`-verified — not a hand-authored guess at what a bundle should look like.

## Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) to get started. Before contributing, you'll need to sign our [Contributor License Agreement](CLA.md). This project follows a [Code of Conduct](CODE_OF_CONDUCT.md); see [SECURITY.md](SECURITY.md) for reporting a security issue.

## License

This project is licensed under the GNU General Public License v3.0 — see the [LICENSE](LICENSE) file for details.
