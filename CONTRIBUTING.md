# Contributing a Vendor Package

This marketplace only accepts vendor packages that meet [`docs/AGENT-FORMAT-SPEC.md`](./docs/AGENT-FORMAT-SPEC.md) in full. That document is the actual contract — this file is the process for submitting against it.

## Before you start

- **Build and verify on a real platform first.** Every `.project.json` in this repo is a real export from a live Itential Platform instance where the project was created and every agent `GET`-verified — tool references resolved, provider populated, approval gates wired. Don't hand-author a bundle from memory and submit it; import failures on a consumer's platform erode trust in the whole marketplace, not just your vendor package.
- **Check whether the vendor already exists.** If it does, you're extending an existing package (new project, new agent, or a documented gotcha), not creating a new top-level folder.

## Submission checklist

1. **Directory**: `vendors/<vendor-slug>/` — lowercase-kebab-case, one folder per vendor (see the format spec for when to split product lines into separate vendor folders instead).
2. **`README.md`**: one-paragraph summary, project index table, prerequisites.
3. **`AGENTS.md`**: domain overview, design principles (state your approval model and modularity rule explicitly, don't assume the reader already knows this marketplace's baseline), capability index, known limitations.
4. **`SKILL.md`**: valid YAML frontmatter (`name`, `description`), when-to-use-this-skill, a complete agent-to-tool map with real `referenceId`s (not paraphrased), patterns, gotchas actually encountered while building (not hypothetical), and a verification checklist.
5. **`projects/*.project.json`**: real exports only. Every agent inside must have been created on a live platform and confirmed via `GET` to have zero broken tool references and a resolved `provider`.
6. **`registry.json` entry**: add your vendor's block following the existing schema (see the header comment in that file). This is not optional — an unregistered vendor package is invisible to any tooling built against this marketplace.

## Design bar

- No agent with more than ~10 tools. If your domain needs more coverage, split into more agents within the same project, or more projects within the vendor.
- Every mutating agent proposes the exact change and gates on human approval before acting — no exceptions, no "it's just a demo."
- No offensive, destructive, or evasive capability — this marketplace is for legitimate operational automation, not adversarial tooling.
- Gotchas in `SKILL.md` must be real, with enough detail (what broke, why, the fix) that someone hitting the same issue doesn't have to re-derive your fix from scratch.

## Review

A PR against this repo should let a reviewer answer "does this meet the format spec" without needing platform access to check — cite what you verified and how (which `GET` calls, what they returned) rather than asserting it worked.
