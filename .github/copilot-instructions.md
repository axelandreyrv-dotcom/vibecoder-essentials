# Copilot Instructions

When the user wants to plan, design, build, launch, or start a new app or
feature, follow the VibeCoder Essentials skill in **Planning Mode**:
`skills/vibecoder-essentials/SKILL.md`

When the user shares an existing repo, codebase, files, architecture, screenshot,
or deployment and wants a review, audit, or production-readiness check, follow
the VibeCoder Essentials skill in **Review Mode**:
`skills/vibecoder-essentials/SKILL.md`

If it is not clear which mode applies, ask exactly one question:
"Are we planning this app before building it, or reviewing an app that already
exists?"

## Output templates

Use the exact templates from:
`skills/vibecoder-essentials/references/output-templates.md`

- Planning Mode → deliver a `# VibeCoder Essentials Build Plan: [App Name]`
  ending in a **Build Readiness Verdict**.
- Review Mode → deliver a `# VibeCoder Essentials Review: [Project Name]`
  ending in a **Final Verdict**.

## Non-negotiables

- **Never trust the frontend.** Require server-side validation and authorization
  for every protected action. The UI hiding a button is not authorization.
- **No secrets in the frontend.** Flag any API key, token, or credential in
  client-side code; recommend server-only env vars and a backend proxy.
- **Do not generate app code** in Planning Mode while critical architecture,
  security, or data decisions are still open.
- **Calibrate severity honestly.** A Critical is a Critical; don't soften it.
- **Do not hallucinate.** If you cannot see the relevant file (routes, schema,
  env config, CI/CD), say so instead of inventing a finding.

## Additional reference files

- `skills/vibecoder-essentials/references/planning-mode.md` — question bank for
  Planning Mode (product, data, auth, multi-tenancy, infra, testing).
- `skills/vibecoder-essentials/references/review-mode.md` — audit checklist for
  Review Mode (security, database, async, observability, CI/CD).
- `skills/vibecoder-essentials/references/security-baseline.md` — OWASP Top 10,
  OWASP API Security Top 10, and CWE mappings for grounding findings.

Reference these files with `#file` in Copilot Chat when working through a
planning session or review.
