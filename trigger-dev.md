---
layout: default
title: "Trigger.dev — Background Jobs"
---

[← Back to Overview](./)

# Trigger.dev vs SQS + Lambda

**Verdict:** Evaluate — pilot on 2-3 queues in Q2-Q3  
**Migration Effort:** 8-12 weeks (full), 2-3 weeks (pilot)  
**Risk:** Medium

---

## What It Is

[Trigger.dev](https://trigger.dev) is an open-source (Apache 2.0) TypeScript-native background jobs framework. v3 architecture provides built-in queuing, retries, concurrency control, and a real-time observability dashboard. Raised $16M Series A — well-funded and actively developed.

## What It Replaces

Brev's current **15+ SQS queues + Lambda consumers + DLQs**:

| Current Pattern | Trigger.dev Pattern |
|----------------|-------------------|
| Define SQS queue in SST | Define task in TypeScript |
| Write Lambda consumer function | Write task `run()` function |
| Configure DLQ separately | Retries built into task config |
| Set visibility timeout (30s–15min) | No timeout limit |
| Monitor via CloudWatch | Built-in real-time dashboard |
| Manual concurrency via SQS settings | Concurrency control per task |

### Current Architecture
```
Event → SQS Queue → Lambda Consumer (30s–15min timeout) → DLQ on failure
```

### Trigger.dev Architecture
```
Event → trigger.dev task (typed, retries built-in, no timeout, live dashboard)
```

## Migration: Queue by Queue

| Brev Queue | Current Timeout | Trigger.dev Benefit | Complexity |
|------------|-----------------|---------------------|------------|
| **RawFileProcessor** | 15 min | No timeout — best candidate for pilot | Medium |
| **Firmgen** | 2 min | Better retry logic, observability | Medium |
| **GoalArtgen** | 10 min | Long AI generation, benefits from no timeout | Medium |
| **ViewArtgen** | 10 min | Same as GoalArtgen | Medium |
| **Nexus** | 5 min | Complex pipeline, benefits from task chaining | Medium |
| **TranscriptProcessingUpdates** | 30s | Simple event — low benefit, low risk pilot | Low |
| **CheckinReminder** | 5 min | Simple notification task | Low |
| **SlackMessenger** | 30s | Simple — SQS is fine here | Low |
| **SlackBotMessage** | 2 min | AI-powered — moderate benefit | Low |
| **CalendarSync** | 5 min | API integration — moderate benefit | Low |
| **ScheduledAgentTrigger** | 12 min | Complex orchestration — high benefit | Medium |
| **WorkflowResume** | 12 min | State management — high benefit | Medium |
| **Gamification** | 30s | Simple — SQS is fine | Low |
| **WebRevalidation** | 30s | Simple — SQS is fine | Low |

### Recommended Pilot: Start with `RawFileProcessor`
- Highest timeout (15 min), most constrained by Lambda limits
- Transcript processing benefits most from unlimited runtime
- Clear before/after comparison

### Code Example

**Before (SQS + Lambda):**
```typescript
// infra/queue.ts
export const rawFileProcessorQueue = new sst.aws.Queue("RawFileProcessorQueue", {
  fifo: true,
  dlq: rawFileProcessorDLQ.arn,
  visibilityTimeout: "15 minutes",
});

rawFileProcessorQueue.subscribe({
  handler: "packages/functions/src/queue-consumers/RawFileProcessorConsumer.main",
  link: [brevDatabase, ...llmSecrets, transcriptsBucket, ...],
  vpc: appVpc,
  timeout: "15 minutes",
});
```

**After (Trigger.dev):**
```typescript
import { task } from "@trigger.dev/sdk/v3";

export const rawFileProcessor = task({
  id: "raw-file-processor",
  retry: { maxAttempts: 3, factor: 2, minTimeoutInMs: 1000 },
  machine: { preset: "large-1x" },
  run: async (payload: { s3Key: string; userId: string; orgId: string }) => {
    // No timeout limit — can process hour-long transcripts
    const transcript = await downloadFromS3(payload.s3Key);
    const processed = await processTranscript(transcript);
    const summary = await generateAISummary(processed);
    await updateGoalArtifacts(payload.orgId, summary);
    return { success: true, transcriptId: processed.id };
  },
});
```

## Pros

| Benefit | Impact for Brev |
|---------|----------------|
| **No timeout limit** | RawFileProcessor, GoalArtgen, ViewArtgen all constrained by 15-min Lambda limit |
| **Built-in retries** | Eliminates 15+ DLQ definitions and DLQ consumer handlers |
| **TypeScript-native** | Full type safety on payloads, same language as rest of stack |
| **Real-time dashboard** | See task progress vs black-box CloudWatch logs |
| **Concurrency control** | Per-task concurrency limits without SQS visibility timeout hacks |
| **Task chaining** | `triggerAndWait()` for pipelines (transcript → summary → artifacts) |
| **Open source** | Can self-host if vendor risk is a concern |

## Cons

| Risk | Severity | Mitigation |
|------|----------|------------|
| **VPC access** | 🔴 High | Trigger.dev tasks run on their infra, not in your VPC. Aurora is VPC-locked. Need self-hosted option OR connection proxy (e.g., AWS RDS Proxy with public endpoint) |
| **Cost model shift** | 🟡 Medium | SQS ≈ $0.40/M requests. Trigger.dev charges per compute-second. At 15+ queues, costs add up. Model carefully before full migration |
| **No SST construct** | 🟡 Medium | No official `sst.aws.TriggerDev`. Manage outside SST or self-host on ECS |
| **Migration scope** | 🟡 Medium | 15+ queues with custom consumer logic. Incremental migration works but is a multi-month effort |
| **Vendor dependency** | 🟢 Low | Open source, self-hostable. $16M funding provides runway |

## Critical Blocker: VPC Access

Your Aurora DB lives inside a VPC. Trigger.dev's managed tasks run on their infrastructure. Options:

1. **Self-host Trigger.dev on ECS** — keeps everything in VPC, but adds ops burden
2. **RDS Proxy with public endpoint** — exposes DB (with IAM auth) outside VPC
3. **API layer** — tasks call your API endpoints instead of direct DB access (adds latency)
4. **Trigger.dev Enterprise** — may offer VPC peering (check with their team)

This is the #1 thing to resolve before committing.

## Recommended Migration Strategy

1. **Week 1-2:** Set up Trigger.dev, resolve VPC access approach
2. **Week 3-4:** Migrate `RawFileProcessor` as pilot
3. **Week 5-8:** Evaluate pilot results — cost, reliability, DX
4. **Week 9-12:** If positive, migrate AI generation queues (Firmgen, GoalArtgen, ViewArtgen)
5. **Later:** Simple notification queues (low priority — SQS works fine for these)

## Decision Criteria

✅ **Adopt if:** VPC access is solvable, cost modeling shows <2x increase, pilot shows meaningful DX improvement  
❌ **Skip if:** VPC access requires exposing DB publicly, costs are significantly higher, team bandwidth is limited
