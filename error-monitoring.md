---
layout: default
title: "Error Monitoring — Sentry"
---

[← Back to Overview](./)

# Error Monitoring: Sentry

**Verdict:** ✅ Recommend — critical missing capability  
**Migration Effort:** 1-2 weeks  
**Risk:** Low

---

## Current State

No dedicated error monitoring tool identified in the codebase. Errors likely surface through:
- CloudWatch logs (Lambda functions)
- Browser console (frontend)
- User reports / manual discovery

This is a significant gap for a production application.

---

## Why Sentry

[Sentry](https://sentry.io) is the industry standard for application error tracking with excellent Next.js and Node.js support.

### What It Adds

| Capability | Current | With Sentry |
|-----------|---------|-------------|
| **Error detection** | Manual / CloudWatch | Automatic capture + alerting |
| **Stack traces** | CloudWatch logs (hard to read) | Rich, source-mapped stack traces |
| **Error grouping** | None | Deduplicated, grouped by root cause |
| **Release tracking** | None | Tie errors to specific deployments |
| **Performance** | None | Web Vitals, transaction tracing |
| **User context** | None | See which users hit errors |
| **Slack alerts** | None | Immediate notification of new errors |

### Integration with Brev's Stack

**Next.js App Router (Frontend):**
```typescript
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // 10% of transactions
  replaysSessionSampleRate: 0.1,
});
```

**Lambda Functions (Backend):**
```typescript
// packages/functions/src/lib/sentry.ts
import * as Sentry from "@sentry/aws-serverless";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.SST_STAGE,
});

// Wrap Lambda handlers
export const withSentry = Sentry.wrapHandler;

// Usage in queue consumer:
export const main = withSentry(async (event: SQSEvent) => {
  // ... existing handler logic
  // Errors automatically captured with full context
});
```

### Key Benefits for Brev

1. **Catch errors before users report them** — especially in SQS consumers where failures silently go to DLQ
2. **Release correlation** — "did this deploy introduce new errors?"
3. **User impact** — "how many users are hitting this bug?"
4. **Performance monitoring** — Web Vitals for the Next.js frontend
5. **Slack integration** — get alerts in your existing Slack workspace

### Pricing

| Tier | Price | Limits |
|------|-------|--------|
| **Developer** | Free | 5k errors/month, 10k performance units |
| **Team** | $26/month | 50k errors, 100k performance |
| **Business** | $80/month | 100k errors, advanced features |

The free tier is likely sufficient to start.

### Alternative: AWS X-Ray + CloudWatch

| Factor | Sentry | X-Ray + CloudWatch |
|--------|--------|-------------------|
| **Setup** | 1-2 days | Already partially set up |
| **DX** | Excellent | Mediocre |
| **Frontend** | ✅ Full support | ❌ Limited |
| **Source maps** | ✅ | ❌ |
| **Alerting** | ✅ Slack, email, etc. | CloudWatch Alarms |
| **Cost** | Free tier or $26/mo | Included in AWS |
| **Lambda** | ✅ Via `@sentry/aws-serverless` | ✅ Native |

**Verdict:** Sentry is significantly better for developer experience. Use X-Ray only if minimizing vendors is a hard constraint.

## Implementation Plan

### Week 1: Setup + Frontend
- Create Sentry project
- Add `@sentry/nextjs` to frontend
- Configure source map uploads in build
- Set up Slack notification channel

### Week 2: Backend + Queue Consumers
- Add `@sentry/aws-serverless` to Lambda functions
- Wrap critical queue consumers (RawFileProcessor, Firmgen, GoalArtgen)
- Configure error grouping rules
- Set up alert thresholds

## Decision

**Essential.** Every production application needs error monitoring. The fact that Brev doesn't have dedicated error tracking is a significant gap. This is a 1-2 week implementation with immediate value. Do it in Q2.
