---
title: "Chunking — How to Split Documents for RAG"
description: "The strategy you use to split documents matters more than the embedding model you choose. Here's what I learned."
date: 2026-08-26
tags: [rag, chunking, embeddings, llm, vector-search]
---

Here's something most RAG tutorials skip:

**Bad chunking kills good embeddings.**

You can have the best embedding model in the world.
If your chunks are wrong, your search results are wrong.

---

## Why Chunking Exists

LLMs have token limits. You can't send a 50-page document
to the LLM in one shot. You need to split it into pieces,
store each piece, and retrieve only the relevant ones.

```
50-page IT Policy Guide
          ↓
Chunk 1: "VPN setup instructions..."
Chunk 2: "VPN certificate expiry policy..."
Chunk 3: "Jira access requirements..."
Chunk 4: "Password reset procedure..."
...
```

User asks about VPN → only chunks 1 and 2 retrieved.
LLM gets focused, relevant context. Not 50 pages of noise.

---

## Strategy 1 — Fixed Size (Don't Do This)

```
Split every 200 characters, no matter what:

"VPN certificates expire every 90 |
days. Renewal is automatic if emp |
loyee is active in HR system."
```

Result: broken sentences, lost context, "days of what?".
Good for: absolutely nothing in production.

---

## Strategy 2 — Sentence Based

```
Split on sentence endings:

Chunk 1: "VPN certificates expire every 90 days."
Chunk 2: "Renewal is automatic if employee is active."
Chunk 3: "Manual renewal needed if auto-renewal fails."
```

Better — no broken sentences.
Problem: single sentence often lacks enough context.
"Renewal is automatic" — renewal of what?

---

## Strategy 3 — Paragraph Based (Good Starting Point)

```
Split on blank lines:

Chunk 1:
"VPN certificates expire every 90 days.
 Renewal is automatic if employee is active.
 Manual renewal needed if auto-renewal fails.
 Contact IT at vpn@acme.com for help."

Chunk 2:
"Jira requires Active Directory group membership.
 Access expires after 60 days of inactivity."
```

Full context preserved per topic.
Natural boundaries.
This is what I use in my projects right now.

---

## Strategy 4 — Sliding Window (Handles Boundaries)

The problem with hard splits: important context gets cut
right at the boundary between chunks.

Solution — overlap:

```
Window: 200 tokens | Overlap: 50 tokens

Chunk 1: tokens 1-200
Chunk 2: tokens 151-350  ← 50 token overlap
Chunk 3: tokens 301-500  ← 50 token overlap
```

The overlap means context from the end of chunk 1
appears at the start of chunk 2. Nothing gets lost.

---

## Strategy 5 — Semantic Chunking (Best, Most Complex)

Split when the TOPIC changes — not when the sentence ends.

Uses embeddings to detect topic boundaries:
- High similarity between sentences = same chunk
- Low similarity = start a new chunk

```
"VPN certificates expire every 90 days.      ← VPN topic
 Renewal is automatic if active.             ← VPN topic
 Manual renewal if auto fails."              ← VPN topic ends
─────────────────────────────────────────────────────────
"Jira requires AD group membership.          ← Jira topic starts
 Access expires after 60 days."              ← Jira topic
```

Most accurate. Each chunk = one complete idea.
More complex to implement. Worth it for production.

---

## Chunk Size — How Big?

```
Too small:
  "VPN expires."
  → Not enough context for LLM to answer from ❌

Too large:
  Entire 50-page document
  → Too many tokens, too expensive, too noisy ❌

Sweet spot:
  200–500 tokens per chunk
  ≈ 1–3 paragraphs
  ≈ enough context, not too much ✅
```

---

## What Gets Stored Per Chunk

```js
{
  id:       "chunk_003",
  content:  "VPN certificates expire every 90 days...",
  vector:   [0.23, 0.87, 0.45, ...],
  metadata: {
    source:       "it-policy.pdf",
    page:         3,
    section:      "VPN Policy",
    chunk_num:    3,
    total_chunks: 24,
  }
}
```

Metadata is how you filter before searching.
"Only search VPN section" → filter by section first.

---

## The Rule That Guides Everything

> One idea = one chunk.
>
> If you can answer a question from the chunk alone
> without needing the rest of the document — chunk size is right.
>
> If the chunk says "as mentioned above..." — too small.
> If the chunk has 5 unrelated topics — too big.

---

## For My Projects

```
IT Support Agent:    Each JSON doc = one chunk already ✅
Project 3 (Search):  Product descriptions → paragraph chunks
Autism Worksheet:    Each ARASAAC symbol = one chunk ✅
Future PDFs:         Semantic chunking + sliding window
```

---

## Key Takeaway

Chunking quality matters MORE than embedding model quality
for most use cases.

A great embedding model with bad chunks = bad results.
A decent embedding model with good chunks = great results.

Get the chunking right first.

*Next: tokens — what they are and why they drive every cost and limit decision in LLM systems.*
