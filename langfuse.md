---
layout: default
title: "Langfuse — LLM Observability"
---

[← Back to Overview](./)

# Langfuse vs Internal Tracing Tool

**Verdict:** Evaluate — run in parallel for 2-4 weeks  
**Migration Effort:** 3-4 weeks  
**Risk:** Low

---

## What It Is

[Langfuse](https://langfuse.com) is an open-source LLM observability platform providing tracing, prompt management, evaluations, and cost tracking. It can be self-hosted on AWS (Docker/ECS) or used as a managed cloud service.

## What It Replaces

Brev currently has an **internal tool** (built in-house) for:
- LLM call tracing and logging
- Prompt management and updates
- (LangSmith keys still in secrets but no longer actively used)

The question is: **does Langfuse provide enough value over the internal tool to justify migration?**

## Langfuse Capabilities

### Tracing
- Full trace visualization with nested spans for multi-step AI pipelines
- See exactly how a transcript flows through: raw processing → summarization → goal alignment → artifact generation
- Latency breakdown per step, token usage per call

### Prompt Management
- Version-controlled prompts with production/staging/dev environments
- Prompt playground for testing changes before deployment
- A/B testing different prompt versions with automatic metrics

### Evaluations
- Human annotation workflows (rate AI outputs)
- LLM-as-judge automated scoring
- Custom evaluation criteria
- Track quality metrics over time

### Cost Tracking
- Per-trace cost calculation across providers
- Cost by feature, user, organization
- Budget alerts and forecasting

### Native Vercel AI SDK Integration
```typescript
import { Langfuse } from "langfuse";
import { observeOpenAI } from "langfuse";
import { generateText } from "ai";

// If using AI SDK — auto-traces every call
const langfuse = new Langfuse();
// Traces automatically captured with model, tokens, latency, cost
```

## Build vs Buy Analysis

### Arguments FOR Langfuse (replacing internal tool)

| Factor | Details |
|--------|---------|
| **Engineering time saved** | Stop maintaining custom observability code — redirect to product features |
| **AI SDK integration** | If you adopt Vercel AI SDK, Langfuse auto-traces every call with zero code |
| **Eval pipelines** | Hard to build well internally — Langfuse has this out of the box |
| **Prompt A/B testing** | Built-in prompt versioning with metrics — the "Prompt Lab replay" feature Chris wants would come nearly free |
| **Self-hostable** | Deploy on ECS/Fargate in your VPC — data never leaves AWS |
| **Active development** | YC-backed, growing fast, regular feature releases |
| **Community** | Large open-source community contributing integrations |

### Arguments FOR Keeping Internal Tool

| Factor | Details |
|--------|---------|
| **Already built** | Sunk cost — it exists and works |
| **Brev-specific** | May be tailored to Brev's unique workflows (goals, artifacts, transcripts) in ways Langfuse isn't |
| **No migration risk** | Existing integrations continue to work |
| **Full control** | Own the data model, UX, and feature roadmap |
| **Tight coupling** | If deeply integrated with prompt management in Lambda consumers, migration could break workflows |
| **Cost** | Internal tool has no per-event costs |

## Key Question: How Much Does the Internal Tool Cost to Maintain?

- **If low-maintenance and meeting needs:** Keep it. Focus engineering elsewhere.
- **If consuming meaningful dev time:** Langfuse is a strong upgrade — especially with AI SDK auto-tracing.
- **If you want eval pipelines / A/B prompt testing you haven't built yet:** Langfuse gives this for free.

## Self-Hosting on AWS

Langfuse can run on your existing AWS infrastructure:

```
ECS/Fargate → Langfuse Docker containers
      ↓
Aurora PostgreSQL (separate DB or same cluster, different schema)
      ↓
S3 for blob storage
```

- Stays in your VPC — no data leaves AWS
- Uses your existing Aurora cluster (or a separate one)
- Estimated infra cost: ~$50-100/month on Fargate

## The Prompt Lab Replay Synergy

Chris's request for "Prompt Lab replay" (rerun prompts against historical parameters) maps directly to Langfuse capabilities:

1. **Traces capture full context** — input, output, model, parameters, system prompt
2. **Prompt playground** — replay any traced call with different prompts/models
3. **Evaluation datasets** — create "golden" test sets from production traces
4. **Compare versions** — side-by-side prompt version output comparison

Building this in the internal tool would take weeks. Langfuse has it built-in.

## Pricing

| Tier | Price | Limits |
|------|-------|--------|
| **Self-hosted** | Free (infra costs only) | Unlimited |
| **Cloud Hobby** | Free | 50k observations/month |
| **Cloud Pro** | $59/month | Included observations + usage-based |
| **Cloud Team** | $499/month | Advanced features, SSO |

**Recommendation:** Self-host to keep data in VPC and avoid per-event costs.

## Migration Plan

### Week 1: Setup + Parallel Running
- Deploy Langfuse on ECS (self-hosted)
- Instrument 2-3 Lambda consumers to send traces to BOTH internal tool and Langfuse
- Compare trace quality

### Week 2-3: Evaluate
- Does Langfuse capture what the internal tool captures?
- Is the prompt playground useful?
- Test eval pipeline with a sample "golden" transcript set
- Measure team adoption — do engineers prefer Langfuse UI?

### Week 4: Decision
- If Langfuse is better: migrate remaining consumers, deprecate internal tool
- If internal tool is better: keep it, use Langfuse insights to improve it
- If mixed: use Langfuse for observability, keep internal tool for Brev-specific features

## Decision Criteria

✅ **Adopt if:** Internal tool requires ongoing maintenance, team wants eval pipelines, Prompt Lab replay is a priority  
⚠️ **Evaluate if:** Internal tool works fine but you're curious about better observability  
❌ **Skip if:** Internal tool is low-maintenance and meeting all needs, team bandwidth is tight
