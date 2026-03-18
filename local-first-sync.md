---
layout: default
title: "Local-First Sync"
---

<h1>Local-First Sync <span class="gradient">Zero / PowerSync</span></h1>
<span class="verdict skip">⏸ Skip — Current Stack Works</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">RxDB + TanStack DB + custom replication</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">6-12 months</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--red)">High</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Revisit Q4+</span></div>
</div>

## Current Stack

Brev's local-first layer:
- **RxDB** with IndexedDB — primary client-side database
- **TanStack DB** — reactive queries over RxDB collections
- **TanStack Query** — server-originated async data (auth, billing, config)
- **Custom replication** — syncs to Aurora PostgreSQL

This is **working in production**. The question is whether newer tools justify a complete rewrite.

---

## Zero (by Rocicorp)

A query-driven sync engine — only syncs rows that active queries need.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Eliminates custom replication</strong> — sync logic handled by the framework, no more maintaining custom sync code</li>
<li><strong>Query-driven sync</strong> — only downloads data that components actually request, avoiding "sync everything" overhead</li>
<li><strong>Works with Aurora Postgres</strong> — direct PostgreSQL integration, not Supabase-only</li>
<li><strong>Instant reads/writes</strong> — client-first operations with automatic server reconciliation</li>
<li><strong>Proven scale</strong> — 1.2M row demo loads in under 2 seconds</li>
<li><strong>Built by Replicache team</strong> — deep expertise in sync systems</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Private beta only</strong> — no public GA date, no production guarantees</li>
<li><strong>Unknown pricing</strong> — Hobby $30/mo, BYOC $1000+/mo (final pricing TBD)</li>
<li><strong>Complete client-side rewrite</strong> — every data access pattern changes from RxDB/TanStack to ZQL</li>
<li><strong>Replaces TanStack DB investment</strong> — lose all existing reactive query patterns</li>
<li><strong>ZQL learning curve</strong> — new query language for the entire team</li>
<li><strong>6-12 month migration</strong> — touches every component that reads/writes data</li>
<li><strong>Not self-hostable</strong> — managed service only (vendor lock-in risk)</li>
</ul>
</div>
</div>

---

## PowerSync

PostgreSQL-to-SQLite sync engine — uses SQLite (via wa-sqlite) on the client.

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Production-ready</strong> — not in beta, more mature than Zero</li>
<li><strong>Open source</strong> — source-available, self-hostable on AWS</li>
<li><strong>SQLite reliability</strong> — more predictable than IndexedDB in browsers</li>
<li><strong>Direct Postgres integration</strong> — change data capture from Aurora</li>
<li><strong>Strong TypeScript SDK</strong> — good developer experience</li>
<li><strong>Standard SQL queries</strong> — no new query language to learn</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>SQLite browser size limits</strong> — storage constraints compared to IndexedDB</li>
<li><strong>Significant data modeling changes</strong> — different schema patterns than RxDB</li>
<li><strong>TanStack DB incompatible</strong> — different query and reactivity patterns</li>
<li><strong>Requires PowerSync service</strong> — additional infrastructure to run/host</li>
<li><strong>Smaller ecosystem</strong> — less community support and fewer plugins than RxDB</li>
<li><strong>4-8 month migration</strong> — still a massive rewrite of the data layer</li>
</ul>
</div>
</div>

---

## Current RxDB — Why Keep It

<div class="pros-cons">
<div class="pros-card">
<h4>✅ What's Working</h4>
<ul>
<li><strong>Already in production</strong> — tested, deployed, serving users</li>
<li><strong>Team knowledge</strong> — developers know the patterns and edge cases</li>
<li><strong>TanStack integration</strong> — TanStack DB + Query well-integrated</li>
<li><strong>No migration risk</strong> — zero disruption to existing users</li>
<li><strong>Offline-first</strong> — IndexedDB provides reliable offline storage</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Known Pain Points</h4>
<ul>
<li><strong>Custom replication maintenance</strong> — sync logic requires ongoing engineering</li>
<li><strong>IndexedDB quirks</strong> — browser compatibility and performance inconsistencies</li>
<li><strong>Sync complexity</strong> — grows with data model complexity</li>
<li><strong>Conflict resolution</strong> — custom logic for concurrent edits</li>
</ul>
</div>
</div>

## Comparison

| Factor | RxDB (Current) | Zero | PowerSync |
|--------|:-:|:-:|:-:|
| **Production status** | ✅ Live | ⚠️ Private beta | ✅ GA |
| **Migration effort** | None | 6-12 months | 4-8 months |
| **Risk level** | None | High | Medium |
| **Custom replication** | Required | Not needed | Not needed |
| **Aurora Postgres** | Via custom sync | Native | Native |
| **Self-hostable** | ✅ | ❌ | ✅ |
| **Team knowledge** | ✅ | ❌ | ❌ |
| **Query language** | RxDB / TanStack | ZQL (new) | SQL |

## Decision

**Keep RxDB.** The effort-to-benefit ratio doesn't justify migration. Revisit when:
- Zero reaches public GA with stable pricing
- Custom replication becomes a meaningful engineering drain
- You hit scaling limits with the current approach
- A major feature needs capabilities RxDB can't provide

**Don't fix what isn't broken** — especially with a 6-12 month rewrite risk.
