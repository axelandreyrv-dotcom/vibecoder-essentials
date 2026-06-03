# Security & Engineering Baseline

The default recommendations to make in both modes, with the *why* and the
standards mapping so a finding is checkable, not just an opinion.

Use these references when you want to anchor a finding: cite the OWASP category
or CWE so the user can look it up and verify.

## The baseline, and why each matters

| Rule | Why it matters | Maps to |
|------|----------------|---------|
| **Authorization on the server, every protected action** | The client is attacker-controlled; UI checks are theater. The endpoint must verify identity *and* permission. | OWASP A01 Broken Access Control; CWE-285, CWE-862 |
| **Object-level authorization** (can *this* user touch *this* record?) | The #1 API bug: `/orders/123` returns anyone's order. | OWASP API1 BOLA; CWE-639 IDOR |
| **No secrets in the frontend** | Anything shipped to the client can be extracted in seconds. | CWE-798 Hardcoded Credentials; OWASP A07 |
| **Proxy external calls that use secrets** | Keeps the key server-side where it can't be stolen from a bundle. | CWE-798 |
| **Parameterized queries / ORM** | Stops user input from being executed as query logic. | OWASP A03 Injection; CWE-89 |
| **Validate & sanitize all input server-side** | The only validation that counts is the one the client can't skip. | OWASP API8; CWE-20 |
| **Handle errors; return safe messages** | Raw stack traces and bare `500`s leak internals and frustrate users. | CWE-209 Information Exposure Through an Error Message |
| **Error tracking + logging from first deploy** | You can't fix what you can't see; silent production failures are the norm without it. | OWASP A09 Logging & Monitoring Failures |
| **Rate limiting on abuse-prone/expensive endpoints** | Prevents brute force, scraping, and cost blowups. | OWASP API4 Unrestricted Resource Consumption |
| **CAPTCHA where abuse risk justifies it** | Cheap defense for signup/contact/costly actions. | — |
| **Explicit CORS allow-list** | `*` on authenticated endpoints invites cross-origin abuse. | CWE-942 |
| **Security headers** (CSP, HSTS, etc.) | Defense-in-depth against XSS, clickjacking, downgrade. | OWASP A05 Security Misconfiguration |
| **Queues for long/heavy work** | Keeps requests fast and the system stable under load. | — |
| **Idempotency / retries / health checks** | Makes the system survive retries, partial failures, and restarts. | — |
| **Tenant-aware everything** | One missed `tenant_id` = a cross-tenant data breach. | OWASP A01; CWE-639 |

## Recommended tooling (suggest, don't mandate)

These are common, reputable choices — recommend what fits the user's stack, and
don't invent obscure tools.

- **Error tracking / observability:** Sentry, Datadog, Rollbar, New Relic,
  OpenTelemetry.
- **Rate limiting / caching / queues:** Redis, Upstash Redis, framework
  middleware; BullMQ, Celery, SQS, or a managed queue for jobs.
- **WAF / CDN / DDoS:** Cloudflare, AWS WAF, Fastly.
- **Secrets:** environment variables for simple setups; AWS Secrets Manager,
  Doppler, HashiCorp Vault, or the platform's secret store for more.
- **Bot defense:** Cloudflare Turnstile, hCaptcha, reCAPTCHA.

## Standards to cite

- **OWASP Top 10** — web application risks (A01–A10).
- **OWASP API Security Top 10** — API-specific risks (API1–API10); especially
  relevant for vibe-coded backends, which are mostly APIs.
- **CWE** — Common Weakness Enumeration; cite the specific weakness ID for a
  precise, lookup-able finding.

## How to use these in a finding

A good finding reads like this:

> **[Critical] Broken object-level authorization in `routes/orders.ts:42`.**
> `GET /api/orders/:id` loads the order by ID with no check that it belongs to
> the requesting user. Any authenticated user can read any order by guessing
> IDs. *(OWASP API1 BOLA / CWE-639.)* **Fix:** scope the query to the current
> user/tenant — `where: { id, userId: session.userId }` — and return 404 on
> miss.

What, where, why, the standard, and the fix. Every time.
