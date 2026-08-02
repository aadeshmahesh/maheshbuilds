---
title: "RAG — You Don't Always Need a Vector Database"
description: "How to build Retrieval Augmented Generation with just a JSON array. Start simple, add complexity only when you need it."
date: 2026-07-22
tags: [rag, llm, nodejs, agentic-ai, retrieval]
---

RAG — Retrieval Augmented Generation.

One of the most important patterns in AI engineering.

Without it, the LLM answers from training data — which may be
outdated, wrong, or hallucinated. With RAG, you search YOUR
documents first, then let the LLM answer from real content.

---

## The Pattern

1. User asks a question
2. Search YOUR documents first
3. Send relevant docs + question to LLM
4. LLM answers based on YOUR real content

The LLM stops guessing. It starts answering from actual knowledge.

---

## What I Built — IT Support Agent

Employee reports: "My VPN is not working"

**Without RAG:**
→ LLM guesses: "Try restarting your computer"
→ Wrong. Unhelpful.

**With RAG:**
→ Agent searches company docs first
→ Finds: "VPN certificates expire every 90 days"
→ LLM reasons: check expiry → if expired → renew it
→ Correct action. Every time.

---

## RAG Without a Database

You don't always need pgvector or Pinecone.
For small doc sets, a JSON array works perfectly:

```js
const docs = [
  {
    tags:    ["vpn", "certificate", "expired"],
    content: "VPN licenses expire every 90 days. Renewal is
              automatic if employee is active in HR system."
  },
  {
    tags:    ["jira", "tool", "access", "expired"],
    content: "Internal tools expire after 60 days of inactivity.
              IT admin can re-enable via provisioning system."
  }
];

function searchKnowledgeBase(query) {
  const keywords = query.toLowerCase().split(" ");
  return docs
    .filter(doc => keywords.some(kw => doc.tags.includes(kw)))
    .map(doc => doc.content);
}
```

Simple. Fast. Zero setup. Works for learning and small doc sets.

---

## When to Upgrade

| Doc Set Size | Approach |
|---|---|
| < 50 docs | JSON keyword search ✅ |
| 50–1000 docs | Postgres Full Text Search |
| 1000+ docs | pgvector with embeddings |
| Mixed/fuzzy queries | Hybrid search |

Start simple. Add complexity only when keyword search fails you.

---

## RAG vs Fine-tuning

| | RAG | Fine-tuning |
|---|---|---|
| Cost | Cheap (just storage) | Expensive ($100s+) |
| Update | Change docs anytime | Retrain model |
| Best for | Facts and knowledge | Style and behavior |

RAG wins for most enterprise use cases.
You can update the docs without touching the model.

---

## Key Insight

RAG is not magic. It's just giving the LLM
the right information before asking it to act.

*Next: how vocabulary mismatch is the real reason
teams reach for embeddings — sooner than they expect.*
