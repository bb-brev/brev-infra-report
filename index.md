---
layout: default
title: Home
---

<h1>Dev Infrastructure <span class="gradient">Research Report</span></h1>
<p class="subtitle">Deep analysis of 12 tools evaluated against Brev's production stack — SST v3, Aurora PostgreSQL, Lambda, SQS, Next.js App Router. Each tool assessed for migration path, effort, risk, and real-world fit.</p>

<div class="meta-bar">
  <div class="meta-item">
    <span class="meta-label">Tools Analyzed</span>
    <span class="meta-value">12</span>
  </div>
  <div class="meta-item">
    <span class="meta-label">Recommended</span>
    <span class="meta-value" style="color: var(--green)">4</span>
  </div>
  <div class="meta-item">
    <span class="meta-label">Evaluate</span>
    <span class="meta-value" style="color: var(--yellow)">4</span>
  </div>
  <div class="meta-item">
    <span class="meta-label">Skip</span>
    <span class="meta-value" style="color: var(--red)">4</span>
  </div>
  <div class="meta-item">
    <span class="meta-label">Date</span>
    <span class="meta-value">March 2026</span>
  </div>
</div>

<p class="section-label">AI & Backend Pipeline</p>
<div class="tool-grid">
  <a href="{{ site.baseurl }}/trigger-dev" class="tool-card">
    <span class="tool-badge eval">Evaluate</span>
    <span class="tool-name">Trigger.dev</span>
    <span class="tool-desc">TypeScript background jobs. Replaces 15+ SQS queues + Lambda consumers. 8-12 week migration.</span>
  </a>
  <a href="{{ site.baseurl }}/vercel-ai-sdk" class="tool-card">
    <span class="tool-badge rec">Recommend</span>
    <span class="tool-name">Vercel AI SDK</span>
    <span class="tool-desc">Unified LLM interface. Replaces direct OpenAI + Anthropic SDKs. 2-3 week migration.</span>
  </a>
  <a href="{{ site.baseurl }}/langfuse" class="tool-card">
    <span class="tool-badge eval">Evaluate</span>
    <span class="tool-name">Langfuse</span>
    <span class="tool-desc">LLM observability. Compare against internal tracing tool. 3-4 week evaluation.</span>
  </a>
</div>

<p class="section-label">Data & Sync</p>
<div class="tool-grid">
  <a href="{{ site.baseurl }}/local-first-sync" class="tool-card">
    <span class="tool-badge skip">Skip</span>
    <span class="tool-name">Zero / PowerSync</span>
    <span class="tool-desc">Local-first sync engines. Current RxDB + TanStack DB stack works. 6-12 month rewrite risk.</span>
  </a>
</div>

<p class="section-label">Auth & Security</p>
<div class="tool-grid">
  <a href="{{ site.baseurl }}/auth" class="tool-card">
    <span class="tool-badge cont">Continue</span>
    <span class="tool-name">Auth — WorkOS / Clerk / Stytch</span>
    <span class="tool-desc">Continue WorkOS migration. Clerk and Stytch not worth switching mid-migration.</span>
  </a>
  <a href="{{ site.baseurl }}/security" class="tool-card">
    <span class="tool-badge skip">Skip</span>
    <span class="tool-name">Arcjet</span>
    <span class="tool-desc">Security middleware. Cloudflare already provides sufficient protection.</span>
  </a>
</div>

<p class="section-label">Developer Experience</p>
<div class="tool-grid">
  <a href="{{ site.baseurl }}/feature-flags" class="tool-card">
    <span class="tool-badge rec">Recommend</span>
    <span class="tool-name">Feature Flags — PostHog</span>
    <span class="tool-desc">Already using PostHog for analytics. Enable flags — zero new vendors. 1-2 weeks.</span>
  </a>
  <a href="{{ site.baseurl }}/error-monitoring" class="tool-card">
    <span class="tool-badge rec">Recommend</span>
    <span class="tool-name">Error Monitoring — Sentry</span>
    <span class="tool-desc">Critical missing capability. Next.js + Lambda error tracking. 1-2 weeks.</span>
  </a>
  <a href="{{ site.baseurl }}/testing" class="tool-card">
    <span class="tool-badge rec">Recommend</span>
    <span class="tool-name">E2E Testing — Playwright</span>
    <span class="tool-desc">Catch regressions like the Microsoft login bug. Cross-browser. 2-4 weeks.</span>
  </a>
</div>

<p class="section-label">Infrastructure</p>
<div class="tool-grid">
  <a href="{{ site.baseurl }}/notifications" class="tool-card">
    <span class="tool-badge eval">Evaluate</span>
    <span class="tool-name">Knock / Resend</span>
    <span class="tool-desc">Notification orchestration. Evaluate if in-app notifications become a priority.</span>
  </a>
  <a href="{{ site.baseurl }}/edge-computing" class="tool-card">
    <span class="tool-badge eval">Evaluate</span>
    <span class="tool-name">Cloudflare Workers</span>
    <span class="tool-desc">Edge compute for latency-sensitive endpoints. Use surgically, not broadly.</span>
  </a>
  <a href="{{ site.baseurl }}/ci-cd" class="tool-card">
    <span class="tool-badge cont">Continue</span>
    <span class="tool-name">CI/CD Improvements</span>
    <span class="tool-desc">Current SST autodeploy works well. Minor optimizations only.</span>
  </a>
</div>

---

## Recommended Timeline

<div class="card">
<h3>🟢 Now</h3>
<p>Vercel AI SDK backend adoption (2-3 weeks) + continue WorkOS migration</p>
</div>

<div class="card">
<h3>🟡 Q2 2026</h3>
<p>PostHog feature flags (1-2 wk) → Sentry (1-2 wk) → Playwright E2E (2-4 wk) → evaluate Langfuse vs internal tool (3-4 wk)</p>
</div>

<div class="card">
<h3>🔵 Q2–Q3 2026</h3>
<p>Pilot Trigger.dev on 2-3 queues (RawFileProcessor first). Evaluate VPC access + cost before committing.</p>
</div>

<div class="card">
<h3>⚪ Q3+ 2026</h3>
<p>Evaluate Knock (if in-app notifications needed), Cloudflare Workers (if latency is measured problem). Reassess Zero/PowerSync maturity.</p>
</div>
