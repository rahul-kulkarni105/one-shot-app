# One-Shot App

> Enter one prompt. Get a fully deployed, production-grade web application.

An AI-orchestrated pipeline that takes a single natural language description of your idea and produces a live, running web application — complete with authentication, database, CI/CD, security hardening, monitoring, and automated maintenance.

**The only things you do:** describe your idea, register OAuth apps (~2 min), and watch the pipeline run.

---

## How It Works

```
You type this:

  IDEA:  A Fantasy Premier League companion app
  GOAL:  Users can view their FPL team, get AI transfer suggestions,
         and track points vs friend groups
  EXTRA: Real-time data, needs FPL public API integration

The pipeline produces this:

  ✓ Live URL:       https://your-app.vercel.app
  ✓ GitHub repo:    private, with branch protection
  ✓ CI/CD:          GitHub Actions (lint → typecheck → tests → deploy)
  ✓ Database:       Neon PostgreSQL, migrated, isolated per PR
  ✓ Auth:           magic links + OAuth, rate-limited
  ✓ Monitoring:     Sentry errors + PostHog analytics + Lighthouse CI
  ✓ Tests:          Vitest unit tests + Playwright E2E smoke tests
  ✓ Docs:           README, ARCHITECTURE.md, SECURITY.md
  ✓ Release:        git tag v1.0.0
```

---

## The 10-Phase Pipeline

The agent executes these phases in strict order. Each phase has a defined quality gate — it cannot advance until the gate passes. All errors are self-corrected before moving on.

| Phase | What Happens | Gate |
|---|---|---|
| **0** · Analysis | Parse the idea. Classify app type. Identify all features, APIs, and data sources. Write `ARCHITECTURE.md`. | ADR written; stack finalised |
| **1** · Scaffolding | `create-next-app` + full dependency install + config files + git repo created + initial commit pushed | All packages installed; project compiles |
| **2** · Code Generation | DB schema → auth → middleware → server actions → UI components → pages → unit tests → E2E tests | Full working application |
| **3** · Bug Review | Re-read every file. Fix TypeScript errors, null safety issues, unhandled edge cases, missing auth guards. | `tsc --noEmit` passes; all checklist items resolved |
| **4** · Security Review | Full OWASP Top 10 audit. Fix access control, injection, misconfigs, SSRF, CSP headers. | All OWASP items addressed |
| **5** · Performance Review | Core Web Vitals audit. Eliminate N+1 queries, add DB indexes, check bundle size, add cursor pagination. | LCP < 2.5s; no N+1s; no chunk > 200KB |
| **6** · Branch Protection | Protect `main`, create `develop`, add all available secrets to GitHub Actions. | Branch protection active |
| **7** · CI/CD Workflows | Write `ci.yml`, `deploy-preview.yml`, `deploy-production.yml`, `security-scan.yml`, `tech-debt.yml`, `dependabot.yml`. Commit to `develop`. | Workflow files committed |
| **8** · Deployment | Provision Neon DB → run initial migration → create Vercel project → set all env vars → first deploy → OAuth registration → redeploy | Live URL confirmed; auth working |
| **9** · Verification | Playwright smoke tests against production. Lighthouse CI (≥ 90 all scores). Security headers audit. | All checks pass |
| **10** · Handoff | Write `README.md`, `ARCHITECTURE.md`, `SECURITY.md`, three code-review reports. Tag `v1.0.0`. | Documentation committed; tag pushed |

### The Two Unavoidable Manual Steps

The pipeline is fully autonomous except for these two:

1. **OAuth app registration (Phase 8)** — GitHub and Google OAuth apps must be registered through their browser-based developer consoles. The agent outputs the exact callback URLs. You register the apps and paste back the client ID and secret. Takes ~2 minutes per provider.

2. **Paid API key (Phase 0, conditional)** — Only triggered if your idea requires an external data source with no free public alternative. The agent asks once. If you decline, it uses mock data and documents the limitation.

---

## Tech Stack

Every generated app uses this exact stack. None of these are configurable — they were chosen for best-in-class DX, free tiers that never expire, and seamless integration.

### Core

| Concern | Tool | Why |
|---|---|---|
| Framework | Next.js 15 (App Router) | Best full-stack React framework; server actions, streaming, edge support |
| Language | TypeScript (strict mode) | Type safety throughout; fewer runtime surprises |
| Styling | Tailwind CSS v4 | Speed, consistency, tiny bundle |
| UI Components | shadcn/ui | Accessible, copy-owned, no vendor lock-in |

### Data & Auth

| Concern | Tool | Free Tier | Why |
|---|---|---|---|
| Database | Neon (serverless PostgreSQL) | 0.5 GB, never pauses | Best free PostgreSQL — Supabase pauses after 1 week; Neon never does |
| ORM | Drizzle ORM | OSS | Type-safe, thin, excellent Neon integration |
| Auth | Auth.js v5 + `@auth/drizzle-adapter` | Free forever | OAuth + magic links; free forever vs Clerk's 10k MAU limit |
| Email | Resend | 3,000/mo, 100/day | Required for magic link auth; best free tier for Next.js |
| Rate limiting | Upstash Redis + `@upstash/ratelimit` | 10,000 cmds/day | Only edge-compatible free-tier rate limiter for Vercel |
| Validation | Zod | OSS | Runtime + compile-time safety on all inputs |

### Infrastructure & Monitoring

| Concern | Tool | Free Tier | Why |
|---|---|---|---|
| Git | GitHub | Free private repos | MCP support, Actions, full ecosystem |
| Deployment | Vercel (Hobby) | Unlimited projects, 100 GB/mo | Best DX for Next.js; edge functions; preview URLs |
| Error tracking | Sentry | 5,000 events/mo | Production error visibility with source maps |
| Analytics | PostHog | 1M events/mo | Client + server-side; open source; generous free tier |
| Performance | Vercel Analytics + Speed Insights | Included | Core Web Vitals in production |

### Testing & Quality

| Tool | Purpose |
|---|---|
| Vitest | Unit tests for all server actions and utilities |
| Playwright | E2E smoke tests — run against Neon branch DBs in CI, production in CD |
| ESLint + `eslint-plugin-security` | Lint + OWASP-aware static analysis |
| Prettier | Code formatting |
| `@next/bundle-analyzer` | Bundle size auditing |
| `@lhci/cli` | Lighthouse CI — blocks deploys if any score drops below 90 |

---

## Prerequisites

Do this once. It works for every app you create with this pipeline.

### Step 1 — Get Service Tokens (~20 minutes)

You need accounts and API tokens for the following services. All have free tiers.

**GitHub PAT**
- Go to: `github.com → Settings → Developer settings → Personal access tokens → Fine-grained`
- Required scopes: `Contents: Write`, `Metadata: Read`, `Pull requests: Write`, `Administration: Write`, `Workflows: Write`, `Secrets: Write`

**Vercel**
```bash
npm install -g vercel
vercel login
```
- Token: `vercel.com → Account Settings → Tokens → Create` (full account scope)

**Neon**
- Sign up at [neon.tech](https://neon.tech)
- `console.neon.tech → Settings → API Keys → Create`
- Note: the free tier allows **one project**. Each pipeline run creates a new project.

**Resend** (magic link email)
- Sign up at [resend.com](https://resend.com)
- `API Keys → Create key`
- Add and verify a sending domain (or use sandbox for testing)

**Upstash Redis** (rate limiting)
- Sign up at [upstash.com](https://upstash.com)
- `Create database → Copy REST URL and REST Token`

**Sentry** (error tracking)
- Sign up at [sentry.io](https://sentry.io)
- Create a project → select "Next.js" → copy the DSN
- `Settings → Auth Tokens → Create` with `project:write` scope → copy as `SENTRY_AUTH_TOKEN`

**PostHog** (analytics)
- Sign up at [posthog.com](https://posthog.com)
- Create a project → copy the Project API Key

---

### Step 2 — Configure MCP Servers in VS Code

Open VS Code and run `Cmd+Shift+P → MCP: Open User Configuration`, then paste and fill in:

```json
{
  "servers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-github-pat>"
      }
    },
    "vercel": {
      "command": "npx",
      "args": ["-y", "@vercel/mcp-server"],
      "env": {
        "VERCEL_TOKEN": "<your-vercel-token>"
      }
    },
    "neon": {
      "command": "npx",
      "args": ["-y", "@neondatabase/mcp-server-neon"],
      "env": {
        "NEON_API_KEY": "<your-neon-api-key>"
      }
    }
  }
}
```

Restart VS Code after saving.

### Step 3 — Verify Everything Works

In Copilot agent mode (or Claude Code / Codex), confirm your MCPs respond:

```
"List my GitHub repositories"   → returns your repos     ✓
"List my Vercel projects"        → returns your projects   ✓
"List my Neon projects"          → returns your databases  ✓
```

Setup is permanently complete. These credentials work for every future pipeline run.

---

## Running the Pipeline

### With GitHub Copilot

1. Open this folder in VS Code
2. Open Copilot Chat → switch to **Agent mode**
3. Type `#create-app.prompt.md` and press Enter
4. The agent asks for your idea in the IDEA / GOAL / EXTRA format
5. Paste your prompt → pipeline executes autonomously

### With Claude Code

1. Open this folder in VS Code with the Claude Code extension
2. Start a new chat
3. Claude reads `CLAUDE.md` automatically and greets you
4. Provide your prompt in the IDEA / GOAL / EXTRA format
5. Pipeline executes autonomously

### With Codex

1. Open this folder in VS Code with the Codex extension
2. Start a new chat
3. Codex reads `AGENTS.md` automatically and greets you
4. Provide your prompt in the IDEA / GOAL / EXTRA format
5. Pipeline executes autonomously

### Prompt Format

All three agents accept the same input format:

```
IDEA:    [What the app is — describe the domain and core purpose]
GOAL:    [What a user should be able to DO with it]
EXTRA:   [Optional: specific integrations, constraints, known requirements]
```

**Examples:**

```
IDEA:    A Fantasy Premier League companion app
GOAL:    Users can view their FPL team, get AI transfer suggestions,
         and track points vs friend groups
EXTRA:   Real-time data, needs FPL public API integration
```

```
IDEA:    An options screener for US stocks
GOAL:    Users filter options chains by IV rank, delta, DTE, and premium;
         get alerts when criteria are met; save watchlists
EXTRA:   Low latency required, mobile-friendly
```

```
IDEA:    A personal finance tracker
GOAL:    Users log income and expenses, visualise spending by category,
         and set monthly budgets with progress indicators
EXTRA:   No external API needed, private data only
```

---

## CI/CD Architecture

The pipeline configures five GitHub Actions workflows and Dependabot.

### Pull Request Flow

```
push to develop or feature branch
  └─ PR opened
       ├─ ci.yml runs:
       │    ├─ lint          (ESLint — zero errors)
       │    ├─ typecheck     (tsc --noEmit — zero errors)
       │    ├─ test-unit     (Vitest)
       │    ├─ test-e2e      (Neon branch created → migrations run →
       │    │                 next build + start → Playwright → Neon branch deleted)
       │    └─ audit         (npm audit --audit-level=high)
       │
       ├─ deploy-preview.yml runs:
       │    └─ Vercel preview URL deployed + commented on PR
       │
       └─ All checks pass → auto-merge to main
            └─ deploy-production.yml runs:
                 ├─ Full CI suite (lint + typecheck + unit + audit)
                 ├─ drizzle-kit migrate against production DB  ← runs BEFORE deploy
                 ├─ vercel --prod
                 ├─ Playwright smoke tests against production URL
                 ├─ Lighthouse CI (fail if any score < 90)
                 └─ On failure: create GitHub issue (no rollback — fix forward)
```

> **Why no rollback:** The DB migration already ran. Rolling back code against a migrated schema causes its own cascade of failures. The correct response is always to push a fix.

### Automated Maintenance

These workflows run permanently after initial deployment, with no manual action required.

| Cadence | Workflow | What It Does |
|---|---|---|
| Every deploy | `deploy-production.yml` | Lighthouse CI — blocks deploy if any score < 90 |
| Every Monday | `security-scan.yml` | `npm audit` → auto-PR with fixes if possible; Semgrep SAST (OWASP ruleset) → GitHub issue if findings |
| 1st of every month | `tech-debt.yml` | `npm outdated` + ESLint complexity check + Vitest coverage report → GitHub issue labelled `tech-debt` |
| Weekly | Dependabot | Dependency update PRs for npm packages and GitHub Actions |

Auto-merge is enabled on `main`. Dependency update PRs merge automatically when CI passes.

### Per-PR Database Isolation

Every pull request gets its own Neon database branch — an instant, copy-on-write snapshot of the production schema. E2E tests run against this isolated branch. The branch is deleted after the CI job completes, regardless of whether tests pass or fail.

This means:
- E2E tests never touch production data
- Every PR has a clean, schema-accurate database to test against
- Failed test runs don't leave orphaned branches that count against storage limits

---

## Project Structure (Generated App)

The scaffolded app follows this canonical structure:

```
<app-name>/
├── src/
│   ├── middleware.ts            ← Auth guard + Upstash rate limiting
│   ├── app/
│   │   ├── (auth)/              ← Sign-in / sign-up pages
│   │   ├── (protected)/         ← Auth-guarded routes
│   │   ├── (public)/            ← Public routes
│   │   └── api/auth/[...nextauth]/route.ts
│   ├── components/
│   │   ├── ui/                  ← shadcn/ui components
│   │   ├── features/            ← Feature-specific components
│   │   └── layouts/             ← Shared layout components
│   ├── db/
│   │   ├── schema/              ← Drizzle table definitions
│   │   ├── migrations/          ← Generated SQL migrations
│   │   └── index.ts             ← Neon serverless DB client
│   ├── lib/
│   │   ├── auth/                ← Auth.js config
│   │   ├── email/               ← Resend client
│   │   ├── rate-limit/          ← Upstash helpers
│   │   ├── analytics/           ← PostHog server-side singleton
│   │   ├── validations/         ← Zod schemas
│   │   └── utils/               ← Shared utilities
│   ├── actions/                 ← Next.js Server Actions
│   ├── hooks/                   ← Custom React hooks
│   └── types/                   ← Global TypeScript types
├── e2e/                         ← Playwright tests
├── .github/workflows/           ← CI/CD workflows
├── .code-reviews/               ← Bug, security, performance audit reports
├── drizzle.config.ts
├── vitest.config.ts
├── playwright.config.ts
├── lighthouserc.cjs
├── .env.example
├── ARCHITECTURE.md
├── SECURITY.md
└── README.md
```

---

## Security Model

Every generated app is built with these security properties by default:

- **Auth guards on every protected server action** — `auth()` called first; session validated before any DB operation
- **Row-level data isolation** — all queries filter by `userId`; users can only access their own data
- **Zod validation on all inputs** — every server action and API route validates inputs before use
- **Parameterised queries throughout** — Drizzle ORM; zero raw SQL interpolation
- **Rate limiting on all API routes** — Upstash sliding window counter in middleware
- **Security headers** — CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy
- **No secrets in files** — all credentials in Vercel env vars and GitHub Actions secrets, set via MCP
- **CSRF protection** — Auth.js handles this natively
- **Semgrep SAST** — OWASP Top 10 ruleset runs weekly in CI

The Phase 4 security review runs a full OWASP Top 10 checklist against every generated file before deployment.

---

## External API Handling

When your idea requires external data, the agent follows this decision tree automatically:

```
External data dependency detected in IDEA/GOAL
              │
              ▼
  Free public API exists for this data type?
              │
       ┌──────┴──────┐
      YES             NO
       │               │
  Use it silently   Freemium tier available?
  (no prompts)           │
                  ┌──────┴──────┐
                 YES             NO
                  │               │
          Ask once:          Document as limitation
          "Add [Service]     in ARCHITECTURE.md.
           for [data]?       Use mock/stub data.
           (yes/no)"         Flag for user to resolve.
```

### Built-in Free API Registry

The pipeline knows about these sources and uses them automatically, with no API key required:

| Domain | Source |
|---|---|
| Fantasy Premier League | `fantasy.premierleague.com/api/` (official, no key) |
| US stock prices | Yahoo Finance (unofficial, no key) |
| Weather | Open-Meteo (`api.open-meteo.com`, no key) |
| Geocoding | Nominatim / OpenStreetMap (no key) |
| Currency / FX rates | Frankfurter (`api.frankfurter.app`, no key) |
| Crypto prices | CoinGecko API (no key, rate limited) |
| News headlines | GNews (`gnews.io`, free tier) |
| Football match data | api-football.com (free: 100 req/day) |
| Maps | Leaflet.js + OpenStreetMap tiles (no key) |

For US options data, the agent will ask about Polygon.io (free tier: 5 req/min). For everything else, it sources free data or documents the gap.

---

## Known Constraints

This is an honest list of what the pipeline cannot fully automate:

| Constraint | Reason | Workaround |
|---|---|---|
| OAuth app registration | GitHub/Google require browser-based consent flows | ~2 min manual step; agent outputs exact callback URLs |
| Neon free tier: 1 project | Free tier is limited to a single Neon project | Upgrade to Launch (~$19/mo) for multiple projects |
| Vercel Hobby function timeout: 60s | Heavy server actions, external API chains, or cold starts can time out | Keep server actions fast; offload heavy work to background jobs |
| GitHub Actions: 2,000 min/mo (private repos) | Each CI run uses ~5–10 min; weekly/monthly jobs add more | Make the repo public (unlimited minutes) or upgrade GitHub plan |
| Context window on very large apps | Agent can't hold 10k+ lines in context simultaneously | Domain-by-domain review strategy; commits between phases preserve progress |
| `npm audit fix --force` may break code | Major version bumps can introduce breaking changes | Only auto-merges when `--force` is not needed; flags the rest as GitHub issues |
| CSP configuration | Strict CSP can break shadcn/ui, Sentry, PostHog in subtle ways | Starts in report-only mode; tightened iteratively after smoke tests pass |
| Semgrep cannot auto-fix | It's a static analysis scanner, not a code writer | Creates GitHub issues with findings; agent can read and fix manually |

---

## Repository Structure

```
one-shot-app/                        ← This repository
├── plan.md                          ← Complete 10-phase pipeline specification
├── create-app.prompt.md             ← Copilot agent mode trigger
├── CLAUDE.md                        ← Claude Code instructions (auto-loaded)
├── AGENTS.md                        ← Codex instructions (auto-loaded)
├── .github/
│   └── copilot-instructions.md      ← Copilot background context (auto-loaded)
└── README.md                        ← You are here

Generated apps are scaffolded as <app-name>/ inside this directory.
That subdirectory is the project root for the generated application.
```

---

## Frequently Asked Questions

**Do I need to write any code?**
No. The agent writes every file. Your input is: the idea prompt, OAuth app credentials (~2 min), and optionally a paid API key if your idea requires one.

**What happens if the agent hits an error?**
The agent self-corrects. TypeScript errors, failed tests, and deployment errors are all resolved autonomously before the pipeline advances to the next phase. If something is truly unresolvable, it surfaces a clear error rather than silently moving on.

**Can I use this for multiple apps?**
Yes. Run `#create-app.prompt.md` (or start a new chat in Claude Code/Codex) in a new VS Code workspace folder for each new app. The MCP tokens are reused. Note: the Neon free tier supports one project — you'll need to upgrade for a second live app.

**Can I modify the generated app afterwards?**
Yes. The generated repo is a normal Next.js project. Once delivered, you own it entirely. The CI/CD and automated maintenance workflows continue to run after handoff.

**Why not use Supabase instead of Neon?**
Supabase pauses databases after 1 week of inactivity on the free tier. Neon never pauses. For hobby apps that may sit unused for periods, Neon is significantly more reliable.

**Why not use Clerk instead of Auth.js?**
Clerk's free tier is capped at 10,000 monthly active users. Auth.js is open-source and free forever, with no MAU limits.

**Why is there no rollback on failed deployments?**
The database migration already ran before the deploy. Rolling back code to the previous version leaves old code running against the new schema, which causes its own cascade of failures. The correct approach is always to push a fix forward — and the pipeline creates a GitHub issue automatically when a production deploy fails.

---

## Acknowledgements

Built on: [Next.js](https://nextjs.org) · [Neon](https://neon.tech) · [Drizzle ORM](https://orm.drizzle.team) · [Auth.js](https://authjs.dev) · [Vercel](https://vercel.com) · [shadcn/ui](https://ui.shadcn.com) · [Upstash](https://upstash.com) · [Sentry](https://sentry.io) · [PostHog](https://posthog.com) · [Resend](https://resend.com)
