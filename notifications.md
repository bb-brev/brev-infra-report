---
layout: default
title: "Notifications — Knock / Resend"
---

[← Back to Overview](./)

# Notifications: Knock + Resend vs SendGrid + Slack

**Verdict:** Keep current approach — evaluate Knock if in-app notifications become a priority  
**Migration Effort:** 4-6 weeks (Knock), 2-4 weeks (Resend)  
**Risk:** Medium

---

## Current State

- **SendGrid** for transactional email (weekly digests, notifications)
- **Slack internal tooling token** for Slack notifications
- **No in-app notifications** — users must check email or Slack

---

## Knock — Notification Orchestration

**What it is:** Cross-channel notification platform that unifies email, in-app, push, Slack, and SMS behind one API.

### What It Would Add

| Channel | Current | With Knock |
|---------|---------|-----------|
| **Email** | SendGrid (manual) | Orchestrated via Knock |
| **Slack** | Internal token (manual) | Orchestrated via Knock |
| **In-app** | ❌ None | ✅ Drop-in notification center |
| **Push** | ❌ None | ✅ Mobile/web push |
| **SMS** | ❌ None | ✅ Via Knock |
| **User preferences** | ❌ None | ✅ Users choose channels |

### Key Features
- **Notification center component** — drop-in React component for in-app notifications
- **User preferences** — let users choose how they want to be notified
- **Digest logic** — batch notifications intelligently (similar to what you built for Slack digests)
- **Cross-channel orchestration** — send to Slack first, email after 1 hour if unread
- **99.99% uptime SLA**

### Integration Example

```typescript
import { Knock } from "@knocklabs/node";

const knock = new Knock(process.env.KNOCK_API_KEY);

// Send a goal check-in reminder across channels
await knock.workflows.trigger("checkin-reminder", {
  recipients: [userId],
  data: {
    goalName: "Reduce critical bug backlog to zero",
    dueDate: "2026-03-21",
    progressUrl: `https://app.brev.io/goals/${goalId}`,
  },
});
// Knock handles: in-app notification + Slack DM + email fallback
```

### Pros
| Benefit | Details |
|---------|---------|
| **In-app notifications** | Currently missing — adds a new engagement channel |
| **User preferences** | Users control notification fatigue |
| **Digest logic** | Could simplify existing Slack/email digest cron jobs |
| **Single API** | One call instead of managing SendGrid + Slack separately |
| **React components** | Drop-in notification bell for the frontend |

### Cons
| Risk | Details |
|------|---------|
| **Another vendor** | Adds billing and dependency |
| **Migration effort** | Need to migrate existing SendGrid templates and Slack logic |
| **Cost** | More expensive than SendGrid for email-only |
| **Overkill** | If in-app notifications aren't a near-term priority |
| **Existing digest logic** | Already built Slack daily/weekly digests — Knock would require rebuilding workflows |

### Pricing
- Free: 10k notifications/month
- Growth: $250/month for 25k notifications
- Enterprise: Custom

---

## Resend — Modern Email API

**What it is:** Developer-friendly email API with React Email template support. Alternative to SendGrid.

### Why Consider
- **React Email** — write email templates as React components (same skills as frontend)
- **Better DX** — cleaner API than SendGrid
- **Good deliverability** — focused on email delivery

### Why Skip (For Now)
| Factor | Details |
|--------|---------|
| **No new capability** | Just email, same as SendGrid |
| **Migration effort** | Rewrite email templates + switch API calls |
| **SendGrid works** | No urgent problems with current email |
| **Smaller scale** | Less proven at high volume than SendGrid |

---

## Recommendation

| Option | Verdict | When |
|--------|---------|------|
| **Keep SendGrid + Slack** | ✅ Default | Now |
| **Add Knock** | Evaluate | When in-app notifications become a priority |
| **Switch to Resend** | Skip | Not enough benefit to justify migration |
| **Knock + Resend** | Skip | Two new vendors for marginal improvement |

### When to Adopt Knock
- Users request in-app notifications
- Notification fatigue becomes a problem (need user preferences)
- You want to consolidate the Slack digest + email digest + checkin reminder logic into one orchestration platform
- Mobile app launch requires push notifications

### When to Consider Resend
- SendGrid becomes a pain point (deliverability, DX, pricing)
- You want to unify email templates with your React component library
