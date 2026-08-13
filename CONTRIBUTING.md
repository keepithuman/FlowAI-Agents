# Contributing to Network and Infrastructure Agents

Thank you for your interest in contributing to Network and Infrastructure Agents! This document covers both the general mechanics of contributing (branches, commits, PRs — mirroring common open-source practice) and the project-specific bar every vendor package must clear.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Contributor License Agreement](#contributor-license-agreement)
- [What you can contribute](#what-you-can-contribute)
- [Vendor Package Submission Checklist](#vendor-package-submission-checklist)
- [Getting Started](#getting-started)
- [Contributing Process](#contributing-process)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Validating Before You Submit](#validating-before-you-submit)
- [Getting Help](#getting-help)

## Code of Conduct

By participating in this project, you are expected to uphold the [Code of Conduct](./CODE_OF_CONDUCT.md).

## Contributor License Agreement

**All contributors must sign the [Contributor License Agreement](./CLA.md) before their contributions can be merged.** When you submit your first pull request, comment with the statement described there. Please complete this before your contribution is reviewed.

## What you can contribute

- A **new vendor package** (`vendors/<slug>/`) — see the checklist below.
- A **new project or agent** added to an existing vendor package.
- A **refinement to an existing vendor's operational procedure** in `SKILL.md` — a missing step, a caveat woven into the wrong place, a clearer explanation of a decision point — these are valuable even without new agents attached.
- Improvements to [`docs/AGENT-FORMAT-SPEC.md`](./docs/AGENT-FORMAT-SPEC.md) itself, if you find the format spec is missing something a real vendor package needed.

## Vendor Package Submission Checklist

Full detail lives in [`docs/AGENT-FORMAT-SPEC.md`](./docs/AGENT-FORMAT-SPEC.md) — this is the short version to check against before opening a PR:

- [ ] **Built and verified on a real platform first.** Every `.project.json` must be a real export from a live Itential Platform instance where the project was created and every agent `GET`-verified (tool references resolved, provider populated). Never hand-author a bundle from memory.
- [ ] Directory is `vendors/<vendor-slug>/` — lowercase-kebab-case.
- [ ] `README.md`, `AGENTS.md`, and `SKILL.md` are all present and each has the required sections from the format spec.
- [ ] No agent exceeds ~10 tools. If your domain needs more, split into more agents or more projects — don't widen one agent's tool list past that point.
- [ ] Every mutating agent proposes the exact change and gates on human approval (`view:WorkFlowEngine:ViewData` or an equivalent) before acting. No exceptions for "it's just a demo."
- [ ] No offensive, destructive, or evasive capability anywhere in the package.
- [ ] `registry.json` updated with your vendor's entry, matching the actual contents of `projects/`.
- [ ] Caveats in `SKILL.md`'s operational procedures are real and woven into the relevant step — not quarantined in a separate list, and not hypothetical.

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally
3. **Create a topic branch** for your changes
4. **Make your changes** — build and verify against a real platform if you're adding or modifying a vendor package
5. **Submit a pull request**

## Contributing Process

### Fork and Pull Model

1. **Fork the repository** to your GitHub account
2. **Create a topic branch** from `main`:
   ```bash
   git checkout main
   git pull upstream main
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes** in logical, atomic commits — commit messages should follow the format below
4. **Push to your fork:**
   ```bash
   git push origin feature/your-feature-name
   ```
5. **Create a pull request** against the `main` branch

### Branch Naming

Format: `<type>/<description>`, lowercase letters/numbers/hyphens only in the description:

| Type | Use for |
|---|---|
| `feature/` | A new vendor package, or a new project/agent within an existing one |
| `fix/` | Correcting a broken tool reference, wrong payload shape, or documentation error |
| `refactor/` | Restructuring an existing vendor package without changing its agents' behavior |
| `docs/` | Format spec or top-level documentation changes |
| `chore/` | Registry regeneration, housekeeping |

Examples: `feature/add-cisco-ios-vendor`, `fix/correct-nslicense-tool-reference`, `docs/clarify-modularity-rule`

### Commit Message Format

Follows [Conventional Commits](https://www.conventionalcommits.org/): `<type>[(scope)]: <description>`

| Type | Meaning |
|---|---|
| `feat` | New vendor package, project, or agent |
| `fix` | Correcting a broken reference, payload, or doc error |
| `docs` | Documentation-only change |
| `refactor` | Restructuring without behavior change |
| `chore` | Housekeeping, registry regeneration |

Examples:
- `feat(citrix): add DNSSEC agent to dns-services project`
- `fix(citrix): remove non-functional createResponderpolicyLbvserverBinding reference`
- `docs: clarify agent tool-count ceiling in format spec`

**Merge commits are discouraged** — prefer squash or rebase to integrate updates from `main`.

## Pull Request Guidelines

### Before Submitting

- [ ] Every `.project.json` you're adding or changing is a real export, not hand-authored
- [ ] `registry.json` reflects the actual current state of `vendors/`
- [ ] `AGENTS.md` and `SKILL.md` follow the required sections in the format spec
- [ ] Signed the CLA (see above)

### Pull Request Description

Include:
1. **Clear title** describing the change
2. **What platform you built/verified against** and what you checked (which `GET` calls, what they returned) — a reviewer without platform access should be able to trust your verification claim without re-running it themselves
3. **Any known limitations or gotchas** worth calling out

## Validating Before You Submit

This repo has no build step, but there is a real bar to check against before opening a PR:

```bash
# Every .project.json must be valid JSON
for f in vendors/*/projects/*.json; do python3 -c "import json,sys; json.load(open(sys.argv[1]))" "$f" || echo "INVALID: $f"; done

# registry.json must be valid JSON and its agent counts must match the actual project files
python3 -c "import json; json.load(open('registry.json'))"
```

If you're adding a vendor package, re-read `docs/AGENT-FORMAT-SPEC.md`'s "Anti-patterns" section and check your package against each item — most rejected PRs will be rejected for one of those, not for anything novel.

## Getting Help

- **Documentation**: Start with the top-level [`README.md`](./README.md) and [`docs/AGENT-FORMAT-SPEC.md`](./docs/AGENT-FORMAT-SPEC.md)
- **Discussions**: Use GitHub Discussions for questions
- **Maintainer**: [@keepithuman](https://github.com/keepithuman)

### Reporting Issues

Include: a clear description, which vendor package/file is affected, expected vs. actual, and (if it's a broken tool reference) how you confirmed it's broken.

## Recognition

Contributors who have PRs merged will be listed in the project's contributors and credited in the relevant vendor package's README where appropriate.

Thank you for contributing to Network and Infrastructure Agents!
