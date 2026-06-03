# Example: Review Mode output

This is an illustrative example of what VibeCoder Essentials produces in **Review
Mode**. It reviews a fictional app to show the format, depth, and tone — it does
not describe any real project, and the snippets are defensive and generic.

> **Input the user gave:** "I built a Next.js app that lets people generate
> marketing copy with the OpenAI API. It calls OpenAI from a client component,
> stores generations in Supabase, and has email/password login. Here's the repo
> — is it safe to launch?"

---

# VibeCoder Essentials Review: CopyGen (Next.js + Supabase + OpenAI)

## Executive Summary

CopyGen works on the happy path and the UI is clean, but it is **not safe to
launch as-is**. The OpenAI API key is exposed to the browser, which means anyone
can extract it and run charges on your account. Authorization for reading and
deleting generations is enforced only by client-side routing, so any logged-in
user can read or delete another user's data by calling the API directly. There is
no rate limiting, no error tracking, and no input validation on the generation
endpoint. The data model and Supabase usage are reasonable; the problems are
concentrated in the trust boundary. Fixing the three critical issues below is a
day or two of work and is mandatory before any public launch.

## Production-Readiness Score

**38 / 100** — a working prototype with critical security gaps at the trust
boundary.

## Overall Risk Rating

**Critical**

## What Looks Good

- The Supabase schema is sensibly normalized; `generations` correctly references
  `users` with a foreign key.
- Passwords are handled by Supabase Auth, so you're not rolling your own hashing.
- The frontend is componentized and readable.

## Critical Issues

**[Critical] OpenAI API key exposed to the client — `app/generate/page.tsx:14`.**
The key is read from `NEXT_PUBLIC_OPENAI_KEY` and used in a client component, so
it ships in the browser bundle and is trivially extractable. Anyone can drain
your OpenAI quota and run up your bill. *(CWE-798 Hardcoded/Exposed Credentials.)*
**Fix:** move the OpenAI call to a server route (e.g. `app/api/generate/route.ts`)
that reads the key from a server-only env var (`OPENAI_API_KEY`, no
`NEXT_PUBLIC_` prefix). The client calls *your* endpoint; the key never leaves
the server.

**[Critical] Broken object-level authorization on generations —
`app/api/generations/[id]/route.ts:9`.** `GET` and `DELETE` load by `id` with no
check that the row belongs to the requester. Any authenticated user can read or
delete anyone's generation by changing the ID. *(OWASP API1 BOLA / CWE-639.)*
**Fix:** scope every query to the authenticated user and return 404 on a miss:

```ts
const { data, error } = await supabase
  .from("generations")
  .select("*")
  .eq("id", params.id)
  .eq("user_id", session.user.id)   // the boundary
  .single();
if (error || !data) return NextResponse.json({ error: "Not found" }, { status: 404 });
```

Also enable Supabase **Row Level Security** so the database enforces this even if
a route forgets to.

**[Critical] No input validation or error handling on the generate endpoint —
`app/api/generate/route.ts`.** The prompt is passed straight to OpenAI with no
size/shape checks, and an OpenAI failure throws an unhandled error that returns a
raw `500` with a stack trace. *(OWASP API8 / CWE-20; CWE-209.)* **Fix:** validate
the request body (e.g. with Zod), cap prompt length, wrap the call in try/catch,
and return a clean message:

```ts
const Body = z.object({ prompt: z.string().min(1).max(2000) });
const parsed = Body.safeParse(await req.json());
if (!parsed.success) return NextResponse.json({ error: "Invalid prompt" }, { status: 400 });
try {
  // ...call OpenAI...
} catch {
  return NextResponse.json({ error: "Generation failed, please try again." }, { status: 502 });
}
```

## High-Priority Improvements

- **No rate limiting on `/api/generate`.** A single user (or a script) can hammer
  the endpoint and run up unbounded OpenAI cost. *(OWASP API4.)* Add per-user
  rate limiting backed by Upstash Redis (e.g. `@upstash/ratelimit`), and consider
  a daily quota per account.
- **No error tracking.** When this breaks in production you'll have no idea why.
  Wire up Sentry (client + server) before launch.
- **No CAPTCHA on signup.** If signups translate to free OpenAI usage, expect
  abuse. Add Turnstile/hCaptcha on signup and on the generate endpoint for
  unauthenticated traffic.

## Architecture & Infrastructure Review

The backend is effectively stateless (good for scaling on Vercel). Generation is
synchronous, which is acceptable at low volume but will cause request timeouts as
prompts get longer or traffic grows — move generation to a queue/worker (or at
least stream the response) before scale. No caching is present; identical
requests re-hit OpenAI. There's no health check endpoint for uptime monitoring.

## Security Review

Covered in Critical/High above. Summary: the trust boundary is the weak point —
secrets on the client (CWE-798), broken object-level authz (OWASP API1/CWE-639),
missing input validation (CWE-20), and leaky errors (CWE-209). CORS is left at
framework defaults; tighten it to your own origins for the API routes. No
security headers (CSP/HSTS) are set — add them via `next.config.js` headers.

## Platform Specifics

N/A — web-only deployment on Vercel.

## Database & Performance

Schema and foreign keys are fine. **Enable Row Level Security** in Supabase — it's
currently off, which is what makes the BOLA issue exploitable end-to-end. Add an
index on `generations.user_id` for the per-user list query. No N+1 patterns
observed. Add pagination to the generations list before a heavy user accumulates
thousands of rows.

## Observability & Testing

No error tracking, no structured logging, no alerts, no health checks — the app
is blind in production. There are no tests at all; at minimum add **authorization
tests** proving user A cannot read or delete user B's generations, and an API
test for the validation on `/api/generate`.

## Recommended Action Plan

**Now (before any public traffic):**

1. Move the OpenAI call server-side; remove the `NEXT_PUBLIC_` key.
2. Add object-level authorization to all generation routes and enable Supabase
   RLS.
3. Add input validation and try/catch to `/api/generate`.

**Before launch:**

4. Add per-user rate limiting (Upstash Redis) and a daily quota.
5. Wire up Sentry for error tracking.
6. Add CAPTCHA on signup and generate.
7. Set security headers and tighten CORS.

**Soon:**

8. Move generation to a queue or stream responses; add caching.
9. Add authorization and API tests; add pagination and a health check.

## Suggested Implementation Examples

See the parameterized, authorization-scoped query and the validated/handled
endpoint shown in the Critical Issues above — those are the two patterns that
remove most of the risk.

## Final Verdict

**Needs security hardening before production.**
