---
layout: default
title: "Vercel AI SDK — Unified LLM Interface"
---

[← Back to Overview](./)

# Vercel AI SDK — Unifying Backend + Frontend

**Verdict:** ✅ Recommend — easiest win of the entire report  
**Migration Effort:** 2-3 weeks  
**Risk:** Low

---

## What It Is

The [Vercel AI SDK](https://sdk.vercel.ai) (`ai` package) is a provider-agnostic TypeScript toolkit for building AI applications. It provides a unified interface for calling any LLM (OpenAI, Anthropic, Google, etc.) with streaming, structured output, and tool calling — all through one API.

**Key fact:** It's open source and runtime-agnostic. Despite the name, it does **not** require Vercel hosting. Works in Lambda, Node.js, Edge, anywhere.

## What It Replaces

**Current state:**
- **Backend (Lambda consumers):** Direct `openai` SDK + `@anthropic-ai/sdk` — different APIs, different patterns for each provider
- **Frontend:** Already using `@ai-sdk/react` and `@ai-sdk/google` (partial adoption)

**Proposed:** Use Vercel AI SDK everywhere — single abstraction for all LLM calls.

## Migration Example

### Before (Direct SDKs)

```typescript
// OpenAI call in a Lambda consumer
import OpenAI from "openai";
const openai = new OpenAI({ apiKey: Resource.OpenaiKey.value });

const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: prompt }],
  temperature: 0.7,
});
const text = response.choices[0].message.content;

// Anthropic call in a different consumer
import Anthropic from "@anthropic-ai/sdk";
const anthropic = new Anthropic({ apiKey: Resource.AnthropicKey.value });

const message = await anthropic.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 4096,
  messages: [{ role: "user", content: prompt }],
});
const text = message.content[0].type === "text" ? message.content[0].text : "";
```

### After (AI SDK)

```typescript
import { generateText, generateObject } from "ai";
import { openai } from "@ai-sdk/openai";
import { anthropic } from "@ai-sdk/anthropic";

// Same API regardless of provider
const { text } = await generateText({
  model: openai("gpt-4o"),  // or anthropic("claude-sonnet-4-20250514")
  prompt,
  temperature: 0.7,
});

// Structured output with Zod — no JSON.parse needed
import { z } from "zod";
const { object: summary } = await generateObject({
  model: anthropic("claude-sonnet-4-20250514"),
  schema: z.object({
    title: z.string(),
    keyPoints: z.array(z.string()),
    actionItems: z.array(z.object({
      owner: z.string(),
      task: z.string(),
      dueDate: z.string().optional(),
    })),
    sentiment: z.enum(["positive", "neutral", "negative"]),
  }),
  prompt: `Summarize this meeting transcript: ${transcript}`,
});
// summary is fully typed — no casting, no parsing
```

## Key Benefits for Brev

### 1. Provider Switching in One Line
```typescript
// Testing a new model? Change one line:
const model = openai("gpt-4o");        // → anthropic("claude-sonnet-4-20250514")
const model = anthropic("claude-sonnet-4-20250514");  // → google("gemini-2.0-flash")
```
Critical for Brev — you already use both OpenAI and Anthropic across different consumers. This lets you A/B test models per-feature without rewriting integration code.

### 2. Structured Output with Zod
No more `JSON.parse()` + manual validation. `generateObject()` returns typed objects validated against your Zod schema. You already use Zod in the codebase — this is a natural fit.

### 3. Streaming (Server + Client)
```typescript
// Server Component or Lambda
const result = streamText({ model: openai("gpt-4o"), prompt });
// Returns a ReadableStream — works in Lambda, Server Actions, Route Handlers

// Client (already using @ai-sdk/react)
const { messages, append } = useChat(); // Already adopted
```

### 4. Tool Calling (Unified)
```typescript
const result = await generateText({
  model: openai("gpt-4o"),
  tools: {
    getGoalProgress: tool({
      description: "Get current progress for a goal",
      parameters: z.object({ goalId: z.string() }),
      execute: async ({ goalId }) => await db.getGoalProgress(goalId),
    }),
  },
  prompt: "What's the progress on our Q2 revenue goal?",
});
```

### 5. Native Langfuse Integration
If you adopt Langfuse later, every AI SDK call is auto-traced. Zero additional instrumentation code.

## Pros

| Benefit | Details |
|---------|---------|
| **Provider-agnostic** | Swap OpenAI ↔ Anthropic ↔ Google without code changes |
| **Structured output** | `generateObject()` + Zod = typed responses, no JSON.parse |
| **Already partially adopted** | Frontend uses `@ai-sdk/react` — backend is additive |
| **Runtime-agnostic** | Works in Lambda — not locked to Vercel hosting |
| **Streaming primitives** | Consistent streaming in Server Components, Actions, and Lambdas |
| **40k+ GitHub stars** | Widely adopted, actively maintained, strong ecosystem |
| **Open source** | MIT license, no vendor lock-in |
| **Incremental migration** | Adopt in new consumers, backfill old ones over time |

## Cons

| Risk | Severity | Details |
|------|----------|---------|
| **Abstraction overhead** | 🟢 Low | Thin wrapper — minimal performance cost |
| **Provider-specific features** | 🟡 Medium | Advanced features (Anthropic prompt caching, extended thinking) may lag behind direct SDK. AI SDK usually catches up within weeks. |
| **Dependency on Vercel org** | 🟢 Low | Open source, MIT. Even if Vercel deprioritizes, community can maintain |
| **Learning curve** | 🟢 Low | API is simpler than direct SDKs — team will be faster, not slower |

## Works in Lambda? Yes.

The core `ai` package uses standard Node.js APIs. No Vercel-specific runtime requirements.

```typescript
// Lambda handler using AI SDK — works fine
export const handler = async (event: SQSEvent) => {
  const { text } = await generateText({
    model: openai("gpt-4o"),
    prompt: event.Records[0].body,
  });
  // ... store result in Aurora
};
```

## Migration Plan

### Week 1: Setup + New Code
- Install `ai`, `@ai-sdk/openai`, `@ai-sdk/anthropic`
- Use AI SDK for all new LLM calls
- Add SST secret links for provider keys (already exist)

### Week 2: Migrate High-Value Consumers
- `RawFileProcessorConsumer` — heaviest AI usage
- `FirmgenQueueConsumer`
- `GoalArtgenQueueConsumer`
- `ViewArtgenQueueConsumer`

### Week 3: Remaining Consumers + Cleanup
- `NexusQueueConsumer`
- `SlackBotMessageConsumer`
- Remove direct `openai` and `@anthropic-ai/sdk` imports
- Update tests

## Decision

This is the **lowest-risk, highest-return** change in the entire report. Already partially adopted on the frontend. Backend migration is incremental and non-breaking. Start immediately.
