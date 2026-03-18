---
layout: default
title: "Security Middleware — Arcjet"
---

[← Back to Overview](./)

# Security Middleware: Arcjet

**Verdict:** Skip — Cloudflare already provides sufficient protection  
**Migration Effort:** 1-2 weeks (low)  
**Risk:** Medium (new/unproven company)

---

## What It Is

[Arcjet](https://arcjet.com) is an AI-powered runtime security platform offering rate limiting, bot detection, PII detection, and email validation — built specifically for Next.js.

## Current State

Brev uses **Cloudflare Turnstile** for bot protection, with Cloudflare CDN providing DDoS protection and general security at the edge.

## What Arcjet Adds

| Feature | Cloudflare (Current) | Arcjet |
|---------|---------------------|--------|
| **Bot detection** | Turnstile (challenge-based) | AI-powered, runtime detection |
| **Rate limiting** | WAF rules (enterprise) | Per-route, code-level |
| **DDoS protection** | ✅ Built-in | ❌ Not its focus |
| **PII detection** | ❌ | ✅ Detect PII in requests |
| **Email validation** | ❌ | ✅ Validate signups |
| **CDN/Edge** | ✅ Full CDN | ❌ App-level only |

## Integration Example

```typescript
// middleware.ts
import arcjet, { detectBot, rateLimit } from "@arcjet/next";

const aj = arcjet({
  key: process.env.ARCJET_KEY!,
  rules: [
    rateLimit({ mode: "LIVE", window: "1m", max: 100 }),
    detectBot({ mode: "LIVE", allow: ["VERIFIED_BOT"] }),
  ],
});

export default async function middleware(req: NextRequest) {
  const decision = await aj.protect(req);
  if (decision.isDenied()) {
    return NextResponse.json({ error: "Blocked" }, { status: 403 });
  }
}
```

## Pros
- Native Next.js App Router integration — middleware-level
- Per-route rate limiting without WAF complexity
- PII detection useful for compliance
- Quick implementation (1-2 days)

## Cons
| Risk | Details |
|------|---------|
| **Duplicates Cloudflare** | Most security is already handled at the edge |
| **New company** | Limited track record, unclear longevity |
| **Performance overhead** | Adds latency to every request (middleware check) |
| **Lambda cold starts** | May worsen cold start times in Lambda |
| **Unclear pricing** | Not well-documented |
| **Defense in depth vs complexity** | Another moving part to debug |

## Recommendation

**Skip.** Cloudflare provides strong edge security. Arcjet's per-route rate limiting is nice but can be achieved with Cloudflare WAF rules or simple middleware. The PII detection feature is interesting but not a current priority.

**Revisit if:**
- You need per-route rate limiting that Cloudflare WAF can't express
- PII compliance becomes a requirement (SOC 2, GDPR data detection)
- Arcjet matures and proves itself in production at scale
