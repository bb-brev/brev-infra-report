---
layout: default
title: "CI/CD Improvements"
---

[← Back to Overview](./)

# CI/CD and Deployment Improvements

**Verdict:** Keep current approach — optimize incrementally  
**Migration Effort:** 1-2 weeks for optimizations  
**Risk:** Low

---

## Current State

- **SST autodeploy** on `dev` branch push
- **PR-based preview stages** (`pr-{number}`)
- Staging and production autodeploy currently commented out in `sst.config.ts`
- **Husky** for pre-commit hooks (typecheck + lint)
- **Vitest** for unit tests

---

## What's Working Well

| Feature | Status |
|---------|--------|
| **Branch deploys** | ✅ Automatic on `dev` push |
| **PR previews** | ✅ Deployed per PR (excluding PR #1) |
| **Pre-commit checks** | ✅ Typecheck + lint via Husky |
| **Infrastructure as code** | ✅ Full stack in SST |

---

## Recommended Optimizations

### 1. Enable Staging + Production Autodeploy

Currently commented out in `sst.config.ts`:

```typescript
// Uncomment when ready:
if (event.type === "branch" && event.branch === "staging" && event.action === "pushed") {
  return { stage: "staging" };
}
if (event.type === "branch" && event.branch === "main" && event.action === "pushed") {
  return { stage: "prod" };
}
```

**Effort:** 5 minutes (uncomment + deploy)

### 2. CI Pipeline Optimization

```yaml
# .github/workflows/ci.yml
name: CI
on: [pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: "pnpm"  # ← Cache dependencies
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck    # Parallel step 1
      - run: pnpm test         # Parallel step 2
      # Add after Playwright setup:
      # - run: pnpm exec playwright test
```

**Key improvements:**
- **Dependency caching** — `pnpm` cache in GitHub Actions
- **Parallel jobs** — typecheck, lint, and tests can run simultaneously
- **Frozen lockfile** — ensure reproducible installs

### 3. Deploy Notifications

```yaml
# Post-deploy notification to Slack
- name: Notify Slack
  if: success()
  run: |
    curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
      -H 'Content-Type: application/json' \
      -d '{"text": "✅ Deployed ${{ github.ref_name }} to ${{ env.STAGE }}"}'
```

### 4. Database Migration Safety

The current setup runs migrations on every deploy via Lambda invocation. Consider adding:

```yaml
# Migration dry-run on PR
- name: Migration Check
  run: |
    pnpm db generate --check  # Verify no pending migrations
```

### 5. PR Preview Database Seeding

PR preview stages deploy infrastructure but may have empty databases. Consider:
- Seed script that runs after PR stage deploy
- Uses a fixed test dataset for consistent QA
- Includes test users, sample goals, mock transcripts

---

## What NOT to Change

| Current Approach | Why Keep |
|-----------------|---------|
| **SST for deploys** | Works well, infrastructure-as-code |
| **Git-based workflow** | Simple, well-understood |
| **Husky pre-commit** | Catches issues early |
| **pnpm** | Fast, disk-efficient |

---

## Future Considerations

| Enhancement | When | Effort |
|------------|------|--------|
| **E2E tests in CI** | After Playwright setup | 1 day |
| **Canary deploys** | When traffic justifies it | 1 week |
| **Rollback automation** | After Sentry integration | 2-3 days |
| **Performance budgets** | After baseline established | 1 day |

---

## Summary

The CI/CD setup is solid. The biggest improvements are:
1. **Enable staging/production autodeploy** (5 minutes)
2. **Add dependency caching** to CI (30 minutes)
3. **Add Slack deploy notifications** (1 hour)
4. **Add E2E tests to CI** (after Playwright, 1 day)

Don't over-engineer what's working. Focus engineering time on higher-impact items.
