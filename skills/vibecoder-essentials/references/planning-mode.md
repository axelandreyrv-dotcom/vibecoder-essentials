# Planning Mode — Question Bank

Use this before building. Ask the questions that matter for *this* app; skip what
is obviously not applicable. Lead with decisions that change the architecture if
answered differently. Ask in small batches, make reasonable senior-engineer
assumptions for the rest, and state those assumptions so the user can correct
them.

The goal is not to fill out a form. The goal is to surface the load-bearing
decisions *before* code locks them in.

## Table of contents

1. Product & scope
2. Users, roles & permissions
3. Data model
4. SaaS & multi-tenancy
5. Auth & secrets
6. External APIs & abuse protection
7. Stack & architecture
8. Async, queues & background jobs
9. Deployment & platform
10. Observability
11. Testing

---

## 1. Product & scope

- What exactly are we building, in one sentence?
- Who will use it, and what problem does it solve for them?
- What is the **MVP scope** — the smallest thing that delivers the core value?
- What features are explicitly *out* of the MVP and can wait?
- What does "done" look like for v1?

## 2. Users, roles & permissions

- Who are the users? Are there multiple roles (admin, member, viewer, billing)?
- What can each role do, and — more importantly — what must each role *not* do?
- **Which actions must be protected server-side?** (List the sensitive ones:
  delete, export, change billing, impersonate, change another user's data.)
- Is there any data one user must never see belonging to another user?

## 3. Data model

- What are the main entities, and how do they relate?
- What data is **sensitive** (PII, payment data, health, credentials, tokens)?
  How will it be stored and protected?
- Do you need **audit logs** (who did what, when)?
- Do you need **soft deletes** (mark-deleted vs. hard-delete) for recoverability
  and audit?
- Are there **file uploads**? Where do files go (object storage vs. DB), and how
  are they validated, scanned, and access-controlled?
- Is there **search**? Over what, and at what scale?
- What is the **backup and restore** plan? Have you tested a restore, not just a
  backup?

## 4. SaaS & multi-tenancy

(Skip entirely if this is a single-tenant or personal app — but confirm that
first.)

- Is this **multi-tenant** (multiple organizations/customers in one system)?
- **Tenant model:** shared schema with a `tenant_id`, schema-per-tenant, or
  database-per-tenant? What are the trade-offs for your scale and isolation
  needs?
- **Tenant isolation:** how do you guarantee one tenant can never read or write
  another tenant's data? Where is that boundary enforced?
- **Tenant-aware authorization:** every authorization check carries the tenant
  context, not just the user.
- **Tenant-aware caching:** cache keys include the tenant so data never leaks
  across tenants via the cache.
- **Tenant-aware background jobs:** jobs run scoped to the correct tenant and
  can't cross the boundary.

## 5. Auth & secrets

- How do users **authenticate** (email/password, OAuth, magic link, SSO)?
- Do you need **MFA**? (Strongly recommended for admin and sensitive accounts.)
- Where do **secrets** live? (Environment variables or a secret manager — never
  in the repo, never in the client bundle.)
- How are sessions/tokens issued, stored, expired, and revoked?

## 6. External APIs & abuse protection

- Which **external APIs** do you call (payments, email, AI, maps, etc.)?
- Do those calls use secret keys? If so, they must go through a **backend
  proxy** — the key never reaches the client.
- **Rate limiting:** which endpoints are abuse-prone or expensive? What limits,
  and backed by what (Redis / Upstash Redis / framework middleware)?
- **CAPTCHA:** does any endpoint (signup, contact, expensive action) warrant it?
- **WAF / CDN / DDoS protection:** is the app exposed enough to need it
  (Cloudflare, AWS WAF, etc.)?
- **Security headers** (CSP, HSTS, X-Content-Type-Options, etc.) — planned?
- **CORS:** which origins are actually allowed? (Explicit allow-list, not `*`.)

## 7. Stack & architecture

- **Frontend framework?** **Backend framework?** **Database?** **ORM?**
- Is the backend **stateless** (so it can scale horizontally)? If not, why not?
- Do you need **queues** for async work?
- Do you need **Redis / Upstash Redis** (caching, rate limiting, sessions,
  queues)?
- What is the **caching** strategy, if any?

## 8. Async, queues & background jobs

- What work is long-running or heavy (emails, exports, image/video processing,
  third-party calls, AI generation)? That work belongs **off the request path**.
- What queue/worker system (e.g. BullMQ, Celery, SQS, a managed queue)?
- Are jobs **idempotent** (safe to run twice)? Do they **retry** on failure with
  backoff? What happens to a job that fails permanently (dead-letter)?

## 9. Deployment & platform

- **Deployment target?** (Vercel, AWS, Fly, Render, a VPS, on-prem…)
- Is this also a **Windows/macOS desktop app**? If so:
  - **Code signing** (so the OS doesn't flag it)?
  - **Auto-updates** (a safe update channel)?
  - Installer/packaging for each OS?
- **CI/CD** pipeline — build, test, deploy?
- **Environments:** development, staging, production — separated, with separate
  data and secrets?
- **Secret management** per environment?
- **Database migrations** — versioned, reversible, run automatically and safely?

## 10. Observability

- **Error tracking** from day one (Sentry, Rollbar, Datadog, etc.)?
- **Logs** — structured, centralized, searchable?
- **Alerts** — who gets paged when something breaks, and on what signal?
- **Health checks** for the app and its dependencies?
- **Retries** and **idempotency** for unreliable downstream calls?

## 11. Testing

Which of these do you need, and at what depth?

- **Unit** tests for core logic
- **Integration** tests across components
- **End-to-end (E2E)** tests for critical user flows
- **API** tests for contract and validation
- **Authorization** tests — proving a user *cannot* do what they shouldn't
- **Tenant isolation** tests — proving one tenant can't reach another's data
- **Webhook** tests (signature verification, replay, idempotency)
- **Payment** tests, if money is involved (success, failure, refund, webhook)
