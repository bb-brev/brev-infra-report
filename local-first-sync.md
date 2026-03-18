---
layout: default
title: "Local-First Sync — Zero / PowerSync"
---

[← Back to Overview](./)

# Local-First Sync: Zero vs PowerSync vs Current RxDB

**Verdict:** Skip for now — revisit when Zero reaches GA  
**Migration Effort:** 6-12 months (massive)  
**Risk:** High

---

## Current State

Brev's local-first stack:
- **RxDB** with IndexedDB storage as the primary client-side database
- **TanStack DB** for reactive queries over RxDB collections
- **TanStack Query** for server-originated async data (auth, billing, config)
- **Custom replication** to Aurora PostgreSQL

This stack is already implemented and working. The question is whether newer alternatives offer enough improvement to justify a complete rewrite.

---

## Zero (by Rocicorp)

**What it is:** A query-driven sync engine that maintains a client-side store of recently used rows and syncs exactly the data you need, when you need it.

### How It Works
- ZQL (Zero Query Language) for client-side queries
- Query-driven sync: only syncs data that active queries request
- Server-side permissions and business logic
- Built for PostgreSQL — works with Aurora

### Key Benefits
- **No custom replication logic** — sync is handled by the framework
- **Query-driven** — avoids "download everything" problem
- **Instant reads/writes** — client-first with automatic reconciliation
- **Proven concept** — 1.2M row demo loads in <2s

### Current Status
- **Private beta** with managed hosting
- Pricing: Hobby $30/month → BYOC $1000+/month (TBD)
- No public GA date announced

### Pros
| Benefit | Details |
|---------|---------|
| Eliminates custom replication | No more maintaining sync logic |
| Works with Aurora Postgres | Direct Postgres integration |
| Smart sync boundaries | Only syncs what's needed |
| Client performance | Proven fast at scale |

### Cons
| Risk | Details |
|------|---------|
| **Private beta** | No production guarantees |
| **Unknown pricing** | Could be expensive at scale |
| **Full rewrite required** | Every client-side data access pattern changes |
| **ZQL learning curve** | New query language for the team |
| **Replaces TanStack DB** | Lose existing investment in TanStack patterns |
| **12+ month effort** | Including testing across all integrations |

---

## PowerSync

**What it is:** PostgreSQL-to-SQLite sync engine with client SDKs for managing local persistence and real-time syncing.

### How It Works
- Uses SQLite on the client (via wa-sqlite in browser)
- Syncs from Postgres via change data capture
- Sync rules defined server-side
- Offline-first with conflict resolution

### Key Benefits
- **SQLite on client** — more predictable than IndexedDB
- **Open source** (source-available core)
- **Mature** — not in beta
- **Direct Postgres integration** — works with Aurora

### Pros
| Benefit | Details |
|---------|---------|
| Production-ready | More mature than Zero |
| Open source | Source-available, self-hostable |
| SQLite reliability | More predictable than IndexedDB |
| Good JS/TS SDK | Strong TypeScript support |

### Cons
| Risk | Details |
|------|---------|
| **SQLite browser limits** | Size constraints vs IndexedDB |
| **Data modeling changes** | Significant rewrite needed |
| **TanStack DB incompatible** | Different query patterns |
| **Requires PowerSync service** | Additional infrastructure |
| **Smaller ecosystem** | Less community support than RxDB |

---

## Current RxDB Approach

### Why Keep It

| Factor | Details |
|--------|---------|
| **Already working** | Implemented, tested, in production |
| **Team knowledge** | Team knows the patterns and quirks |
| **TanStack integration** | TanStack DB + TanStack Query are well-integrated |
| **Proven scale** | Working at current user base |
| **No migration risk** | No disruption to existing users |

### Known Pain Points (Evaluate)

- Custom replication logic maintenance burden
- IndexedDB browser compatibility quirks
- Sync complexity grows with data model complexity
- Potential performance issues at scale

---

## Comparison Matrix

| Factor | RxDB (Current) | Zero | PowerSync |
|--------|----------------|------|-----------|
| **Status** | Production | Private Beta | Production |
| **Migration Effort** | None | 6-12 months | 4-8 months |
| **Risk** | None | High | Medium |
| **Postgres Integration** | Custom | Native | Native |
| **Offline Support** | ✅ | ✅ | ✅ |
| **Query Language** | RxDB/TanStack | ZQL | SQL |
| **Custom Replication** | Required | Not needed | Not needed |
| **Team Knowledge** | ✅ | ❌ | ❌ |
| **Self-Hostable** | ✅ | ❌ (managed) | ✅ |

## Recommendation

**Keep RxDB for now.** The migration effort (6-12 months) for either alternative is enormous and the current stack works. Revisit when:

1. Zero reaches public GA with clear pricing
2. Custom replication becomes a significant maintenance burden
3. You hit scaling limits with the current approach
4. A major feature requires capabilities the current stack can't provide

**Don't fix what isn't broken** — especially with a 6-12 month rewrite risk.
