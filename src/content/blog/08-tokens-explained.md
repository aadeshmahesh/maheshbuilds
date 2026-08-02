---
title: "Tokens — What They Are and Why They Drive Every LLM Decision"
description: "Tokens are not words. Not characters. Understanding them changes how you design, optimize, and price AI systems."
date: 2026-09-02
tags: [tokens, llm, cost, context-window, agentic-ai]
---

Every LLM decision — speed, cost, limits, quality —
comes down to tokens.

Yet most tutorials barely explain what they are.

---

## What a Token Actually Is

```
A token is NOT a word.
A token is NOT a character.
A token is a PIECE of text.
```

Real examples:

```
"Hello"          → 1 token
"VPN"            → 1 token
"certificates"   → 2 tokens  (certific + ates)
"unbelievable"   → 3 tokens  (un + believ + able)
```

Rule of thumb for English:
```
1 token ≈ 4 characters ≈ 0.75 words

100 tokens  ≈ 75 words
1000 tokens ≈ 750 words  ≈ 1.5 pages
4000 tokens ≈ 3000 words ≈ 6 pages
```

---

## Why LLMs Count Tokens

LLMs don't read text. They read tokens.

Every API call:
- Counts **input tokens** (what you send)
- Counts **output tokens** (what LLM returns)
- You pay for BOTH
- There's a LIMIT on both

---

## Context Window

Every LLM has a maximum number of tokens it can process at once.

```
Model                  Context Window
────────────────────────────────────
claude-sonnet-4-6      200,000 tokens
llama3.1:8b (Ollama)   128,000 tokens
qwen2.5-coder:7b       32,000 tokens

200,000 tokens ≈ 500 pages of text
```

The context window is everything the LLM can "see" in one call.
It's NOT memory — the LLM forgets everything after the call ends.

---

## Tokens in the Agentic Loop

Here's what happens to token count across turns:

```
Turn 1 request:  system prompt + user message
                 ≈ 500 tokens

Turn 2 request:  system prompt + user + assistant + tool_result
                 ≈ 800 tokens

Turn 3 request:  everything above + more tool results
                 ≈ 1200 tokens

Turn 4 request:  ≈ 1600 tokens
```

The messages[] array grows every turn.
Token count grows with it.
Cost grows with it.

---

## Real Cost for My IT Support Agent

```
Anthropic Claude Sonnet 4.6:
  Input:  $3 per million tokens
  Output: $15 per million tokens

One IT Support session (4 turns):
  Input tokens:  ~2000
  Output tokens: ~400

Cost: (2000 × $3/1M) + (400 × $15/1M)
    = $0.006 + $0.006
    = ~$0.012 per session

1000 sessions/day = $12/day = $360/month
```

Cheap at small scale. Adds up fast at production scale.

---

## Token Budget in Code

```js
let totalInputTokens  = 0;
let totalOutputTokens = 0;
const MAX_TOKENS = 50000;

while (turn < maxTurns) {
  const response = await callLLM(messages);

  totalInputTokens  += response.usage?.input_tokens  || 0;
  totalOutputTokens += response.usage?.output_tokens || 0;

  if (totalInputTokens + totalOutputTokens > MAX_TOKENS) {
    console.warn("Token budget reached — stopping");
    break;
  }
}
```

Always track token usage in production.
Always set a budget. Runaway agents are expensive.

---

## Why Chunking Saves Tokens

```
Without chunking:
  Entire 50-page doc = ~25,000 tokens
  Sent to LLM every search
  Cost: massive ❌

With chunking (500 tokens per chunk):
  Only relevant chunk sent
  Cost: tiny ✅
```

Token efficiency is the real reason we chunk —
not just search accuracy.

---

## System Prompt Tokens — Hidden Cost

```
Your system prompt = ~200 tokens
Sent on every single LLM turn
4 turns per session = 800 tokens just for system prompt

100 users/day = 80,000 tokens/day just for system prompts

Keep system prompts short and precise.
Every word costs money × every turn × every user.
```

---

## Common Misconceptions

```
❌ "1 token = 1 word"
✅ 1 token ≈ 0.75 words

❌ "Context window = how much LLM remembers"
✅ LLM has NO memory. Context window = what you send THIS call.

❌ "Longer responses cost more"
✅ Input usually costs more — you send full history every turn

❌ "More context = better answers"
✅ Noise hurts quality. Focused context = better answers.
```

---

## Key Takeaway

> Token = the unit of text LLMs process
>
> Why it matters:
> → Cost (you pay per token)
> → Speed (more tokens = slower response)
> → Limits (context window max)
> → Quality (too many tokens = noisy context)
>
> Track usage. Set budgets. Keep prompts concise.

*Next: workflow vs LLM agent — when to use each and the production sweet spot.*
