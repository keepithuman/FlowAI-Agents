# Security Policy

## Supported Versions

This repository is a content marketplace (documentation + exported agent definitions), not a running service — there isn't a version matrix of "supported releases" in the traditional sense. The `main` branch is always the supported, current state.

## Reporting a Vulnerability

If you discover a security issue related to this project — for example, a published `.project.json` bundle that would grant an imported agent overly broad or unintended access, a credential or live token accidentally committed, or a documented pattern in `AGENT-FORMAT-SPEC.md` that encourages an insecure practice:

1. **Do not** open a public GitHub issue for it.
2. Report it privately via [GitHub Security Advisories](https://github.com/keepithuman/FlowAI-Agents/security/advisories/new).
3. Include in your report:
   - Description of the issue
   - Which file(s)/vendor package are affected
   - Impact assessment
   - Suggested fix (if any)

We will acknowledge your report within a reasonable timeframe and follow coordinated disclosure — no public disclosure before a fix (or an agreed mitigation) is in place.

## What "security" means for a marketplace of agent definitions

This repo doesn't run code against production systems itself, but every `.project.json` here is *meant* to be imported and run against a real platform, so the bar is still real:

- **No credentials, tokens, or live secrets in any committed file.** Project exports should only ever contain `provider` references by profile/model UUID (environment-specific, not a secret) — never an API key, password, or bearer token. If you find one, report it as a security issue, not a normal bug.
- **No agent ships with a tool list broader than its stated purpose.** An agent with unrelated write-capable tools attached is itself a security smell — see `docs/AGENT-FORMAT-SPEC.md`'s modularity rule.
- **Every mutating agent gates on human approval before acting.** A submitted vendor package that skips this for any write-capable agent will be treated as a security-relevant defect, not a style nitpick.
- **No offensive, destructive, or evasive capability.** This marketplace is for legitimate operational automation. Anything resembling a DoS primitive, credential-harvesting tool, or detection-evasion technique will be rejected outright, not just flagged.
