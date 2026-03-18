---
layout: default
title: "Testing — Playwright E2E"
---

[← Back to Overview](./)

# Testing Infrastructure: Playwright E2E

**Verdict:** ✅ Recommend — essential for production confidence  
**Migration Effort:** 2-4 weeks  
**Risk:** Low

---

## Current State

- **Vitest** for unit/integration testing (in package.json)
- **Cypress** config exists (`cypress:open`, `cypress:run` scripts) — unclear if actively used
- No evidence of regular E2E test runs in CI

---

## Why Playwright

[Playwright](https://playwright.dev) is Microsoft's end-to-end testing framework — fast, reliable, and built for modern web apps.

### Playwright vs Cypress

| Factor | Playwright | Cypress |
|--------|-----------|---------|
| **Multi-browser** | Chrome, Firefox, Safari, Edge | Chrome, Firefox, Edge (no Safari) |
| **Speed** | Parallel by default | Sequential by default |
| **Architecture** | Headless, out-of-process | In-process (limitations) |
| **Network interception** | Full control | Limited |
| **Mobile testing** | Device emulation | Limited |
| **Auto-wait** | Built-in smart waits | Requires manual waits |
| **Pricing** | Free, open source | Free core, paid dashboard |
| **Next.js support** | Official integration | Community |

### Why It Matters for Brev

The Microsoft login bug (blocking ALL external users for >1 year) was caught during a team sync, not by automated tests. With E2E tests:

```typescript
// tests/auth/external-login.spec.ts
test("external Microsoft user can log in", async ({ page }) => {
  await page.goto("/login");
  await page.click("[data-testid='microsoft-login']");
  // Verify external tenant user can authenticate
  await expect(page).toHaveURL(/dashboard/);
});
```

This single test would have caught the bug on the first deploy.

### Key Test Scenarios for Brev

| Area | Test Cases |
|------|-----------|
| **Authentication** | Google login, Microsoft login (internal + external tenants), WorkOS SSO |
| **Onboarding** | 4-step onboarding flow (Benn's current work) |
| **Goals** | Create, edit, check-in, progress tracking |
| **Transcripts** | Upload, processing status, summary display |
| **Integrations** | Slack connection, Calendar sync setup |
| **Billing** | Subscription flow, plan switching |

### Implementation

```typescript
// playwright.config.ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e",
  fullyParallel: true,
  reporter: [["html"], ["github"]],
  use: {
    baseURL: process.env.BASE_URL || "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  projects: [
    { name: "chromium", use: { browserName: "chromium" } },
    { name: "firefox", use: { browserName: "firefox" } },
    { name: "webkit", use: { browserName: "webkit" } },
  ],
  webServer: {
    command: "pnpm dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

### CI Integration

```yaml
# .github/workflows/e2e.yml
name: E2E Tests
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: pnpm install
      - run: pnpm exec playwright install --with-deps
      - run: pnpm exec playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## Testing Strategy

### Layer 1: Vitest (Keep)
Unit and integration tests for:
- Utility functions
- Hooks and sync logic
- API route handlers
- Queue consumer logic

### Layer 2: Playwright (Add)
E2E tests for:
- Critical user journeys (auth, onboarding, goals)
- Cross-browser compatibility
- Visual regression (screenshot comparison)
- Integration smoke tests

### Layer 3: Automated Staging Checks (Future)
- Agent-driven periodic checks on staging
- External user login verification
- Integration health monitoring

## Implementation Plan

### Week 1: Setup + Auth Tests
- Install Playwright, configure for Next.js
- Write auth flow tests (Google, Microsoft — internal AND external)
- Set up CI pipeline

### Week 2: Core Flows
- Onboarding flow tests
- Goal CRUD tests
- Transcript upload + status tests

### Week 3: Integration + Visual
- Integration setup flows
- Visual regression baseline
- Cross-browser verification

### Week 4: Stabilize + Documentation
- Fix flaky tests
- Document test patterns
- Train team on writing E2E tests

## Decision

**Essential.** The Microsoft login bug proves the need. E2E tests on critical paths (auth, onboarding, core features) would catch regressions that unit tests can't. The Cypress config already exists but may not be active — Playwright is the better modern choice. 2-4 weeks to a solid foundation.
