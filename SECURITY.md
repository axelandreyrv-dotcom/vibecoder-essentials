# Security Policy

VibeCoder Essentials is a **documentation-only Agent Skill**: it contains
Markdown instructions that guide an AI assistant to plan and review software. It
ships **no executable code, no scripts, no binaries, and no dependencies**. That
keeps its own attack surface essentially zero — but we still take the security of
this project and its users seriously.

## What this skill does and doesn't do

- ✅ It **advises** on secure design and reviews code for security issues.
- ✅ It cites **OWASP Top 10**, **OWASP API Security Top 10**, and **CWE** to make
  findings checkable.
- ❌ It does **not** generate exploit code, malware, or attack tooling.
- ❌ It does **not** include real secrets, credentials, or API keys.
- ❌ Example snippets are **illustrative and defensive only** — e.g. how to add a
  server-side authorization check — never how to attack a system.

If a contribution attempts to add exploit code, malware, real credentials, or
content designed to facilitate unauthorized access, it will be rejected. See
[CONTRIBUTING.md](CONTRIBUTING.md).

## Reporting a vulnerability

If you find a security issue **in this repository** (for example, a malicious or
misleading instruction in a skill file, or a supply-chain concern in the docs):

1. **Do not** open a public issue for anything sensitive.
2. Email the maintainers or use **GitHub's private vulnerability reporting**
   (Security → Report a vulnerability) on the repo.
3. Include: what you found, where (file and line), and why it's a concern.

We aim to acknowledge reports within **5 business days** and to address valid
issues promptly.

## Scope

**In scope**

- Misleading, unsafe, or malicious guidance in any skill or reference file.
- Instructions that could cause an assistant to leak secrets or produce harmful
  output.
- Inaccurate security advice that could lead users to ship vulnerable software.

**Out of scope**

- Vulnerabilities in *your own* application that the skill reviewed. The skill is
  an aid, not a guarantee — a clean review does not certify your app as secure.
- The behavior of third-party AI platforms (Claude, Codex, Copilot) themselves.

## A note on the advice itself

This skill helps you find problems; it does not absolve you of responsibility for
your software. Treat its output as a knowledgeable second opinion, not a
compliance certificate. Always validate critical security controls with real
testing and, for high-stakes systems, a professional audit.
