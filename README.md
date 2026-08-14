# Network and Infrastructure Agents

[![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)

A vendor-organized library of network and infrastructure operational skills — real procedures, real API operations, zero platform lock-in. Load a vendor's skill into whatever skill registry or agent framework you already use; nothing here assumes a specific orchestration platform.

## Why this exists

Most agent examples floating around are either a single hero demo or a wall of undocumented JSON, and most are tied to one specific orchestration platform whether they need to be or not, or to one specific platform's live build state to even verify. This repo aims to be neither: each vendor's `SKILL.md` teaches the real operational procedure — sequencing, decision points, load-bearing caveats — and then gives you the exhaustive tool reference — every real operation, named exactly as the vendor's own API/CLI names it, with a plain-English description — in the same file. It doesn't require live access to a running instance of the target product to build or verify — it's sourced from the vendor's own public documentation, which means this format scales to covering a lot of vendors, not just the ones someone happens to have lab access to.

The documentation format reuses a convention that already has independent industry traction — Claude's `SKILL.md` — so a vendor package here is legible to any tool that already understands it, with no adapter layer.

## How it's organized

```
vendors/<vendor-slug>/
├── README.md            — what's here, at a glance
└── SKILL.md              — the real operational procedure + exhaustive tool reference, vendor-neutral
```

The exact contract every vendor package must satisfy is defined in [`docs/AGENT-FORMAT-SPEC.md`](./docs/AGENT-FORMAT-SPEC.md) — read that before adding a new vendor or judging whether an existing one is "done."

[`registry.json`](./registry.json) is the machine-readable index of every vendor in this repo — the thing a catalog UI or search tool would actually query, rather than crawling markdown.

## Using a skill from this marketplace

1. Find the vendor you need via [`registry.json`](./registry.json) or by browsing `vendors/`.
2. Read that vendor's `SKILL.md` for the real operational procedure and its capability index (which section answers which kind of request).
3. Load `SKILL.md` into your skill registry / agent framework of choice — it's already in the Claude Skills shape (YAML frontmatter + markdown), and portable to anything else that consumes similarly-shaped skill documents.
4. When you (or an agent you build) need the exact operation name for a step in the procedure, look it up in the same `SKILL.md`'s Tools section.

## Vendors currently in the marketplace

| Vendor | Domain | Real operations documented |
|---|---|---|
| [Citrix](./vendors/citrix/) | NetScaler ADC — load balancing, traffic routing, SSL, GSLB, security, remote access, system administration, networking, clustering/HA, DNS, bot management, traffic optimization | 211 |
| [VMware](./vendors/vmware/) | vSphere — VM operations, infrastructure inventory, storage, resource pools, content library, guest customization, tagging, RBAC, certificates, VM encryption, cluster configuration, performance metrics, diagnostics | 85 |

## Principles that apply across every vendor, not just one

- **Human approval before any state-changing action.** Every procedure that can mutate a real system's state says so explicitly, and describes proposing the exact change before acting on it. This repo doesn't prescribe *how* you implement that gate — that's a decision for whoever builds on top of a skill — but every procedure is written assuming one exists.
- **No agent architecture baked into the skill.** No tool-count ceilings, no named UI mechanism, no "agents" as objects this repository defines. A skill is domain knowledge; how many tools you hand an LLM at once and how you gate a write are downstream decisions in whatever framework you're using.
- **Nothing published that wasn't confirmed against a real source.** Every row in every vendor's Tools table traces back to a specific, cited API spec or doc — never invented, never inferred without saying so.

## Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) to get started. Before contributing, you'll need to sign our [Contributor License Agreement](CLA.md). This project follows a [Code of Conduct](CODE_OF_CONDUCT.md); see [SECURITY.md](SECURITY.md) for reporting a security issue.

## License

This project is licensed under the GNU General Public License v3.0 — see the [LICENSE](LICENSE) file for details.
