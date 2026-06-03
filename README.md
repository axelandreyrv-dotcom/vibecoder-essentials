# VibeCoder Essentials

> Plan, build, and review vibe-coded software like a senior engineer.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](CHANGELOG.md)
[![Agent Skill](https://img.shields.io/badge/type-Agent%20Skill-7c3aed.svg)](skills/vibecoder-essentials/SKILL.md)

**VibeCoder Essentials** is an [Agent Skill](https://docs.anthropic.com/en/docs/claude-code/skills)
that turns your AI coding assistant into a senior software engineer, security
auditor, and systems architect — so the apps you vibe-code don't just *look*
finished, they're actually ready for real users and real data.

It works at **both ends** of the build:

- **Planning Mode** — *before* you build. It asks the critical product, data,
  security, and infrastructure questions a senior engineer would, then hands you
  a concrete **Build Plan** with a readiness verdict.
- **Review Mode** — *after* you build. It audits your repo, code, architecture,
  screenshot, or live deployment for security, scalability, and
  production-readiness, then hands you a **Review** with a final verdict.

---

## Why this exists

AI-assisted development made it trivial to ship something that works on the happy
path. The hard parts — authorization, secrets, multi-tenancy, data modeling,
failure handling, observability, scaling — stay invisible until they fail in
production.

Most vibe-coded apps die from the same wounds:

- An **API key shipped in the frontend** bundle.
- **Authorization checked only in the UI** — the endpoint is wide open.
- **One tenant able to read another tenant's data.**
- A **`500` that leaks a stack trace** to users.
- **No logs** when it breaks at 2am.
- A **synchronous endpoint** that collapses the moment it gets traffic.

VibeCoder Essentials catches these *early* (a sentence to fix) instead of *late*
(an incident to clean up). Its rule number one: **never trust the frontend.**

---

## What it checks

Across both modes, it reasons about the things that actually make software
production-grade:

- **Security** — server-side authorization, secrets handling, exposed API keys,
  input validation, SQL injection, CORS, rate limiting, CAPTCHA, WAF/DDoS,
  security headers. Grounded in **OWASP Top 10**, **OWASP API Security Top 10**,
  and **CWE**.
- **Architecture** — stateless backends, decoupling, queues, caching,
  idempotency, retries, resilience, high availability, scalability.
- **Data** — schema design, normalization, indexes, migrations, N+1 queries,
  backups, soft deletes, audit logs.
- **Multi-tenancy** — tenant isolation, tenant-aware authorization, caching, and
  background jobs.
- **Operations** — error tracking, structured logging, alerts, health checks,
  CI/CD, environments, secret management.
- **Testing** — unit, integration, E2E, API, **authorization**, **tenant
  isolation**, webhooks, payments.
- **Platform** — Windows/macOS installability, code signing, auto-updates.

---

## Installation

The skill lives at [`skills/vibecoder-essentials/SKILL.md`](skills/vibecoder-essentials/SKILL.md).
Pick your tool below.

### Claude.ai (web / desktop)

Skills are available on paid plans via **Settings → Capabilities → Skills**.

1. Download or clone this repo.
2. Zip the skill folder so `SKILL.md` is at the **root** of the zip:
   ```bash
   cd skills/vibecoder-essentials
   zip -r vibecoder-essentials.zip SKILL.md references/
   ```
3. In Claude.ai, open **Settings → Capabilities → Skills** (Pro/Max/Team/Enterprise).
4. Click **Upload skill** and select `vibecoder-essentials.zip`.
5. Start a new chat and say *"I want to plan a new app"* or *"Review my repo for
   production-readiness."* Claude will load the skill automatically.

### Claude Code

Skills can live at the user level (all projects) or project level (one repo).

**User-level (recommended — available everywhere):**

```bash
# macOS / Linux
mkdir -p ~/.claude/skills
cp -r skills/vibecoder-essentials ~/.claude/skills/

# Windows (PowerShell)
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills"
Copy-Item -Recurse skills\vibecoder-essentials "$env:USERPROFILE\.claude\skills\"
```

**Project-level (only this repo):**

```bash
mkdir -p .claude/skills
cp -r skills/vibecoder-essentials .claude/skills/
```

Then in Claude Code, run `/doctor` (or just start a session) to confirm the skill
is detected. It triggers automatically on planning/review requests, or you can
ask for it by name.

### Codex

Codex reads agent instructions from `AGENTS.md`. This repo ships one at the root.

1. Copy the skill into your project (or point Codex at this repo):
   ```bash
   cp -r skills/vibecoder-essentials /path/to/your/project/skills/
   cp AGENTS.md /path/to/your/project/
   ```
2. `AGENTS.md` instructs Codex to apply VibeCoder Essentials in Planning or
   Review Mode and points it to `skills/vibecoder-essentials/SKILL.md` and the
   `references/` files.
3. Ask Codex to *plan* or *review* an app — it will follow the skill's modes and
   output templates.

### GitHub Copilot / VS Code

Copilot Chat respects repo-level custom instructions.

1. Copy the skill into your repo:
   ```bash
   cp -r skills/vibecoder-essentials .
   ```
2. Create `.github/copilot-instructions.md` with:
   ```markdown
   # Copilot Instructions

   When the user wants to plan, build, or review an app, follow the
   VibeCoder Essentials skill at `skills/vibecoder-essentials/SKILL.md`.
   Detect Planning Mode (building something new) vs. Review Mode (an existing
   app) and use the matching output template from
   `skills/vibecoder-essentials/references/output-templates.md`.
   Never trust the frontend; require server-side validation and authorization.
   ```
3. Reload VS Code. In Copilot Chat, ask to plan or review your app, and reference
   the files with `#file` if needed.

> **Note:** Native Agent Skills (auto-discovery) are a Claude feature. On Codex
> and Copilot, the same content is applied via `AGENTS.md` /
> `copilot-instructions.md`, which is why this repo ships both.

---

## Usage

Just talk to your assistant naturally. The skill detects the mode for you.

**Planning:**
> "I want to build a SaaS app where teams track their freelance invoices. Help me
> plan it before I start coding."

**Reviewing:**
> "Here's my repo. Is this safe to deploy to production?"

If it's ambiguous, the skill asks exactly one question: *"Are we planning this
app before building it, or reviewing an app that already exists?"*

### Try these prompts

**Planning Mode**
> "I'm about to vibe-code a habit-tracking web app with a friend. It'll have
> accounts, daily check-ins, and a streak leaderboard. Walk me through planning
> it like a senior engineer before we write any code."

**Review Mode**
> "I built a Next.js app that calls the OpenAI API from the client and stores
> notes in Supabase. Here's the repo — review it for security and
> production-readiness before I launch."

**SaaS multi-tenant review**
> "Review my multi-tenant B2B dashboard (Node + Postgres + Prisma). Multiple
> companies share one database with a `tenant_id` column. I'm worried about
> tenant isolation, authorization, and whether it'll scale. Audit it."

---

## Repository structure

```
vibecoder-essentials/
├── README.md                  # You are here
├── SECURITY.md                # Security policy & scope
├── CONTRIBUTING.md            # How to contribute
├── CHANGELOG.md               # Version history
├── AGENTS.md                  # Instructions for Codex & other agents
├── LICENSE                    # MIT
├── examples/
│   └── sample-review.md       # An example Review Mode output
└── skills/
    └── vibecoder-essentials/
        ├── SKILL.md           # The skill (modes, attitude, output contracts)
        └── references/
            ├── planning-mode.md      # Planning question bank
            ├── review-mode.md        # Review audit checklist
            ├── security-baseline.md  # OWASP / CWE baseline
            └── output-templates.md   # Build Plan & Review templates
```

---

## Compatibility

| Platform | Mechanism | Status |
|----------|-----------|--------|
| Claude.ai (Pro/Max/Team/Enterprise) | Native Agent Skill (upload) | ✅ |
| Claude Code | Native Agent Skill (`~/.claude/skills`) | ✅ |
| Codex | `AGENTS.md` + skill files | ✅ |
| GitHub Copilot / VS Code | `copilot-instructions.md` + skill files | ✅ |

---

## Contributing

Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Security policy
is in [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE) © VibeCoder Essentials contributors.
