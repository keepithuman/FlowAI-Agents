# Contributing to Network and Infrastructure Skills

Thank you for your interest in contributing to Network and Infrastructure Skills! This document covers both the general mechanics of contributing (branches, commits, PRs — mirroring common open-source practice) and the project-specific bar every skill must clear.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Contributor License Agreement](#contributor-license-agreement)
- [What you can contribute](#what-you-can-contribute)
- [Skill Submission Checklist](#skill-submission-checklist)
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

- A **new skill** within an existing vendor (`vendors/<slug>/<new-skill-slug>/SKILL.md`) — a functional domain that vendor's package doesn't cover yet.
- A **new vendor package** (`vendors/<slug>/`, one or more skill folders inside it) — see the checklist below.
- A **refinement to an existing skill's operational procedure or tool reference** — a missing step, a caveat woven into the wrong place, a clearer explanation of a decision point, a missing or incorrectly-described operation — these are valuable on their own.
- Improvements to [`docs/SKILL-FORMAT-SPEC.md`](./docs/SKILL-FORMAT-SPEC.md) itself, if you find the format spec is missing something a real skill needed.

## Skill Submission Checklist

Full detail lives in [`docs/SKILL-FORMAT-SPEC.md`](./docs/SKILL-FORMAT-SPEC.md) — this is the short version to check against before opening a PR:

- [ ] **Every operation named in the skill's `SKILL.md` is real and confirmed against the vendor's own documentation.** Never invent or paraphrase an operation name — cite the exact OpenAPI spec, API doc, or CLI reference in the vendor's `README.md`'s Source section.
- [ ] Vendor directory is `vendors/<vendor-slug>/`, skill directory is `vendors/<vendor-slug>/<skill-slug>/` — both lowercase-kebab-case.
- [ ] The skill maps to one functional domain, not a bundle of unrelated domains crammed together to avoid adding another folder.
- [ ] `SKILL.md` has the required sections from the format spec, and the vendor's `README.md` lists the new skill.
- [ ] `SKILL.md` does not bake in orchestration architecture — no tool-count ceilings, no named approval-UI mechanism, no calling-software object model this package defines. That's a downstream decision for whoever consumes the skill.
- [ ] No offensive, destructive, or evasive capability anywhere in the package.
- [ ] `registry.json` updated with the new/changed skill entry, matching the actual contents of `vendors/<slug>/`.
- [ ] Caveats in the procedure are real and woven into the relevant step — not quarantined in a separate list, and not hypothetical.

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally
3. **Create a topic branch** for your changes
4. **Make your changes** — every operation name you add must be traceable to a real source (OpenAPI spec, API doc, CLI reference)
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
| `feature/` | A new vendor package or a new skill within an existing vendor |
| `fix/` | Correcting a wrong operation name, description, or documentation error |
| `refactor/` | Restructuring an existing skill without changing its documented behavior |
| `docs/` | Format spec or top-level documentation changes |
| `chore/` | Registry regeneration, housekeeping |

Examples: `feature/add-cisco-ios-vendor`, `feature/add-citrix-appflow-skill`, `fix/correct-nslicense-operation-description`, `docs/clarify-tools-section-requirements`

### Commit Message Format

Follows [Conventional Commits](https://www.conventionalcommits.org/): `<type>[(scope)]: <description>`

| Type | Meaning |
|---|---|
| `feat` | New vendor package or skill |
| `fix` | Correcting a wrong operation name, description, or doc error |
| `docs` | Documentation-only change |
| `refactor` | Restructuring without behavior change |
| `chore` | Housekeeping, registry regeneration |

Examples:
- `feat(cisco-ios): add vendor package covering IOS device configuration`
- `feat(citrix): add dns-services skill`
- `fix(citrix): correct non-functional createResponderpolicyLbvserverBinding reference`
- `docs: clarify required Tools section columns in format spec`

**Merge commits are discouraged** — prefer squash or rebase to integrate updates from `main`.

## Pull Request Guidelines

### Before Submitting

- [ ] Every operation name you're adding or changing in a skill's Tools section is traceable to a real, cited source — not hand-authored from memory
- [ ] `registry.json` reflects the actual current state of `vendors/`
- [ ] `README.md` (vendor level) and `SKILL.md` (skill level) follow the required sections in the format spec
- [ ] Signed the CLA (see above)

### Pull Request Description

Include:
1. **Clear title** describing the change
2. **What source you verified against** (spec name + version, doc URL, CLI reference version) — a reviewer without access to that source should be able to trust your verification claim without re-running it themselves
3. **Any known limitations or gotchas** worth calling out

## Validating Before You Submit

This repo has no build step, but there is a real bar to check against before opening a PR:

```bash
# registry.json must be valid JSON
python3 -c "import json; json.load(open('registry.json'))"

# Every skill in registry.json must have a matching SKILL.md on disk, and every vendor a README.md
python3 -c "
import json, os
reg = json.load(open('registry.json'))
for v in reg['vendors']:
    assert os.path.exists(v['readme']), f\"missing {v['readme']}\"
    for s in v['skills']:
        assert os.path.exists(s['path']), f\"missing {s['path']}\"
print('ok')
"
```

If you're adding a vendor package or skill, re-read `docs/SKILL-FORMAT-SPEC.md`'s "Anti-patterns" section and check your package against each item — most rejected PRs will be rejected for one of those, not for anything novel.

## Getting Help

- **Documentation**: Start with the top-level [`README.md`](./README.md) and [`docs/SKILL-FORMAT-SPEC.md`](./docs/SKILL-FORMAT-SPEC.md)
- **Discussions**: Use GitHub Discussions for questions
- **Maintainer**: [@keepithuman](https://github.com/keepithuman)

### Reporting Issues

Include: a clear description, which vendor/skill file is affected, expected vs. actual, and (if it's a wrong operation name or description) how you confirmed the correct one.

## Recognition

Contributors who have PRs merged will be listed in the project's contributors and credited in the relevant vendor package's README where appropriate.

Thank you for contributing to Network and Infrastructure Skills!
