---
layout: default
title: "Error Monitoring"
---

<h1>Error Monitoring <span class="gradient">Sentry</span></h1>
<span class="verdict recommend">✅ Recommend — Critical Gap</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">Nothing (new capability)</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">1-2 weeks</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--green)">Low</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Q2 2026</span></div>
</div>

## Current State

**No dedicated error monitoring** identified in the codebase. Errors likely surface through CloudWatch logs (hard to search), browser console (invisible to team), and user reports (too late).

> This is a significant gap for a production SaaS application serving enterprise customers.

---

## Why Sentry

[Sentry](https://sentry.io) is the industry standard for application error tracking with first-class Next.js and AWS Lambda support.

### What Changes

| | Before (Current) | After (With Sentry) |
|---|---|---|
| **Error detection** | Manual discovery or CloudWatch | Automatic capture + Slack alert |
| **Stack traces** | Raw CloudWatch logs | Source-mapped, readable traces |
| **Error grouping** | None — every log is separate | Deduplicated by root cause |
| **Release tracking** | None | Errors tied to specific deploys |
| **User impact** | "Someone reported a bug" | "247 users hit this error since deploy" |
| **Performance** | None | Web Vitals, transaction tracing |

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Catch errors before users report</strong> — especially in SQS consumers where failures silently go to DLQ</li>
<li><strong>Release correlation</strong> — "this deploy introduced 3 new error types"</li>
<li><strong>User impact visibility</strong> — "142 users affected by this bug"</li>
<li><strong>Source maps</strong> — readable stack traces from minified production code</li>
<li><strong>Slack integration</strong> — instant alerts in existing workspace</li>
<li><strong>Performance monitoring</strong> — Web Vitals for the Next.js frontend</li>
<li><strong>Session replay</strong> — see exactly what users did before hitting an error</li>
<li><strong>First-class Next.js SDK</strong> — official <code>@sentry/nextjs</code> with App Router support</li>
<li><strong>AWS Lambda SDK</strong> — <code>@sentry/aws-serverless</code> wraps handlers automatically</li>
<li><strong>Free tier sufficient</strong> — 5k errors/month free</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Another vendor</strong> — adds a dependency and billing relationship</li>
<li><strong>Can be noisy</strong> — requires tuning alert thresholds to avoid fatigue</li>
<li><strong>Bundle size</strong> — Sentry SDK adds ~30-40KB to client bundle (mitigated with lazy loading)</li>
<li><strong>Privacy considerations</strong> — captures user context by default (configurable)</li>
<li><strong>Cost at scale</strong> — can get expensive at high error volumes (fix your bugs!)</li>
</ul>
</div>
</div>

### Real-World Impact

The **Microsoft login bug** (blocking ALL external users for >1 year) would have been caught immediately:

```
🔴 Sentry Alert: New error in auth.handler
   TypeError: Invalid tenant ID for external Microsoft user
   ├── 847 events in last 24h
   ├── Affected users: 312 unique
   ├── First seen: 2025-01-15 (deploy abc123)
   └── [View in Sentry →]
```

Instead, it was discovered in a team sync meeting — over a year later.

## Integration

### Frontend (Next.js App Router)

```typescript
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
});
```

### Backend (Lambda Consumers)

```typescript
import * as Sentry from "@sentry/aws-serverless";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.SST_STAGE,
});

// Wrap any Lambda handler
export const main = Sentry.wrapHandler(async (event: SQSEvent) => {
  // Existing handler logic unchanged
  // Errors automatically captured with full SQS context
});
```

### Pricing

| Tier | Price | Included |
|------|-------|----------|
| **Developer** | Free | 5k errors, 10k perf, 50 replays/month |
| **Team** | $26/mo | 50k errors, 100k perf, 500 replays |
| **Business** | $80/mo | 100k errors, advanced features |

Free tier is likely sufficient to start.

## Migration Plan

<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <h4>Week 1: Frontend + Critical Lambda</h4>
    <p>Add <code>@sentry/nextjs</code> to frontend. Configure source map uploads. Wrap the 3-4 most critical queue consumers with <code>@sentry/aws-serverless</code>. Set up Slack notification channel.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">2</span>
  <div class="step-content">
    <h4>Week 2: Full Coverage + Tuning</h4>
    <p>Wrap all remaining Lambda handlers. Configure error grouping rules. Set alert thresholds. Add SST secret for Sentry DSN. Create team dashboard.</p>
  </div>
</div>

## Decision

**Essential.** Every production SaaS needs error monitoring. The Microsoft login bug proves the cost of not having it. 1-2 weeks, free tier, immediate value. Do it.
