---
title: "The Agentic Loop — How AI Agents Actually Work"
description: "The while loop that powers every serious AI agent. From calendar scheduling to GitHub Copilot — it's all the same pattern."
date: 2026-07-15
tags: [agentic-ai, tool-use, nodejs, anthropic, ollama]
---

Workflows are great — predictable, fast, cheap.
But they break the moment input gets messy or steps change.

> "Schedule a meeting" → workflow handles it fine.
> "Find time when my whole team is free next week and avoid anyone's focus blocks" → workflow gives up.

That's where the **Agentic Loop** takes over.

---

## The Pattern

Every AI agent runs on this same loop:

1. Send **system prompt + tool definitions + user message** to the LLM
2. LLM responds with a **tool to call** — not text
3. YOU execute that tool against real APIs
4. Send the **result back** to the LLM
5. LLM decides — call another tool or give final answer
6. Repeat until the goal is done

The LLM is the **decision maker**.
Your code is the **executor**.
The `messages[]` array is the **memory**.

---

## In Code

```js
while (response.stop_reason === "tool_use") {
  const toolCalls = response.content
    .filter(b => b.type === "tool_use");

  for (const tool of toolCalls) {
    const result = await executeTool(tool.name, tool.input);

    messages.push({
      role: "user",
      content: [{
        type:        "tool_result",
        tool_use_id: tool.id,
        content:     JSON.stringify(result)
      }]
    });
  }

  response = await callLLM(messages);
}
```

---

## The Key Insight — Messages Array

Every turn you send the **entire conversation history**:

```
Turn 1: [ user_message ]
Turn 2: [ user_message, assistant(tool_use), tool_result ]
Turn 3: [ user_message, assistant, tool_result, assistant, tool_result ]
```

The LLM has no memory between API calls.
The `messages[]` array IS the agent's memory.

---

## What I Built — Calendar Scheduling Agent

User types: "Schedule a standup tomorrow at 10am"

```
Turn 1: LLM → check_availability({ date: "2026-07-16", time: "10:00" })
Turn 2: { available: true } → LLM → schedule_event({ title: "Standup" })
Turn 3: { success: true } → LLM → "Your standup is booked! ✅"
```

Three turns. Two tools. One goal achieved.

---

## Workflow vs Agentic Loop

| | Workflow | Agentic Loop |
|---|---|---|
| Input | Structured, known | Natural language |
| Steps | Hardcoded by you | LLM decides |
| Cost | Near zero | Token cost per turn |
| Best for | Predictable tasks | Complex, variable tasks |

**Production sweet spot:** LLM to UNDERSTAND → Workflow to EXECUTE.

---

## Same Pattern, Bigger Scale

- **GitHub Copilot Workspace** — reads code, writes fix, runs tests
- **Intercom Fin** — searches docs, checks account, resolves ticket
- **Salesforce Agentforce** — qualifies lead, drafts proposal, updates CRM

The difference is scale and observability — not the core pattern.

*Building in public — one agent at a time.*
