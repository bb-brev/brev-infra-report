---
layout: default
title: "Langfuse"
---

<h1>Langfuse <span class="gradient">LLM Observability</span></h1>
<span class="verdict evaluate">🔍 Evaluate — Compare Against Internal Tool</span>

<div class="meta-bar">
  <div class="meta-item"><span class="meta-label">Replaces</span><span class="meta-value">Internal tracing/prompt tool</span></div>
  <div class="meta-item"><span class="meta-label">Effort</span><span class="meta-value">3-4 weeks</span></div>
  <div class="meta-item"><span class="meta-label">Risk</span><span class="meta-value" style="color: var(--green)">Low</span></div>
  <div class="meta-item"><span class="meta-label">Timeline</span><span class="meta-value">Q2 2026</span></div>
</div>

## What It Is

[Langfuse](https://langfuse.com) is an open-source LLM observability platform — tracing, prompt management, evaluations, and cost tracking. Self-hostable on AWS or available as managed cloud.

## Current State

Brev has an **internal tool built in-house** for:
- LLM call tracing and logging
- Prompt management and updates

LangSmith keys still exist in secrets but are no longer actively used.

## The Core Question: Build vs Buy

This isn't about replacing nothing — it's about whether Langfuse provides enough value over your existing internal tooling to justify the switch.

## What Langfuse Offers

| Capability | Details |
|-----------|---------|
| **Trace visualization** | Nested spans for multi-step pipelines — see how a transcript flows through processing → summarization → goal alignment → artifacts |
| **Prompt versioning** | Version-controlled prompts with prod/staging/dev environments. Promote versions without deploys. |
| **Prompt playground** | Test prompt changes in a UI before deploying |
| **Prompt A/B testing** | Run multiple prompt versions, compare output quality with metrics |
| **Cost tracking** | Per-trace, per-user, per-feature cost calculation across providers |
| **Eval pipelines** | Human annotation workflows + LLM-as-judge scoring + custom criteria |
| **AI SDK auto-tracing** | If using Vercel AI SDK, every `generateText`/`streamText` call is automatically traced with zero code |

<div class="pros-cons">
<div class="pros-card">
<h4>✅ Pros — Why Langfuse Could Be Better</h4>
<ul>
<li><strong>Stop maintaining custom code</strong> — redirect engineering time from observability tooling to product features</li>
<li><strong>AI SDK auto-tracing</strong> — if you adopt Vercel AI SDK, Langfuse traces every call automatically. Zero instrumentation code.</li>
<li><strong>Prompt Lab replay for free</strong> — the feature Chris wants (rerun prompts against historical params) is built-in</li>
<li><strong>Eval pipelines are hard to build</strong> — Langfuse has human annotation, LLM-as-judge, custom scoring out of the box</li>
<li><strong>Self-hostable on AWS</strong> — deploy on ECS/Fargate in your VPC. Data never leaves your infrastructure.</li>
<li><strong>Active development</strong> — YC-backed, fast release cycle, growing community</li>
<li><strong>Cost dashboards</strong> — see exactly what you're spending per feature/user on LLM calls</li>
</ul>
</div>
<div class="cons-card">
<h4>✗ Cons — Why Keep the Internal Tool</h4>
<ul>
<li><strong>Already built and working</strong> — sunk cost, no migration needed</li>
<li><strong>Brev-specific customization</strong> — may be tailored to goals, artifacts, transcripts in ways Langfuse isn't</li>
<li><strong>No per-event costs</strong> — internal tool has no usage-based pricing</li>
<li><strong>Full control</strong> — own the data model, UX, and roadmap</li>
<li><strong>Tight integration risk</strong> — if deeply coupled to Lambda consumers, migration could break workflows</li>
<li><strong>Another system to maintain</strong> — even self-hosted Langfuse needs updates, monitoring</li>
<li><strong>Team context switching</strong> — new UI/workflow for prompt management</li>
</ul>
</div>
</div>

## The Prompt Lab Replay Synergy

Chris's request for "Prompt Lab replay" (rerun prompts against exact historical parameters) maps directly to Langfuse:

1. **Traces capture full context** — input, output, model, parameters, system prompt for every call
2. **Prompt playground** — click any trace, modify the prompt, re-run with same inputs
3. **Evaluation datasets** — create "golden" test sets from production traces
4. **Version comparison** — side-by-side output from different prompt versions

Building this in the internal tool: **weeks of work**. In Langfuse: **built-in**.

## Self-Hosting on AWS

```
Your VPC
├── ECS/Fargate → Langfuse Docker containers
├── Aurora PostgreSQL (separate schema or instance)
└── S3 → blob storage for large traces
```

- Stays in VPC — no data leaves AWS
- Estimated infra cost: ~$50-100/month on Fargate
- Uses your existing Aurora cluster (separate schema) or a dedicated small instance

## Pricing

| Option | Cost | Best For |
|--------|------|----------|
| **Self-hosted** | Free (+ ~$50-100/mo infra) | Full control, VPC data residency |
| **Cloud Hobby** | Free | Testing/evaluation |
| **Cloud Pro** | $59/month | Small teams |
| **Cloud Team** | $499/month | SSO, advanced features |

**Recommendation:** Self-host for VPC data residency and zero per-event costs.

## Migration Plan

<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <h4>Week 1: Deploy + Parallel Run</h4>
    <p>Deploy Langfuse on ECS (self-hosted). Instrument 2-3 Lambda consumers to send traces to BOTH internal tool and Langfuse simultaneously.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">2</span>
  <div class="step-content">
    <h4>Week 2-3: Evaluate Side-by-Side</h4>
    <p>Compare: Does Langfuse capture what the internal tool captures? Is the prompt playground useful? Test eval pipeline with a "golden" transcript set. Gauge team preference.</p>
  </div>
</div>
<div class="step">
  <span class="step-num">3</span>
  <div class="step-content">
    <h4>Week 4: Decision</h4>
    <p>If Langfuse wins: migrate remaining consumers, deprecate internal tool. If internal tool wins: keep it, take Langfuse insights to improve it. If mixed: use Langfuse for observability + evals, keep internal tool for Brev-specific features.</p>
  </div>
</div>

## Decision Criteria

**Adopt if:** Internal tool requires ongoing maintenance, team wants eval pipelines/prompt A/B testing, Prompt Lab replay is a priority, or adopting AI SDK makes auto-tracing compelling.

**Keep internal tool if:** It's low-maintenance, meeting all needs, and team bandwidth is tight.

**Hybrid option:** Use Langfuse for tracing + evals, keep internal tool for Brev-specific prompt management.
