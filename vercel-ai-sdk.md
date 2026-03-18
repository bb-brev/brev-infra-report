---
layout: default
title: "Vercel AI SDK"
---

<h1>Vercel AI SDK <span class="gradient">Unified LLM Interface</span></h1>
<span class="verdict recommend">✅ Recommend — Easiest Win</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">Direct OpenAI + Anthropic SDKs</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">2-3 weeks</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--green)">Low</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Now</span></div>
</div>

## What It Is

The [Vercel AI SDK](https://sdk.vercel.ai) is a provider-agnostic TypeScript toolkit for LLM applications. One API for calling OpenAI, Anthropic, Google, and others — with streaming, structured output, and tool calling built in.

**Key fact:** Open source, MIT license. Works in Lambda, Node.js, Edge — does **not** require Vercel hosting despite the name.

## What It Replaces

| | Current | With AI SDK |
|---|---|---|
| **Backend (Lambda)** | `openai` SDK + `@anthropic-ai/sdk` — different APIs per provider | Single `generateText()` / `generateObject()` API |
| **Frontend** | Already using `@ai-sdk/react` + `@ai-sdk/google` | No change needed — already adopted |
| **Provider switching** | Rewrite integration code per provider | Change one line: `openai("gpt-4o")` → `anthropic("claude-sonnet-4-20250514")` |
| **Structured output** | `JSON.parse()` + manual validation | `generateObject()` with Zod schema — fully typed |
| **Streaming** | Different per provider | Unified `streamText()` everywhere |

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros</h4>
<ul>
<li><strong>Provider-agnostic</strong> — swap OpenAI ↔ Anthropic ↔ Google without changing call sites</li>
<li><strong>Structured output with Zod</strong> — <code>generateObject()</code> returns typed objects, no <code>JSON.parse()</code></li>
<li><strong>Already partially adopted</strong> — frontend uses <code>@ai-sdk/react</code>, backend is additive</li>
<li><strong>Works in Lambda</strong> — runtime-agnostic, no Vercel hosting required</li>
<li><strong>Streaming primitives</strong> — consistent streaming across Server Components, Actions, and Lambdas</li>
<li><strong>Tool calling</strong> — unified tool definition with Zod parameter schemas across all providers</li>
<li><strong>40k+ GitHub stars</strong> — widely adopted, actively maintained, strong ecosystem</li>
<li><strong>Native Langfuse integration</strong> — if you adopt Langfuse later, every AI SDK call is auto-traced</li>
<li><strong>Incremental migration</strong> — adopt in new consumers, backfill old ones over time</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons</h4>
<ul>
<li><strong>Provider-specific features may lag</strong> — Anthropic prompt caching, extended thinking may take weeks to appear in AI SDK</li>
<li><strong>Abstraction dependency</strong> — if Vercel deprioritizes, you maintain a wrapper (mitigated: MIT license, huge community)</li>
<li><strong>Thin but present overhead</strong> — additional layer between your code and the provider (minimal perf impact)</li>
<li><strong>Team relearning</strong> — developers familiar with direct SDKs need to learn AI SDK patterns (low barrier)</li>
</ul>
</div>
</div>

## Before & After

### Direct SDKs (Current)

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

// Anthropic call in a DIFFERENT consumer — different API
import Anthropic from "@anthropic-ai/sdk";
const anthropic = new Anthropic({ apiKey: Resource.AnthropicKey.value });

const message = await anthropic.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 4096,
  messages: [{ role: "user", content: prompt }],
});
const text = message.content[0].type === "text" ? message.content[0].text : "";
```

### AI SDK (Proposed)

```typescript
// Same API for both providers
import { generateText, generateObject } from "ai";
import { openai } from "@ai-sdk/openai";
import { anthropic } from "@ai-sdk/anthropic";

// Simple text generation — works with ANY provider
const { text } = await generateText({
  model: openai("gpt-4o"),         // swap to anthropic("claude-sonnet-4-20250514") in one line
  prompt,
  temperature: 0.7,
});

// Structured output — typed result, no JSON.parse
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
// summary.title, summary.keyPoints, etc. — fully typed
```

## Migration Plan

<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <h4>Week 1: Setup + New Code</h4>
    <p>Install <code>ai</code>, <code>@ai-sdk/openai</code>, <code>@ai-sdk/anthropic</code>. Use AI SDK for all new LLM calls going forward. SST secret links already exist for provider keys.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">2</span>
  <div class="step-content">
    <h4>Week 2: Migrate High-Value Consumers</h4>
    <p>Convert the heaviest AI consumers: <code>RawFileProcessorConsumer</code>, <code>FirmgenQueueConsumer</code>, <code>GoalArtgenQueueConsumer</code>, <code>ViewArtgenQueueConsumer</code>.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">3</span>
  <div class="step-content">
    <h4>Week 3: Remaining + Cleanup</h4>
    <p>Migrate <code>NexusQueueConsumer</code>, <code>SlackBotMessageConsumer</code>. Remove direct <code>openai</code> and <code>@anthropic-ai/sdk</code> imports. Update tests.</p>
  </div>
</div>

## Works in Lambda — Confirmed

The `ai` package uses standard Node.js APIs. No Vercel-specific requirements.

```typescript
// Lambda handler — works fine
import { SQSEvent } from "aws-lambda";
import { generateText } from "ai";
import { openai } from "@ai-sdk/openai";

export const main = async (event: SQSEvent) => {
  for (const record of event.Records) {
    const { text } = await generateText({
      model: openai("gpt-4o"),
      prompt: record.body,
    });
    await saveToAurora(text);
  }
};
```

## Decision

Lowest-risk, highest-return change in the entire report. Already partially adopted on the frontend. Backend migration is incremental, non-breaking, and reversible. **Start immediately.**
