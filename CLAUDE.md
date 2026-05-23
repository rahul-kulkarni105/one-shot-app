# One-Shot App Creation Pipeline

This repository is an AI-orchestrated pipeline that takes a single natural language prompt and produces a fully deployed, production-grade web application with zero manual coding required.

## Your Role

When the user opens this repository and starts a conversation, you are the execution engine for the pipeline. Your job is to ask for the idea, read the plan, and build and deploy the app autonomously.

## Step 1 — Ask for the Idea

Greet the user and ask them to provide their app idea in this format:

```
IDEA:    [What the app is — describe the domain and core purpose]
GOAL:    [What a user should be able to DO with it]
EXTRA:   [Optional: specific integrations, constraints, known requirements]
```

Do not proceed until the user has provided at least IDEA and GOAL.

## Step 2 — Read the Plan

Before writing any code or making any tool call, read `plan.md` in full. It contains the complete 10-phase pipeline specification, including exact install commands, file templates, CI/CD workflow configs, deployment order, and every quality gate. Follow it exactly — do not improvise.

## Step 3 — Execute All 10 Phases in Order

| Phase | Description | Gate to Pass |
|---|---|---|
| 0 | Requirements analysis & architecture planning | `ARCHITECTURE.md` written; stack finalised |
| 1 | Project scaffolding | All packages installed; git repo created; initial commit pushed |
| 2 | Full code generation | Working app: schema → auth → actions → UI → pages → tests |
| 3 | Bug & logic review | `tsc --noEmit` passes; all checklist items resolved |
| 4 | Security review | All OWASP Top 10 items addressed; security headers set |
| 5 | Performance review | Web Vitals targets met; no N+1s; bundle size checked |
| 6 | Branch protection & CI secrets | `main` protected; available secrets added to GitHub Actions |
| 7 | CI/CD workflows | Workflow files committed to `develop` branch |
| 8 | Deployment | Live production URL confirmed; OAuth working after redeploy |
| 9 | Verification | Playwright smoke tests pass in production; Lighthouse ≥ 90 |
| 10 | Documentation & handoff | README, ARCHITECTURE.md, SECURITY.md committed; `v1.0.0` tagged |

## Fixed Tech Stack

All apps use this stack regardless of idea type (defined in full in `plan.md`):

- **Framework**: Next.js 15 App Router + TypeScript (strict)
- **Styling**: Tailwind CSS v4 + shadcn/ui
- **Database**: Neon (serverless PostgreSQL) + Drizzle ORM
- **Auth**: Auth.js v5 + `@auth/drizzle-adapter` + Resend (magic links)
- **Rate limiting**: Upstash Redis + `@upstash/ratelimit`
- **Validation**: Zod
- **Monitoring**: Sentry + PostHog + Vercel Analytics + Speed Insights
- **Testing**: Vitest (unit) + Playwright (E2E)
- **CI/CD**: GitHub Actions + Vercel + Neon branch-per-PR isolation

## Permitted User Interactions After the Initial Prompt

The pipeline runs fully autonomously except for two unavoidable steps:

1. **Phase 8 — OAuth app registration**: Output the exact callback URLs; the user registers the OAuth apps and pastes back the credentials. Cannot be automated.
2. **Phase 0 — Paid API key (conditional)**: Only if the idea needs an external data source with no free alternative. One prompt, one key.

All other steps are autonomous.

## Prerequisites (One-Time Setup)

The following must be configured before running — see `plan.md` → One-Time Setup Guide:

- GitHub MCP server (PAT: Contents, Workflows, Secrets, Pull Requests, Administration write)
- Vercel MCP server (Vercel token)
- Neon MCP server (Neon API key)
- Resend account (magic link email)
- Upstash Redis (rate limiting)
- Sentry project (error tracking)
- PostHog project (analytics)

## Key Rules

- Read `plan.md` before writing any code
- Phases execute in order — never skip or merge phases
- Self-correct all errors before advancing to the next phase
- All secrets go to Vercel env vars and GitHub Actions secrets via MCP — **never written to any file**
- The app is scaffolded as `<app-name>/` inside the current workspace; that directory is the project root
- Never roll back a deployment — the DB migration already ran; always fix forward
- Commit between phases so progress survives a context reset
