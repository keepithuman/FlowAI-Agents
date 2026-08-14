# Security Policy

## Supported Versions

This repository is a content marketplace (vendor operational skills — procedures and API references), not a running service — there isn't a version matrix of "supported releases" in the traditional sense. The `main` branch is always the supported, current state.

## Reporting a Vulnerability

If you discover a security issue related to this project — for example, a credential or live token accidentally committed, an operation described in a vendor's `SKILL.md` that's actually destructive/offensive/evasive, or a documented pattern in `AGENT-FORMAT-SPEC.md` that encourages an insecure practice:

1. **Do not** open a public GitHub issue for it.
2. Report it privately via [GitHub Security Advisories](https://github.com/keepithuman/network-infrastructure-agents/security/advisories/new).
3. Include in your report:
   - Description of the issue
   - Which file(s)/vendor package are affected
   - Impact assessment
   - Suggested fix (if any)

We will acknowledge your report within a reasonable timeframe and follow coordinated disclosure — no public disclosure before a fix (or an agreed mitigation) is in place.

## What "security" means for a marketplace of pure skill content

This repo doesn't run code against production systems itself, and it doesn't ship agent definitions — it ships domain knowledge that something else (whatever agent framework a consumer uses) acts on. The bar is still real:

- **No credentials, tokens, or live secrets in any committed file.** Nothing in a `SKILL.md` or `README.md` should ever contain a real API key, password, or bearer token — examples should use obvious placeholders.
- **Every procedure that mutates a real system's state says so, and describes proposing the exact change before acting on it.** A submitted vendor package that describes a write operation without that framing will be treated as a security-relevant defect, not a style nitpick.
- **No offensive, destructive, or evasive capability.** This marketplace is for legitimate operational automation. Anything resembling a DoS primitive, credential-harvesting tool, or detection-evasion technique will be rejected outright, not just flagged.
