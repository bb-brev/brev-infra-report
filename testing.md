---
layout: default
title: "E2E Testing"
---

<h1>E2E Testing <span class="gradient">Playwright</span></h1>
<span class="verdict recommend">✅ Recommend — Production Confidence</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">Cypress (if active) / adds E2E</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">2-4 weeks</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--green)">Low</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Q2 2026</span></div>
</div>

## Current State

- **Vitest** for unit/integration tests
- **Cypress** scripts exist (`cypress:open`, `cypress:run`) — unclear if actively used
- No evidence of regular E2E test runs in CI

---

## Why Playwright Over Cypress

| Factor | Playwright | Cypress |
|--------|:-:|:-:|
| **Multi-browser** | Chrome, Firefox, **Safari**, Edge | Chrome, Firefox, Edge (no Safari) |
| **Speed** | Parallel by default | Sequential by default |
| **Architecture** | Out-of-process (reliable) | In-process (limitations) |
| **Auto-wait** | Built-in smart waits | Manual waits often needed |
| **Network** | Full control | Limited interception |
| **Mobile** | Device emulation | Limited |
| **Pricing** | Free, open source | Free core, paid dashboard |
| **Next.js** | Official integration | Community support |

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Catches regressions unit tests can't</strong> — the Microsoft login bug (blocking external users for 1+ year) would have been caught on first deploy</li>
<li><strong>Cross-browser including Safari</strong> — critical for a SaaS app; Cypress can't test Safari</li>
<li><strong>Parallel by default</strong> — fast CI runs even with growing test suite</li>
<li><strong>Auto-wait</strong> — waits for elements, navigation, network automatically. Fewer flaky tests.</li>
<li><strong>Visual regression</strong> — screenshot comparisons catch UI regressions</li>
<li><strong>Trace viewer</strong> — step-by-step replay of failed tests with screenshots and network logs</li>
<li><strong>Official Next.js support</strong> — built-in <code>webServer</code> config for dev server</li>
<li><strong>Free and open source</strong> — no vendor lock-in, no paid tier needed</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Test maintenance</strong> — E2E tests are inherently more fragile than unit tests</li>
<li><strong>Slower than unit tests</strong> — each test runs a real browser</li>
<li><strong>Test data management</strong> — need seed data, test users, cleanup strategies</li>
<li><strong>Auth complexity</strong> — testing OAuth flows requires mock providers or test accounts</li>
<li><strong>CI time</strong> — adds minutes to PR pipelines (mitigated by parallelism)</li>
<li><strong>Learning curve</strong> — team needs to learn Playwright patterns and best practices</li>
</ul>
</div>
</div>

## Key Test Scenarios for Brev

| Area | Tests | Impact |
|------|-------|--------|
| **Authentication** | Google login, Microsoft login (internal + external tenants), WorkOS SSO | Prevents another year-long auth bug |
| **Onboarding** | 4-step flow (Benn's current work) | Ensures new users can get started |
| **Goals** | Create, edit, check-in, progress tracking, AI summary display | Core product functionality |
| **Transcripts** | Upload, processing status, summary display | Revenue-critical feature |
| **Integrations** | Slack connection, Calendar sync setup | Common support requests |
| **Billing** | Subscription flow, plan switching | Revenue protection |

## The Microsoft Login Bug — What E2E Prevents

```typescript
// tests/auth/microsoft-external.spec.ts
import { test, expect } from "@playwright/test";

test("external Microsoft tenant user can authenticate", async ({ page }) => {
  await page.goto("/login");
  await page.click("[data-testid='microsoft-login']");

  // Mock Microsoft OAuth with EXTERNAL tenant ID
  await page.route("**/oauth/token", (route) => {
    route.fulfill({
      body: JSON.stringify({
        access_token: "mock-token",
        tenant_id: "external-tenant-id",  // NOT Brev's tenant
      }),
    });
  });

  await expect(page).toHaveURL(/dashboard/);
  await expect(page.locator("[data-testid='user-menu']")).toBeVisible();
});
```

This single test would have caught the bug on the first deploy — saving 1+ year of blocked external users.

## Testing Strategy (3 Layers)

### Layer 1: Vitest (Keep — Unit/Integration)
```
Utility functions, hooks, sync logic, API handlers, queue consumer logic
→ Fast, isolated, run on every commit
```

### Layer 2: Playwright (Add — E2E)
```
Auth flows, onboarding, goals CRUD, transcript upload, integrations, billing
→ Run on PR, catch cross-component regressions
```

### Layer 3: Automated Staging Checks (Future)
```
Periodic health checks on staging — external login, integration health, critical paths
→ Agent-driven, catches environment-specific issues
```

## Migration Plan

<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <h4>Week 1: Setup + Auth Tests</h4>
    <p>Install Playwright, configure for Next.js. Write auth flow tests — Google and Microsoft (internal AND external tenants). Set up GitHub Actions CI pipeline.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">2</span>
  <div class="step-content">
    <h4>Week 2: Core User Journeys</h4>
    <p>Onboarding flow, Goal CRUD (create, edit, check-in, view progress), Transcript upload + processing status display.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">3</span>
  <div class="step-content">
    <h4>Week 3: Integrations + Visual</h4>
    <p>Integration setup flows (Slack, Calendar). Visual regression baseline screenshots. Cross-browser verification (Chrome, Firefox, Safari).</p>
  </div>
</div>
<div class="step">
  <span class="step-num">4</span>
  <div class="step-content">
    <h4>Week 4: Stabilize + CI</h4>
    <p>Fix flaky tests. Document test patterns and conventions. Train team on writing E2E tests. Add to PR required checks.</p>
  </div>
</div>

## CI Integration

```yaml
name: E2E Tests
on: [pull_request]
jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: pnpm install --frozen-lockfile
      - run: pnpm exec playwright install --with-deps
      - run: pnpm exec playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## Decision

**Essential.** The Microsoft login bug is the proof. E2E tests on critical paths catch what unit tests can't — cross-component regressions, auth flow breakages, integration failures. 2-4 weeks to a solid foundation that protects every subsequent deploy.
