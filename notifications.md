---
layout: default
title: "Notifications"
---

<h1>Notifications <span class="gradient">Knock / Resend</span></h1>
<span class="verdict evaluate">⏳ Evaluate — When In-App Notifications Become a Priority</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">SendGrid (partially) + adds in-app</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">4-6 weeks (Knock)</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--yellow)">Medium</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Q3+ (when needed)</span></div>
</div>

## Current State

- **SendGrid** — transactional email (weekly digests, check-in reminders)
- **Slack internal tooling token** — Slack notifications
- **No in-app notifications** — users must check email or Slack
- **Custom digest logic** — Slack daily/weekly digest cron jobs built in-house

---

## Knock — Notification Orchestration

Unifies email, in-app, push, Slack, and SMS behind one API. The compelling piece: **in-app notification center** as a drop-in React component.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Adds in-app notifications</strong> — currently missing, biggest new capability</li>
<li><strong>Drop-in React component</strong> — notification bell/center that works with Next.js immediately</li>
<li><strong>User preferences</strong> — users choose how they want to be notified (reduces fatigue)</li>
<li><strong>Cross-channel orchestration</strong> — "Slack first, email after 1hr if unread"</li>
<li><strong>Built-in digest logic</strong> — could simplify existing cron-based Slack/email digests</li>
<li><strong>Single API</strong> — one call instead of managing SendGrid + Slack separately</li>
<li><strong>99.99% uptime SLA</strong> — reliable delivery</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Another vendor</strong> — new billing, dependency, and integration to maintain</li>
<li><strong>Migration effort</strong> — rewrite SendGrid templates and Slack notification logic</li>
<li><strong>Existing digest logic wasted</strong> — already built Slack daily/weekly digests with custom cron jobs</li>
<li><strong>More expensive than SendGrid</strong> — for email-only use case, SendGrid is cheaper</li>
<li><strong>Overkill right now</strong> — if in-app notifications aren't a near-term requirement</li>
<li><strong>Learning curve</strong> — notification workflow design patterns are non-trivial</li>
</ul>
</div>
</div>

### Knock Pricing

| Tier | Price | Included |
|------|-------|----------|
| Free | $0 | 10k notifications/month |
| Growth | $250/mo | 25k notifications |
| Enterprise | Custom | Advanced features |

---

## Resend — Modern Email API

Developer-friendly email with React Email templates. Direct SendGrid replacement.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>React Email templates</strong> — write emails as React components (same skills as frontend)</li>
<li><strong>Better developer experience</strong> — cleaner API than SendGrid</li>
<li><strong>Good deliverability</strong> — focused on email delivery</li>
<li><strong>Modern tooling</strong> — active development, good docs</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>No new capability</strong> — just email, same as SendGrid</li>
<li><strong>Migration effort without reward</strong> — rewrite templates for marginal improvement</li>
<li><strong>SendGrid works</strong> — no urgent problems to solve</li>
<li><strong>Smaller scale</strong> — less proven at high volume</li>
</ul>
</div>
</div>

---

## What Would Change

### Current Flow (Check-in Reminder)
```
EventBridge Cron → Lambda (nexus.main)
  → query Aurora for due check-ins
  → SQS (CheckinReminderQueue) → Lambda consumer
    → SendGrid API (email)
    → Slack API (internal tooling token)
```

### With Knock
```
EventBridge Cron → Lambda (nexus.main)
  → query Aurora for due check-ins
  → Knock API: trigger("checkin-reminder", { user, data })
    → Knock handles: in-app + Slack + email (with user preferences)
```

## Decision

| Option | Verdict |
|--------|---------|
| **Keep SendGrid + Slack** | ✅ Default for now |
| **Add Knock** | Evaluate when in-app notifications become a requirement |
| **Switch to Resend** | Skip — not enough benefit to justify migration |

### When to Adopt Knock
- Users request in-app notifications
- Notification fatigue becomes a support issue
- Mobile app launch requires push notifications
- You want to consolidate digest logic into one orchestration platform
