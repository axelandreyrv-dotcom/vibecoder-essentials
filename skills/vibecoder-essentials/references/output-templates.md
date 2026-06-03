# Output Templates

The two primary deliverables have fixed templates. Use them exactly so output is
predictable and skimmable. Fill every section; if a section is genuinely N/A,
say so in one line rather than deleting it (the reader should see you considered
it).

---

## Build Plan (Planning Mode)

Use this exact structure:

```markdown
# VibeCoder Essentials Build Plan: [App Name]

## Product Summary
One paragraph: what it is, who it's for, the problem it solves.

## MVP Scope
The smallest thing that delivers the core value. Bullet the in-scope features
and explicitly list what's deferred.

## Recommended Architecture
Frontend, backend, database, ORM, hosting, and how they fit together. Note
stateless backend, queues, caching, Redis/Upstash if applicable. Justify the
choices in one line each.

## Security-by-Design Requirements
Auth, server-side authorization, secrets handling, backend proxy for external
keys, input validation, CORS, rate limiting, CAPTCHA/WAF if warranted, security
headers. Tie each to why it matters here.

## SaaS / Multi-Tenant Design
Tenant model and how isolation, authorization, caching, and background jobs all
stay tenant-aware. (Write "N/A — single-tenant" if it doesn't apply.)

## Data Model Draft
Main entities and relationships. Note sensitive data, audit logs, soft deletes,
file uploads, search, backups/restore.

## Backend Rules
The non-negotiables for this app's backend: what must be validated and
authorized server-side, error handling, what the client must never be trusted
with.

## Async, Queues, and Background Jobs
What work goes off the request path, the queue/worker approach, idempotency,
retries, dead-letter handling.

## Observability Plan
Error tracking, logging, alerts, health checks — from the first deploy.

## Testing Plan
Which test types matter for this app (unit, integration, E2E, API,
authorization, tenant isolation, webhooks, payments) and why.

## Deployment Plan
Target platform, environments (dev/staging/prod), CI/CD, secret management,
migrations, rollback. Desktop specifics (code signing, auto-updates) if relevant.

## Remaining Questions
The open decisions still blocking a clean start, phrased so the user can answer
them quickly.

## Build Readiness Verdict
Exactly one of:
- Ready to start building
- Ready after answering remaining questions
- Not ready to build yet
- Architecture must be clarified first
- Security model must be clarified first
```

---

## Review (Review Mode)

Use this exact structure:

```markdown
# VibeCoder Essentials Review: [Project Name / Inferred Stack]

## Executive Summary
3–6 sentences a busy founder/lead can read: what this is, the headline risks,
and the bottom line.

## Production-Readiness Score
A score out of 100 with a one-line justification. Be honest, not generous.

## Overall Risk Rating
Critical / High / Medium / Low — for the system as a whole.

## What Looks Good
Genuine strengths, stated once. Don't pad.

## Critical Issues
Must-fix-before-production. Each: what, where (file:line), why it matters,
OWASP/CWE if applicable, and the fix. Lead with these.

## High-Priority Improvements
Important but not blocking-everything. Same format.

## Architecture & Infrastructure Review
Statelessness, decoupling, HA, queues, caching, idempotency, resilience,
scalability.

## Security Review
Auth, authorization, secrets/key exposure, frontend exposure, input validation,
injection, CORS, rate limiting, DDoS/WAF, headers. Cite OWASP/CWE.

## Platform Specifics
Desktop installability, code signing, auto-updates — or "N/A" if web-only.

## Database & Performance
Schema, normalization, indexes, migrations, N+1, pagination, caching, hot paths.

## Observability & Testing
Error tracking, logging, alerts, health checks; test coverage including
authorization and tenant-isolation tests.

## Recommended Action Plan
Ordered, concrete steps. Group by Now / Before launch / Soon. Each step is
something the user can actually do.

## Suggested Implementation Examples
Short, safe, illustrative snippets for the most important fixes (e.g. a
server-side authorization check, a parameterized query, a rate-limit
middleware). Keep them generic and non-malicious — no exploit code, no real
secrets.

## Final Verdict
Exactly one of:
- Safe to deploy
- Safe to deploy after minor fixes
- Not safe to deploy yet
- Needs architecture refactor before production
- Needs security hardening before production
```

---

## Notes on using both templates

- **Lead with what's critical.** Reader attention is most valuable at the top.
- **Every finding is checkable:** what, where, why, and the fix. Cite OWASP/CWE
  where it applies.
- **Don't hallucinate.** If a section depends on a file you couldn't see, say
  what's missing instead of inventing it.
- **One verdict, no hedging.** Pick the single verdict that's true and own it.
