---
layout: default
title: "Edge Computing"
---

<h1>Edge Computing <span class="gradient">Cloudflare Workers</span></h1>
<span class="verdict evaluate">🔍 Evaluate — Surgical Use Only</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">Specific Lambda functions (selective)</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">Varies by use case</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--yellow)">Medium</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Q3+ (measure first)</span></div>
</div>

## Current State

- **Cloudflare** for DNS and CDN (already in place)
- **Next.js on Lambda** via SST for server rendering + API routes
- **Lambda functions** for all backend processing
- All compute in a single AWS region

---

## What Workers Add

| | Lambda (Current) | Workers |
|---|---|---|
| **Cold starts** | 100-500ms | ~0ms (V8 isolates) |
| **Global distribution** | Single region | 330+ cities |
| **Runtime** | Full Node.js | V8 isolates (limited) |
| **DB access** | VPC-native to Aurora | Requires proxy/API |
| **Max execution** | 15 minutes | 30s free / 15min paid |

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Zero cold starts</strong> — V8 isolates start instantly vs 100-500ms Lambda</li>
<li><strong>Global latency</strong> — sub-50ms for 95% of internet-connected population</li>
<li><strong>Already using Cloudflare</strong> — incremental addition to existing infra</li>
<li><strong>Cheap</strong> — free tier: 100k req/day. Paid: $5/mo + $0.50/M requests</li>
<li><strong>KV storage</strong> — edge-native key-value for cached data</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Split architecture</strong> — SST manages Lambda, Workers managed separately (Wrangler). Two deployment targets, harder to reason about.</li>
<li><strong>Can't access Aurora</strong> — Workers are far from your VPC. Every DB query needs a proxy or API call.</li>
<li><strong>Limited runtime</strong> — V8 isolates, not full Node.js. No native modules.</li>
<li><strong>Conflicts with SST model</strong> — SST has no native Worker construct. You'd manage it outside your IaC.</li>
<li><strong>Debugging across platforms</strong> — request flows through Cloudflare → AWS. Two monitoring systems.</li>
<li><strong>Premature optimization</strong> — are Lambda cold starts actually impacting user experience?</li>
</ul>
</div>
</div>

## Where Workers Make Sense

| Use Case | Benefit | Worth It? |
|----------|---------|-----------|
| **Auth middleware** | Instant JWT validation globally | Maybe — measure cold start impact first |
| **Cached API responses** | Sub-10ms for frequently accessed data | Yes, if specific endpoints are slow |
| **Feature flag evaluation** | No layout shift, edge-side decisions | Maybe — PostHog client SDK may be sufficient |
| **A/B routing** | Edge-level traffic splitting | Only if needed |

## Where Workers Don't Make Sense

| Use Case | Why Not |
|----------|---------|
| **AI processing** | Needs full Node.js, long execution |
| **Database queries** | Aurora is in VPC, Workers can't reach it |
| **Queue consumers** | SQS + Lambda is the right pattern |
| **File processing** | Size limits, runtime constraints |
| **Any core business logic** | Splits architecture for marginal gain |

## Before Adopting: Measure First

> Are Lambda cold starts actually impacting user experience?

1. **Add performance monitoring** (Sentry Performance or Web Vitals) — measure actual TTFB
2. **Check CloudWatch** — how frequent are cold starts on critical Lambda functions?
3. **Try simpler solutions first:**
   - Provisioned concurrency on critical Lambdas (stays within SST)
   - CloudFront caching for semi-static responses
   - Lambda SnapStart (when available for Node.js)

## Decision

**Don't migrate core logic to Workers.** SST + Lambda works well. Consider Workers only for specific, measured latency problems at the edge — and only after confirming simpler solutions (provisioned concurrency, caching) don't solve it.
