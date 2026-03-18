---
layout: default
title: "Arcjet"
---

<h1>Security Middleware <span class="gradient">Arcjet</span></h1>
<span class="verdict skip">⏸ Skip — Cloudflare Covers This</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">Nothing (additive)</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">1-2 weeks</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--yellow)">Medium</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">—</span></div>
</div>

## What It Is

[Arcjet](https://arcjet.com) — AI-powered runtime security for Next.js. Rate limiting, bot detection, PII detection, email validation at the middleware level.

## Current State

Brev uses **Cloudflare Turnstile** for bot protection + Cloudflare CDN for DDoS protection at the edge.

## What Arcjet Adds vs Cloudflare

| Feature | Cloudflare (Current) | Arcjet |
|---------|:-:|:-:|
| **Bot detection** | ✅ Turnstile (challenge) | ✅ AI-powered runtime |
| **DDoS protection** | ✅ Built-in | ❌ Not its focus |
| **Rate limiting** | ✅ WAF rules | ✅ Per-route, code-level |
| **PII detection** | ❌ | ✅ Detect PII in requests |
| **Email validation** | ❌ | ✅ Validate signups |
| **CDN/Edge** | ✅ Full CDN | ❌ App-level only |

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Native Next.js middleware</strong> — per-route security in code, not WAF config</li>
<li><strong>PII detection</strong> — useful for compliance (SOC 2, GDPR)</li>
<li><strong>Email validation</strong> — block disposable emails at signup</li>
<li><strong>Quick implementation</strong> — 1-2 days for basic setup</li>
<li><strong>Defense in depth</strong> — additional layer beyond Cloudflare</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Largely duplicates Cloudflare</strong> — bot protection and rate limiting already handled at edge</li>
<li><strong>New/unproven company</strong> — limited production track record</li>
<li><strong>Performance overhead</strong> — middleware check on every request adds latency</li>
<li><strong>May worsen Lambda cold starts</strong> — additional initialization in middleware</li>
<li><strong>Unclear pricing</strong> — not well-documented</li>
<li><strong>One more dependency</strong> — another moving part to debug when things go wrong</li>
</ul>
</div>
</div>

## Decision

**Skip.** Cloudflare provides strong edge security. The unique value (PII detection, email validation) is niche and not a current priority.

**Revisit if:** PII compliance becomes a requirement, or you need per-route rate limiting that Cloudflare WAF can't express cleanly.
