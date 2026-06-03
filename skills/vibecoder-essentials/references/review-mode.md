# Review Mode — Audit Checklist

Use this to audit an existing app like a pre-production review. Open the real
files, cite real paths and lines, and ground findings in OWASP / CWE where they
apply (see [security-baseline.md](security-baseline.md)).

Work top to bottom, but spend your time where the risk is: the trust boundary
(server vs. client), the data layer, and anything that touches secrets or
another tenant's data.

## Table of contents

1. Security & auth
2. Secrets & key exposure
3. Backend API safety
4. CORS, rate limiting & abuse
5. Database & performance
6. Async, resilience & architecture
7. Multi-tenancy
8. Observability & testing
9. CI/CD & deployment readiness
10. Platform specifics (desktop)

---

## 1. Security & auth

- **Authentication:** how are users identified? Are sessions/tokens issued,
  validated, expired, and revoked correctly? Any auth bypass?
- **Authorization:** is every protected action checked on the **server**, for
  both *who you are* and *what you're allowed to do*? Look for the classic vibe-
  coding bug: the UI hides the button but the API endpoint is wide open.
  (OWASP A01: Broken Access Control; IDOR / CWE-639.)
- **Frontend exposure:** is any security decision made only in the client? Treat
  it as not made at all.
- **Password/credential handling:** hashing (bcrypt/argon2), no plaintext, no
  secrets in logs.

## 2. Secrets & key exposure

- Are any **API keys, tokens, or DB credentials** present in the frontend bundle,
  client-side env vars, or committed to the repo? (CWE-798 hardcoded creds.)
- Are secrets pulled from **environment variables or a secret manager**?
- Do external calls that require a secret go through a **backend proxy**, or does
  the client hold the key directly?
- Is `.env` gitignored? Is there a safe `.env.example` with no real values?

## 3. Backend API safety

- **Input validation:** is every input validated server-side for type, shape,
  size, and range? (OWASP API8; CWE-20.)
- **Injection:** are queries **parameterized / via ORM**, or is user input
  concatenated into SQL? (OWASP A03; CWE-89.) Check NoSQL, command, and template
  injection too.
- **Error handling:** does every route handle errors (try/catch or equivalent)?
  Are user-facing errors **safe and useful**, not raw stack traces or bare
  `500`s that leak internals? (CWE-209.)
- **Mass assignment / over-posting:** can a client set fields it shouldn't (role,
  tenant, price)? (OWASP API6; CWE-915.)

## 4. CORS, rate limiting & abuse

- **CORS:** explicit origin allow-list, or a wildcard `*` on authenticated
  endpoints? (CWE-942.)
- **Rate limiting:** present on abuse-prone and expensive endpoints? Backed by
  what (Redis / Upstash / middleware)? (OWASP API4: Unrestricted Resource
  Consumption.)
- **CAPTCHA / bot protection** where abuse risk justifies it (signup, contact,
  costly actions).
- **DDoS / WAF:** is the app behind a CDN/WAF if its exposure warrants it?
- **Security headers:** CSP, HSTS, X-Content-Type-Options, X-Frame-Options.

## 5. Database & performance

- **Normalization & schema:** is the schema sane, or is data duplicated/denormal-
  ized in ways that will cause anomalies?
- **Indexes:** are queries indexed for their access patterns? Any obvious full
  scans?
- **Migrations:** versioned, reversible, applied safely? Any destructive
  migration without a path back?
- **N+1 queries:** loops issuing one query per item instead of a join/batch?
- **Performance:** obvious hot paths doing too much work synchronously? Missing
  pagination on large lists?
- **Caching:** is anything cached that should be? Anything cached that will go
  stale or leak across users/tenants?

## 6. Async, resilience & architecture

- **Async processing / queues:** is long or heavy work offloaded, or does it
  block the request and risk timeouts?
- **Idempotency:** are retried operations (webhooks, payments, jobs) safe to run
  twice?
- **Resilience:** retries with backoff, timeouts, and circuit breaking on
  external calls? What happens when a dependency is down?
- **High availability:** single points of failure? Is the backend **stateless**
  so it can scale horizontally?
- **Decoupled architecture:** are components loosely coupled, or will one change
  ripple everywhere?

## 7. Multi-tenancy

(Only if the app is multi-tenant — confirm first.)

- **Tenant isolation:** can one tenant read or write another's data? Is the
  `tenant_id` (or equivalent) enforced on **every** query, not just most?
- **Tenant-aware authorization:** checks carry tenant context.
- **Tenant-aware caching:** cache keys include the tenant — no cross-tenant
  leakage.
- **Tenant-aware background jobs:** jobs are scoped to the right tenant.

## 8. Observability & testing

- **Error tracking:** Sentry / Rollbar / Datadog / etc. wired up, or is the app
  blind in production?
- **Logging:** structured, centralized, and free of secrets/PII?
- **Alerts & health checks:** is anyone notified when it breaks? Are there
  liveness/readiness checks?
- **Testing:** is there meaningful coverage of critical flows — unit,
  integration, E2E, API, **authorization**, **tenant isolation**, webhooks, and
  payments where relevant? Tests that prove a user *can't* do the forbidden thing
  are worth more than happy-path tests.

## 9. CI/CD & deployment readiness

- **CI/CD:** is there a pipeline that builds, tests, and deploys? Do tests gate
  deploys?
- **Environments:** dev / staging / production separated, with isolated data and
  secrets?
- **Secret management** per environment?
- **Migrations** run safely as part of deploy?
- **Rollback:** can you roll back a bad deploy quickly?

## 10. Platform specifics (desktop)

(Only if the app ships as a Windows/macOS desktop app.)

- **Installability:** clean install/uninstall on each target OS?
- **Code signing:** signed so the OS doesn't block or warn?
- **Auto-updates:** a safe, verified update channel?
- **Local data & secrets:** stored securely on the user's machine, not in
  plaintext where another process can read them?
