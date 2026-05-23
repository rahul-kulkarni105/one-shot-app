# One-Shot AI-Powered Website Creation Pipeline

> **Vision**: Enter a single prompt describing an idea → receive a fully deployed, production-grade website — with zero manual coding and near-zero user input beyond the prompt itself.

---

## Table of Contents

1. [Core Concept](#core-concept)
2. [What "Knowledge Base A" Means](#what-knowledge-base-a-means)
3. [The Prompt Interface](#the-prompt-interface)
4. [System Architecture](#system-architecture)
5. [Tech Stack — Fixed Decisions](#tech-stack--fixed-decisions)
6. [Tech Stack Decision Engine](#tech-stack-decision-engine)
7. [The 10-Phase Pipeline](#the-10-phase-pipeline)
8. [External API Key Handling](#external-api-key-handling)
9. [MCP Server Integration Layer](#mcp-server-integration-layer)
10. [Code Review Framework](#code-review-framework)
11. [CI/CD & Automation](#cicd--automation)
12. [Tech Debt & Maintenance](#tech-debt--maintenance)
13. [Minimum Unavoidable User Inputs](#minimum-unavoidable-user-inputs)
14. [One-Time Setup Guide](#one-time-setup-guide)
15. [Implementation Roadmap](#implementation-roadmap)
16. [Known Constraints & Limitations](#known-constraints--limitations)
17. [Decisions Log](#decisions-log)
18. [Issue Audit Log](#issue-audit-log)

---

## Core Concept

This system is an **AI Orchestration Pipeline** powered by GitHub Copilot (agent mode) + MCP servers. It takes a natural language idea and autonomously:

- Selects the best tech stack for the use case
- Scaffolds the entire project
- Detects required external APIs and sources free alternatives first
- Writes all production-quality code
- Runs 3 mandatory code review passes (bugs → security → performance)
- Creates a GitHub repository, commits, and pushes code
- Configures CI/CD with GitHub Actions
- Deploys to a live Vercel URL
- Monitors ongoing health, security, and tech debt automatically

The pipeline deploys **straight to production** — no manual PR approval needed. Auto-PRs handle ongoing maintenance.

---

## What "Knowledge Base A" Means

> "Knowledge Base A is the whole knowledge of creating the website from one statement — everything which is needed for that to be accomplished."

**Knowledge Base A is not a folder you maintain. It IS this pipeline system.**

It is the sum of:

| Component | What It Contains |
|---|---|
| **This plan** | The 10-phase pipeline, quality gates, review checklists |
| **The agent instructions file** | The `.prompt.md` that tells the agent exactly how to execute every phase |
| **The stack blueprints** | Canonical implementations of each supported stack, baked into the agent instructions |
| **The security checklists** | OWASP Top 10 checks written into the review phases |
| **The performance checklists** | Core Web Vitals audit baked into Phase 5 |
| **The deployment runbooks** | Vercel + GitHub Actions config templates embedded in the agent instructions |
| **The AI's training knowledge** | Next.js, TypeScript, Drizzle, Auth.js, Neon, Tailwind, shadcn/ui — the agent already knows these |

When you invoke the pipeline, the agent operates with all of the above simultaneously. There are no external docs to fetch at runtime — the knowledge is encoded in the agent instructions themselves.

**The only work required to "build Knowledge Base A" is writing excellent agent instructions.** That is the implementation task in Phase A of the roadmap below.

---

## The Prompt Interface

### User-Facing Input Format

```
IDEA:    [What the app is — describe the domain and core purpose]
GOAL:    [What a user should be able to DO with it]
EXTRA:   [Optional: specific integrations, constraints, known requirements]
```

### Example Prompts

```
IDEA:    A Fantasy Premier League companion app
GOAL:    Users can view their FPL team, get AI transfer suggestions,
         track points vs friend groups, and see upcoming fixture difficulty
EXTRA:   Real-time data, needs FPL public API integration
```

```
IDEA:    An options screener for US stocks
GOAL:    Users filter options chains by IV rank, delta, DTE, and premium;
         get alerts when criteria are met; save watchlists
EXTRA:   Low latency required, mobile-friendly
```

That single block is the ONLY user input required at runtime (plus any unavoidable single-prompt for API keys — see the External API section).

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                      USER PROMPT                             │
│              (IDEA + GOAL + EXTRA)                           │
└───────────────────────────┬──────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│               ORCHESTRATION AGENT                            │
│          (GitHub Copilot — Agent Mode)                       │
│                                                              │
│  ┌────────────────────────┐  ┌────────────────────────────┐ │
│  │   Knowledge Base A     │  │  MCP Integration Layer     │ │
│  │  (encoded in agent     │  │  GitHub | Vercel | Neon    │ │
│  │   instructions)        │  │                            │ │
│  └────────────────────────┘  └────────────────────────────┘ │
└───────────────────────────┬──────────────────────────────────┘
                            │
               ┌────────────▼────────────┐
               │    10-PHASE PIPELINE    │
               └────────────┬────────────┘
                            │
    ┌───────────────────────▼────────────────────────┐
    │                  OUTPUTS                       │
    │  • Live website URL (*.vercel.app)             │
    │  • GitHub repository (private)                 │
    │  • CI/CD pipeline (GitHub Actions)             │
    │  • Architecture docs + review reports          │
    │  • Scheduled maintenance workflows             │
    └────────────────────────────────────────────────┘
```

---

## Tech Stack — Fixed Decisions

These are non-negotiable across all app types. Chosen for free tiers, best DX, and hobby-scale suitability.

| Concern | Tool | Free Tier | Reason |
|---|---|---|---|
| Framework | Next.js 15 (App Router) | OSS | Best full-stack React framework |
| Language | TypeScript (strict mode) | OSS | Type safety, fewer runtime bugs |
| Styling | Tailwind CSS v4 | OSS | Speed, consistency, tiny bundle |
| UI Components | shadcn/ui | OSS | Accessible, copy-owned, no lock-in |
| Database | **Neon** (serverless PostgreSQL) | 0.5GB, never pauses | Best free PostgreSQL, no cold starts |
| ORM | **Drizzle ORM** | OSS | Type-safe, thin, excellent Neon integration |
| Auth | **Auth.js v5** (NextAuth) + `@auth/drizzle-adapter` | OSS | Free forever; OAuth + magic link + credentials |
| Email (for magic links) | **Resend** | 3k/mo, 100/day free | Native Next.js SDK, required by Auth.js magic link |
| Rate limiting | **`@upstash/ratelimit`** + **Upstash Redis** | 10k cmds/day free | Edge-compatible, the standard for Next.js |
| Validation | Zod | OSS | Runtime + compile-time safety |
| File storage | Vercel Blob | 500MB free | When app needs file uploads |
| Linting | ESLint + Prettier | OSS | Code consistency |
| Testing | Vitest + Playwright | OSS | Unit + E2E |
| Error tracking | Sentry | 5k events/mo free | Production error visibility |
| Analytics | PostHog (`posthog-js` + `posthog-node`) | 1M events/mo free | Client + server-side analytics |
| Git platform | GitHub | Free private repos | MCP support, Actions, full ecosystem |
| Deployment | Vercel | Hobby tier free | Best DX, edge functions, preview URLs |

**Why Neon over Supabase**: Supabase pauses databases after 1 week of inactivity on the free tier. Neon never pauses.

**Why Auth.js over Clerk**: Clerk has a 10k MAU limit before paid plans. Auth.js is free forever.

**Why Resend**: Auth.js magic link authentication requires an email provider. Without one, magic links silently fail. Resend has the best free tier and an official Auth.js adapter.

**Why Upstash**: Rate limiting on auth routes requires a persistent counter store. Upstash Redis is the only edge-compatible free-tier option that works in Vercel edge middleware.

---

## Tech Stack Decision Engine

The agent classifies the app idea and selects additional tools. The base stack above is always used.

```
What is the primary nature of the app?
│
├── Content-heavy / Blog / Marketing site
│   └── ADD: next-mdx-remote for content (NOT Contentlayer — deprecated)
│       SKIP: Neon (no DB needed for pure content)
│
├── CRUD app / SaaS / Dashboard  ← DEFAULT
│   ├── Needs real-time updates?
│   │   └── ADD: Pusher Channels (free tier: 200k msg/day)
│   │
│   ├── Needs payments?
│   │   └── ADD: Stripe (test mode free; live mode % fee only)
│   │
│   └── Base stack is sufficient for most cases
│
├── Data / Analytics / Screener / Dashboard
│   └── ADD: shadcn/ui charts (Recharts-based, no extra dep)
│       ADD: TanStack Table for data grids
│       OPTIMIZE: aggressive caching, cursor pagination
│
├── E-commerce
│   └── ADD: Stripe + Stripe Checkout
│       ADD: Vercel Blob for product images
│
└── API / External data integration heavy
    └── ADD: tRPC for end-to-end type-safe API layer
        ADD: external API clients with Zod-validated responses
        OPTIMIZE: rate limiting all external API calls, cache responses in Neon
```

---

## The 10-Phase Pipeline

Each phase must complete successfully before the next begins. The agent self-corrects all errors before advancing. **No human input is needed between phases** except for the OAuth ordering exception documented in Phase 1.

---

### Phase 0 — Requirements Analysis & Architecture Planning

**Input**: User prompt | **Output**: Architecture Decision Record (ADR)

1. Parse IDEA + GOAL + EXTRA
2. Classify app type using the decision tree above
3. Identify all required features: auth type (OAuth / magic link / credentials), payments, real-time, uploads, external APIs
4. **Run the External API Key Handling flow** (see dedicated section)
5. Select final tech stack additions
6. Design database schema: tables, columns, relations, indexes, constraints
7. Define all server actions and API routes
8. Define page structure and component hierarchy
9. Write `ARCHITECTURE.md` — the full plan before any code is written

**This phase is planning-only — zero code files created.**

---

### Phase 1 — Project Scaffolding

**Input**: ADR | **Output**: Initialized project with all config, zero app code yet

1. `npx create-next-app@latest <app-name> --typescript --tailwind --app --eslint --src-dir --import-alias "@/*" --use-npm` — pass all flags to avoid interactive prompts
2. `npx shadcn@latest init -d` — installs and initializes shadcn/ui using defaults to avoid interactive prompts (`-d` / `--defaults` flag). The old `npx shadcn-ui@latest init` package name was deprecated; the correct current name is `shadcn`.
3. Install runtime ORM deps: `npm install drizzle-orm @neondatabase/serverless`
   Install Drizzle CLI as dev dependency: `npm install -D drizzle-kit`
   (`drizzle-kit` is a CLI tool only — never imported at runtime; installing it as a regular dependency bloats the production `node_modules`)
4. Install: `next-auth@5 @auth/drizzle-adapter` (Auth.js v5 graduated from beta — use the `@5` dist-tag, not `@beta` which may be stale or no longer updated)
5. Install: `resend` (for magic link email — do NOT install `@auth/core` separately; it is a peer dependency of `next-auth@5`, and pinning a separate version causes auth runtime conflicts)
6. Install: `@upstash/ratelimit @upstash/redis`
7. Install: `zod @sentry/nextjs posthog-js posthog-node`
8. Install (dev): `npm install -D vitest @vitejs/plugin-react jsdom @vitest/coverage-v8` — test tooling must be in `devDependencies`, not `dependencies`
9. Install (dev): `npm install -D @playwright/test` → then immediately run `npx playwright install --with-deps chromium` (installs browser binaries required locally and in CI) — must be a devDependency
10. Install: `@vercel/analytics @vercel/speed-insights`
11. Install: `npm install -D @next/bundle-analyzer eslint-plugin-security @lhci/cli` (all dev dependencies — `@next/bundle-analyzer` used in Phase 5 bundle size audit; `eslint-plugin-security` required by Phase 1 step 15 ESLint config; `@lhci/cli` used by `deploy-production.yml` on every production deploy — must be in `package.json` before the first deploy in Phase 8)
12. Create `vitest.config.ts`:
    ```ts
    import { defineConfig } from 'vitest/config'
    import react from '@vitejs/plugin-react'
    import path from 'path'

    export default defineConfig({
      plugins: [react()],
      test: { environment: 'jsdom', globals: true },
      resolve: { alias: { '@': path.resolve(__dirname, './src') } },
    })
    ```
13. Create `playwright.config.ts`:
    ```ts
    import { defineConfig, devices } from '@playwright/test'

    export default defineConfig({
      testDir: './e2e',
      timeout: 30_000,
      fullyParallel: true,
      retries: process.env.CI ? 2 : 0,
      use: {
        baseURL: process.env.PLAYWRIGHT_BASE_URL ?? 'http://localhost:3000',
        trace: 'on-first-retry',
      },
      projects: [{ name: 'chromium', use: { ...devices['Desktop Chrome'] } }],
    })
    ```
    > `webServer` is intentionally omitted: in CI the server is started explicitly before Playwright runs; locally `npm run dev` is expected to already be running.
14. Create `drizzle.config.ts` — pointing to `src/db/schema/`, using `DATABASE_URL_UNPOOLED`. Drizzle-kit does not automatically load Next.js's `.env.local` — explicitly load it at the top of the config so local `db:*` scripts work:
    ```ts
    import { config } from 'dotenv'
    config({ path: '.env.local' }) // load .env.local for local drizzle-kit commands (Next.js convention)
    import { defineConfig } from 'drizzle-kit'

    export default defineConfig({
      schema: './src/db/schema/',
      out: './src/db/migrations/',
      dialect: 'postgresql',
      dbCredentials: { url: process.env.DATABASE_URL_UNPOOLED! },
    })
    ```
    `dotenv` is a dependency of many packages already, but add it explicitly as a devDependency: `npm install -D dotenv`
15. Configure ESLint with `eslint-plugin-security`; also add the `complexity` rule so `tech-debt.yml` has something to detect:
    ```json
    "rules": {
      "complexity": ["warn", { "max": 10 }]
    }
    ```
16. Configure Prettier
17. Add npm scripts to `package.json`:
    ```json
    "test": "vitest run",
    "test:coverage": "vitest run --coverage",
    "db:push": "drizzle-kit push",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:studio": "drizzle-kit studio",
    "analyze": "ANALYZE=true next build"
    ```
    > **`test` and `test:coverage` scripts are required**: `ci.yml`'s `test-unit` job and `tech-debt.yml`'s coverage report both invoke tests via these npm scripts. Without them, the agent would fall back to bare `npx vitest run`, inconsistent with every other tool in the plan.
    > **`db:push` footgun warning**: `db:push` bypasses migration files — it is a dev convenience only. Never use it for CI-consistency-critical workflows. If confused, always use `db:migrate`.
    And configure `next.config.mjs` to wire up `@next/bundle-analyzer` (without this wrapper the `analyze` script is a no-op):
    ```js
    import withBundleAnalyzer from '@next/bundle-analyzer'

    const bundleAnalyzer = withBundleAnalyzer({
      enabled: process.env.ANALYZE === 'true',
    })

    /** @type {import('next').NextConfig} */
    const nextConfig = {
      // ... other config
    }

    export default bundleAnalyzer(nextConfig)
    ```
18. Create `.env.example` with ALL variables documented:
    ```
    # Database
    DATABASE_URL=                    # Neon pooled (for app queries)
    DATABASE_URL_UNPOOLED=           # Neon direct (for drizzle-kit migrations)
    
    # Auth
    AUTH_SECRET=                     # openssl rand -base64 32
    AUTH_URL=                        # https://your-app.vercel.app  ← CRITICAL
    AUTH_GITHUB_ID=                  # GitHub OAuth App client ID
    AUTH_GITHUB_SECRET=              # GitHub OAuth App client secret
    
    # Email (for magic links)
    RESEND_API_KEY=                  # From resend.com
    AUTH_RESEND_KEY=                 # Same value as RESEND_API_KEY, used by Auth.js Resend email provider
    
    # Rate limiting
    UPSTASH_REDIS_REST_URL=          # From upstash.com
    UPSTASH_REDIS_REST_TOKEN=        # From upstash.com
    
    # Monitoring
    NEXT_PUBLIC_SENTRY_DSN=          # From sentry.io project settings
    SENTRY_AUTH_TOKEN=               # From sentry.io auth tokens (project:write) — used for source map upload at build time
    NEXT_PUBLIC_POSTHOG_KEY=
    NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com
    ```
19. Configure `tsconfig.json`:
    - Path alias: `@/` → `./src/`
    - Add Vitest global types so `tsc --noEmit` does not error on `describe`, `it`, `expect` etc. used without imports in test files:
      ```json
      "compilerOptions": {
        "types": ["vitest/globals", "node"]
      }
      ```
      **Append to the existing `types` array — do not replace any other entries.** `create-next-app` does not set a `types` field by default, so this creates it. Both entries are required:
      - `"vitest/globals"` — Vitest's `describe`, `it`, `expect` ambient declarations
      - `"node"` — Node.js ambient globals (`process.env`, `Buffer`, etc.) used throughout Next.js server-side code. When TypeScript's `types` array is set, it restricts ambient type auto-discovery to only listed packages; omitting `"node"` causes `tsc --noEmit` to fail with "Cannot find name 'process'" in server components, server actions, and route handlers.
20. Create `lighthouserc.cjs` in the project root (must be created here in Phase 1 and committed — `deploy-production.yml` runs `lhci autorun` starting from the Phase 8 step 10 merge to main, which happens BEFORE Phase 9; if the config file is not committed before that merge, Lighthouse CI has no config and fails or runs with defaults):
    ```js
    /** @type {import('@lhci/cli').LighthouseRcConfig} */
    module.exports = {
      ci: {
        collect: {
          url: [process.env.LHCI_URL ?? 'http://localhost:3000'],
          numberOfRuns: 1,
        },
        assert: {
          // Do NOT add `preset: 'lighthouse:recommended'` here — it stacks dozens of
          // individual audit-level assertions on top of these category checks. A generated
          // app can score 93 in Performance but still fail CI because one preset audit hits
          // its own threshold, producing opaque failures. Keep only category-level gates.
          assertions: {
            'categories:performance':    ['error', { minScore: 0.9 }],
            'categories:accessibility':  ['error', { minScore: 0.9 }],
            'categories:best-practices': ['error', { minScore: 0.9 }],
            'categories:seo':            ['error', { minScore: 0.9 }],
          },
        },
        upload: { target: 'temporary-public-storage' },
      },
    }
    ```
    Use `.cjs` extension (not `.js`) because `create-next-app` sets `"type": "module"` in `package.json`, making plain `.js` files ES modules; `module.exports` is CommonJS syntax and throws `ReferenceError: module is not defined` in ESM context. **Commit this file before Phase 8 step 10.**

> **⚠ IMPORTANT — Git starts HERE**: After scaffolding is complete, run `git init`, create the GitHub repo via MCP, and make the initial scaffold commit. This ensures version history exists throughout code generation. (Full git setup happens again in Phase 6 for branch protection, but the repo and history must exist from Phase 1.)

21. Create canonical folder structure:

```
src/
├── middleware.ts            ← Auth.js route protection + Upstash rate limiting (combined into single file)
├── app/
│   ├── (auth)/              ← Sign in / sign up pages
│   ├── (protected)/         ← Auth-guarded routes
│   ├── (public)/            ← Public routes
│   └── api/
│       └── auth/[...nextauth]/route.ts  ← Auth.js handler
├── components/
│   ├── ui/                  ← shadcn/ui components
│   ├── features/            ← Feature-specific components
│   └── layouts/             ← Shared layout components
├── db/
│   ├── schema/              ← Drizzle table definitions (one file per domain)
│   ├── migrations/          ← Generated SQL migrations
│   └── index.ts             ← Neon serverless DB client
├── lib/
│   ├── auth/                ← Auth.js config (providers, adapter, callbacks)
│   ├── email/               ← Resend client config
│   ├── rate-limit/          ← Upstash ratelimit helpers
│   ├── validations/         ← Zod schemas (one file per domain entity)
│   └── utils/               ← Shared utilities
├── types/                   ← Global TypeScript types / interfaces
├── hooks/                   ← Custom React hooks
└── actions/                 ← Next.js Server Actions (one file per domain)
```

---

### Phase 2 — Full Code Generation

**Input**: ADR + scaffolded project | **Output**: Complete working application

Generation respects dependency order — each layer complete before the next:

1. **DB schema** — Drizzle schema files per domain entity, including Auth.js required tables (users, accounts, sessions, verificationTokens) via `@auth/drizzle-adapter` schema helpers
2. **DB migrations** — `npm run db:generate` → SQL files created in `src/db/migrations/` (the `out` directory in `drizzle.config.ts`) → **immediately `git add src/db/migrations/ && git commit -m "chore: add initial migrations"`** (use the npm script, not bare `drizzle-kit generate` — the binary is in `node_modules/.bin/` not in global PATH)
   ⚠ The migration SQL files MUST be committed before Phase 8 CI runs. `npm run db:migrate` in both the CI E2E job and `deploy-production.yml` reads committed migration files from the repo — if they are only on disk but not committed, `drizzle-kit migrate` finds no pending migrations, runs nothing, and every table is missing; E2E tests crash with "relation does not exist" errors.
3. **TypeScript types** — Inferred from Drizzle using `typeof table.$inferSelect` / `typeof table.$inferInsert` (the current Drizzle ORM API). Do NOT use the old `InferSelectModel` / `InferInsertModel` named exports — they are deprecated in drizzle-orm v0.29+ and may not exist in newer versions.
4. **Zod schemas** — One schema per input form / server action, derived from DB types
5. **Auth config** — Auth.js v5 with Drizzle adapter, providers (OAuth + magic link via Resend), session callbacks
6. **Sentry wiring** — `@sentry/nextjs` requires three config files and editing `next.config.mjs`; without all four, the package is installed but captures nothing:
   - `sentry.client.config.ts` — `Sentry.init({ dsn, tracesSampleRate, replaysOnErrorSampleRate })`
   - `sentry.server.config.ts` — `Sentry.init({ dsn, tracesSampleRate })`
   - `sentry.edge.config.ts` — `Sentry.init({ dsn })` (minimal, no profiling on edge)
   - **Edit** `next.config.mjs` (created in Phase 1 step 17) — add the Sentry import and change the final export:
     ```js
     import { withSentryConfig } from '@sentry/nextjs'
     // ... existing bundleAnalyzer import and nextConfig ...
     export default withSentryConfig(bundleAnalyzer(nextConfig), {
       // silent: true was REMOVED in @sentry/nextjs v8 — do NOT include it; it is ignored and
       // triggers a deprecation warning in newer versions
       widenClientFileUpload: true,  // upload more source files for better stack traces
       hideSourceMaps: true,         // prevent source maps from being bundled into the client
       disableLogger: true,          // suppress Sentry CLI build-time output
     })
     ```
     The `bundleAnalyzer(nextConfig)` wrapper from Phase 1 must be preserved — do not overwrite it.
   - `SENTRY_AUTH_TOKEN` is required at build time for source map uploads (without it builds warn and stack traces in Sentry are unreadable minified code). This secret must be in Vercel env vars and GitHub Actions secrets — see Phase 6 step 3 and Phase 8 step 6.
7. **`src/middleware.ts`** — Next.js has exactly one middleware file. This single file must combine: (a) Auth.js `auth()` call to protect `(protected)/` routes from unauthenticated access; (b) Upstash rate limiting on `/api/auth/*` and mutation routes. **Without this file, `(protected)/` routes are publicly accessible — this is a required generation step.** Use this canonical pattern:
   ```ts
   import { auth } from '@/lib/auth'
   import { Ratelimit } from '@upstash/ratelimit'
   import { Redis } from '@upstash/redis'
   import { NextResponse } from 'next/server'

   const ratelimit = new Ratelimit({
     redis: Redis.fromEnv(),
     limiter: Ratelimit.slidingWindow(10, '10 s'),
   })

   export default auth(async (req) => {
     // auth() types req as NextAuthRequest (NextRequest & { auth: Session | null })
     // — do NOT add an explicit type annotation; it overrides Auth.js inference and
     //   forces an (req as any).auth cast that breaks strict-mode tsc.
     const { pathname } = req.nextUrl

     // Rate limit all API routes
     if (pathname.startsWith('/api/')) {
       const ip = req.headers.get('x-forwarded-for') ?? '127.0.0.1'
       const { success } = await ratelimit.limit(ip)
       if (!success) return NextResponse.json({ error: 'Too many requests' }, { status: 429 })
     }

     // Protect authenticated routes — replace with your actual protected URL paths
     // NOTE: App Router route groups like (protected)/ do NOT appear in URLs;
     //       match by actual URL path (e.g. /dashboard, /settings).
     if (pathname.startsWith('/dashboard') || pathname.startsWith('/settings')) {
       if (!req.auth) return NextResponse.redirect(new URL('/sign-in', req.url))
     }

     return NextResponse.next()
   })

   export const config = {
     // Run middleware on all routes except static files, images, and favicon
     matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
   }
   ```
   > **Path matching note**: The App Router route group `(protected)/` is a file-system convention only — it does not appear in the URL. Protect routes by their actual URL paths (e.g., `/dashboard`, `/settings`), not by the folder name `(protected)`.

8. **Server actions** — All CRUD operations: auth guard → Zod parse → business logic → DB query
9. **Data fetching** — Server Component fetches (primary pattern); `useFormStatus` + `useOptimistic` for client interactions; no separate state management library unless complexity demands it
10. **UI components** — Atoms → molecules → organisms, composed bottom-up
11. **Pages** — Full App Router pages with metadata, composed from components
12. **Layouts** — Root layout must include all of the following (missing any one silently breaks that service):
    - `<Analytics />` from `@vercel/analytics`
    - `<SpeedInsights />` from `@vercel/speed-insights`
    - **PostHog provider** — `posthog-js` calls browser APIs (`window`, `document`) and **cannot run in a Server Component**. Generate a separate `src/app/providers.tsx` Client Component:
      ```tsx
      'use client'
      import posthog from 'posthog-js'
      import { PHProvider } from 'posthog-js/react'
      import { useEffect } from 'react'

      export function Providers({ children }: { children: React.ReactNode }) {
        useEffect(() => {
          posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
            api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
            capture_pageview: false, // handled by PostHogPageView component below
          })
        }, [])
        return <PHProvider client={posthog}>{children}</PHProvider>
      }
      ```
      Also generate a `PostHogPageView` component (in the same file or `src/components/PostHogPageView.tsx`) that fires `$pageview` on every route change — required because `capture_pageview: false` disables PostHog's own auto-tracking (which doesn't work correctly in Next.js App Router SPA navigations):
      ```tsx
      'use client'
      import { usePathname, useSearchParams } from 'next/navigation'
      import { useEffect } from 'react'
      import { usePostHog } from 'posthog-js/react'

      export function PostHogPageView() {
        const pathname = usePathname()
        const searchParams = useSearchParams()
        const posthog = usePostHog()
        useEffect(() => {
          if (pathname && posthog) {
            posthog.capture('$pageview', { $current_url: window.location.href })
          }
        }, [pathname, searchParams, posthog])
        return null
      }
      ```
      Render `<PostHogPageView />` inside `<Providers>` AND inside a `<Suspense>` boundary — **both wrappers are simultaneously required**: `usePostHog()` needs PostHog context from `<Providers>`, and `useSearchParams()` must be inside `<Suspense>` or Next.js opts the entire root layout out of static rendering. The correct structure in the root layout is:
      ```tsx
      <Providers>
        <Suspense fallback={null}>
          <PostHogPageView />
        </Suspense>
        {children}
      </Providers>
      ```
      Without this component, pageviews are never recorded despite PostHog being initialized.
      Then import `<Providers>` into the root layout (which remains a Server Component). **Do NOT put `posthog.init()` directly in the layout — it will crash with "window is not defined".**
    - **Server-side PostHog (`posthog-node`)** — for capturing analytics from Server Actions or Route Handlers. Generate `src/lib/analytics/posthog-server.ts`:
      ```ts
      import { PostHog } from 'posthog-node'

      // Singleton — do NOT create a new PostHog instance per request
      export const posthogServer = new PostHog(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
        host: process.env.NEXT_PUBLIC_POSTHOG_HOST ?? 'https://app.posthog.com',
        flushAt: 1,    // flush immediately (serverless environment — no long-lived process)
        flushInterval: 0,
      })
      ```
      Usage in a server action: `posthogServer.capture({ distinctId: session.user.id, event: 'item_created', properties: { ... } }); await posthogServer.flushAsync()`. ⚠ `capture()` is synchronous (void return) — `await posthogServer.flushAsync()` is required to ensure the event is sent before the serverless function terminates. An `await` on `capture()` itself is a no-op.
      **`posthog-node` captures nothing if this singleton is never created** — it is not auto-initialised like the browser SDK.
    - Auth layout and protected layout (nav/sidebar)
13. **Error boundaries** — `error.tsx` per route segment. **`error.tsx` MUST have `'use client'` as its first line** — Next.js App Router enforces this at build time and throws "Error component must be a client component" without it. The `reset` prop passed to error components is a function, which cannot be serialised across the server/client boundary.
14. **Loading states** — `loading.tsx` with skeleton UIs per route segment
15. **Empty states** — UI for when lists/tables contain no data
16. **Unit tests** — Vitest tests for all server actions and utility functions
17. **E2E tests** — Playwright tests for: page load, sign up, core feature, sign out (these will run against a Neon branch DB — see CI/CD section)

---

### Phase 3 — Review Round 1: Bug & Logic Audit

> **Context window strategy**: Agent reviews domain-by-domain (DB layer → auth layer → server actions → UI layer), not all files at once. Each domain is reviewed completely before moving to the next.

Agent re-reads every generated file and fixes every issue before continuing.

- [ ] `tsc --noEmit` with `strict: true` passes with zero errors
- [ ] All async/await operations in try/catch or using Result types
- [ ] No unhandled promise rejections
- [ ] All external data (API responses, DB results) null-checked before use
- [ ] Form submissions correctly handle: loading, success, and error states
- [ ] Error messages are user-friendly and do not leak internals (stack traces, SQL errors)
- [ ] All DB mutations check resource existence before updating/deleting
- [ ] All list queries have pagination — no unbounded `SELECT *`
- [ ] No hardcoded values that belong in environment variables
- [ ] No TODO/FIXME comments — all resolved
- [ ] Edge cases: empty string, max length, special characters, concurrent requests
- [ ] No dead code paths, unreachable branches, or unused imports
- [ ] Every protected server action calls `auth()` and validates the session first
- [ ] Middleware correctly protects configured protected URL paths (e.g., `/dashboard`, `/settings`) — match by actual URL path, not the route-group folder name `(protected)/` which never appears in URLs
- [ ] Middleware `auth()` handler correctly checks `req.auth` for null/undefined and redirects unauthenticated requests to `/sign-in` — verify the pattern uses `req.auth` directly on the `NextAuthRequest` argument (NOT an `authorized` callback, which is a separate, incompatible Auth.js v5 middleware approach; do not add one)

**Output**: Issue list with severity ratings + confirmation all issues fixed.

---

### Phase 4 — Review Round 2: Security Audit

Audits against OWASP Top 10 + Next.js/Auth.js-specific patterns. Fixes every issue before continuing.

**A01 — Broken Access Control**
- [ ] Every server action verifies `session.user.id` before operating on data
- [ ] Queries filter by `userId` — users can only touch their own data
- [ ] No security logic in Client Components (can be bypassed)
- [ ] API routes return 401 for unauthenticated requests

**A02 — Cryptographic Failures**
- [ ] Zero secrets in the repository (`.env*` in `.gitignore`)
- [ ] `AUTH_SECRET` is a cryptographically strong random value (generated via `openssl rand -base64 32`)
- [ ] No sensitive data in `localStorage` or JS-accessible cookies
- [ ] HTTPS enforced at Vercel level (automatic)

**A03 — Injection**
- [ ] All DB queries use Drizzle ORM parameterized methods — zero raw SQL interpolation
- [ ] All user inputs pass Zod `.parse()` before use
- [ ] No `eval()`, `new Function()`, or unsanitized `dangerouslySetInnerHTML`

**A04 — Insecure Design**
- [ ] Rate limiting applied to all auth routes via Upstash ratelimit middleware
- [ ] No mass assignment: server actions explicitly pick only expected fields
- [ ] All business logic enforced server-side

**A05 — Security Misconfiguration**
- [ ] `next.config.mjs` security headers configured (note: `create-next-app` generates `.mjs` by default; rename to `.ts` only if TypeScript config imports are explicitly required):
  - `Content-Security-Policy` (crafted carefully — overly strict CSP breaks shadcn/ui and Sentry)
  - `Strict-Transport-Security: max-age=63072000; includeSubDomains`
  - `X-Frame-Options: DENY`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy: camera=(), microphone=(), geolocation=()`
- [ ] CORS restricted to expected origins on API routes
- [ ] No debug/admin endpoints exposed in production
- [ ] `.env.local` confirmed in `.gitignore`

**A06 — Vulnerable Components**
- [ ] `npm audit --audit-level=high` passes with zero findings
- [ ] No `^` version ranges on major versions in `package.json`
  > **Corrective action**: `npm install` always writes `^` by default, so this will flag most packages. Do not blindly remove all carets — instead pin security-sensitive and deeply integrated packages where a major version bump would silently break things (e.g., `next-auth`, `drizzle-orm`, `@sentry/nextjs`). For each, remove the `^` prefix in `package.json` and run `npm install` to update the lock file. Lower-risk utility packages can retain `^`.

**A07 — Authentication Failures**
- [ ] Passwords never appear in logs
- [ ] Auth.js cookies: `httpOnly: true`, `secure: true`, `sameSite: lax`
- [ ] Sign-in returns identical error messages for "user not found" vs "wrong password"
- [ ] CSRF protection active (Auth.js handles this natively)
- [ ] `AUTH_URL` env var is set correctly in Vercel → OAuth callbacks will not 404

**A08 — Software and Data Integrity Failures**
- [ ] No `npm install` of packages from untrusted or unscoped sources; all packages resolve from the official npm registry
- [ ] `package-lock.json` committed and `npm ci` used in CI (not `npm install`) — ensures reproducible, integrity-verified installs
- [ ] GitHub Actions workflow files pinned to specific SHA or tagged versions for third-party actions (e.g., `neondatabase/create-branch-action@v5`) to prevent supply chain attacks via mutable tags
- [ ] No `--ignore-scripts` bypass in CI that would prevent npm lifecycle hooks from running integrity checks
- [ ] `deploy-production.yml` runs migration before deploy (not after) — DB and code changes applied atomically in the correct order to prevent data integrity failures

**A09 — Logging & Monitoring**
- [ ] Sentry configured and capturing unhandled exceptions in both client and server
- [ ] No PII (email, name, IDs) in Sentry breadcrumb or log data

**A10 — SSRF (Server-Side Request Forgery)**
- [ ] No server action or API route accepts a user-supplied URL and performs a server-side fetch with it
- [ ] Any server-side URL construction uses only hardcoded base URLs from environment variables — not user input
- [ ] If a feature genuinely requires user-supplied URLs, validate against a strict allowlist of expected domains before fetching

**Output**: Security report with OWASP mapping + all issues fixed.

---

### Phase 5 — Review Round 3: Performance Audit

- [ ] **LCP < 2.5s**: Above-fold images use `priority`; fonts use `font-display: swap`
- [ ] **CLS = 0**: All images have explicit width + height; no layout-shifting dynamic inserts
- [ ] **INP < 200ms**: No synchronous heavy work blocking the main thread on interaction
- [ ] **N+1 eliminated**: List views use JOIN queries or batch fetches, never per-row queries
- [ ] **DB indexes**: Every column in WHERE / ORDER BY / JOIN ON has a Drizzle index defined in schema
- [ ] **Cursor pagination**: All paginated endpoints use cursor-based, not offset-based
- [ ] **Caching**: Public static pages use ISR `revalidate`; user-specific pages use `no-store`
- [ ] **Images**: All images via `next/image`; no raw `<img>` tags
- [ ] **Bundle size**: `@next/bundle-analyzer` confirms no chunk > 200KB parsed
- [ ] **Dynamic imports**: Heavy components (charts, editors) use `dynamic(() => import(...), { ssr: false })`
- [ ] **Server Components**: All data fetching in Server Components; Client Components only where interactivity is needed
- [ ] **Suspense**: Granular `<Suspense>` boundaries for streaming; no sequential request waterfalls

**Output**: Performance report + Lighthouse score targets + all issues fixed.

---

### Phase 6 — Git Branch Protection & CI Setup

> Note: The repo was already created and initial commits made at the end of Phase 1. This phase finalises branch protection rules and sets up CI secrets.

1. Configure branch protection on `main` via GitHub MCP:
   - Required status checks: `CI / lint`, `CI / typecheck`, `CI / test-unit`, `CI / test-e2e`, `CI / audit`
     GitHub Actions reports status checks as `WorkflowName / JobName`. The `ci.yml` workflow must have `name: CI` at the top — if the workflow name differs, these check names won't match and branch protection silently allows all merges regardless of CI result. The exact job names (`lint`, `typecheck`, `test-unit`, `test-e2e`, `audit`) are defined in Phase 7's `ci.yml`; the `CI /` prefix comes from the workflow's `name:` field.
   - No force pushes
   - **Required pull request reviews: 0** (explicitly set to allow auto-merge)
   - Allow auto-merge when all checks pass
2. Create `develop` branch
3. Via GitHub MCP, add repository secrets for GitHub Actions:
   - `VERCEL_TOKEN` — from Vercel account settings
   - `NEON_API_KEY` — for Neon branch creation and migration runner in CI (account-level key, available from One-Time Setup)
   - **Operator-level reusable secrets** (same values across all projects — set once from One-Time Setup credentials):
     `RESEND_API_KEY`, `AUTH_RESEND_KEY`,
     `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`,
     `NEXT_PUBLIC_POSTHOG_KEY`, `NEXT_PUBLIC_POSTHOG_HOST`,
     `NEXT_PUBLIC_SENTRY_DSN`, `SENTRY_AUTH_TOKEN`
     (plus any external API keys collected in Phase 0)
   - **`AUTH_URL`** — the production URL (`https://<app>.vercel.app`). This is project-specific but must be added now because `deploy-production.yml` reads it to derive `PLAYWRIGHT_BASE_URL` and `LHCI_URL`. Use the Vercel project name chosen in Phase 0 to form the URL — it will match the actual deployment.
   > **Secrets deferred because values don't exist yet:**
   > - `AUTH_SECRET` — generated and added as GitHub Secret in Phase 8 step 6a; set in Vercel env vars in Phase 8 step 6b
   > - `DATABASE_URL`, `DATABASE_URL_UNPOOLED` — Neon not provisioned until Phase 8 step 1
   > - `NEON_PROJECT_ID` — Neon project not created until Phase 8 step 1; add immediately after creation alongside `DATABASE_URL`
   > - `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID` — Vercel project not created until Phase 8 step 3
   > **Do not set placeholder values** for any deferred secret — a placeholder causes the workflow to fail silently when triggered.

---

### Phase 7 — CI/CD Pipeline Configuration

> **Branch target for Phase 7**: All workflow files created in this phase must be committed to the **`develop` branch** (created in Phase 6 step 2), NOT to `main`. Committing to `main` immediately triggers `deploy-production.yml`, which will fail because `AUTH_SECRET`, `DATABASE_URL`, `VERCEL_ORG_ID`, and `VERCEL_PROJECT_ID` are not yet in GitHub Secrets (those are added in Phase 8). The first legitimate push to `main` happens after Phase 8 is fully complete.

Creates `.github/workflows/`:

**`ci.yml`** — Every pull request (workflow must have `name: CI` — Phase 6 required status checks are configured as `CI / lint`, `CI / typecheck`, etc.; if the workflow name differs, branch protection silently stops working):
```yaml
name: CI
on: pull_request
jobs:
  lint:       npm ci → ESLint — fail on any error
  typecheck:  npm ci → tsc --noEmit (strict: true)
  test-unit:  npm ci → Vitest
  test-e2e:
    - npm ci
    - Sanitize branch name for Neon: `BRANCH_NAME=$(echo "$GITHUB_REF_NAME" | tr '/' '-' | cut -c1-63)` (branch names cannot contain `/` or exceed 63 chars)
    - **Create Neon DB branch using the official `neondatabase/create-branch-action`** — do NOT use raw `curl` against the Neon API; the action handles response parsing and exposes typed outputs:
      ```yaml
      - uses: neondatabase/create-branch-action@v5
        id: neon-branch
        with:
          project_id: ${{ secrets.NEON_PROJECT_ID }}
          branch_name: ${{ env.BRANCH_NAME }}
          api_key: ${{ secrets.NEON_API_KEY }}
      ```
      This action outputs `db_url` (unpooled direct connection) and `db_url_with_pooler` (pooled). Use `steps.neon-branch.outputs.db_url` for `DATABASE_URL_UNPOOLED` (drizzle-kit migrate requires a direct connection) and `steps.neon-branch.outputs.db_url_with_pooler` for `DATABASE_URL` (app queries).
    - **Override DATABASE_URL and DATABASE_URL_UNPOOLED** with the Neon branch connection strings from the action outputs, added to the step `env:` block:
      ```yaml
      env:
        DATABASE_URL: ${{ steps.neon-branch.outputs.db_url_with_pooler }}
        DATABASE_URL_UNPOOLED: ${{ steps.neon-branch.outputs.db_url }}
      ```
      ⚠ The GitHub Secret `DATABASE_URL` is the production connection string — if it is not overridden here, `next start` connects to production, not the branch, and the entire isolation strategy is broken.
    - Run drizzle-kit migrate against the branch DB: `npm run db:migrate` (uses the `db:migrate` script defined in Phase 1 step 17, which runs `drizzle-kit migrate`; always use the npm script rather than calling `drizzle-kit` directly since it may not be in PATH)
    - Inject remaining app env vars from GitHub Secrets: `AUTH_SECRET`, `RESEND_API_KEY`, `AUTH_RESEND_KEY`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`, `NEXT_PUBLIC_POSTHOG_KEY`, `NEXT_PUBLIC_POSTHOG_HOST`, `NEXT_PUBLIC_SENTRY_DSN`, `SENTRY_AUTH_TOKEN`, and any external API keys from Phase 0. (`NEXT_PUBLIC_POSTHOG_HOST` is a build-time variable embedded by `next build` — if omitted, PostHog silently sends events to the wrong host or not at all.)
    - **Override AUTH_URL to `http://localhost:3000`** — the GitHub Secret contains the production URL; Auth.js uses AUTH_URL to build callback URLs and validate CSRF tokens. With the production URL active, auth redirects and sign-in fail against localhost.
    - npx playwright install --with-deps chromium (browser binaries must be installed on the CI runner)
    - `npm run build` — do NOT use bare `next build`; `next` is not in global PATH on CI runners, only in `node_modules/.bin/`. The npm script resolves it correctly. Note: Next.js does not connect to the database at build time in the App Router model; DB calls only happen inside request handlers (Server Components, Server Actions, Route Handlers). Ensure no top-level module-scope DB calls exist — `tsc` and the Phase 3 review should catch these.
    - `npm run start &` — again use the npm script, not bare `next start &`
    - npx wait-on http://localhost:3000 (use `npx wait-on` — `wait-on` is not a global binary on CI runners; `npx` resolves it without a prior install)
    - Run Playwright tests against localhost:3000
    - **Delete Neon branch** using `neondatabase/delete-branch-action` (**must use `if: always()` on this step** — without it, a Playwright test failure skips the cleanup step, leaving an orphaned Neon branch; every failed PR run accumulates a branch that counts against the 0.5GB free tier storage limit and eventually breaks all future branch creation):
      ```yaml
      - uses: neondatabase/delete-branch-action@v3
        if: always()
        with:
          project_id: ${{ secrets.NEON_PROJECT_ID }}
          branch_id: ${{ steps.neon-branch.outputs.branch_id }}
          api_key: ${{ secrets.NEON_API_KEY }}
      ```
      `steps.neon-branch.outputs.branch_id` is the branch identifier returned by `create-branch-action` — this is how the delete action knows which branch to remove.
  audit:      npm audit --audit-level=high
```

> **Why Neon branches**: Neon allows creating instant copy-on-write DB branches per PR — free, isolated, identical schema. This prevents E2E tests from touching production data.

**`deploy-preview.yml`** — Pull requests:
```yaml
on: pull_request
- npm ci
- npm install -g vercel  (vercel is not pre-installed on CI runners)
- Inject Vercel secrets as env vars: VERCEL_TOKEN, VERCEL_ORG_ID, VERCEL_PROJECT_ID
  (the CLI reads VERCEL_ORG_ID and VERCEL_PROJECT_ID from env; without them it cannot identify which project to deploy)
- vercel deploy --token=$VERCEL_TOKEN
  ⚠ Do NOT use `vercel --prod` here — that deploys to the production alias. `vercel deploy` (without --prod) creates a preview URL only.
- Capture the preview URL and comment it on the PR. `vercel deploy` prints the URL as the last line of stdout — capture it and post a PR comment:
  ```yaml
  - name: Deploy preview
    id: vercel-deploy
    run: |
      PREVIEW_URL=$(vercel deploy --token=$VERCEL_TOKEN)
      echo "url=$PREVIEW_URL" >> $GITHUB_OUTPUT
    env:
      VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
      VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
      VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
  - name: Comment preview URL on PR
    run: |
      gh pr comment ${{ github.event.pull_request.number }} \
        --body "🚀 Preview deployed: ${{ steps.vercel-deploy.outputs.url }}"
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```
  > **Permissions required** — add a `permissions:` block to `deploy-preview.yml`:
  > ```yaml
  > permissions:
  >   contents: read
  >   pull-requests: write
  > ```
  > Without `pull-requests: write`, the `gh pr comment` step fails with a 403. `GITHUB_TOKEN` is a built-in GitHub Actions token — it does NOT need to be added as a repository secret, but it DOES need to be explicitly mapped in the step's `env:` block.
```

**`deploy-production.yml`** — Push to main:
```yaml
on:
  push:
    branches: [main]
- npm ci
- Run full CI suite (lint, typecheck, test-unit, audit — skip E2E to save time)
- npm install -g vercel
- Run `npm run db:migrate` against production DB (using DATABASE_URL_UNPOOLED secret) **BEFORE deploying**
  This uses the `db:migrate` npm script (Phase 1 step 17) which runs `drizzle-kit migrate`. Use `DATABASE_URL_UNPOOLED` from GitHub Secrets as the env var.
  ⚠ **GitHub Actions secrets are not automatically available as env vars** — they must be explicitly mapped in the workflow step's `env:` block: `DATABASE_URL_UNPOOLED: ${{ secrets.DATABASE_URL_UNPOOLED }}`. Without this, the migration step runs with `DATABASE_URL_UNPOOLED` undefined and silently fails or uses the wrong database.
  ⚠ Migration MUST run before `vercel --prod`. If the order is reversed, new code goes live against the old
  schema and every request that touches a new column/table crashes. If migration fails, abort — do not deploy.
- vercel --prod
  ⚠ **Inject Vercel secrets as explicit env vars** — same requirement as `deploy-preview.yml`: `VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}`, `VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}`, `VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}` must be in the step's `env:` block. The Vercel CLI reads these from the environment; without the mapping, `vercel --prod` cannot identify which project to deploy.
- Derive production URL: read AUTH_URL GitHub Actions secret (which equals https://<app>.vercel.app)
  and export it as PLAYWRIGHT_BASE_URL for the smoke test step.
  (AUTH_URL is added as a GitHub Actions secret in Phase 6 step 3; reuse it rather than duplicating a new secret)
  ⚠ **GitHub Actions secrets are not automatically available as shell variables** — `AUTH_URL` must be explicitly mapped in the step's `env:` block: `AUTH_URL: ${{ secrets.AUTH_URL }}`. Without this, `$AUTH_URL` evaluates to empty in both the Playwright step and the Lighthouse step — Playwright tests fail with connection refused and Lighthouse fails to collect from an empty URL.
- npx playwright install --with-deps chromium
- Run Playwright smoke tests — playwright.config.ts picks up PLAYWRIGHT_BASE_URL automatically
- Run Lighthouse CI: `LHCI_URL=$AUTH_URL npx lhci autorun` (lighthouserc.cjs reads LHCI_URL via process.env)
  Fail the workflow if any Lighthouse score drops below 90 (`minScore: 0.9` in lighthouserc.cjs)
- Create GitHub issue on failure using `if: failure()` to catch any step failure and post a tracking issue:
  ```yaml
  - name: Create issue on deployment failure
    if: failure()
    run: |
      gh issue create \
        --title "Production deploy failed $(date +%Y-%m-%d)" \
        --body "The deploy-production.yml workflow failed. Check the Actions run: ${{ github.server_url }}/${{ github.repository }}/actions/run/${{ github.run_id }}" \
        --label "deployment-failure"
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```
  > **Permissions required** — add a `permissions:` block to `deploy-production.yml`:
  > ```yaml
  > permissions:
  >   issues: write
  > ```
  > Without it, `gh issue create` fails with a 403. `GITHUB_TOKEN` must be explicitly mapped in the step's `env:` block.
  > ⚠ Do NOT auto-rollback — the DB migration already ran; rolling back code against a migrated schema causes further failures; fix forward.
```

**`security-scan.yml`** — Every Monday 09:00 UTC:
```yaml
on:
  schedule:
    - cron: '0 9 * * 1'
- npm ci
- npm audit --json → parse findings
- If high/critical: run `npm audit fix`, then create a PR with the changes:
  ```yaml
  - name: Run npm audit fix and open PR
    run: |
      npm audit fix
      # If nothing changed, exit silently
      git diff --quiet && exit 0
      git config user.name "github-actions[bot]"
      git config user.email "github-actions[bot]@users.noreply.github.com"
      BRANCH="security/audit-fix-$(date +%Y%m%d)"
      git checkout -b "$BRANCH"
      git add package.json package-lock.json
      git commit -m "fix(deps): npm audit fix $(date +%Y-%m-%d)"
      git push origin "$BRANCH"
      gh pr create \
        --title "fix(deps): npm audit fix $(date +%Y-%m-%d)" \
        --body "Automated security fix from security-scan.yml" \
        --base main \
        --head "$BRANCH"
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```
  > **`GITHUB_TOKEN` must be mapped** — it is a special GitHub Actions token, but steps still need it in their `env:` block to allow the `gh` CLI to authenticate. The push also requires the workflow to have `contents: write` and `pull-requests: write` permissions; add a `permissions:` block to the workflow:
  > ```yaml
  > permissions:
  >   contents: write
  >   pull-requests: write
  > ```
- Semgrep SAST scan using the official GitHub Action:
    uses: semgrep/semgrep-action@v1
    with: { config: "p/owasp-top-ten p/nodejs" }
  → if findings: create GitHub issue with the Semgrep report. The `semgrep/semgrep-action` action sets the exit code to non-zero on findings; use a separate step with `if: failure()` to create the issue:
  ```yaml
  - name: Create issue for Semgrep findings
    if: failure()
    run: |
      gh issue create \
        --title "Security: Semgrep findings $(date +%Y-%m-%d)" \
        --body "Semgrep SAST scan found issues. Run \`semgrep --config p/owasp-top-ten p/nodejs\` locally to see details." \
        --label "security"
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```
  > **Permissions required** — `security-scan.yml` needs `issues: write` added to its `permissions:` block (alongside `contents: write` and `pull-requests: write` already required for the npm audit PR step):
  > ```yaml
  > permissions:
  >   contents: write
  >   pull-requests: write
  >   issues: write
  > ```
  (Semgrep reports only; it cannot write fixes automatically)
```

**`tech-debt.yml`** — 1st of every month:
```yaml
on:
  schedule:
    - cron: '0 0 1 * *'
- npm ci  (required — ESLint and Vitest coverage both need node_modules)
- npm outdated → report
- ESLint complexity rules → flag files with cyclomatic complexity > 10
- Vitest coverage → flag modules below 70%
- Create GitHub issue labeled "tech-debt" with the full report. Capture all output to a variable and post using `gh issue create`:
  ```yaml
  - name: Run tech-debt checks and create issue
    run: |
      OUTDATED=$(npm outdated --json 2>/dev/null || true)
      COVERAGE=$(npm run test:coverage -- --reporter=json 2>/dev/null | tail -5 || true)
      gh issue create \
        --title "Tech Debt Report $(date +%Y-%m)" \
        --body "$(printf '## npm outdated\n```\n%s\n```\n## Coverage\n```\n%s\n```' "$OUTDATED" "$COVERAGE")" \
        --label "tech-debt"
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```
  > **Permissions required** — `tech-debt.yml` needs a `permissions:` block with `issues: write`:
  > ```yaml
  > permissions:
  >   issues: write
  > ```
  > Without it, `gh issue create` fails with a 403. `GITHUB_TOKEN` must be explicitly mapped in the step's `env:` block.
```

**`.github/dependabot.yml`** — Weekly dependency updates:
```yaml
updates:
  - package-ecosystem: npm
    directory: "/"
    schedule:
      interval: weekly
    open-pull-requests-limit: 5
  - package-ecosystem: github-actions
    directory: "/"
    schedule:
      interval: weekly
    open-pull-requests-limit: 3
# Auto-merge is handled by GitHub's auto-merge + CI gate
```

---

### Phase 8 — Deployment

1. Create Neon project via **Neon MCP** → get `DATABASE_URL` (pooled), `DATABASE_URL_UNPOOLED` (direct), and `NEON_PROJECT_ID`; immediately add **all three** as GitHub Actions secrets via GitHub MCP:
   - `DATABASE_URL` — needed by `next build` (app references it at build time in some configurations)
   - `DATABASE_URL_UNPOOLED` — used by `drizzle-kit migrate` in the deploy workflow
   - `NEON_PROJECT_ID` — required by the CI E2E job to create/delete per-PR Neon branches (`POST /projects/{project_id}/branches`)
   All three were deferred from Phase 6 because the Neon project did not exist yet.
2. **Run initial DB migration immediately** using the `DATABASE_URL_UNPOOLED` value just obtained:
   ```
   DATABASE_URL_UNPOOLED="<value>" npm run db:migrate
   ```
   Use `npm run db:migrate` (not bare `npx drizzle-kit migrate`) for consistency — the `db:migrate` npm script (Phase 1 step 17) resolves `drizzle-kit` from `node_modules`. The value is passed as an inline env var and never written to any file. `drizzle.config.ts` calls `dotenv` to load `.env.local`, but dotenv does not override existing env vars, so the inline value takes precedence. This seeds the DB schema **before** the first deploy — without this step, the app crashes on first request because tables don’t exist.
3. Create Vercel project via **Vercel MCP** → get `VERCEL_ORG_ID` and `VERCEL_PROJECT_ID`
4. Add `VERCEL_ORG_ID` and `VERCEL_PROJECT_ID` as GitHub Actions secrets via GitHub MCP (deferred from Phase 6 until project IDs were known)
5. Run `vercel link --yes --project=<project-name>` in the project root, using the project name returned by the Vercel MCP create step — required so that `vercel --prod` CLI commands target the correct project. The `--project` flag is required; omitting it causes `vercel link` to prompt interactively or, if multiple projects exist, potentially link to the wrong one.
6a. Generate `AUTH_SECRET`: run `openssl rand -base64 32` in the terminal and capture the output. Then immediately add it as a GitHub Actions secret via GitHub MCP (deferred from Phase 6 — it didn't exist yet).
6b. Set ALL environment variables in Vercel via MCP (never written to any local file):
   - `DATABASE_URL` + `DATABASE_URL_UNPOOLED`
   - `AUTH_SECRET` (the value generated in step 6a above)
   - `AUTH_URL=https://<project-name>.vercel.app`
   - `RESEND_API_KEY` + `AUTH_RESEND_KEY`
   - `UPSTASH_REDIS_REST_URL` + `UPSTASH_REDIS_REST_TOKEN`
   - `NEXT_PUBLIC_SENTRY_DSN` + `SENTRY_AUTH_TOKEN` (both required — DSN enables capture; auth token enables source map upload so stack traces are readable)
   - `NEXT_PUBLIC_POSTHOG_KEY` + `NEXT_PUBLIC_POSTHOG_HOST` (both required — `POSTHOG_HOST` is a build-time variable; omitting it causes PostHog to send events to the wrong endpoint)
   - Any external API keys collected in Phase 0
7. **Deploy first (without OAuth)**: `vercel --prod` — This gives us the production URL needed to register OAuth apps
8. **OAuth app registration** (the one unavoidable ordering step):
   - Agent outputs the exact callback URLs: `https://<app>.vercel.app/api/auth/callback/github`, `/callback/google` etc.
   - User registers OAuth apps on each provider using these URLs (GitHub: Settings → Developer → OAuth Apps; Google: console.cloud.google.com)
   - User pastes `CLIENT_ID` and `CLIENT_SECRET` back
   - Agent sets them in Vercel via MCP: `AUTH_GITHUB_ID`, `AUTH_GITHUB_SECRET`
9. **Redeploy** with OAuth credentials: `vercel --prod`
10. **Merge `develop` → `main` via PR** — This is the CI/CD handoff step. Open a PR from `develop` to `main` via GitHub MCP. Because the workflow files now exist in the PR branch, GitHub Actions picks them up and runs `ci.yml` against the PR. Once CI passes, merge the PR. This is the first time `deploy-production.yml` runs from a push to `main` — from this point forward, all changes flow through the PR process and CI/CD is fully active. **Without this step, the workflow files from Phase 7 stay on `develop` forever and `deploy-production.yml` never triggers on any future `main` push.**
11. All subsequent migrations run automatically via the `deploy-production.yml` CI job on every push to `main` (uses `DATABASE_URL_UNPOOLED` GitHub Actions secret). The one-time inline migration in step 2 is the only manual migration needed.
12. Add `<Analytics />` (from `@vercel/analytics`) and `<SpeedInsights />` (from `@vercel/speed-insights`) to `src/app/layout.tsx` if not already added in Phase 2
13. Verify production URL returns HTTP 200

> **On OAuth ordering**: This is unavoidable. You cannot register a GitHub/Google OAuth app without the redirect URI, and the URI doesn't exist until the first deploy. The correct flow is always: deploy first (auth will fail) → register OAuth apps → set credentials → redeploy (auth works). This costs one extra deploy but cannot be avoided.

---

### Phase 9 — Verification & Smoke Testing

Agent runs these against the live production URL and fixes any failures before completing:

1. HTTP health check → 200 OK
2. Playwright smoke tests vs production URL:
   - [ ] Homepage renders, zero console errors
   - [ ] Sign-in flow: if using **OAuth**, automate with test credentials; if using **magic link only**, magic links arrive in a real inbox and cannot be intercepted in production automation — instead, pre-seed a test user via Neon MCP and inject a forced session cookie so the core feature test can proceed without a real login
   - [ ] Core feature (from GOAL) works end-to-end with a test account
   - [ ] Sign-out clears session correctly
3. Lighthouse CI:
   - `@lhci/cli` is already installed (added in Phase 1 step 11 as a devDependency)
   - `lighthouserc.cjs` is already created and committed (Phase 1 step 20) — do NOT recreate it here; the file is already in the repo and `deploy-production.yml` already depends on it
   - Run: `LHCI_URL=https://<app>.vercel.app npx lhci autorun` (`lhci` auto-discovers `lighthouserc.cjs` in the project root)
   - Targets: Performance ≥ 90, Accessibility ≥ 90, Best Practices ≥ 90, SEO ≥ 90
4. Security headers audit → minimum A rating on `securityheaders.com`
5. SSL certificate valid, HTTPS redirect enforced

**If any check fails → agent identifies root cause → fixes → redeploys → re-runs check. Does not advance until all pass.**

> **On smoke test failures**: Do NOT run `vercel rollback`. The DB migration already ran — rolling back code to the previous version leaves old code against the new schema, which causes its own cascade of failures. Fix forward: identify the root cause, push a fix via a new commit, let `deploy-production.yml` re-run end-to-end.

---

### Phase 10 — Documentation & Handoff

Generated files, committed to `main`:

- **`README.md`** — App description, architecture overview, local dev setup (3 commands: `cp .env.example .env.local` → fill in values → `npm run db:migrate` → `npm run dev`), env vars reference
  > Use `db:migrate` (not `db:push`) to stay consistent with the CI/CD flow. `db:push` bypasses migration files and creates divergence between local and CI database states.
- **`ARCHITECTURE.md`** — Stack choices + rationale, DB schema ERD (text), action/API reference, key decisions
- **`SECURITY.md`** — Security controls, how to report vulnerabilities, auth model
- **`.code-reviews/round-1-bugs.md`** — Issues found + fixes applied
- **`.code-reviews/round-2-security.md`** — Security audit report
- **`.code-reviews/round-3-performance.md`** — Performance audit report

Git tag `v1.0.0` created. **Production URL delivered to user. Pipeline complete.**

---

## External API Key Handling

Some ideas require external data APIs. This flow runs in Phase 0, autonomously:

```
Agent detects external data dependency in IDEA/GOAL
            │
            ▼
Does a free public API exist for this data type?
            │
     ┌──────┴──────┐
    YES             NO
     │               │
Use free API     Is there a freemium tier?
(zero prompts)        │
                ┌─────┴──────────┐
               YES                NO
                │                  │
       Ask user ONCE:         Note in ARCHITECTURE.md
       "Would you like         as limitation.
        to add [Service]       Use mock/stub data for dev.
        for better [data]?     Flag for user to resolve.
        (yes/no)"
                │
           User says YES
                │
       "Please paste your
        [Service] API key:"
                │
       Key saved directly to
       Vercel env vars via MCP.
       Never written to any file.
```

### Free API Registry (baked into agent instructions)

| Domain | Free Source | Notes |
|---|---|---|
| Fantasy Premier League | `https://fantasy.premierleague.com/api/` (official) | No key needed |
| US stock prices | Yahoo Finance (unofficial) | No key; rate limit applies |
| US options data | No reliable free source → prompt for Polygon.io | Free: 5 req/min |
| Weather | Open-Meteo (`api.open-meteo.com`) | No key needed |
| Geocoding | Nominatim (OpenStreetMap) | No key; 1 req/sec limit |
| Currency / FX rates | Frankfurter (`api.frankfurter.app`) | No key needed |
| Crypto prices | CoinGecko API (`api.coingecko.com`) | No key (rate limited) |
| News headlines | ~~NewsAPI~~ → **GNews** (`gnews.io`) free tier | NewsAPI free tier is localhost-only; GNews works in production |
| Football match scores | api-football.com | Free: 100 req/day |
| Maps | Leaflet.js + OpenStreetMap tiles | No key needed |

> **⚠ Warning on NewsAPI**: The NewsAPI free plan only works from `localhost`. It returns 426 errors on any deployed domain. GNews or The Guardian API are valid production-safe free alternatives.

---

## MCP Server Integration Layer

### MCPs Used

| MCP Server | Purpose | Key Actions |
|---|---|---|
| **GitHub MCP** | Repository automation | Create repo, push, branch protection, add secrets, create issues, open PRs |
| **Vercel MCP** | Deployment automation | Create project, set env vars, trigger deploys, get project IDs |
| **Neon MCP** | Database automation | Create project, get connection strings, create/delete DB branches |

### MCP Configuration (`mcp.json`)

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

> **Note on Vercel MCP package name**: The package `@vercel/mcp-server` is listed here but verify the exact current package name at `npmjs.com/org/vercel` before setup — MCP package names are still stabilising. The alternative is using the Vercel CLI (`npm i -g vercel`) and calling it directly from agent terminal commands, which is more stable.

All three services have free tiers that cover hobby use indefinitely:
- **GitHub** — Free private repos, Actions minutes included
- **Vercel** — Hobby tier: unlimited projects, 100GB bandwidth/month
- **Neon** — 0.5GB storage, never pauses, **1 project on free tier** (upgrade to Launch ~$19/mo for multiple projects; branches within a single project are unlimited)

---

## Code Review Framework

### The 3-Review Guarantee

No code ships without passing all three reviews. Each review results in actual fixes.

```
Phase 3: Bug & Logic    → Fix TypeScript errors, edge cases, null safety
          ↓
Phase 4: Security       → Fix OWASP violations, misconfigurations
          ↓
Phase 5: Performance    → Fix Web Vitals, N+1 queries, bundle issues
          ↓
Git finalised → CI/CD → Deploy → Verify → Done ✓
```

Each review saves a structured report to `.code-reviews/` (severity, root cause, fix applied, confirmation).

### Context Window Management During Reviews

Large apps can exceed the agent's working context. The review strategy:

1. **Review by domain layer**, not by file: DB schema → auth layer → server actions → API routes → UI components → pages
2. **Use search tools** to locate specific patterns across files (e.g., grep for `dangerouslySetInnerHTML`, `raw(`, unhandled `.then(`)
3. **Commit after each review round** so progress is preserved even if the agent context resets

---

## CI/CD & Automation

### Pull Request Flow

```
push to develop / feature branch
  → PR opened
    → CI runs:
        lint + typecheck + test-unit + npm audit
        Neon branch created → migrations run → next build+start → Playwright E2E → Neon branch deleted
    → Vercel preview URL deployed
    → Preview URL commented on PR
    → All checks pass → auto-merge to main
      → Production deploy triggered
        → drizzle-kit migrate runs against production
          → Playwright smoke tests run against production URL
            → Pass → deploy confirmed ✓
            → Fail → GitHub issue created (no auto-rollback — the DB migration already ran;
                      rolling back code against a migrated schema causes its own failures;
                      investigate and fix forward via a new push)
```

### Quality Gates (block merge if any fail)

- `tsc --noEmit` (strict: true): zero errors
- ESLint: zero errors
- Vitest: all unit tests pass
- Playwright: all E2E tests pass (against Neon branch DB)
- `npm audit`: zero high/critical CVEs

---

## Tech Debt & Maintenance

Self-healing automation runs permanently after initial deploy.

| Cadence | What Runs | Outcome |
|---|---|---|
| Every deploy | Bundle size baseline check | Fail build if bundle grows >20% |
| Every deploy | Lighthouse CI | Run `npx lhci autorun` in `deploy-production.yml` after smoke tests pass; fail deploy if any score drops below 90 (set as `minScore: 0.9` in `lighthouserc.cjs`) |
| Weekly | `npm audit` | Auto-PR with `npm audit fix` changes if fixable; GitHub issue if not |
| Weekly | Semgrep SAST scan | GitHub issue with report (Semgrep cannot auto-fix) |
| Monthly | `npm outdated` + complexity check | GitHub issue labeled `tech-debt` |
| Monthly | Test coverage report | Flag modules below 70% |
| Quarterly | Full Phase 4 OWASP re-audit | Re-run complete security checklist |

**Auto-merge applies to dependency update PRs when CI passes.** You are only notified on failures or when manual action is required.

---

## Minimum Unavoidable User Inputs

| Input | When | Why Unavoidable |
|---|---|---|
| **The idea prompt** | Once, at start | The system can't know your idea |
| **GitHub PAT** | One-time setup | API auth for repo creation and Actions secrets |
| **Vercel token** | One-time setup | API auth for project creation and deployment |
| **Neon API key** | One-time setup | API auth for DB provisioning and branching |
| **Resend API key** | One-time setup (if using magic links) | Email sending for Auth.js magic links |
| **Upstash credentials** | One-time setup | Redis for rate limiting |
| **OAuth app setup** | Once per project using OAuth | Can't automate OAuth app registration on GitHub/Google |
| **Paid API key (optional)** | Once per project, only if user opts in | Only when no free alternative exists |

> **Note on OAuth**: This requires ~2 minutes per provider. The agent outputs the exact URLs to paste. It cannot be automated because GitHub and Google OAuth apps are registered through their respective browser-based developer consoles.

---

## One-Time Setup Guide

Do this once. Works for every future project.

### Step 1 — Get Service Tokens (~20 minutes)

**GitHub PAT**
- github.com → Settings → Developer settings → Personal access tokens → Fine-grained
- Required permissions: `Contents: Write`, `Metadata: Read`, `Pull requests: Write`, `Administration: Write`, **`Workflows: Write`** (needed to create `.github/workflows/` files), **`Secrets: Write`** (needed to add GitHub Actions secrets via GitHub MCP in Phase 6 step 3 and Phase 8 — without this, all `addSecret` MCP calls fail with 403)

**Vercel CLI**
- `npm install -g vercel` (one-time; required for `vercel link` and `vercel --prod` commands in Phase 8)
- Log in: `vercel login`

**Vercel Token**
- vercel.com → Account Settings → Tokens → Create token (full account scope)

**Neon API Key**
- console.neon.tech → Settings → API Keys → Create key

**Resend API Key** (for magic link auth)
- resend.com → Sign up (free) → API Keys → Create key
- Add and verify your sending domain, or use the Resend sandbox for testing

**Upstash Redis** (for rate limiting)
- upstash.com → Sign up (free) → Create database → Copy REST URL and REST Token

**Sentry** (for error tracking)
- sentry.io → Sign up (free) → Create project → Select "Next.js" → Copy the DSN (`NEXT_PUBLIC_SENTRY_DSN`)
- Settings → Auth Tokens → Create token with `project:write` scope → copy as `SENTRY_AUTH_TOKEN`
- Free tier: 5,000 errors/month

**PostHog** (for analytics)
- posthog.com → Sign up (free) → Create project → Copy the Project API Key (`NEXT_PUBLIC_POSTHOG_KEY`)
- Free tier: 1 million events/month

### Step 2 — Configure MCP Servers in VS Code

- `Cmd+Shift+P` → "MCP: Open User Configuration"
- Paste the JSON block from the MCP section above with your real tokens
- Restart VS Code

### Step 3 — Verify MCPs Work

In Copilot agent mode:
- "List my GitHub repositories" → returns your repos ✓
- "List my Vercel projects" → returns your projects ✓
- "List my Neon projects" → returns your databases ✓

Setup is permanently done.

### Step 4 — Run the Pipeline

Open an empty folder in VS Code and send:

```
IDEA:    Your idea here
GOAL:    What users should be able to do
EXTRA:   Any specific requirements (optional)
```

---

## Implementation Roadmap

One-time work to build the pipeline system itself:

### Phase A — Write the Agent Instructions File (core deliverable, ~1 week)

The file: `one-shot-pipeline.prompt.md`

This encodes Knowledge Base A — the complete 10-phase pipeline as executable agent instructions. Quality of this file = quality of every generated app.

Structure:
```
├── System context (role, constraints, quality bar, non-negotiable stack)
├── Phase 0 instructions (analysis steps, decision tree logic)
├── Phase 1 instructions (exact install commands, drizzle.config.ts template, folder structure)
├── Phase 2 instructions (generation order, patterns per feature type)
│   ├── Auth.js + Drizzle adapter canonical setup
│   ├── Rate limiting canonical setup (Upstash)
│   ├── Server Action template (auth guard + Zod + DB)
│   └── Drizzle schema template (with Auth.js tables)
├── Phase 3 checklist (inline, every item actionable)
├── Phase 4 checklist (OWASP items, Next.js-specific caveats)
├── Phase 5 checklist (performance items, Lighthouse targets)
├── Phase 6 instructions (branch protection with reviews=0 for auto-merge)
├── Phase 7 workflow templates (ci.yml with Neon branch strategy, deploy workflows)
├── Phase 8 instructions (OAuth ordering, Vercel env setup, migration steps)
├── Phase 9 verification steps (smoke test scripts, fix-forward failure handling)
├── Phase 10 doc templates
├── External API registry (free sources per domain, with gotcha notes)
└── Context window strategy (domain-by-domain review approach)
```

### Phase B — MCP Setup (~30 minutes)

Complete the One-Time Setup Guide. Verify all five services respond correctly (GitHub, Vercel, Neon, Resend, Upstash).

### Phase C — Pilot: Simple App (~1 day)

Run the pipeline on a todo/notes app. Document every manual step. Each = a gap to fix in the instructions.

### Phase D — Pilot: Real Idea (~2-3 days)

Run on FPL companion app or options screener. Iterate instructions until zero manual interventions needed.

### Phase E — Pipeline v1.0.0

Declare ready when: prompt → live URL, zero manual steps (excluding the unavoidable OAuth registration and initial token setup). Tag `v1.0.0`.

---

## Known Constraints & Limitations

Honest list of things this pipeline cannot fully automate:

| Constraint | Why | Workaround |
|---|---|---|
| OAuth app registration | GitHub/Google require browser-based consent flows | ~2 min manual step per provider; agent outputs exact URLs |
| Vercel MCP package name instability | MCP packages are still maturing | Fall back to Vercel CLI in terminal commands |
| Semgrep cannot auto-fix | It's a scanner, not a code writer | Creates GitHub issues; agent can read and fix manually |
| `npm audit fix --force` may break code | Major version bumps can introduce breaking changes | Only auto-merge if `--force` not needed; flag rest as issues |
| Context window on very large apps | Agent can't hold 10k+ lines in context at once | Domain-by-domain review strategy; commit between phases |
| CSP is hard to get right | Strict CSP can break shadcn/ui, Sentry, PostHog | Start with report-only mode, tighten after smoke tests pass |
| Playwright in CI needs correct server start | `next start` must be ready before tests run | Use `wait-on` package in CI workflow |
| Neon free tier: 1 project only | Free tier allows a single project; second project creation fails | Upgrade to Launch plan (~$19/mo) or reuse the same project with multiple schemas |
| Vercel Hobby function timeout: 60s | Server actions or API routes with heavy queries, external API chains, or cold starts can time out | Keep server actions fast; offload heavy work to background jobs; upgrade to Pro for 300s limit |
| GitHub Actions free minutes: 2,000/month (private repos) | Each CI run (lint + typecheck + unit tests + E2E) uses ~5–10 min; security scans and monthly jobs add more | Switch repo to public (Actions minutes unlimited) or monitor usage; paid plan adds minutes |

---

## Decisions Log

| Decision | Choice | Rationale |
|---|---|---|
| Knowledge Base A | The pipeline system itself (agent instructions + this plan) | User intent: "everything needed to go from one sentence to a live website" |
| Hosting | Vercel | Best DX, edge functions, preview URLs, generous hobby tier |
| Database | Neon over Supabase | Supabase pauses free DBs after 1 week; Neon never pauses |
| ORM | Drizzle ORM | Type-safe, thin, excellent Neon integration |
| Auth | Auth.js v5 + `@auth/drizzle-adapter` | Free forever; needs adapter for DB session persistence |
| Email | Resend | Required for magic link auth; best free tier for Next.js |
| Rate limiting | Upstash Redis + `@upstash/ratelimit` | Only edge-compatible free option |
| MCP servers | GitHub + Vercel + Neon | All have free tiers and working MCP servers |
| Scale | Hobby / personal use | Simpler monitoring, free tiers sufficient |
| Domain | *.vercel.app subdomain acceptable | No custom domain automation needed initially |
| External APIs | Free-first, optional prompt for paid | Minimum prompts while avoiding unnecessary cost |
| API key storage | Vercel env vars via MCP | Keys never written to files; encrypted at Vercel |
| Maintenance PRs | Auto-created, auto-merged if CI passes | Fully autonomous maintenance |
| Pre-deploy review | Straight to production | Minimum input is the primary goal |
| Git init timing | End of Phase 1 (not Phase 6) | Need version history during code generation |
| E2E test isolation | Neon DB branching per PR | Free, instant, prevents production data corruption |
| OAuth deploy order | Deploy first → register OAuth → redeploy | Unavoidable — OAuth apps need the production URL |

---

## Issue Audit Log

Issues found and resolved during plan review (v1 → v2):

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 1 | Breaking | OAuth chicken-and-egg: can't register without prod URL | Documented two-deploy strategy in Phase 8 |
| 2 | Breaking | GitHub Actions missing Vercel secrets | Added secret setup step to Phase 6; updated Phase 7 workflow |
| 3 | Breaking | Playwright in CI had no server to test against | Added `next build && next start` + `wait-on` to CI workflow |
| 4 | Breaking | E2E tests had no DB isolation (would corrupt prod) | Added Neon branch strategy to Phase 7 CI workflow |
| 5 | Breaking | `AUTH_URL` env var missing | Added to `.env.example` and Phase 8 Vercel setup |
| 6 | Breaking | `@auth/drizzle-adapter` missing | Added to Phase 1 installs and Phase 2 DB schema generation |
| 7 | High | NewsAPI free tier is localhost-only (fails in production) | Replaced with GNews in Free API Registry, added warning |
| 8 | High | FPL API URL wrong (`api.fpl.pl`) | Corrected to `https://fantasy.premierleague.com/api/` |
| 9 | High | Semgrep described as auto-fixing code | Corrected: Semgrep reports only; create issue not PR |
| 10 | High | `exchangerate.host` requires registration | Replaced with `frankfurter.app` (truly free, no auth) |
| 11 | High | Contentlayer is deprecated | Replaced with `next-mdx-remote` |
| 12 | High | Rate limiting tool never specified | Added Upstash Redis + `@upstash/ratelimit` to fixed stack |
| 13 | High | `drizzle.config.ts` never mentioned | Added to Phase 1 scaffolding steps |
| 14 | High | Email provider missing for magic links | Added Resend to fixed stack, Phase 1 install, Phase 8 env |
| 15 | High | Git init placed after review phases (too late) | Moved git init to end of Phase 1 |
| 16 | High | `@vercel/mcp-adapter` is wrong package (builds MCPs, not connects) | Corrected to `@vercel/mcp-server`, added stability note |
| 17 | High | GitHub PAT missing `Workflows: Write` permission | Added to One-Time Setup Guide |
| 18 | High | DB migrations not run in CI before E2E tests | Added migration step to CI E2E job |
| 19 | Medium | SWR listed for mutations (not idiomatic in Next.js 15) | Changed to Server Actions + `useFormStatus` / `useOptimistic` |
| 20 | Medium | Branch protection blocked auto-merge (reviews not set to 0) | Added explicit `required reviews: 0` to Phase 6 |
| 21 | Medium | Rollback mechanism mentioned but not explained | Added `vercel rollback` to Phase 9 and CI failure flow |
| 22 | Medium | Context window limits not addressed | Added domain-by-domain review strategy to Phase 3 and review section |
| 23 | Medium | Neon preview DB branches not mentioned | Added to Phase 7 CI workflow with explanation |
| 24 | Medium | `posthog-node` missing for server-side analytics | Added to Phase 1 installs and tech stack table |
| 25 | Medium | Upstash not in tech stack table | Added to fixed stack with free tier info |
| 26 | Medium | Resend not in tech stack | Added to fixed stack |

**v2 → v3 Review (Third pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 27 | Breaking | `create-next-app` hangs on interactive prompts without CLI flags | Added `--typescript --tailwind --app --eslint --src-dir --import-alias --use-npm` flags to Phase 1 step 1 |
| 28 | Breaking | Neon free tier described as "unlimited projects" — free tier is 1 project only | Corrected in MCP section; added to Known Constraints |
| 29 | Breaking | `vitest.config.ts` never created — tests fail to resolve `@/` aliases and `jsdom` environment | Added `vitest.config.ts` generation as Phase 1 step 11 |
| 30 | Breaking | `playwright.config.ts` never created — Playwright has no base URL or browser config | Added `playwright.config.ts` generation as Phase 1 step 12 |
| 31 | Breaking | `src/middleware.ts` never generated in Phase 2 — `(protected)/` routes publicly accessible | Made `middleware.ts` an explicit Phase 2 step 6; added to Phase 1 folder structure |
| 32 | Breaking | `NEON_PROJECT_ID` missing from CI secrets — Neon branch API requires project ID | Added `NEON_PROJECT_ID` to Phase 6 secrets list |
| 33 | Breaking | Playwright smoke tests can't automate magic link sign-in — links arrive in real inbox | Documented workaround: pre-seed test user via Neon MCP + forced session cookie |
| 34 | Breaking | DB migrations in Phase 8 listed as local terminal command — contradicts "keys never written to files" | Clarified: migrations run via `deploy-production.yml` CI job using `DATABASE_URL_UNPOOLED` GitHub Actions secret |
| 35 | High | `@auth/core` installed separately — can conflict with `next-auth@beta` peer dependency | Removed from Phase 1 install; added explicit warning |
| 36 | High | `@vercel/analytics` and `@vercel/speed-insights` packages never installed | Added to Phase 1 step 10; Phase 2 layouts step; Phase 8 step 10 |
| 37 | High | Playwright browser binaries never installed (`npx playwright install`) | Added to Phase 1 step 9 and Phase 7 CI `test-e2e` job |
| 38 | High | CI E2E job missing all required app env vars — `next start` crashes without them | Added env var injection note to Phase 7 CI `test-e2e` job |
| 39 | High | `vercel link` step missing — `vercel --prod` CLI targets wrong/new project without it | Added `vercel link --yes` as Phase 8 step 4 |
| 40 | High | Vercel Hobby function timeout (60s) not documented | Added to Known Constraints |
| 41 | Medium | `next.config.ts` referenced in Phase 4 but `create-next-app` generates `next.config.mjs` | Corrected reference to `.mjs` in Phase 4 security headers check |
| 42 | Medium | Dependabot only covers `npm` — GitHub Actions versions go unmonitored | Added `github-actions` ecosystem to `.github/dependabot.yml` |
| 43 | Medium | OWASP A10 (SSRF) missing from Phase 4 security audit | Added A10 SSRF checks to Phase 4 |
| 44 | Medium | Neon branch names can't contain `/` — PR branches like `feature/x` cause API failure | Added branch name sanitization (`tr '/' '-'`) to Phase 7 CI E2E job |
| 45 | Medium | `VERCEL_ORG_ID`/`VERCEL_PROJECT_ID` as placeholders in Phase 6 cause silent CI failures | Removed from Phase 6; added only in Phase 8 after project IDs are known |
| 46 | Medium | Local dev "3 commands" in README not defined — agent would improvise | Defined explicitly: `cp .env.example .env.local` → `npm run db:push` → `npm run dev`; added `db:*` npm scripts to Phase 1 step 16 |

**v3 → v4 Review (Fourth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 47 | Breaking | `vercel` CLI never installed — `vercel link` and `vercel --prod` fail with "command not found" | Added `npm install -g vercel` + `vercel login` to One-Time Setup Guide; added `npm install -g vercel` to `deploy-production.yml` CI job |
| 48 | Breaking | Initial DB migration runs after first deploy — app crashes on first request (zero tables) | Added explicit initial migration step (inline env var, not written to file) as Phase 8 step 2, before first deploy |
| 49 | Breaking | `next-auth@beta` dist-tag stale by May 2026 (Auth.js v5 graduated to stable) | Changed to `next-auth@5` throughout Phase 1 |
| 50 | High | `@next/bundle-analyzer` used in Phase 5 checklist but never installed or scripted | Added to Phase 1 installs (step 11); added `analyze` npm script to Phase 1 step 17 |
| 51 | High | Semgrep in `security-scan.yml` has no installation method — CI runner doesn't have it | Replaced with `uses: semgrep/semgrep-action@v1` official GitHub Action with OWASP ruleset |
| 52 | High | Lighthouse CI in Phase 9 has no tooling, no config, no invocation — cannot run | Added `@lhci/cli` install, `lighthouserc.json` config template, and `lhci autorun` command to Phase 9 |
| 53 | High | `PLAYWRIGHT_BASE_URL` not set in `deploy-production.yml` — smoke tests hit `localhost:3000` (nothing there) | Added `PLAYWRIGHT_BASE_URL` env var set to production URL in `deploy-production.yml` |
| 54 | High | shadcn/ui init command wrong — `shadcn-ui` package was deprecated in mid-2024 | Corrected Phase 1 step 2 to `npx shadcn@latest init` |
| 55 | Medium | ESLint `complexity` rule never configured — `tech-debt.yml` complexity check produces no findings | Added `complexity: ["warn", { max: 10 }]` to ESLint config in Phase 1 step 15 |
| 56 | Medium | GitHub Actions free tier: 2,000 min/month for private repos — not documented | Added to Known Constraints |
| 57 | Medium | `wait-on` not installed — CI command fails without `npx` prefix | Changed to `npx wait-on` in Phase 7 CI E2E job |

**v4 → v5 Review (Fifth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 58 | Breaking | `lighthouserc.json` uses `$LHCI_URL` shell syntax — JSON does not interpolate env vars, so `lhci` tries to collect from the literal string `$LHCI_URL` | Switched to `lighthouserc.js` (JS config) using `process.env.LHCI_URL`; run command updated to `LHCI_URL=... npx lhci autorun` |
| 59 | Breaking | `npx shadcn@latest init` has interactive prompts and hangs — no `--defaults` flag was specified | Added `-d` flag: `npx shadcn@latest init -d` |
| 60 | High | `@sentry/nextjs` installed but never wired up — requires 3 config files + `withSentryConfig()` wrapper or it captures nothing | Added Sentry wiring as Phase 2 step 6 with all required files and `next.config.mjs` wrapper |
| 61 | High | `@next/bundle-analyzer` installed and scripted but `next.config.mjs` never wrapped with `withBundleAnalyzer()` — `ANALYZE=true` is silently ignored | Added `withBundleAnalyzer` wrapper to Phase 1 step 17 `next.config.mjs` config |
| 62 | High | App runtime secrets (`AUTH_SECRET`, `DATABASE_URL`, etc.) set in Vercel but never added as GitHub Actions secrets — CI `next build` fails without them | Added all app secrets to Phase 6 step 3 GitHub Secrets list with explanation |
| 63 | Medium | PostHog requires `<PHProvider>` wrapper in root layout — without it `posthog-js` is installed but no events are tracked | Added `<PHProvider>` to Phase 2 layouts step with explicit warning |
| 64 | Medium | `PLAYWRIGHT_BASE_URL` source undefined in `deploy-production.yml` — agent has no instruction on where this value comes from | Clarified: reuse the `AUTH_URL` GitHub Actions secret (already set in Phase 8); export it as `PLAYWRIGHT_BASE_URL` in the workflow step |

**v5 → v6 Review (Sixth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 65 | Breaking | CI E2E job injects `DATABASE_URL` from GitHub Secrets (production connection) but never overrides it with the Neon branch URL — app connects to production, defeating isolation entirely | Added explicit step: extract branch connection strings from Neon API response and export as `DATABASE_URL` / `DATABASE_URL_UNPOOLED`, overriding the secrets |
| 66 | Breaking | `deploy-production.yml` ran `vercel --prod` before `drizzle-kit migrate` — new code goes live against old schema, crashing all requests that touch new columns/tables | Swapped order: migration runs first; if it fails the deploy is aborted |
| 67 | High | `eslint-plugin-security` configured in ESLint but never installed — `lint` CI job fails immediately with "Cannot find module" | Merged into Phase 1 step 11 alongside `@next/bundle-analyzer` |
| 68 | High | `posthog-js` / `<PHProvider>` placed directly in the root Server Component layout — `window is not defined` crash on every page load | Added `src/app/providers.tsx` Client Component pattern (`'use client'` + `useEffect` + `posthog.init`) imported into the layout; explicit warning against calling `posthog.init()` in Server Components |
| 69 | High | `AUTH_URL` GitHub Secret (production URL) not overridden to `http://localhost:3000` in CI E2E job — Auth.js builds wrong callback URLs and CSRF validation fails against localhost | Added explicit `AUTH_URL=http://localhost:3000` override inside the `test-e2e` job environment |
| 70 | Medium | Maintenance table promised "Every deploy: Lighthouse CI" but `deploy-production.yml` had no `lhci autorun` step | Added `LHCI_URL=$AUTH_URL npx lhci autorun` step to `deploy-production.yml` after smoke tests; maintenance table updated to reference the workflow step |
| 71 | Medium | `deploy-preview.yml` calls `vercel deploy` without installing the CLI — CI runners do not have `vercel` pre-installed | Added `npm install -g vercel` as first step in `deploy-preview.yml` |

**v6 → v7 Review (Seventh pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 72 | Breaking | Phase 6 listed `DATABASE_URL` and `DATABASE_URL_UNPOOLED` as GitHub Secrets to add — but Neon doesn't exist until Phase 8; agent has no values to set and would either block or write blanks causing silent CI failures | Removed database secrets from Phase 6 step 3; added them to Phase 8 step 1 immediately after Neon project creation with explicit note they were deferred |
| 73 | High | `SENTRY_AUTH_TOKEN` completely absent from the plan — `withSentryConfig()` uses it to upload source maps at build time; without it every production error shows unreadable minified stack traces | Added to Phase 6 secrets list, Phase 8 Vercel env vars, `.env.example` (with comment), Phase 2 step 6 warning, and One-Time Setup Guide Step 1 |
| 74 | High | Phase 2 step 6 said "wrap `next.config.mjs` export with `withSentryConfig(...)`" without clarifying this requires *editing* the file created in Phase 1 — an agent may regenerate the file, silently dropping the `withBundleAnalyzer` wrapper | Clarified as an edit to the existing file; included explicit code showing both wrappers composed; added warning not to overwrite the bundle analyzer wrapper |
| 75 | High | No step anywhere in the plan creates a Sentry project or PostHog project — agent references `NEXT_PUBLIC_SENTRY_DSN` and `NEXT_PUBLIC_POSTHOG_KEY` throughout but has no instructions for obtaining them | Added Sentry and PostHog project creation steps to One-Time Setup Guide Step 1 |
| 76 | Medium | Phase 8 step 1 added only `DATABASE_URL_UNPOOLED` as GitHub Secret — `DATABASE_URL` (pooled) was missing; `next build` can reference it at build time causing build failures | Added `DATABASE_URL` alongside `DATABASE_URL_UNPOOLED` in Phase 8 step 1 |

**v7 → v8 Review (Eighth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 77 | Breaking | `AUTH_SECRET` was listed in Phase 6 GitHub Secrets as a value "known from One-Time Setup" — but it is generated per-project in Phase 8 step 6; no value exists at Phase 6 | Removed `AUTH_SECRET` from Phase 6 secrets list; added explicit Phase 8 step 6a (generate with `openssl rand -base64 32`, then add as GitHub Secret immediately) |
| 78 | Breaking | `RESEND_API_KEY`, `AUTH_RESEND_KEY`, `UPSTASH_*` described as "known from One-Time Setup" without making clear these are operator-level shared credentials reused across projects, not per-project values | Renamed category to "Operator-level reusable secrets" with explicit explanation they are the same values across all projects |
| 79 | High | README local dev setup instructed `npm run db:push` — inconsistent with CI/CD which uses `drizzle-kit migrate`; running `db:push` locally bypasses migration history and causes schema divergence between local and CI DB | Changed README instruction to `npm run db:migrate` with explanatory note |
| 80 | High | Phase 8 step 6 said `AUTH_SECRET` "(generated: `openssl rand -base64 32`)" without explicit sequencing — agent may try to set it before generating it, or skip the GitHub Secret step | Split into step 6a (generate + add as GitHub Secret) and step 6b (set in Vercel) with explicit ordering |
| 81 | High | CI E2E `next build` step had no note about DB connections at build time — if any module-scope DB call exists the build fails with a connection error against the production DB | Added explicit note: App Router model means DB calls only happen inside request handlers; confirmed `next build` does not connect to DB; flagged to verify no top-level module-scope DB calls |
| 82 | Medium | `AUTH_URL` never added as a GitHub Actions secret anywhere — `deploy-production.yml` reads it to derive `PLAYWRIGHT_BASE_URL` and `LHCI_URL` but GitHub Actions cannot read Vercel env vars directly | Added `AUTH_URL` to Phase 6 step 3 GitHub Secrets list with clear explanation |
| 83 | Medium | `lighthouserc.js` using `module.exports` fails in ESM context — `create-next-app` sets `"type": "module"` making plain `.js` files ES modules; `module.exports` throws `ReferenceError` | Renamed to `lighthouserc.cjs` throughout (Phase 9, deploy-production.yml); updated `lhci autorun` note to confirm `.cjs` is auto-discovered |

**v8 → v9 Review (Ninth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 84 | Breaking | CI/CD flow diagram said "Fail → GitHub issue created, vercel rollback executed" — directly contradicts Phase 7 deploy-production.yml "do NOT auto-rollback". Auto-rollback is also dangerous: the DB migration already ran; reverting code to a previous version leaves old code against the new schema, potentially causing its own cascade of failures | Updated CI/CD flow diagram to "no auto-rollback"; updated deploy-production.yml failure text to explain why rollback is unsafe; fix-forward is the correct strategy |
| 85 | Breaking | `drizzle.config.ts` never loads `.env.local` — drizzle-kit reads `.env` by default, not Next.js's `.env.local`. Running `npm run db:migrate` locally fails with `DATABASE_URL_UNPOOLED` undefined | Added explicit `dotenv` loading to `drizzle.config.ts` template (`config({ path: '.env.local' })`); added `dotenv` as devDependency in Phase 1 step 14 |
| 86 | High | Lighthouse threshold inconsistency — `lighthouserc.cjs` asserts `minScore: 0.9` (90%) but Phase 7 deploy-production.yml and the maintenance table both said "fail if score drops below 85". Config is ground truth; 85 was wrong | Updated all text references from 85 to 90 to match the `minScore: 0.9` assertion in the config |
| 87 | High | `@lhci/cli` installed in Phase 9 step 3 but the first production deploy happens in Phase 8 step 7, which triggers `deploy-production.yml` with `lhci autorun` — `@lhci/cli` is not in `package.json` yet at that point | Moved `@lhci/cli` install to Phase 1 step 11 alongside other devDependencies; Phase 9 updated to note it's already installed |
| 88 | Medium | `NEXT_PUBLIC_POSTHOG_HOST` not listed explicitly in CI E2E env injection — hidden under "etc."; it is a build-time `NEXT_PUBLIC_*` variable embedded by `next build`; if omitted PostHog silently sends to wrong host | Added `NEXT_PUBLIC_POSTHOG_HOST` explicitly to CI E2E env injection list, Phase 6 secrets list, and Phase 8 Vercel env vars with explanation |
| 89 | Medium | `vercel link --yes` without `--project` flag — if multiple Vercel projects exist the command may link to the wrong one or prompt interactively, breaking automated execution | Changed to `vercel link --yes --project=<project-name>` using the project name from the MCP create step |

**v9 → v10 Review (Tenth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 90 | Breaking | Phase 9 "On failures that require rollback" still instructed `vercel rollback` — directly contradicting issue #84's no-rollback fix applied to Phase 7 and the CI/CD flow diagram. The DB migration already ran; rolling back code against the migrated schema causes its own failures | Replaced Phase 9 rollback note with explicit no-rollback instruction; same rationale as #84; fix-forward via a new push is the correct approach |
| 91 | Breaking | Phase 7 creates `.github/workflows/` with no instruction on which branch to commit them to. If committed to `main`, `deploy-production.yml` triggers immediately — at that point `AUTH_SECRET`, `DATABASE_URL`, `VERCEL_ORG_ID`, and `VERCEL_PROJECT_ID` are not yet in GitHub Secrets (Phase 8 hasn't run), causing the workflow to fail with missing credentials | Added explicit branch note to Phase 7 header: all workflow files must be committed to `develop`, not `main`; first legitimate push to `main` happens only after Phase 8 is fully complete |
| 92 | High | GitHub PAT in One-Time Setup Guide missing `Secrets: Write` permission. Phase 6 step 3 and Phase 8 both add GitHub Actions secrets via the GitHub MCP (`PUT /repos/{owner}/{repo}/actions/secrets/{secret_name}`); this requires the `Secrets: Write` permission on fine-grained PATs. Without it, every `addSecret` MCP call fails with 403 | Added `Secrets: Write` to the required PAT permissions list with explanation |
| 93 | High | PostHog providers.tsx sets `capture_pageview: false` with comment "handled manually with usePathname" but no `PostHogPageView` component is ever instructed to be created anywhere in the plan — pageviews are never recorded | Added complete `PostHogPageView` component to Phase 2 step 12, using `usePathname` + `useSearchParams` + `usePostHog`, rendered inside `<Providers>` in a `<Suspense>` boundary |
| 94 | Medium | All CI jobs (lint, typecheck, test-unit, test-e2e) and both deploy workflows missing `npm ci` as their first step. An agent implementing these workflow descriptions literally generates jobs that fail immediately because `node_modules` doesn't exist on the fresh CI runner | Added `npm ci` as first step to all five jobs/workflows in Phase 7 descriptions |

**v10 → v11 Review (Eleventh pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 95 | Breaking | `deploy-preview.yml` described only as "Build and deploy to Vercel preview URL" / "Comment preview URL on PR" — no actual commands. An agent implementing this will likely write `vercel --prod` (which deploys to production) instead of `vercel deploy` (preview). Also missing `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`, `VERCEL_TOKEN` injection and no PR comment mechanism specified | Rewrote `deploy-preview.yml` with explicit `vercel deploy --token=$VERCEL_TOKEN`, prominent warning against `--prod`, secrets injection note, and PR comment via `gh pr comment` / `actions/github-script` |
| 96 | High | `security-scan.yml` and `tech-debt.yml` scheduled workflows had no `npm ci` step. `tech-debt.yml` runs ESLint and Vitest coverage — both require `node_modules`. `security-scan.yml` runs `npm audit fix` and opens a PR — requires a working npm install to resolve the dependency tree | Added `npm ci` as first step to both scheduled workflows with explanatory comments |
| 97 | High | `drizzle-kit migrate` invocation in `ci.yml` test-e2e and `deploy-production.yml` written as prose — not an actual shell command. `drizzle-kit` is not in PATH on CI runners; calling it directly fails. The correct command is `npm run db:migrate` (the npm script defined in Phase 1 step 17) | Changed both occurrences to `npm run db:migrate` with explanation that the npm script resolves the binary correctly |
| 98 | Medium | `vitest.config.ts` sets `globals: true` (enables `describe`, `it`, `expect` without imports) but `tsconfig.json` is never updated with `"types": ["vitest/globals"]`. TypeScript does not know about these globals, so `tsc --noEmit` in Phase 3 fails with "Cannot find name 'describe'" on every test file using globals | Added `"types": ["vitest/globals"]` to the `tsconfig.json` configuration step (Phase 1 step 19) |

**v11 → v12 Review (Twelfth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 99 | High | `audit` job defined in `ci.yml` and listed in Quality Gates as blocking merge, but NOT in Phase 6 step 1 required status checks (`lint`, `typecheck`, `test-unit`, `test-e2e`). A failing `npm audit` does not actually block PR merges — the gate is documented but not enforced by branch protection | Added `audit` to Phase 6 step 1 required status checks list with explanatory note |
| 100 | Medium | Implementation Roadmap Phase A structure still listed "├── Phase 9 verification steps (smoke test scripts, rollback command)" — contradicts the no-rollback policy established in issues #84 and #90, which removed all rollback instructions throughout the plan | Changed to "fix-forward failure handling" matching the no-rollback policy |
| 101 | High | Phase 1 step 19 `tsconfig.json` update specified only `"types": ["vitest/globals"]`. When TypeScript's `types` array is set, it restricts ambient type auto-discovery to ONLY the listed packages. Omitting `"node"` means `@types/node` is no longer auto-included, causing `tsc --noEmit` to fail with "Cannot find name 'process'" in server components, server actions, and route handlers — which use `process.env` throughout | Added `"node"` to the types array: `["vitest/globals", "node"]`; added explanation that both entries are required and that this creates (not replaces) the array since `create-next-app` does not set `types` by default |

**v12 → v13 Review (Thirteenth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 102 | Breaking | `ci.yml` E2E job description used bare commands `next build` and `next start &`. `next` is not a global binary on CI runners — it's installed in `node_modules/.bin/`. Calling it directly fails with "command not found". The npm scripts (`npm run build`, `npm run start &`) resolve the binary correctly | Changed to `npm run build` and `npm run start &` in the E2E job description; added explicit warning not to use bare `next` commands |
| 103 | High | Phase 6 step 1 required status checks were specified as bare job names (`lint`, `typecheck`, etc.). GitHub Actions reports status check names as `WorkflowName / JobName` — if `ci.yml` has `name: CI`, the checks are `CI / lint`, `CI / typecheck`, etc. Using bare names means branch protection matches nothing and all PRs merge regardless of CI result | Updated Phase 6 step 1 to use `CI / lint`, `CI / typecheck`, `CI / test-unit`, `CI / test-e2e`, `CI / audit`; added note to Phase 7 `ci.yml` header that the workflow must be named `CI` |
| 104 | Medium | Phase 8 step 2 used `npx drizzle-kit migrate` — inconsistent with issue #97's fix establishing `npm run db:migrate` as the standard. While `npx` works, consistency across all migration invocations reduces agent confusion | Changed to `npm run db:migrate` with explanation that dotenv won't override the inline env var, so the inline `DATABASE_URL_UNPOOLED` value correctly takes precedence |

**v13 → v14 Review (Fourteenth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 105 | Breaking | Phase 2 step 13 instructed generating `error.tsx` per route segment without the mandatory `'use client'` directive. Next.js App Router enforces this at build time — without it, `next build` fails with "Error component must be a client component". The `reset` prop is a function, which cannot be serialised across the server/client boundary | Added explicit `'use client'` requirement and build-time error description to Phase 2 step 13 |
| 106 | High | Phase 2 step 2 used bare `drizzle-kit generate` — same class of issue as #97/#104. `drizzle-kit` is not in global PATH; calling it directly fails. This is the command run during local code generation to create SQL migration files from the schema | Changed to `npm run db:generate` with the same PATH-in-PATH rationale |
| 107 | High | Phase 2 step 3 referenced `InferSelectModel` / `InferInsertModel` — deprecated named exports removed in drizzle-orm v0.29+. The current API is `typeof table.$inferSelect` / `typeof table.$inferInsert`. An agent following the old API generates type errors against any recent drizzle-orm version | Updated to use `typeof table.$inferSelect` / `typeof table.$inferInsert` with deprecation note |

**v14 → v15 Review (Fifteenth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 108 | Breaking | `NEON_PROJECT_ID` listed as a Phase 6 step 3 secret alongside `NEON_API_KEY`. Unlike `NEON_API_KEY` (an account-level key from One-Time Setup), `NEON_PROJECT_ID` is the project-specific Neon project ID — it does not exist until Phase 8 step 1 when the Neon project is created. An agent following the plan would attempt to set it in Phase 6 without a valid value; per the plan's own rule "do not set placeholder values", this either blocks the agent or produces a silent failure when CI later tries to create Neon branches | Moved `NEON_PROJECT_ID` out of the main Phase 6 secrets list and into the "Secrets deferred" blockquote; updated Phase 8 step 1 to add `NEON_PROJECT_ID` alongside `DATABASE_URL` and `DATABASE_URL_UNPOOLED` as a third secret to set immediately after Neon project creation |
| 109 | High | Phase 2 step 12 described rendering `PostHogPageView` as "inside `<Providers>` (or inside a `<Suspense>` boundary)". The "or" phrasing implies these are alternative placements. They are not: `usePostHog()` requires `<Providers>` context AND `useSearchParams()` requires `<Suspense>` simultaneously. An agent reading this could generate `<PostHogPageView />` inside `<Providers>` without a `<Suspense>` wrapper, causing Next.js to opt the entire root layout out of static rendering and log dev-mode warnings | Changed wording to "AND inside a `<Suspense>` boundary — both wrappers are simultaneously required" with explicit correct JSX structure showing `<Providers> → <Suspense> → <PostHogPageView />`  |
| 110 | Medium | Phase 6 deferred secrets note said "`AUTH_SECRET` — generated in Phase 8 step 6a; added as GitHub Secret in Phase 8 step 6b". Wrong: Phase 8 step 6a explicitly generates the value AND adds it as a GitHub Secret in one step. Phase 8 step 6b sets it in Vercel env vars. The cross-reference in Phase 6 pointed to the wrong step for the GitHub Secret addition | Corrected to "generated and added as GitHub Secret in Phase 8 step 6a; set in Vercel env vars in Phase 8 step 6b" |

**v15 → v16 Review (Sixteenth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 111 | High | CI E2E job "Delete Neon branch after tests complete" had no `if: always()` guard — when Playwright tests fail, GitHub Actions skips subsequent steps by default, leaving orphaned Neon branches on every failed PR run; they accumulate against the 0.5GB free tier storage limit and eventually break all future branch creation | Added explicit `if: always()` requirement to the delete step with explanation |
| 112 | High | `lighthouserc.cjs` included both `preset: 'lighthouse:recommended'` AND the four `categories:*` assertions. The preset stacks dozens of individual audit-level assertions on top; a generated app can score 93 in Performance but still fail CI because a single preset audit hits its own threshold, producing opaque failures that are difficult to debug and fix | Removed `preset: 'lighthouse:recommended'`; kept only the four `categories:*` assertions; added inline comment explaining why the preset must not be added |
| 113 | Medium | Phase 4 A06 check "No `^` version ranges" had no corrective action. `npm install` always writes `^` by default, so the check flags every package; an agent has no instruction on what to do and will either skip it or attempt to remove all carets blindly (which can break packages that require floating ranges) | Added corrective action: pin only security-sensitive and deeply integrated packages (e.g., `next-auth`, `drizzle-orm`, `@sentry/nextjs`); lower-risk utility packages can retain `^` |
| 114 | Low | `deploy-production.yml` migration step said "using `DATABASE_URL_UNPOOLED` secret" but never specified the explicit env var mapping (`DATABASE_URL_UNPOOLED: ${{ secrets.DATABASE_URL_UNPOOLED }}`). GitHub Actions secrets are not automatically available as env vars — without the mapping the migration step runs with the variable undefined, silently failing or connecting to the wrong DB | Added explicit env block mapping note and warning that secrets require explicit injection |

**v16 → v17 Review (Seventeenth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 115 | Breaking | Phase 7 commits all workflow files to `develop` and Phase 8 does two manual CLI deploys — but no step anywhere merges `develop` into `main`. GitHub Actions reads workflow files from the branch being pushed to; if `deploy-production.yml` never lands on `main`, every future push to `main` triggers zero GitHub Actions and the entire CI/CD system from Phase 7 is permanently inert | Added Phase 8 step 10: open a PR from `develop` → `main` via GitHub MCP, let `ci.yml` run on it, merge — this is the CI/CD handoff point; all previous steps renumbered accordingly |
| 116 | High | `deploy-production.yml` `vercel --prod` step only had a parenthetical "(using VERCEL_TOKEN, VERCEL_ORG_ID, VERCEL_PROJECT_ID secrets)" with no explicit `env:` block requirement — unlike `deploy-preview.yml` which had an explicit injection note. An agent generating the YAML without explicit env mapping produces a workflow where `vercel --prod` cannot identify the project | Added the same explicit env block injection note as `deploy-preview.yml`: all three Vercel secrets must be mapped in the step's `env:` block |
| 117 | Medium | Phase 1 steps 8 and 9 installed test tooling (`vitest`, `@playwright/test`) without any `-D` / `--save-dev` flag; step 11 said "all dev dependencies" but only covered its own step. An agent following steps 8–9 literally installs test runners into `dependencies`, bloating the production bundle | Changed steps 8 and 9 to use `npm install -D` with explicit note that these are devDependencies |

**v17 → v18 Review (Eighteenth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 118 | High | Phase 2 step 2 said `npm run db:generate` → "SQL files committed to repo" but never gave an explicit `git add drizzle/ && git commit` instruction. CI's `npm run db:migrate` (in both the E2E job and `deploy-production.yml`) reads committed migration files; if the files are only on disk and not committed, `drizzle-kit migrate` finds nothing to run, all tables are absent, and E2E tests crash with "relation does not exist" | Rewrote step 2 to make the `git add drizzle/ && git commit` command explicit and added a ⚠ warning explaining the CI breakage that results from skipping it |
| 119 | High | All five workflow files (`ci.yml`, `deploy-preview.yml`, `deploy-production.yml`, `security-scan.yml`, `tech-debt.yml`) described their triggers only in prose headings but none of the YAML blocks included an `on:` key. A workflow file with no trigger is completely ignored by GitHub Actions — it never runs regardless of what events occur | Added the correct `on:` block to each workflow's YAML snippet: `ci.yml` and `deploy-preview.yml` → `on: pull_request`; `deploy-production.yml` → `on: push: branches: [main]`; `security-scan.yml` → `on: schedule: cron: '0 9 * * 1'`; `tech-debt.yml` → `on: schedule: cron: '0 0 1 * *'` |
| 120 | Medium | Phase 1 step 3 installed `drizzle-orm drizzle-kit @neondatabase/serverless` without a `-D` flag, putting `drizzle-kit` (a CLI-only tool never imported at runtime) into `dependencies`. This bloats production `node_modules` and is inconsistent with the `-D` fixes applied to steps 8, 9, and 11 in prior passes | Split step 3: `drizzle-orm @neondatabase/serverless` installed as runtime deps; `drizzle-kit` installed with `npm install -D` |
| 121 | Medium | Phase 1 step 11 said "all dev dependencies" in parenthetical prose but the install command had no `-D` flag, putting `@next/bundle-analyzer`, `eslint-plugin-security`, and `@lhci/cli` into `dependencies` | Changed to `npm install -D @next/bundle-analyzer eslint-plugin-security @lhci/cli` |

**v18 → v19 Review (Nineteenth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 122 | Breaking | Phase 2 step 2: the description said "SQL files created in `drizzle/` directory" and the git add command was `git add drizzle/`. But `drizzle.config.ts` (Phase 1 step 14) specifies `out: './src/db/migrations/'` — generated migration files land in `src/db/migrations/`, not a `drizzle/` directory. Running `git add drizzle/` silently adds nothing (the directory doesn't exist), the commit is empty, and CI still sees no committed migration files — silently undoing the entire #118 fix | Corrected both the description ("SQL files created in `src/db/migrations/`") and the git add path (`git add src/db/migrations/`) to match the `out` config in `drizzle.config.ts` |
| 123 | High | `deploy-production.yml` uses `AUTH_URL` in two shell commands (`PLAYWRIGHT_BASE_URL` derivation and `LHCI_URL=$AUTH_URL npx lhci autorun`) but has no `env:` block mapping for the `AUTH_URL` secret. GitHub Actions secrets are not automatically available as shell variables — the plan explicitly documents this for `DATABASE_URL_UNPOOLED` (#114) and the three Vercel secrets (#116) but omitted `AUTH_URL`. Without `AUTH_URL: ${{ secrets.AUTH_URL }}` in the step env block, `$AUTH_URL` is empty: Playwright tests fail with connection refused and Lighthouse fails to collect a URL | Added explicit `env:` block mapping requirement with ⚠ warning, same pattern as the existing #114/#116 fixes |

**v19 → v20 Review (Twentieth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 124 | High | Phase 1 step 17 npm scripts missing `"test": "vitest run"` and `"test:coverage": "vitest run --coverage"`. The `ci.yml` `test-unit` job and `tech-debt.yml` coverage check both require these; without them agents fall back to bare `npx vitest run`, inconsistent with the rest of the plan | Added `test` and `test:coverage` scripts to Phase 1 step 17; added warning note explaining why they are required |
| 125 | High | Phase 2 step 7 described `src/middleware.ts` combining Auth.js + Upstash but provided zero code. Composing Auth.js v5 `auth()` middleware with Upstash rate limiting is non-trivial (ordering, response handling, `config.matcher`) — without a template an agent is likely to implement one but omit the other or compose them incorrectly | Added canonical `middleware.ts` template to Phase 2 step 7; added important note that App Router group `(protected)/` does not appear in URLs — protect by actual URL paths |
| 126 | Medium | `posthog-node` installed in Phase 1 but no server-side initialization pattern anywhere. The package silently captures nothing without a `PostHog` singleton; an agent has no instructions for creating `src/lib/analytics/posthog-server.ts` | Added `posthog-server.ts` singleton template to Phase 2 step 12 with `flushAt: 1` / `flushInterval: 0` for serverless and a usage example |
| 127 | Medium | Phase 2 step 6 `withSentryConfig` options included `silent: true` — deprecated and removed in `@sentry/nextjs` v8 (released 2024). Since no version is pinned, agents install the latest and get the deprecation warning or silent misconfiguration | Replaced with `widenClientFileUpload: true`, `hideSourceMaps: true`, `disableLogger: true` (the correct v8 options); added comment explicitly warning not to add `silent: true` |
| 128 | Low | `db:push` script defined in Phase 1 step 17 with no warning. The plan's own README section actively discourages it; having it silently available is a footgun for confused users | Added inline warning note to the scripts block: `db:push` is dev-only and bypasses migration history; always use `db:migrate` for CI-consistency |

**v20 → v21 Review (Twenty-first pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 129 | Breaking | `middleware.ts` template typed the `auth()` callback argument as `req: NextRequest & { auth: unknown }` — an explicit annotation that overrides Auth.js v5's own inference (which types `req` as `NextAuthRequest` with `auth: Session \| null`). This forced a `(req as any).auth` cast to suppress TS errors, causing the Phase 3 `tsc --noEmit --strict` check to fail (or pass only because `any` suppresses the error while destroying type safety) | Removed the explicit type annotation; let TypeScript infer the correct `NextAuthRequest` type from the `auth()` wrapper; replaced `(req as any).auth` with `req.auth`; added a comment explaining why an explicit annotation must not be added |
| 130 | High | `middleware.ts` rate-limit condition was `pathname.startsWith('/api/auth') \|\| pathname.startsWith('/api/')` — the first branch is a strict subset of the second and is always subsumed by it. It implied `/api/auth` might be treated differently (e.g., stricter limit) but actually both branches applied the same 10-req/10s window, making the first condition dead/misleading code | Removed the redundant first condition; kept only `pathname.startsWith('/api/')` |
| 131 | High | `middleware.ts` template had `pathname.startsWith('/(protected)')` as the protection guard — this is ALWAYS false because App Router route groups never appear in URLs. The authentication redirect code was dead and no routes were ever protected, despite a note below the template saying to use actual paths | Replaced the always-false guard with real example paths (`/dashboard`, `/settings`); added an inline comment repeating the URL-path requirement directly in the code, not just in a note below |
| 132 | High | `posthog-server.ts` usage example showed `await posthogServer.capture(...)`. `PostHog.capture()` returns `void`, not a Promise; `await` is a no-op and does NOT wait for the HTTP flush. In serverless environments the function terminates before the flush completes, silently dropping events | Corrected the usage to `posthogServer.capture(...); await posthogServer.flushAsync()` and added a ⚠ explaining that `capture()` is synchronous and `flushAsync()` is required for delivery |
| 133 | Medium | Phase 3 checklist item "Middleware correctly protects `(protected)/` route group" contradicts the Phase 2 Step 7 note that `(protected)/` never appears in URLs, and would cause an agent reviewing the middleware to incorrectly approve it if the middleware checked `(protected)/` (always-false guard) | Rewrote the checklist item to "Middleware correctly protects configured protected URL paths (e.g., `/dashboard`, `/settings`) — match by actual URL path, not the route-group folder name" |

**v21 → v22 Review (Twenty-second pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 134 | High | Phase 3 checklist referenced "Auth.js `authorized` callback in middleware handles both session-present and session-absent cases". The middleware template uses the `auth(handler)` pattern where the handler receives `NextAuthRequest` and checks `req.auth` directly. The `authorized` callback is a completely different, incompatible Auth.js v5 middleware approach. An agent auditing the code for an `authorized` callback would not find one and incorrectly flag the correct implementation as broken — or worse, try to add an `authorized` callback alongside `auth(handler)`, breaking the middleware | Rewrote the checklist item to check for `req.auth` null-check in the `auth()` handler and explicitly note not to add an `authorized` callback |
| 135 | High | `ci.yml` E2E job described Neon branch creation as "via Neon API → API returns branch connection strings → Override DATABASE_URL and DATABASE_URL_UNPOOLED" with no instructions on: (a) the actual API endpoint or HTTP call syntax, (b) how to parse the JSON response with `jq`, (c) how to export values to `$GITHUB_ENV`, or (d) how to capture `branch_id` for the cleanup step. Raw Neon REST API calls are complex to get right in GitHub Actions YAML; the vague description would produce broken CI | Replaced with explicit `neondatabase/create-branch-action@v5` (outputs `db_url`, `db_url_with_pooler`, `branch_id`) and `neondatabase/delete-branch-action@v3` (consumes `branch_id`); added YAML snippets showing `steps.neon-branch.outputs.*` wiring |
| 136 | High | `security-scan.yml` said "If high/critical: run `npm audit fix`, open PR with the changes" with no instructions on how to create a branch, commit the changed lockfile, push, or open the PR. The workflow would either silently apply `npm audit fix` and never create a PR, or fail — security vulnerabilities would go unfixed with no notification | Added explicit shell script: `git checkout -b security/audit-fix-DATE`, `git add package.json package-lock.json`, `git commit`, `git push`, `gh pr create`; added required `GITHUB_TOKEN` env mapping and `permissions: contents: write, pull-requests: write` block to the workflow |

**v22 → v23 Review (Twenty-third pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 137 | High | `lighthouserc.cjs` was only created in Phase 9 step 3. But `deploy-production.yml` (Phase 7) runs `npx lhci autorun` — which reads `lighthouserc.cjs` — starting from the Phase 8 step 10 merge to main. Phase 8 step 10 happens BEFORE Phase 9 begins. At that point, `lighthouserc.cjs` is not yet committed to the repo; `lhci autorun` has no config and either fails or runs with defaults, causing the first production CI deploy to fail | Moved `lighthouserc.cjs` creation to Phase 1 step 20 (alongside `vitest.config.ts`, `playwright.config.ts`, `drizzle.config.ts`); added explicit note that it must be committed before Phase 8 step 10; updated Phase 9 step 3 to reference the already-created file instead of recreating it |
| 138 | High | `deploy-preview.yml` said "Capture the URL output from the previous step and comment it on the PR using the GitHub Actions `actions/github-script` action or `gh pr comment`" with no implementation. `vercel deploy` outputs the URL as the last line of stdout — there were no instructions on (a) capturing it with `$GITHUB_OUTPUT`, (b) how to call `gh pr comment` with the PR number, (c) the required `GITHUB_TOKEN` env mapping, or (d) the required `permissions: pull-requests: write` block. An agent implementing this generates a broken workflow | Added explicit YAML: `PREVIEW_URL=$(vercel deploy ...) >> $GITHUB_OUTPUT`, then `gh pr comment ${{ github.event.pull_request.number }} --body "..."` with `GITHUB_TOKEN` mapped and `permissions: pull-requests: write` block |
| 139 | High | `tech-debt.yml` said "Create GitHub issue labeled 'tech-debt' with full report" with no implementation. No `gh issue create` command, no `GITHUB_TOKEN` env mapping, no `permissions: issues: write` block. The report data (npm outdated, coverage) was never captured into a variable to include in the issue body — the workflow would either not create any issue or fail with 403 | Added explicit `gh issue create` command with shell variable capture of npm outdated and coverage output; added `GITHUB_TOKEN` env mapping and `permissions: issues: write` block |
| 140 | High | `security-scan.yml` Semgrep step said "if findings: create GitHub issue with report" — same class of gap as #139. No `gh issue create` command, no `GITHUB_TOKEN` mapping, no `permissions: issues: write`. The `semgrep/semgrep-action` exits non-zero on findings; there was no `if: failure()` step to catch it and create the issue | Added explicit `if: failure()` step with `gh issue create`, `GITHUB_TOKEN` mapping, and `issues: write` added to the workflow `permissions:` block |

**v23 → v24 Review (Twenty-fourth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 141 | High | `deploy-production.yml` "Create GitHub issue on failure" was prose-only with zero implementation — no `gh issue create`, no `GITHUB_TOKEN` env mapping, no `if: failure()` step, no `permissions: issues: write` block. Identical class of gap as #139 (`tech-debt.yml`) and #140 (`security-scan.yml`), both already fixed. Without this, any production deploy failure creates no GitHub issue and goes unnoticed | Added explicit `if: failure()` step with `gh issue create`, `GITHUB_TOKEN` mapping, and `permissions: issues: write` block; added ⚠ fix-forward note |

**v24 → v25 Review (Twenty-fifth pass)**

| # | Severity | Issue | Resolution |
|---|---|---|---|
| 142 | Medium | Phase 1 had two steps numbered "20" — `lighthouserc.cjs` creation (added by issue #137) and the canonical folder structure (original last step), both labeled `20.`. An agent implementing Phase 1 would either skip one step or process them in the wrong order | Renumbered folder structure from step 20 to step 21 |
| 143 | Medium | OWASP A08 (Software and Data Integrity Failures) was entirely absent from Phase 4 security audit — the checklist jumped directly from A07 to A09. A08 is relevant for CI/CD pipeline integrity, reproducible builds (`npm ci` vs `npm install`), and supply-chain attack prevention via pinned Actions | Added A08 section with five actionable checks: npm registry trust, `npm ci` in CI, pinned third-party Action versions, lifecycle script integrity, and deploy-before-migrate ordering |
