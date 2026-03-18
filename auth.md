---
layout: default
title: "Auth"
---

<h1>Auth <span class="gradient">WorkOS / Clerk / Stytch</span></h1>
<span class="verdict continue">🔵 Continue WorkOS Migration</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Current</span><span class="meta-value">SST Auth → WorkOS (in progress)</span></div>
  <div class="meta-item"><span class="meta-label">Owner</span><span class="meta-value">Chris</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--green)">Low</span></div>
  <div class="meta-item"><span class="meta-label">Action</span><span class="meta-value">Don't disrupt</span></div>
</div>

## Current State

- **SST Auth** currently in production
- **WorkOS migration in progress** — Chris is actively building this
- Dual-user model: WorkOS user object linked to Brev user table via WorkOS user ID
- Org provisioning deferred to later phase

---

## WorkOS ✅ Continue

Already the migration target. Enterprise-grade auth with SSO, SCIM, and directory sync.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Migration already underway</strong> — Chris is building this now. Switching would waste weeks of work.</li>
<li><strong>Enterprise SSO</strong> — SAML/OIDC with 20+ identity providers out of the box</li>
<li><strong>SCIM provisioning</strong> — automated user lifecycle for enterprise customers</li>
<li><strong>Admin Portal</strong> — self-serve SSO setup for customer IT teams</li>
<li><strong>Directory Sync</strong> — sync from Active Directory, Okta, etc.</li>
<li><strong>Proven at scale</strong> — used by Discord, Perplexity, Plaid</li>
<li><strong>Free up to 1M MAUs</strong> — generous pricing for AuthKit</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>No billing included</strong> — still need Stripe separately (Clerk could consolidate)</li>
<li><strong>Less polished UI components</strong> — pre-built components aren't as refined as Clerk's</li>
<li><strong>No built-in fraud detection</strong> — need separate solution for bot/abuse prevention</li>
<li><strong>No built-in analytics</strong> — auth metrics require custom dashboards</li>
</ul>
</div>
</div>

---

## Clerk — Not Now

$50M Series C. Beautiful UI components. Recently added billing. Could consolidate auth + payments.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ What's Appealing</h4>
<ul>
<li><strong>Auth + Billing consolidation</strong> — could replace WorkOS + some Stripe functionality</li>
<li><strong>Best-in-class UI components</strong> — drop-in React components that look great immediately</li>
<li><strong>Excellent Next.js integration</strong> — built specifically for App Router</li>
<li><strong>Fraud prevention</strong> — built-in bot detection and abuse prevention</li>
<li><strong>Well-funded</strong> — $50M Series C, not going anywhere</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Why Skip</h4>
<ul>
<li><strong>Would abandon WorkOS migration</strong> — weeks of Chris's work wasted</li>
<li><strong>More expensive</strong> for enterprise SSO/SCIM features</li>
<li><strong>Less mature enterprise auth</strong> — WorkOS is purpose-built for enterprise</li>
<li><strong>Billing features are new/unproven</strong> — Stripe is battle-tested</li>
<li><strong>6+ month restart</strong> — complete migration reboot</li>
</ul>
</div>
</div>

---

## Stytch — Not Now

"Identity platform for humans & AI agents." Advanced MCP authentication toolkit.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ What's Appealing</h4>
<ul>
<li><strong>AI agent authentication</strong> — most advanced MCP auth toolkit available</li>
<li><strong>Multi-tenancy</strong> — native organization support</li>
<li><strong>Fraud detection</strong> — real-time abuse protection</li>
<li><strong>Forward-thinking</strong> — positioned for AI-first future</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Why Skip</h4>
<ul>
<li><strong>AI agent auth is premature</strong> — not a current need for Brev</li>
<li><strong>Smaller company</strong> — less proven at enterprise scale than WorkOS/Clerk</li>
<li><strong>Would abandon WorkOS migration</strong> — same problem as Clerk</li>
<li><strong>6+ month restart</strong> — complete migration reboot</li>
</ul>
</div>
</div>

---

## Comparison

| Feature | WorkOS (Current) | Clerk | Stytch |
|---------|:-:|:-:|:-:|
| **Enterprise SSO** | ✅ Best-in-class | ✅ Good | ✅ Good |
| **SCIM** | ✅ | ✅ | ✅ |
| **Billing** | ❌ (need Stripe) | ✅ New | ❌ |
| **UI Components** | ⚠️ Functional | ✅ Beautiful | ⚠️ Functional |
| **AI Agent Auth** | ❌ | ❌ | ✅ |
| **Next.js** | ✅ | ✅ Best | ✅ |
| **Migration effort** | In progress | 6+ months | 6+ months |

## Decision

**Finish the WorkOS migration.** Don't disrupt active work for marginal improvements. Revisit in 6 months:
- If Clerk's billing matures → could consolidate auth + payments
- If AI agent auth becomes critical → Stytch's MCP toolkit could be relevant
- WorkOS is also evolving → may add AI agent features
