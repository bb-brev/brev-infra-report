---
layout: default
title: "Auth — Clerk / WorkOS / Stytch"
---

[← Back to Overview](./)

# Auth Consolidation: Clerk vs WorkOS vs Stytch

**Verdict:** ✅ Continue WorkOS migration (already in progress)  
**Migration Effort:** Already underway (Chris)  
**Risk:** Low

---

## Current State

- **SST Auth** currently in production
- **WorkOS migration in progress** — Chris is actively building this
- Dual-user model: WorkOS user object linked to Brev's user table via WorkOS user ID
- Org provisioning deferred to later phase

---

## WorkOS (Current Migration Target)

**What it is:** Enterprise-ready auth platform with SSO, SCIM, directory sync, and organization management.

### Why It's the Right Choice (Already)

| Feature | Details |
|---------|---------|
| **Enterprise SSO** | SAML/OIDC with 20+ identity providers |
| **SCIM provisioning** | Automated user lifecycle management |
| **Admin Portal** | Self-serve SSO setup for enterprise customers |
| **Directory Sync** | Sync user data from AD, Okta, etc. |
| **Migration in progress** | Sunk cost — Chris is building this now |

### Pros
- Migration already underway — switching would waste weeks of work
- Strong enterprise features (SSO, SCIM, Directory Sync)
- Great developer experience and documentation
- Used by Discord, Perplexity, Plaid — proven at scale
- Reasonable pricing: free up to 1M MAUs for AuthKit

### Cons
- Doesn't include billing/payments (still need Stripe separately)
- Less polished pre-built UI components than Clerk
- No built-in analytics or fraud detection

---

## Clerk — Why Not Now

**What it is:** Complete user management with auth, orgs, and recently added billing.

| Factor | Assessment |
|--------|-----------|
| **$50M Series C** | Well-funded, growing fast |
| **Auth + Billing** | Could theoretically consolidate with Stripe — but billing features are new/unproven |
| **Beautiful UI** | Best drop-in React components in the market |
| **Next.js integration** | Excellent App Router support |

### Why Skip
- **Would abandon WorkOS work in progress** — weeks of Chris's migration work wasted
- **More expensive** for enterprise features (SSO/SCIM)
- **Less mature enterprise SSO** than WorkOS
- **Billing consolidation unproven** — still need Stripe for complex subscription management
- **Migration effort: 6+ months** to restart from scratch

---

## Stytch — Why Not Now

**What it is:** "Identity platform for humans & AI agents" — auth + authorization + security.

| Factor | Assessment |
|--------|-----------|
| **AI agent auth** | Most advanced toolkit for MCP authentication |
| **Multi-tenancy** | Native organization support |
| **Fraud detection** | Real-time abuse protection |
| **Embeddable admin** | White-label admin portal |

### Why Skip
- **AI agent auth is forward-looking** — premature for current needs
- **Smaller company** than WorkOS/Clerk — less proven at scale
- **Would abandon WorkOS migration** — same problem as Clerk
- **Migration effort: 6+ months**

---

## Recommendation

| Option | Verdict | Reason |
|--------|---------|--------|
| **Continue WorkOS** | ✅ Do this | Already in progress, strong enterprise features |
| **Switch to Clerk** | ❌ Skip | Would waste current migration work |
| **Switch to Stytch** | ❌ Skip | AI agent features premature, smaller ecosystem |

### Future Considerations
- **If Clerk's billing matures** (6-12 months): Could consolidate auth + billing, reducing Stripe dependency. Worth revisiting in Q4.
- **If AI agent auth becomes critical** (Brev agents need authenticated API access): Stytch's MCP auth could be relevant. Monitor their progress.
- **WorkOS is also adding AI features** — they may cover AI agent auth needs as they evolve.

**Bottom line:** Don't disrupt an active migration. Finish WorkOS, ship it, reassess in 6 months.
