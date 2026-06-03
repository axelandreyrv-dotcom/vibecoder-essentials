# Example: Planning Mode output

This is an illustrative example of what VibeCoder Essentials produces in
**Planning Mode**. It plans a fictional B2B SaaS app before any code is written,
to show the format, depth, and tone. It does not describe any real project.

> **Input the user gave:** "I want to build a SaaS app where small marketing
> agencies can manage their clients' social media content. Multiple agencies,
> each with their own team members and clients. I want to start coding this
> week — help me plan it before I write a single line."

---

# VibeCoder Essentials Build Plan: ContentDesk

## Product Summary

ContentDesk is a multi-tenant B2B SaaS platform for small marketing agencies.
Each agency (tenant) manages multiple client accounts, creates and schedules
social media posts, and collaborates across a team. The product solves the
problem of scattered tools and context-switching: one workspace per agency,
isolated from every other agency on the platform.

Target users: agency owners (admins), agency team members (editors/managers),
and optionally external clients with view-only access to their own content
calendar.

## MVP Scope

**In scope for v1:**

- Tenant onboarding (agency signup, team invite by email).
- Role system: `owner`, `editor`, `viewer` (client-facing).
- Client accounts — each belongs to exactly one tenant.
- Content posts — draft, scheduled, published states, attached to a client.
- Basic content calendar view (list + simple calendar).
- Manual publish action (no social API integration in v1).
- Email notifications for key events (invite, post approved).

**Explicitly deferred:**

- OAuth connections to social platforms (Twitter/X, Instagram, LinkedIn).
- Automated publishing via social APIs.
- Analytics and reporting.
- Billing and subscriptions.
- AI-assisted copy generation.
- White-label / custom domain per tenant.

## Recommended Architecture

| Layer | Choice | Rationale |
|---|---|---|
| Frontend | Next.js (App Router) | SSR + RSC; easy auth with Supabase; strong ecosystem. |
| Backend | Next.js API routes + Server Actions | Stateless; colocated with frontend for MVP speed. |
| Database | PostgreSQL (Supabase) | Row Level Security enables tenant isolation at DB layer. |
| ORM | Prisma | Type-safe queries; migration tooling; easy tenant scoping. |
| Auth | Supabase Auth | Email/password + magic link; JWT session; handles invites. |
| File storage | Supabase Storage | For post media attachments; scoped by bucket policy. |
| Email | Resend | Transactional email; simple API; generous free tier. |
| Queue | Inngest (or BullMQ + Redis) | Async jobs for email delivery, scheduled post processing. |
| Caching | None in v1 (add Redis/Upstash post-MVP) | Premature for MVP traffic; add when query cost shows up. |
| Hosting | Vercel (frontend + API) + Supabase (DB/Auth/Storage) | Zero-ops for MVP. |
| CI/CD | GitHub Actions | Lint, test, deploy on merge to `main`. |

**Stateless backend:** yes — all state in Postgres/Supabase; scales horizontally
on Vercel without config.

## Security-by-Design Requirements

- **Tenant isolation enforced at the database layer.** Every table that holds
  tenant data has a `tenant_id` column. Every Prisma query scopes to
  `where: { tenantId: session.tenantId }`. Enable Supabase **Row Level Security**
  as a second layer — if a query forgets the scope, RLS catches it.
- **Authorization on the server for every protected action.** The UI hiding a
  button is not authorization. Every API route and Server Action verifies both
  identity (`session.userId`) and permission (`userRole >= required`).
- **No secrets in the frontend.** Supabase anon key is the only client-visible
  key; it is restricted by RLS policies. Social API keys (post-MVP), Resend key,
  and any third-party secrets live in server-only env vars.
- **Input validation on every route.** Use Zod to parse request bodies
  server-side. Validate type, shape, length, and enum membership. Reject invalid
  input with `400` + a safe message, never pass raw user input to the database.
- **Error handling on every route.** Wrap all async logic in try/catch. Return
  safe, useful messages to the client — never raw stack traces or bare `500`s.
- **CORS:** Next.js API routes are same-origin by default; add an explicit
  allow-list if you expose a public API endpoint.
- **Security headers:** set `Content-Security-Policy`, `Strict-Transport-Security`,
  `X-Content-Type-Options`, and `X-Frame-Options` via `next.config.js` headers.
- **Rate limiting** on auth endpoints (login, invite, magic link) and any
  expensive action using Upstash Redis + `@upstash/ratelimit` as middleware
  (add post-MVP or at first sign of abuse risk).
- **File upload validation:** validate MIME type, file size (server-side), and
  scan for malicious content before storing in Supabase Storage.

## SaaS / Multi-Tenant Design

**Tenant model:** shared schema, single database. Every tenant-scoped table
carries `tenant_id UUID NOT NULL`. This is the right choice for a small-to-mid
scale MVP — straightforward to operate, low cost, and easy to migrate if you
later need schema-per-tenant isolation for compliance reasons.

**Tenant isolation checklist:**

| Concern | Implementation |
|---|---|
| Database queries | `where: { tenantId }` on every Prisma query. Never query without it. |
| RLS backup | Supabase RLS policy: `tenant_id = auth.jwt() ->> 'tenantId'`. |
| Authorization | Every check carries `tenantId` + `userId` + `role`. |
| File storage | Bucket path prefix: `/{tenantId}/{clientId}/{filename}`. Bucket policy enforces prefix ownership. |
| Background jobs | Jobs are enqueued with `tenantId` in the payload and re-validate it before acting. |
| Caching (post-MVP) | Cache keys include `tenantId` — never cache a response that crosses the tenant boundary. |

**Tenant onboarding flow:** agency signs up → a new `Tenant` row is created →
the user becomes the `owner` role → they invite team members by email → invited
users accept and are created as `editor` or `viewer` within that tenant.

## Data Model Draft

```text
Tenant
  id         UUID PK
  name       TEXT
  slug       TEXT UNIQUE   -- used in URLs
  createdAt  TIMESTAMP

User
  id         UUID PK       -- matches Supabase Auth user id
  tenantId   UUID FK → Tenant
  email      TEXT
  role       ENUM(owner, editor, viewer)
  createdAt  TIMESTAMP

Client
  id         UUID PK
  tenantId   UUID FK → Tenant
  name       TEXT
  timezone   TEXT
  createdAt  TIMESTAMP

Post
  id           UUID PK
  tenantId     UUID FK → Tenant
  clientId     UUID FK → Client
  authorId     UUID FK → User
  title        TEXT
  body         TEXT
  mediaUrls    TEXT[]
  status       ENUM(draft, scheduled, published, archived)
  scheduledAt  TIMESTAMP NULL
  publishedAt  TIMESTAMP NULL
  createdAt    TIMESTAMP
  updatedAt    TIMESTAMP
  deletedAt    TIMESTAMP NULL    -- soft delete
```

**Notes:**

- `tenantId` denormalized on `Post` for fast RLS enforcement (avoids join).
- Soft deletes on `Post` via `deletedAt` — recoverable, preserves audit trail.
- Add an `AuditLog` table post-MVP:
  `(id, tenantId, userId, action, entityType, entityId, createdAt)`.
- Indexes needed: `Post(tenantId, clientId)`,
  `Post(tenantId, status, scheduledAt)`, `User(tenantId, email)`.

## Backend Rules

- **Never trust the client for `tenantId` or `role`.** Read them from the
  server-side session only. The client is attacker-controlled.
- **Every API route and Server Action authenticates first**, then authorizes
  (checks role), then acts.
- **Ownership check pattern** for any action on a record (update, delete,
  publish a post):
  ```ts
  const post = await prisma.post.findUnique({
    where: { id: params.id, tenantId: session.tenantId }
  });
  if (!post) return notFound(); // 404, not 403 — don't leak existence
  ```
- **Zod on all inputs.** If the shape is wrong, return `400` before touching the
  database.
- **Return safe error messages.** `"Something went wrong, please try again"` is
  always better than a Prisma error message.

## Async, Queues, and Background Jobs

Work that leaves the request path (enqueue and return immediately; process in a
worker):

| Job | Trigger | Why async |
|---|---|---|
| Send invite email | User invited | Resend latency; retry on failure |
| Send notification email | Post status changes | Latency; retries |
| Process scheduled posts (post-MVP) | Cron every minute | Long-running; batch |
| Media processing (post-MVP) | File uploaded | Variable latency |

**Queue system:** [Inngest](https://www.inngest.com/) is the lowest-ops choice
for Vercel + Next.js (serverless-native, retries, replay, local dev support).
BullMQ + Upstash Redis is a valid alternative if you want Redis anyway.

**Idempotency:** all jobs must be safe to run twice. Use the invite token or
post ID as the idempotency key. Log on completion; skip if already done.

**Retries:** Inngest retries automatically with exponential backoff. Define a
`maxAttempts` (3–5) and a dead-letter handler that alerts on permanent failure.

## Observability Plan

Start on day one. You cannot fix what you cannot see.

- **Error tracking:** [Sentry](https://sentry.io/) — client + server. Configure
  `beforeSend` to scrub PII from error reports. Alert on new issues and on error
  rate spikes.
- **Logging:** Vercel's built-in log drain to a log aggregator (Axiom, Datadog,
  Logtail). Structure logs as JSON with `tenantId` (never PII) on every line.
- **Alerts:**
  - Error rate > X in 5 minutes → PagerDuty/Slack.
  - Job dead-letter queue receives an item → Slack.
  - Scheduled post processing falls behind → Slack.
- **Health check:** `GET /api/health` returns `200 { status: "ok", db: "ok" }`.
  Vercel checks it; uptime monitor (Better Uptime, UptimeRobot) alerts if it
  goes down.

## Testing Plan

| Type | Scope | Tool |
|---|---|---|
| Unit | Core business logic (role checks, slug generation, status transitions) | Vitest |
| Integration | API routes + DB (Prisma + test database) | Vitest + test DB |
| Authorization | Prove a user cannot perform actions outside their role/tenant | Vitest (integration) |
| Tenant isolation | Prove tenant A cannot read/write tenant B's data | Vitest (integration) |
| E2E | Critical user flows (sign up, invite, create post, publish) | Playwright |
| Email | Confirm correct template sent on invite/notification | Inngest test mode |

**Priority:** authorization and tenant isolation tests are the most critical.
A test that proves tenant A's editor cannot read tenant B's posts is worth more
than a dozen happy-path unit tests.

## Deployment Plan

**Target:** Vercel (app) + Supabase (DB/Auth/Storage).

**Environments:**

| Env | Branch | Data | Secrets |
|---|---|---|---|
| Development | local | Local Supabase instance | `.env.local` (gitignored) |
| Staging | `staging` | Staging Supabase project | Vercel env vars (staging) |
| Production | `main` | Production Supabase project | Vercel env vars (production) |

**CI/CD (GitHub Actions):**

1. On PR → lint + type-check + unit/integration tests.
2. On merge to `staging` → run tests → deploy to Vercel staging.
3. On merge to `main` → run tests → run migrations → deploy to Vercel production.

**Migrations:** Prisma Migrate. Migrations are version-controlled, reviewed in
PR, and run as a deploy step (`prisma migrate deploy`) — never run manually in
production.

**Rollback:** Vercel instant rollback to the previous deployment. DB rollback
requires a reversible migration — write every migration with a `down` path.

**Secret management:** server-only env vars in Vercel per-environment. Never
commit `.env.local` or any real secret. Keep `.env.example` with placeholder
values committed.

## Remaining Questions

1. **Billing:** is this in scope for v1 or deferred? If v1, Stripe integration
   adds significant auth surface area (webhooks, idempotency, test/live keys).
2. **Client portal access:** can external clients log in and see their content
   calendar? If yes, `viewer` role scoped to a single `clientId` needs a
   separate auth flow (magic link preferred over password for clients).
3. **Post approval workflow:** is there a review/approval step before publishing,
   or can any `editor` publish directly?
4. **Data residency / compliance:** any requirement for GDPR, SOC 2, or data
   stored in a specific region? Affects Supabase project region selection.
5. **Team size per tenant:** expected maximum? Affects whether per-seat pricing
   and seat limits need to be built into the data model from day one.

## Build Readiness Verdict

**Ready after answering remaining questions.**

The architecture, security model, and data model are well-defined and ready to
build. Questions 1 (billing scope) and 2 (client portal) affect the data model
and auth flow in ways that are much harder to retrofit than to design upfront.
Answer those two, and you can start building immediately.
