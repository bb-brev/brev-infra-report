---
layout: default
title: "Edge Computing — Cloudflare Workers"
---

[← Back to Overview](./)

# Edge Computing: Cloudflare Workers

**Verdict:** Evaluate selectively — use for specific latency-sensitive endpoints only  
**Migration Effort:** Varies by use case  
**Risk:** Medium (architectural complexity)

---

## Current State

- **Cloudflare** for DNS and CDN
- **Next.js on Lambda** via SST for server rendering and API routes
- **Lambda functions** for all backend processing
- All compute runs in a single AWS region (us-east-1)

---

## What Cloudflare Workers Would Add

| Factor | Lambda (Current) | Workers (Proposed) |
|--------|-----------------|-------------------|
| **Cold starts** | 100-500ms | ~0ms (V8 isolates) |
| **Global distribution** | Single region | 330+ cities |
| **Runtime** | Full Node.js | V8 isolates (limited) |
| **DB access** | VPC-native (Aurora) | Requires proxy/API |
| **Max execution** | 15 minutes | 30 seconds (free), 15 min (paid) |
| **Cost** | Per-invocation + duration | Per-request (cheaper for short tasks) |

---

## Where Workers Make Sense for Brev

### ✅ Good Candidates

| Use Case | Current | With Workers | Benefit |
|----------|---------|-------------|---------|
| **Auth middleware** | Lambda cold start on every request | Instant JWT validation at edge | Faster page loads globally |
| **API rate limiting** | None or Cloudflare WAF | Code-level rate limiting per user/route | More granular control |
| **Feature flag evaluation** | PostHog client-side | Edge-side flag evaluation | No layout shift, faster decisions |
| **Static API responses** | Lambda + Aurora query | KV-cached responses at edge | Sub-10ms for cached data |

### ❌ Bad Candidates

| Use Case | Why Not |
|----------|---------|
| **AI processing** | Needs full Node.js + long execution |
| **DB queries** | Aurora is in VPC, Workers can't access directly |
| **Queue consumers** | Already SQS + Lambda, Workers adds complexity |
| **File processing** | Size limits, runtime constraints |

---

## Architectural Tension

The core problem: **SST is built around AWS Lambda.** Adding Workers creates a split architecture:

```
User Request
    ├── Cloudflare Worker (edge logic)
    │   ├── Auth validation
    │   ├── Rate limiting
    │   └── Cached responses
    │
    └── Lambda via SST (core logic)
        ├── DB queries
        ├── AI processing
        └── Queue management
```

This adds complexity:
- Two deployment pipelines (SST + Wrangler)
- Two monitoring systems (CloudWatch + Workers Analytics)
- Two sets of environment variables/secrets
- Debugging spans across two platforms

---

## Pros

| Benefit | Details |
|---------|---------|
| **Zero cold starts** | Workers start instantly — no 100-500ms Lambda penalty |
| **Global latency** | Sub-50ms for 95% of internet-connected population |
| **Already using Cloudflare** | Incremental addition to existing infrastructure |
| **Cost-effective** | Free tier: 100k requests/day. Paid: $5/month + $0.50/M requests |
| **KV/D1 storage** | Edge-native key-value and SQLite for cached data |

## Cons

| Risk | Severity | Details |
|------|----------|---------|
| **Split architecture** | 🟡 Medium | Two deployment targets, harder to reason about |
| **SST conflict** | 🟡 Medium | Workers managed outside SST (or via custom constructs) |
| **Limited runtime** | 🟡 Medium | V8 isolates — no native Node.js modules |
| **Data locality** | 🔴 High | Workers are far from Aurora — can't query DB directly |
| **Debugging** | 🟡 Medium | Request flows across two platforms |

---

## Recommendation

**Don't migrate core application logic to Workers.** The SST + Lambda architecture works well.

**Do consider Workers for:**
1. **Auth middleware** if Lambda cold starts are impacting UX (measure first)
2. **Cached API responses** via KV for frequently accessed, rarely changed data
3. **A/B routing** if you need edge-level traffic splitting

**Measure before adopting:**
- What's the actual Lambda cold start impact on user experience?
- Are there specific pages where latency is a problem?
- Would CloudFront caching solve the same problem more simply?

**If Lambda cold starts ARE a problem**, consider these alternatives first:
- **Provisioned concurrency** on critical Lambda functions (stays within SST)
- **CloudFront caching** for static/semi-static API responses
- **Lambda SnapStart** (if available for Node.js)

Workers add value at the margins but introduce architectural complexity. Use surgically, not broadly.
