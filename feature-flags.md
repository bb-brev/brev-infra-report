---
layout: default
title: "Feature Flags — Statsig / LaunchDarkly / PostHog"
---

[← Back to Overview](./)

# Feature Flags: Statsig vs LaunchDarkly vs PostHog

**Verdict:** ✅ Recommend PostHog Feature Flags (already using PostHog for analytics)  
**Migration Effort:** 1-2 weeks  
**Risk:** Low

---

## Current State

- **No dedicated feature flag system** in production
- **PostHog** already used for analytics
- Feature rollouts are currently all-or-nothing deploys

---

## PostHog Feature Flags ✅ Recommended

**What it is:** Feature flags built into the PostHog platform you're already using.

### Why This Wins

| Factor | Details |
|--------|---------|
| **Zero new vendors** | Already using PostHog — flags are a configuration toggle away |
| **Analytics integration** | Flags tied directly to your existing PostHog events |
| **Free tier** | 1M flag requests/month free |
| **Low complexity** | Simple API, good React/Next.js SDK |

### Implementation

```typescript
// Server-side (Lambda or Server Component)
import PostHog from "posthog-node";

const posthog = new PostHog(process.env.POSTHOG_KEY!);
const isEnabled = await posthog.isFeatureEnabled("new-onboarding-flow", userId);

// Client-side (already have PostHog initialized)
import { useFeatureFlagEnabled } from "posthog-js/react";

function OnboardingFlow() {
  const showNewFlow = useFeatureFlagEnabled("new-onboarding-flow");
  return showNewFlow ? <NewOnboarding /> : <LegacyOnboarding />;
}
```

### Pros
- No additional vendor or billing relationship
- Instant access — already paying for PostHog
- Feature flags + A/B testing + analytics in one platform
- Good enough for current needs
- Percentage rollouts, user targeting, multivariate flags

### Cons
- Less sophisticated targeting than LaunchDarkly
- No approval workflows or audit logs (enterprise features)
- Limited SDK compared to dedicated platforms
- Flag evaluation speed may be slower than purpose-built solutions

### Pricing
- **Free:** 1M requests/month
- **Beyond:** $0.0001/request (~$100 per billion requests)

---

## Statsig — For Later

**What it is:** Product development platform with feature flags + experimentation + analytics.

### When to Consider
- If PostHog analytics prove insufficient for product decisions
- If you need advanced experimentation (holdouts, power analysis, statistical rigor)
- If you want to consolidate PostHog + flags into one more powerful platform

### Pros
- Superior experimentation engine
- Could replace PostHog entirely (analytics + flags + experiments)
- Strong Next.js integration
- Generous free tier for startups

### Cons
- Requires migrating from PostHog analytics (disruption)
- More complex than needed for basic feature flags
- Another platform to learn
- Overlap with existing PostHog investment

### Verdict
**Evaluate later** — if PostHog flags aren't sufficient after 3-6 months of use.

---

## LaunchDarkly — Skip

**What it is:** Enterprise feature management platform — the market leader.

### Why Skip
| Factor | Details |
|--------|---------|
| **Cost** | Expensive for startups — enterprise pricing |
| **Overkill** | Complex interface for simple feature rollouts |
| **No analytics** | Still need PostHog separately |
| **Enterprise focus** | Approval workflows, audit logs — premature for Brev's stage |

### When to Revisit
- If you have 50+ engineers needing governance and approval workflows
- If you need 45 trillion daily evaluations-level scale
- If enterprise customers require it for compliance

---

## Recommendation

| Platform | Verdict | Effort | When |
|----------|---------|--------|------|
| **PostHog Flags** | ✅ Adopt | 1-2 weeks | Q2 |
| **Statsig** | Evaluate | 4-6 weeks | If PostHog insufficient |
| **LaunchDarkly** | Skip | — | Premature for stage |

**Start with PostHog feature flags.** Zero vendor overhead, immediate access, good enough for feature rollouts and basic A/B testing. Graduate to Statsig if you need advanced experimentation.
