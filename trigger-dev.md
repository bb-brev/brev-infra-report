---
layout: default
title: "Trigger.dev"
---

<h1>Trigger.dev <span class="gradient">Background Jobs</span></h1>
<span class="verdict evaluate">⚡ Evaluate — Pilot 2-3 Queues</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">SQS + Lambda + DLQs</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">8-12 weeks (full)</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--yellow)">Medium</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Q2-Q3 2026</span></div>
</div>

## What It Is

[Trigger.dev](https://trigger.dev) is an open-source (Apache 2.0) TypeScript-native background jobs framework. v3 provides built-in queuing, retries, concurrency control, and a real-time dashboard. **Raised $16M Series A** — well-funded and actively developed.

## What It Replaces

Brev currently runs **15+ SQS queues**, each with a Lambda consumer and a Dead Letter Queue. That's ~45 infrastructure resources just for background jobs.

| | Current (SQS + Lambda) | Trigger.dev |
|---|---|---|
| **Queue definition** | SST `sst.aws.Queue` + DLQ | TypeScript `task()` function |
| **Consumer** | Lambda handler (30s–15min timeout) | Task `run()` (no timeout) |
| **Retry logic** | DLQ + separate DLQ consumer | Built-in exponential backoff |
| **Monitoring** | CloudWatch logs | Real-time dashboard |
| **Concurrency** | SQS visibility timeout | Per-task concurrency config |
| **Task chaining** | Publish to another SQS queue | `triggerAndWait()` |

## Current Flow vs Proposed

```
CURRENT:
Upload → S3 → SQS (RawFileProcessor, 15min timeout)
       → Lambda → direct openai.chat.completions.create()
       → on failure → DLQ → DLQ Lambda consumer
       → on success → SQS (GoalArtgen, 10min timeout) → Lambda → ...

PROPOSED:
Upload → S3 → Trigger.dev task (rawFileProcessor, no timeout)
       → generateText() via AI SDK (auto-retried)
       → on failure → automatic retry with backoff (visible in dashboard)
       → on success → triggerAndWait(goalArtgen) → ...
```

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>No 15-minute timeout</strong> — RawFileProcessor, GoalArtgen, ViewArtgen all constrained by Lambda limit today</li>
<li><strong>Eliminates ~30 DLQ resources</strong> — retries with exponential backoff built into every task</li>
<li><strong>Real-time visibility</strong> — see task progress, queued items, failures in a dashboard instead of CloudWatch</li>
<li><strong>TypeScript-native</strong> — full type safety on payloads, same language as the rest of the stack</li>
<li><strong>Task chaining</strong> — <code>triggerAndWait()</code> for pipelines (transcript → summary → artifacts) without managing SQS pub/sub</li>
<li><strong>Concurrency control</strong> — set per-task limits without SQS visibility timeout hacks</li>
<li><strong>Open source</strong> — self-hostable on AWS if vendor risk is a concern</li>
<li><strong>Dev experience</strong> — local development with <code>npx trigger.dev dev</code>, hot reload</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>VPC access is a blocker</strong> — Trigger.dev tasks run on their infra, but Aurora is VPC-locked. Must self-host or use RDS Proxy</li>
<li><strong>Cost model shift</strong> — SQS ≈ $0.40/M requests (near-free). Trigger.dev charges per compute-second. At 15+ busy queues, costs add up</li>
<li><strong>No SST construct</strong> — managed outside SST infrastructure, or self-hosted on ECS</li>
<li><strong>Migration scope</strong> — 15+ queues with custom consumer logic. Even incremental migration is months of work</li>
<li><strong>Simple queues don't benefit</strong> — SlackMessenger (30s), Gamification (30s) work fine on SQS</li>
<li><strong>Self-hosting adds ops</strong> — if you go self-hosted for VPC access, you're running more infrastructure</li>
</ul>
</div>
</div>

## Queue-by-Queue Assessment

| Queue | Timeout | Trigger.dev Benefit | Priority |
|-------|---------|-------------------|----------|
| **RawFileProcessor** | 15 min | 🔴 High — constrained by Lambda timeout, long transcripts | Pilot candidate |
| **GoalArtgen** | 10 min | 🟡 Medium — AI generation benefits from no timeout | Phase 2 |
| **ViewArtgen** | 10 min | 🟡 Medium — same as GoalArtgen | Phase 2 |
| **ScheduledAgentTrigger** | 12 min | 🟡 Medium — complex orchestration | Phase 2 |
| **WorkflowResume** | 12 min | 🟡 Medium — state management | Phase 2 |
| **Firmgen** | 2 min | 🟢 Low — works fine on Lambda | Phase 3 |
| **Nexus** | 5 min | 🟢 Low — works fine on Lambda | Phase 3 |
| **CheckinReminder** | 5 min | 🟢 Low — simple notification | Not worth migrating |
| **SlackMessenger** | 30s | ⚪ None — SQS is perfect for this | Don't migrate |
| **Gamification** | 30s | ⚪ None — SQS is perfect for this | Don't migrate |

## Critical Blocker: VPC Access

> Aurora PostgreSQL lives inside a VPC. Trigger.dev managed tasks run on their infrastructure. This must be resolved before any migration.

**Options:**

| Approach | Pros | Cons |
|----------|------|------|
| **Self-host on ECS** | Stays in VPC, full control | Ops burden, must maintain Trigger.dev infra |
| **RDS Proxy (public)** | Simple, works with managed Trigger.dev | Exposes DB endpoint (IAM auth mitigates risk) |
| **API layer** | Tasks call your API, not DB directly | Adds latency, requires API for every DB operation |
| **Trigger.dev Enterprise** | VPC peering (if available) | Enterprise pricing, unknown availability |

## Migration Plan

<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <h4>Week 1-2: Setup & VPC Resolution</h4>
    <p>Install Trigger.dev, resolve VPC access approach. Set up local dev with <code>npx trigger.dev dev</code>.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">2</span>
  <div class="step-content">
    <h4>Week 3-4: Pilot — RawFileProcessor</h4>
    <p>Migrate the highest-value queue. Run in parallel with existing SQS queue to compare reliability and performance.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">3</span>
  <div class="step-content">
    <h4>Week 5-8: Evaluate Results</h4>
    <p>Measure cost, reliability, DX improvement. Compare dashboard observability vs CloudWatch. Get team feedback.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">4</span>
  <div class="step-content">
    <h4>Week 9-12: Expand (if positive)</h4>
    <p>Migrate AI generation queues (GoalArtgen, ViewArtgen, Firmgen). Keep simple notification queues on SQS.</p>
  </div>
</div>

## Code Example

```typescript
// Before: infra/queue.ts (SST)
export const rawFileProcessorQueue = new sst.aws.Queue("RawFileProcessorQueue", {
  fifo: true,
  dlq: rawFileProcessorDLQ.arn,
  visibilityTimeout: "15 minutes",
});

rawFileProcessorQueue.subscribe({
  handler: "packages/functions/src/queue-consumers/RawFileProcessorConsumer.main",
  link: [brevDatabase, ...llmSecrets, transcriptsBucket],
  vpc: appVpc,
  timeout: "15 minutes",
});

// After: packages/functions/src/tasks/raw-file-processor.ts
import { task } from "@trigger.dev/sdk/v3";
import { generateText } from "ai";
import { openai } from "@ai-sdk/openai";

export const rawFileProcessor = task({
  id: "raw-file-processor",
  retry: { maxAttempts: 3, factor: 2, minTimeoutInMs: 1000 },
  run: async (payload: { s3Key: string; orgId: string }) => {
    const transcript = await downloadFromS3(payload.s3Key);
    const processed = await processTranscript(transcript);

    const { text: summary } = await generateText({
      model: openai("gpt-4o"),
      prompt: `Summarize this transcript: ${processed.text}`,
    });

    await db.insert(transcriptSummaries).values({
      orgId: payload.orgId,
      summary,
      transcriptId: processed.id,
    });

    // Chain to goal artifact generation
    await goalArtgen.triggerAndWait({ orgId: payload.orgId, transcriptId: processed.id });

    return { success: true };
  },
});
```

## Decision

**Adopt if:** VPC access is solvable, cost modeling shows <2x increase, pilot shows meaningful DX improvement on RawFileProcessor.

**Skip if:** VPC access requires exposing DB publicly with unacceptable risk, costs are significantly higher than SQS, or team bandwidth is too limited for an 8-12 week migration.
