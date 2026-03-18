---
layout: default
title: "Feature Flags"
---

<h1>Feature Flags <span class="gradient">PostHog / Statsig / LaunchDarkly</span></h1>
<span class="verdict recommend">✅ Recommend PostHog Feature Flags</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">Nothing (new capability)</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">1-2 weeks</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--green)">Low</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Q2 2026</span></div>
</div>

## Current State

**No dedicated feature flag system.** Feature rollouts are all-or-nothing deploys. PostHog is already used for analytics — feature flags are a built-in capability that just needs to be enabled.

---

## PostHog Feature Flags ✅ Recommended

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Zero new vendors</strong> — already using PostHog for analytics, flags are built-in</li>
<li><strong>Tied to analytics</strong> — measure impact of feature rollouts with existing PostHog events</li>
<li><strong>Free tier</strong> — 1M flag requests/month at no additional cost</li>
<li><strong>Percentage rollouts</strong> — gradually roll features to 10%, 25%, 50%, 100%</li>
<li><strong>User targeting</strong> — target by user properties, cohorts, or specific users</li>
<li><strong>Multivariate flags</strong> — A/B/C test different variants</li>
<li><strong>React hooks</strong> — <code>useFeatureFlagEnabled()</code> works with Next.js immediately</li>
<li><strong>Server-side SDK</strong> — evaluate flags in Lambda consumers and Server Components</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Less sophisticated targeting</strong> — fewer rules/conditions than LaunchDarkly</li>
<li><strong>No approval workflows</strong> — anyone with access can toggle flags (no governance)</li>
<li><strong>No audit logs</strong> — limited visibility into who changed what</li>
<li><strong>Evaluation speed</strong> — may be slower than purpose-built flag systems for real-time use cases</li>
<li><strong>Limited SDK features</strong> — fewer advanced patterns than dedicated platforms</li>
</ul>
</div>
</div>

### Implementation

```typescript
// Server-side (Lambda or Server Component)
import PostHog from "posthog-node";
const posthog = new PostHog(process.env.POSTHOG_KEY!);

const showNewFlow = await posthog.isFeatureEnabled("new-onboarding-flow", userId);

// Client-side (PostHog already initialized)
import { useFeatureFlagEnabled } from "posthog-js/react";

function OnboardingFlow() {
  const showNewFlow = useFeatureFlagEnabled("new-onboarding-flow");
  return showNewFlow ? <NewOnboarding /> : <LegacyOnboarding />;
}
```

---

## Statsig — For Later

Full product development platform: feature flags + experimentation + analytics.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Superior experimentation</strong> — statistical rigor, holdouts, power analysis</li>
<li><strong>Could replace PostHog</strong> — analytics + flags + experiments in one platform</li>
<li><strong>Strong Next.js integration</strong> — React SDK with SSR support</li>
<li><strong>Generous free tier</strong> — good for startups</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Requires PostHog migration</strong> — move analytics to justify the switch</li>
<li><strong>More complex than needed</strong> — overkill for basic feature flags</li>
<li><strong>Another platform to learn</strong> — new dashboard, SDK, patterns</li>
<li><strong>Overlap with PostHog</strong> — paying for capabilities you already have</li>
</ul>
</div>
</div>

**Verdict:** Evaluate after 3-6 months if PostHog flags feel limiting.

---

## LaunchDarkly — Skip

Enterprise feature management. 45 trillion daily evaluations. Built for 500+ engineer orgs.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Most mature platform</strong> — market leader with proven reliability</li>
<li><strong>Advanced governance</strong> — approval workflows, audit logs, RBAC</li>
<li><strong>Automated rollbacks</strong> — flag-level rollback on error spikes</li>
<li><strong>80+ integrations</strong> — connects to everything</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Expensive</strong> — enterprise pricing, not startup-friendly</li>
<li><strong>Overkill</strong> — complex interface for simple rollouts</li>
<li><strong>No analytics</strong> — still need PostHog separately</li>
<li><strong>Premature</strong> — governance features aren't needed at current team size</li>
</ul>
</div>
</div>

**Verdict:** Skip. Revisit when you have 50+ engineers needing governance.

## Migration Plan

<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <h4>Day 1-2: Enable in PostHog</h4>
    <p>Enable feature flags in PostHog dashboard. Create first flag for a current feature rollout (e.g., new onboarding flow).</p>
  </div>
</div>
<div class="step">
  <span class="step-num">2</span>
  <div class="step-content">
    <h4>Day 3-5: Client Integration</h4>
    <p>Add <code>useFeatureFlagEnabled()</code> hooks to components. PostHog React provider is likely already configured for analytics.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">3</span>
  <div class="step-content">
    <h4>Week 2: Server Integration</h4>
    <p>Add <code>posthog-node</code> to Lambda consumers for server-side flag evaluation. Wrap new features behind flags by default.</p>
  </div>
</div>

## Decision

**Start with PostHog feature flags.** Zero vendor overhead, immediate access, good enough for feature rollouts and basic experimentation. Graduate to Statsig only if you need advanced statistical rigor.
