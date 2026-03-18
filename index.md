---
layout: default
title: Home
---

# Brev Dev Infrastructure Research Report

Deep analysis of developer infrastructure tools evaluated against Brev's actual production stack. Each tool is assessed for migration complexity, pros/cons, and fit with our SST v3 + AWS + Aurora PostgreSQL + Lambda + Next.js architecture.

---

## Current Architecture Reference

| Layer | Technology |
|-------|-----------|
| **Database** | AWS Aurora Serverless PostgreSQL 16.6 via SST |
| **ORM** | Drizzle Kit |
| **Infra-as-code** | SST v3 (Ion), home=AWS |
| **Networking** | Custom VPC, bastion + NAT (EC2) |
| **Background Jobs** | 15+ SQS queues (FIFO + standard) + Lambda consumers + DLQs |
| **Cron** | ~10 AWS EventBridge cron jobs via `sst.aws.Cron` |
| **AI (Backend)** | OpenAI SDK, Anthropic SDK |
| **AI (Frontend)** | Vercel AI SDK partial (`@ai-sdk/google`, `@ai-sdk/react`) |
| **LLM Observability** | Internal tool (built in-house) |
| **Vector DB** | Pinecone |
| **Transcription** | AssemblyAI |
| **Client** | RxDB + TanStack DB + TanStack Query, Next.js App Router |
| **Storage** | S3, DynamoDB |
| **Auth** | SST Auth + WorkOS migration in progress |
| **Email** | SendGrid |
| **Payments** | Stripe |
| **Security** | Cloudflare Turnstile |
| **Analytics** | Intercom, PostHog |

---

## Tool Categories

### AI & Backend Pipeline
- [Trigger.dev — Background Jobs](trigger-dev) — vs SQS + Lambda
- [Vercel AI SDK — Unified LLM Interface](vercel-ai-sdk) — vs direct OpenAI/Anthropic SDKs
- [Langfuse — LLM Observability](langfuse) — vs internal tracing tool

### Data & Sync
- [Local-First Sync — Zero / PowerSync](local-first-sync) — vs RxDB replication

### Auth & Security
- [Auth — Clerk / WorkOS / Stytch](auth) — vs current SST Auth + WorkOS migration
- [Security Middleware — Arcjet](security) — vs Cloudflare Turnstile

### Developer Experience
- [Feature Flags — Statsig / LaunchDarkly / PostHog](feature-flags)
- [Error Monitoring — Sentry](error-monitoring)
- [Testing — Playwright E2E](testing)

### Infrastructure
- [Notifications — Knock / Resend](notifications) — vs SendGrid + Slack
- [Edge Computing — Cloudflare Workers](edge-computing) — vs Lambda
- [CI/CD Improvements](ci-cd)

---

## Summary Matrix

| Tool | Verdict | Priority | Effort | Risk |
|------|---------|----------|--------|------|
| **Trigger.dev** | Evaluate (pilot 2-3 queues) | Q2-Q3 | 8-12 weeks | Medium |
| **Vercel AI SDK** | ✅ Recommend | Now | 2-3 weeks | Low |
| **Langfuse** | Evaluate vs internal tool | Q2 | 3-4 weeks | Low |
| **Zero / PowerSync** | Skip (too early) | — | 6-12 months | High |
| **WorkOS** | ✅ Continue migration | Now | In progress | Low |
| **Clerk / Stytch** | Skip | — | — | — |
| **Arcjet** | Skip | — | 1-2 weeks | Medium |
| **PostHog Feature Flags** | ✅ Recommend | Q2 | 1-2 weeks | Low |
| **Sentry** | ✅ Recommend | Q2 | 1-2 weeks | Low |
| **Playwright** | ✅ Recommend | Q2 | 2-4 weeks | Low |
| **Knock** | Evaluate if needed | Q3 | 4-6 weeks | Medium |
| **Resend** | Skip | — | — | — |
| **Cloudflare Workers** | Evaluate selectively | Q3 | Varies | Medium |

### Recommended Timeline

- **Now:** Vercel AI SDK backend adoption + continue WorkOS migration
- **Q2:** PostHog feature flags + Sentry + Playwright E2E + evaluate Langfuse
- **Q2-Q3:** Pilot Trigger.dev on 2-3 queues
- **Q3:** Evaluate Knock (in-app notifications), Cloudflare Workers (edge)
- **Q4:** Reassess Zero/PowerSync maturity, Clerk billing consolidation

**Total high-priority engineering effort: 4-8 weeks**
