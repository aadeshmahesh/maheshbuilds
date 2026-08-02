---
title: "Embeddings — How Text Becomes Numbers That Capture Meaning"
description: "What embeddings are, why they matter for RAG, and how 'can't reach the work network' matches VPN documentation."
date: 2026-08-19
tags: [embeddings, rag, vectors, llm, semantic-search]
---

After publishing my RAG post, a Product Director left a comment
that stopped me in my tracks:

> "The VPN-cert example works because the answer is a fact on a schedule,
> not a judgment call. Where keyword scoring actually breaks isn't doc size,
> it's vocabulary: someone types 'can't reach the work network' and your
> 'VPN' keyword never fires."

He was right. And it led me to understand embeddings properly.

---

## The Vocabulary Mismatch Problem

Keyword search matches WORDS, not MEANING.

```
Document: "VPN certificates expire every 90 days"
User types: "can't reach the work network"

Keyword search:
  Looking for: "can't" "reach" "work" "network"
  Found in doc: NONE
  Score: 0
  Result: not returned ❌

But the user MEANS VPN.
The document IS about VPN.
They should match — but keyword search misses it.
```

Same concept, different words. Keyword search is blind to this.

---

## What Embeddings Are

Embeddings convert text → numbers that capture meaning.

```
"VPN not working"
→ [0.23, 0.87, 0.45, 0.12, ...]

"can't reach the work network"
→ [0.21, 0.85, 0.47, 0.14, ...]
```

The numbers are **very similar** even though the words are completely different.
That's because both phrases mean the same thing.

Similar meaning = similar numbers. That's the key insight.

---

## How Embeddings Are Trained

The model was trained on billions of sentences from the internet.

It learned that "VPN" appears near "network", "remote", "work access".
It learned that "can't reach" appears near "not working", "down", "unavailable".

So when you embed both phrases, they point in the same direction.
Same direction = similar numbers = high similarity score.

---

## Cosine Similarity — The Comparison

```
"VPN not working"          → points in direction A
"can't reach work network" → points almost same direction

Similarity score: 0.97 out of 1.0 ✅

"VPN not working"   vs  "best pizza recipe"
Completely different directions
Similarity score: 0.02 ❌
```

---

## The Two Phases

**Phase 1 — Indexing (done once)**
```
Your docs → embedding model → numbers → stored in DB

"VPN expires every 90 days" → [0.23, 0.87, ...]  → saved
"Jira access expires..."    → [0.45, 0.12, ...]  → saved
"Password reset policy..."  → [0.78, 0.34, ...]  → saved
```

**Phase 2 — Querying (every search)**
```
User query → same embedding model → numbers
                    ↓
        Compare against stored numbers
                    ↓
        VPN doc: similarity 0.97 ✅
        Jira doc: similarity 0.23 ❌
                    ↓
        Return VPN doc to LLM
```

Critical: **must use the same model both times**.
Numbers from different models aren't comparable.

---

## Vocabulary Coverage — Why It Hits Early

The Product Director's insight was that you hit the vocabulary
problem on day one of real users — not when you have 10,000 docs.

```
100 users search for VPN help:
  "VPN not working"          → keyword finds it ✅
  "can't reach work network" → keyword misses ❌
  "tunnel is down"           → keyword misses ❌
  "remote access broken"     → keyword misses ❌
  "network connection failed"→ keyword misses ❌

60% of searches miss. On day one.
```

That's why teams reach for embeddings sooner than expected.

---

## Tools for Embeddings

```
Local (free):    ollama pull nomic-embed-text
Production:      Voyage AI (pairs best with Anthropic Claude)
Most popular:    OpenAI text-embedding-3-small
```

Start with Ollama locally. Zero cost. Zero API key.

---

## In Code

```js
// Generate embedding with Ollama
async function getEmbedding(text) {
  const response = await fetch("http://localhost:11434/api/embeddings", {
    method: "POST",
    body: JSON.stringify({
      model: "nomic-embed-text",
      prompt: text
    })
  });
  const data = await response.json();
  return data.embedding;  // [0.23, 0.87, ...]
}

// Search by meaning (pgvector)
const results = await db.execute(sql`
  SELECT content,
         1 - (vector <=> ${queryVector}::vector) AS score
  FROM documents
  ORDER BY score DESC
  LIMIT 5
`);
```

---

## Key Takeaway

> Keyword search matches WORDS.
> Embeddings match MEANING.
>
> "can't reach the work network" → keyword: 0 matches
>                                → embeddings: VPN doc found ✅

The vocabulary gap is the real driver for embeddings —
not document count. You'll hit it earlier than you expect.

*Next: chunking — how to split large documents into searchable pieces.*
