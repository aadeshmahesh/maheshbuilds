---
title: "Production AI Agents — Queues, Rate Limits, and Cost Control"
description: "What separates a demo from production. The patterns that make AI agents reliable, affordable, and scalable."
date: 2026-09-16
tags: [production, agentic-ai, bullmq, rate-limiting, nodejs, cost]
---

Building an AI agent that works for one user is easy.

Building one that works for 1000 concurrent users
without crashing, overspending, or hitting API limits —
that's engineering.

Here's what I learned about production patterns for AI agents.

---

## The Problem at Scale

```
1000 users hit your IT Support Agent at the same time

Each session:
  4 LLM turns × 2–5 seconds each = 8–20 seconds
  Multiple Anthropic API calls
  Multiple DB queries
  Multiple external tool calls

Without handling:
  Server crashes ❌
  Anthropic 429 rate limit errors ❌
  Costs spike uncontrollably ❌
  Users get failed responses ❌
```

---

## Layer 1 — Job Queue (BullMQ)

Don't run agents directly on HTTP requests.
Queue them. Process with limited concurrency.

```js
import { Queue, Worker } from "bullmq";

// Producer — return job ID instantly
app.post("/support", async (req, res) => {
  const job = await agentQueue.add("support", req.body, {
    attempts: 3,
    backoff: { type: "exponential", delay: 2000 }
  });
  res.json({ job_id: job.id, status: "queued" });
});

// Consumer — only 5 agents run at once
const worker = new Worker("ai-agent", async (job) => {
  return await runITSupportAgent(job.data);
}, {
  concurrency: 5   // ← this is the key setting
});
```

User gets job ID instantly. Agent runs when slot is available.
5 concurrent agents maximum. No crashes.

---

## Layer 2 — Rate Limiter (Bottleneck)

Anthropic limits how many requests per minute you can make.
Bottleneck wraps every LLM call and spaces them out.

```js
import Bottleneck from "bottleneck";

const limiter = new Bottleneck({
  maxConcurrent: 5,
  minTime: 200,          // min 200ms between calls
  reservoir: 50,         // 50 requests available
  reservoirRefreshAmount: 50,
  reservoirRefreshInterval: 60 * 1000  // refill every 60s
});

const callLLM = limiter.wrap(async (params) => {
  return await anthropic.messages.create(params);
});
```

No more 429 errors. Every call is spaced correctly.

---

## Layer 3 — Token Budget

Set a hard limit on tokens per session.
Runaway agents are expensive.

```js
let totalTokens = 0;
const MAX_TOKENS = 50000;

while (response.stop_reason === "tool_use") {
  totalTokens += response.usage.input_tokens
               + response.usage.output_tokens;

  if (totalTokens >= MAX_TOKENS) {
    console.warn("Token budget exhausted");
    break;
  }
}

console.log(`Session cost: ~$${(totalTokens / 1000000 * 3).toFixed(4)}`);
```

---

## Layer 4 — Circuit Breaker (Opossum)

If Anthropic API starts failing — stop hitting it.
Let it recover. Try again after a cooldown.

```js
import CircuitBreaker from "opossum";

const breaker = new CircuitBreaker(callLLM, {
  timeout: 30000,
  errorThresholdPercentage: 50,  // open at 50% failure rate
  resetTimeout: 60000            // try again after 60s
});

breaker.on("open",  () => console.warn("API failing — stopping calls"));
breaker.on("close", () => console.log("API recovered — resuming"));
```

---

## Layer 5 — Cache Repeated Calls

Same user asking the same VPN question?
Return cached answer. Skip the LLM call.

```js
const cacheKey = `llm:${hash(systemPrompt + userMessage)}`;
const cached = await redis.get(cacheKey);

if (cached) return JSON.parse(cached);

const response = await callLLM(params);
await redis.setEx(cacheKey, 3600, JSON.stringify(response)); // 1hr cache
return response;
```

---

## Layer 6 — Job Status Polling

User gets job ID immediately.
Frontend polls until complete.

```js
// Backend
app.get("/job/:id", async (req, res) => {
  const job   = await agentQueue.getJob(req.params.id);
  const state = await job.getState();

  res.json({
    status: state,
    result: state === "completed" ? job.returnvalue : null,
    error:  state === "failed"    ? job.failedReason : null,
  });
});

// Frontend — polls every 2 seconds
const poll = setInterval(async () => {
  const { status, result } = await fetch(`/job/${jobId}`).then(r => r.json());
  if (status === "completed") { clearInterval(poll); showResult(result); }
  if (status === "failed")    { clearInterval(poll); showError(); }
}, 2000);
```

---

## The Full Stack

```
Request → Rate limit check (per user)
        → BullMQ Queue (instant job ID returned)
        → Worker Pool (5 concurrent max)
        → Bottleneck (space out Anthropic calls)
        → Cache check (skip if seen before)
        → Circuit Breaker (stop if API failing)
        → Agentic Loop (with token budget)
        → Result saved to DB
        → User polls job status
```

---

## Cost at Scale

```
Without optimization:
  1000 sessions/day × $0.02 = $20/day = $600/month

With caching (30% cache hit rate):
  700 sessions/day × $0.02 = $14/day = $420/month

With token budget (avg 30% reduction):
  $420 × 0.7 = $294/month

Same user experience. 50% lower cost.
```

---

## Key Takeaway

> This is what separates a demo from production.
>
> Queue → Worker → Rate Limit → Cache → Circuit Break → Budget
>
> Each layer solves one problem.
> Together they make AI agents production-ready.

*Next: the site search agent — building RAG with real embeddings and pgvector.*
