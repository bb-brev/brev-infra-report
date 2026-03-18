---
layout: default
title: "CI/CD"
---

<h1>CI/CD <span class="gradient">Improvements</span></h1>
<span class="verdict continue">🔵 Keep + Optimize</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Current</span><span class="meta-value">SST autodeploy + PR stages</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">1-2 weeks (optimizations)</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--green)">Low</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Incremental</span></div>
</div>

## Current Setup

- **SST autodeploy** on `dev` branch push
- **PR preview stages** — `pr-{number}` deployed per PR
- Staging/production autodeploy currently **commented out** in `sst.config.ts`
- **Husky** pre-commit hooks (typecheck + lint)
- **Vitest** for unit tests

---

## What's Working

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Strengths</h4>
<ul>
<li><strong>SST autodeploy</strong> — infrastructure and app deploy together, single pipeline</li>
<li><strong>PR preview environments</strong> — every PR gets its own deployment for testing</li>
<li><strong>Pre-commit hooks</strong> — typecheck + lint catches issues before push</li>
<li><strong>Git-based workflow</strong> — simple, well-understood, no custom tooling</li>
<li><strong>pnpm</strong> — fast, disk-efficient package management</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Gaps</h4>
<ul>
<li><strong>No staging/prod autodeploy</strong> — commented out, requires manual deploy</li>
<li><strong>No dependency caching in CI</strong> — reinstalls everything on each run</li>
<li><strong>No deploy notifications</strong> — team doesn't know when deploys happen</li>
<li><strong>No E2E tests in CI</strong> — only typecheck + lint pre-commit</li>
<li><strong>No migration safety check</strong> — DB migrations run on deploy without dry-run</li>
<li><strong>Empty PR preview databases</strong> — PR stages have no test data</li>
</ul>
</div>
</div>

## Quick Wins

<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <h4>Enable Staging + Production Autodeploy (5 minutes)</h4>
    <p>Uncomment the staging/production blocks in <code>sst.config.ts</code>. Push to staging branch → auto-deploys. Push to main → auto-deploys production.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">2</span>
  <div class="step-content">
    <h4>Add Dependency Caching (30 minutes)</h4>
    <p>Add <code>cache: "pnpm"</code> to <code>actions/setup-node</code> in CI. Use <code>--frozen-lockfile</code> for reproducible installs. Saves 1-3 minutes per CI run.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">3</span>
  <div class="step-content">
    <h4>Slack Deploy Notifications (1 hour)</h4>
    <p>Post to a <code>#deploys</code> channel on successful deploys with branch, stage, and commit message. Simple webhook integration.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">4</span>
  <div class="step-content">
    <h4>E2E Tests in CI (1 day, after Playwright setup)</h4>
    <p>Add Playwright tests to the PR pipeline. Upload test reports as artifacts on failure. Block merge if critical tests fail.</p>
  </div>
</div>

## What NOT to Change

| Current Approach | Why Keep |
|-----------------|---------|
| **SST for deploys** | Works well, full infrastructure as code |
| **Git-based workflow** | Simple, everyone knows it |
| **Husky pre-commit** | Catches issues before they hit CI |
| **pnpm** | Fast, reliable, disk-efficient |

## Future Considerations

| Enhancement | When | Effort |
|------------|------|--------|
| Canary deploys | When traffic justifies | 1 week |
| Rollback automation | After Sentry integration | 2-3 days |
| Performance budgets | After Web Vitals baseline | 1 day |
| PR preview DB seeding | When QA uses previews regularly | 2-3 days |

## Decision

**Don't over-engineer.** The CI/CD setup is solid. Focus on the 4 quick wins above — total investment ~2 days — and move engineering time to higher-impact items.
