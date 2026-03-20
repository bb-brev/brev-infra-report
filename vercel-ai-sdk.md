---
layout: default
title: "Vercel AI SDK + Prompt Management Dashboard"
---

<h1>Vercel AI SDK <span class="gradient">Unified LLM Interface + Prompt Management Dashboard</span></h1>
<span class="verdict recommend">✅ Recommend — Easiest Win</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">Direct OpenAI + Anthropic SDKs</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">3-4 weeks (migration + dashboard)</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--green)">Low</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Now</span></div>
</div>

## Current State (Updated March 2026)

> **This assessment has been updated to reflect what already exists.** The original report understated Brev's current capabilities. See [What Already Exists](#what-already-exists) below.

### Frontend — Already Adopted ✅

The frontend (`packages/frontend`) already uses AI SDK:
- `ai` (v6.x) — core SDK
- `@ai-sdk/react` — React hooks for streaming UI
- `@ai-sdk/google` — Google provider

**No frontend migration needed.**

### Backend — Still on Direct SDKs

`packages/functions/src/llm/` uses provider-specific SDKs across **10 files**:

| File | SDK | Complexity |
|------|-----|------------|
| `TranscriptBroker.ts` | `openai` | High — 7+ methods, heaviest consumer |
| `GoalGenerationBroker.ts` | `openai` | High — goal + view artifact generation |
| `OnboardingBroker.ts` | `openai` | Medium |
| `OpenAIBroker.ts` | `openai` | Low — facade/factory |
| `OpenAIAgentProvider.ts` | `openai` | Medium — agent chat + tool calling + streaming |
| `AnthropicAgentProvider.ts` | `@anthropic-ai/sdk` | Medium — agent chat + tool calling |
| `GoogleGenAIBroker.ts` | `google` | Low |
| `SheetRowEnricher.ts` | `openai` | Low — batch row enrichment |
| `StreamingGoalBroker.ts` | `openai` | Medium — streaming generation |
| `llm-stream-consumer.ts` | `openai` | Medium — queue consumer |

### What Already Exists

**The original report implied Brev needs to build prompt management from scratch. That's wrong.** Brev already has ~70% of a prompt management dashboard:

**Prompt Lab** (`packages/internal/app/(authenticated)/prompts-lab/`):
- Prompt listing with runtime model labels
- Per-prompt profile pages with metadata, versioning, parameter configuration
- **Test panel** — select version, override model, fill parameters, run, see output + tokens + cost + duration
- **Replay dialog** — rehydrate parameters from a tracing log entry, re-run with a different version or model
- Tracing logs table per prompt

**LLM Tracing** (`LlmTracingLogDbRepository`):
- Every LLM call is logged: entity type/UUID, stage name, prompt ID + version + snapshot
- Full LLM response, provider, model, tokens in/out, cost (cents), duration
- Status, error, source (`production` / `test`), triggered-by, context metadata
- Organization + user UUID scoping

**Prompts Registry** (`PromptsRegistryBroker`):
- Dynamic prompt fetching from DB with `{% raw %}{{parameter}}{% endraw %}` substitution
- Metadata for tracing integration
- Version management

**Model Config** (`model-config.ts`):
- Centralized processing stage → model mapping
- Feature-key-to-stage resolution
- Runtime model selection

**Transcript Tracing Visualization**:
- D3-based DAG showing transcript → stages → chapters → goals → insights
- Node detail panels with timing and output data

---

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
<li><strong>Incremental migration</strong> — adopt in new consumers, backfill old ones over time</li>
<li><strong>Unlocks dashboard model-switching</strong> — Prompt Lab test panel and replay become truly provider-agnostic after migration</li>
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

// Anthropic call — completely different API
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

---

## Implementation Plan

### Phase 1: AI SDK Backend Migration (1.5–2 weeks)

This is the prerequisite. Without it, model switching in the dashboard only works for OpenAI-compatible models.

<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <h4>Setup — AI SDK Client Layer</h4>
    <p>Install <code>@ai-sdk/openai</code> + <code>@ai-sdk/anthropic</code> in <code>packages/functions</code>. Create <code>packages/functions/src/llm/ai-sdk-client.ts</code> — thin wrapper that initializes providers with SST Resource keys. Pin to the same major version as frontend (<code>ai@^6.x</code>).</p>
  </div>
</div>
<div class="step">
  <span class="step-num">2</span>
  <div class="step-content">
    <h4>Week 1: Heavy Consumers</h4>
    <p>Migrate <code>TranscriptBroker.ts</code> (7+ methods, highest coverage of tracing patterns) and <code>GoalGenerationBroker.ts</code>. These two files cover the majority of LLM calls. Keep tracing integration intact — AI SDK's <code>onFinish</code> callback maps cleanly to <code>LlmTracingLogDbRepository.createTracingLog()</code>.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">3</span>
  <div class="step-content">
    <h4>Week 2: Agent Providers + Remaining</h4>
    <p>Migrate <code>OpenAIAgentProvider.ts</code>, <code>AnthropicAgentProvider.ts</code> (unify into single provider), <code>OnboardingBroker.ts</code>, <code>SheetRowEnricher.ts</code>, <code>StreamingGoalBroker.ts</code>, <code>llm-stream-consumer.ts</code>. Update <code>OpenAIBroker.ts</code> facade to create AI SDK providers. Remove direct <code>openai</code> and <code>@anthropic-ai/sdk</code> from <code>packages/functions/package.json</code>.</p>
  </div>
</div>

### Phase 2: Prompt Management Dashboard Enhancements (1.5–2 weeks)

Building on the existing Prompt Lab. All data sources already exist in `llm_tracing_log`.

#### 2A: Token Usage Dashboard (2–3 days)

**What:** Aggregate view of LLM spend across all stages, models, and time periods.

**Data source:** `llm_tracing_log` table — already has `tokens_input`, `tokens_output`, `cost_total_cents`, `model_used`, `stage_name`, `created_at`.

**Build:**
- New API route: `packages/internal/app/internal-api/llm-usage/route.ts`
  - Group by model, stage, day/week/month; sum tokens and cost
  - Filters: date range, model, stage, organization
- New page: `packages/internal/app/(authenticated)/llm-usage/page.tsx`
  - **Summary cards:** total spend, total tokens, avg cost per call, call count
  - **Chart:** cost over time by model (line/area chart)
  - **Table:** breakdown by stage × model with tokens + cost
  - **Drill-down:** click a row → filtered tracing logs

#### 2B: True Model Switching on Re-run (1–2 days, after Phase 1)

**What:** The Prompt Lab test panel and replay dialog already support model override dropdowns. After AI SDK migration, this becomes truly provider-agnostic instead of OpenAI-only.

**Changes:**
- Update `packages/internal/app/actions/prompts.ts` — swap direct OpenAI call for AI SDK `generateText` with dynamic provider selection based on model string prefix (`gpt-*` → openai, `claude-*` → anthropic, `gemini-*` → google)
- **New feature:** Model comparison view — run same prompt against 2+ models side by side, display outputs + cost + latency in columns

#### 2C: Output Tracking & Diff View (2–3 days)

**What:** Compare outputs across runs of the same prompt with different versions, parameters, or models.

**Data:** Already stored in `llm_tracing_log.llm_response` + `prompt_snapshot`.

**Build:**
- New component: `OutputDiffViewer.tsx` — side-by-side or inline diff of two tracing log outputs
- Integrate into `TracingLogReplayDialog` — after a re-run, show diff between original and new output
- **"Pin as baseline"** action on any tracing log entry, then diff new runs against the pinned baseline

#### 2D: Prompt Test Suites (3–5 days)

**What:** Batch testing of a prompt version against a set of saved golden inputs. Addresses the "golden transcript benchmark dataset" action item from the March 19 team sync.

**Build:**
- New table: `prompt_test_cases` — stores named parameter sets per prompt
- New UI: "Test Suite" tab on prompt profile page
  - Manage test cases (add, edit, delete, import from tracing logs)
  - Run batch: execute all test cases against a prompt version + model
  - Results matrix: rows = test cases, columns = version/model combos, cells = pass/fail + output preview + cost
- **Golden dataset management:** Import high-quality transcript excerpts as reusable test inputs
- **Regression detection:** Compare batch results between prompt versions to catch regressions before deploying

---

## Architecture Decisions Needed

These need input from the team before starting implementation:

### 1. AI SDK Version Pinning
Frontend uses `ai@^6.0.73`. Pin the same major version in `packages/functions` to avoid drift. **Recommendation:** Pin `ai@^6.x` across both packages.

### 2. Provider Key Management
AI SDK providers read from env vars by default (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`). SST Resource bindings need mapping.

| Option | Approach | Trade-off |
|--------|----------|-----------|
| **A** | Set env vars in Lambda config | Simplest, follows AI SDK conventions |
| **B** | Pass `apiKey` explicitly to provider constructors | More explicit, matches current pattern |

**Recommendation:** Option B — explicit keys via SST Resource bindings. Consistent with existing code and avoids implicit env var coupling.

### 3. Tracing Adapter Strategy
Do we use AI SDK's built-in telemetry (`experimental_telemetry`) to feed the tracing table, or keep the manual `createTracingLog` pattern?

| Option | Approach | Trade-off |
|--------|----------|-----------|
| **A** | AI SDK `experimental_telemetry` | Automatic, less code, but less control over schema |
| **B** | Manual `createTracingLog` via `onFinish` | Full control, matches existing data model exactly |

**Recommendation:** Option B — manual tracing. Preserves existing tracing schema, avoids telemetry API churn, and keeps the data model under our control.

### 4. Structured Output Migration Timing
Several brokers do `JSON.parse(response)` with manual validation. Migrating to `generateObject()` + Zod is a clear win but changes error handling patterns.

**Recommendation:** Defer to Phase 1.5 after the base migration is stable. Don't mix the SDK swap with output parsing changes in the same PRs.

---

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

---

## Timeline Summary

```
Phase 1: AI SDK Backend Migration          ~2 weeks
├── Setup + AI SDK client layer             Day 1
├── TranscriptBroker + GoalGenerationBroker Week 1
└── Agent providers + remaining brokers     Week 2

Phase 2: Dashboard Enhancements             ~2 weeks
├── 2A: Token Usage Dashboard               2-3 days
├── 2B: Model Switching (post-Phase 1)      1-2 days
├── 2C: Output Diff View                    2-3 days
└── 2D: Prompt Test Suites                  3-5 days
```

**Total: 3–4 weeks.** Phase 2A and 2C can run in parallel with late Phase 1 work.

## What This Does NOT Cover

- **Langfuse / external observability** — Brev has its own tracing system. No need for Langfuse unless cross-team dashboards outside the internal tool are required.
- **Edge runtime** — AI calls are Lambda-based queue consumers, not latency-sensitive edge routes. No change needed.
- **Frontend changes** — frontend already uses AI SDK. No migration needed.

## Decision

Lowest-risk, highest-return change in the entire report. Already partially adopted on frontend. Backend migration is incremental, non-breaking, and reversible. The prompt management dashboard is ~70% built — the remaining work is additive features on proven infrastructure. **Start immediately.**
