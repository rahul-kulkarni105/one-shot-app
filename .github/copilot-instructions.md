# One-Shot App Creation Pipeline — Copilot Context

This repository is an AI-orchestrated pipeline that takes a single natural language prompt and produces a fully deployed, production-grade web application with zero manual coding required.

## What This Repo Does

When the pipeline is invoked it executes a 10-phase process defined in `plan.md`:

| Phase | What Happens |
|---|---|
| 0 | Requirements analysis, stack selection, architecture planning |
| 1 | Project scaffolding (Next.js 15, TypeScript, Tailwind, shadcn/ui, Drizzle, Auth.js, Upstash, Sentry, PostHog) |
| 2 | Full code generation (DB schema → auth → server actions → UI → pages → tests) |
| 3 | Bug & logic review (TypeScript strict, null safety, edge cases) |
| 4 | Security review (OWASP Top 10, Next.js-specific checks) |
| 5 | Performance review (Core Web Vitals, N+1 queries, bundle size) |
| 6 | Git branch protection + CI secrets |
| 7 | CI/CD workflow configuration (GitHub Actions + Neon DB branching per PR) |
| 8 | Deployment (Neon → Vercel → OAuth → live URL) |
| 9 | Verification (Playwright smoke tests + Lighthouse CI against production) |
| 10 | Documentation and handoff |

## Fixed Tech Stack

All apps use this stack regardless of idea type:
- **Framework**: Next.js 15 App Router + TypeScript (strict)
- **Styling**: Tailwind CSS v4 + shadcn/ui
- **Database**: Neon (serverless PostgreSQL) + Drizzle ORM
- **Auth**: Auth.js v5 + `@auth/drizzle-adapter` + Resend (magic links)
- **Rate limiting**: Upstash Redis + `@upstash/ratelimit`
- **Validation**: Zod
- **Monitoring**: Sentry + PostHog + Vercel Analytics + Speed Insights
- **Testing**: Vitest (unit) + Playwright (E2E)
- **CI/CD**: GitHub Actions + Vercel + Neon branch-per-PR isolation

## How to Invoke the Pipeline

Use `#create-app.prompt.md` in Copilot agent mode. The agent will ask for the idea, then execute all 10 phases autonomously.

## Prerequisites

The following must be configured before running (see `plan.md` → One-Time Setup Guide):

- GitHub MCP server (PAT with Contents, Workflows, Secrets, Pull Requests, Administration write)
- Vercel MCP server (Vercel token)
- Neon MCP server (Neon API key)
- Resend account (for Auth.js magic link email)
- Upstash Redis database (for rate limiting)
- Sentry project (for error tracking)
- PostHog project (for analytics)

## Key Behavioural Rules

When executing the pipeline:

1. Always read `plan.md` in full before writing any code
2. Phases must complete in order — never skip or merge phases
3. Self-correct all errors before advancing to the next phase
4. The only permitted user interactions after the initial prompt are the OAuth registration step (Phase 8) and optional paid API key prompts (Phase 0)
5. All secrets are stored in Vercel env vars and GitHub Actions secrets via MCP — never written to any file
6. The new app is scaffolded as a subdirectory of the current workspace (Phase 1 step 1 creates `<app-name>/`); that subdirectory is the project root
