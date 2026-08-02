---
title: "6 Months of Building AI Agents — What I Learned"
description: "16 years of frontend engineering meets agentic AI. What actually matters, what surprised me, and where I'm going next."
date: 2026-09-30
tags: [agentic-ai, learning, career, reflection, ai-engineering]
---

Six months ago I made a deliberate decision.

A 16-year e-commerce program I'd led had wound down
when the client migrated to Shopify. Instead of
finding the next similar role, I went deep on AI engineering.

Not prompting. Building.

Here's what I learned.

---

## What I Built

```
Project 1: Calendar Scheduling Agent
  Agentic loop, tool use, Ollama + Anthropic
  Provider-agnostic via normalizeResponse()

Project 2: IT Support Agent
  RAG with in-memory JSON, parallel tool calls
  System prompt as agent behavior controller

Project 3: Site Search Agent (in progress)
  Real embeddings, pgvector, semantic search
  Vocabulary mismatch problem solved
```

---

## The Thing That Surprised Me Most

The agentic loop is just a while loop.

```js
while (response.stop_reason === "tool_use") {
  // execute tools
  // feed results back
  // call LLM again
}
```

That's it. That's GitHub Copilot Workspace.
That's Intercom Fin. That's Salesforce Agentforce.

The sophistication is in the tools, the prompts,
the error handling, and the production patterns
around that loop — not the loop itself.

Once I understood this, everything clicked.

---

## What Actually Matters

**System prompt design is underrated.**

Most tutorials focus on the code.
But the system prompt is where agent behavior lives.

```
"ALWAYS call search_knowledge_base first"
```

That one line changed everything in the IT Support Agent.
The LLM follows it. Every. Single. Time.

**RAG is not about vectors — it's about grounding.**

Keyword search works for structured small doc sets.
You hit the vocabulary problem sooner than expected.
But the concept — retrieve then generate — matters more
than the retrieval method you use.

**Tokens are the unit of everything.**

Cost, speed, limits, quality — all driven by tokens.
I now think in tokens the way I think in KB for images.

---

## What I Got Wrong

**I underestimated the vocabulary problem.**

My IT Support Agent post said "keyword breaks at scale."
A Product Director corrected me:
"keyword breaks on vocabulary mismatch — day one, not at scale."

He was right. Real users type things you never anticipated.
I updated my understanding publicly. That interaction
led to my deepest understanding of embeddings.

**I thought more tools = smarter agent.**

More tools = more confusion for the LLM.
Start with the minimum tools needed.
Add tools only when the agent needs them.

---

## What 16 Years Gave Me

Frontend engineering taught me to think in systems.

Component boundaries = tool boundaries.
State management = messages[] management.
Error boundaries = try/catch in executeTool().
Performance budgets = token budgets.

The patterns translate. The vocabulary changes.
The engineering instinct stays the same.

---

## The Stack I'd Choose Today

```
Local dev:    Ollama (free, private, fast iteration)
Production:   Anthropic Claude Sonnet
Embeddings:   Voyage AI (pairs best with Claude)
Vector DB:    Neon Postgres + pgvector
Backend:      Hono on Cloudflare Workers
Frontend:     React 19 + TanStack Query
Queue:        BullMQ + Redis
```

---

## What's Next

```
→ Site Search Agent with real pgvector embeddings
→ Add streaming to all agents (most asked in interviews)
→ HR Onboarding Agent (human-in-the-loop + webhook)
→ Code Review Multi-Agent (orchestrator pattern)
→ Add ARASAAC symbol search to Autism Worksheet Studio
→ CCA-F certification (Claude Certified Architect)
```

---

## For Anyone Starting This Journey

Start with the calendar agent.

Understand the while loop.
Understand the messages[] array.
Understand why the LLM never calls APIs directly.

Everything else — RAG, embeddings, chunking, multi-agent —
is that same pattern with one new concept layered on top.

Build first. Read after. The concepts make sense
once you've seen them work in your own code.

---

## One Last Thing

A comment from a Product Director on my second LinkedIn post
taught me more about vocabulary mismatch than any tutorial had.

Learning in public accelerates learning.

You share what you know.
Someone smarter corrects you.
You update your model.
You share again.

That loop is the real agentic loop.

*Thanks for following along. More agents coming.*
