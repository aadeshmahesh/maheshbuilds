---
title: "Building an IT Support Agent — RAG + Parallel Tool Calls"
description: "How I built an AI agent that diagnoses and fixes VPN and tool access issues using RAG and parallel tool execution."
date: 2026-07-29
tags: [agentic-ai, rag, tool-use, nodejs, react, parallel-tools]
---

My second AI agent goes beyond the calendar scheduler.

It diagnoses IT issues, searches company documentation,
checks system status, and fixes problems — all in one
agentic loop. Two new concepts: RAG and parallel tool calls.

---

## What It Does

Employee types: "My VPN is not working and I can't access Jira"

```
Turn 1: LLM → search_knowledge_base("VPN not working")
              search_knowledge_base("Jira access")     ← parallel!

Turn 2: RAG returns policy docs → LLM →
              check_vpn_status(EMP-4521)
              check_tool_access(EMP-4521, "Jira")      ← parallel!

Turn 3: Both expired → LLM →
              enable_vpn(EMP-4521)
              enable_tool_access(EMP-4521, "Jira")     ← parallel!

Turn 4: Both fixed → LLM →
  "Fixed both! VPN renewed until Aug 22.
   Jira access restored. ✅"
```

---

## New Concept 1 — RAG First

The system prompt forces the agent to search docs
before taking any action:

```
ALWAYS call search_knowledge_base first.
Never check status without reading policy.
Never enable access without checking status first.
```

This grounds every decision in real company policy.
The agent doesn't guess — it reads, then acts.

---

## New Concept 2 — Parallel Tool Calls

In the calendar agent, tools ran one at a time.
Here, the LLM calls multiple tools in the same turn:

```js
// LLM response — two tools in one turn
content: [
  {
    type: "tool_use",
    name: "check_vpn_status",
    input: { employee_id: "EMP-4521" }
  },
  {
    type: "tool_use",
    name: "check_tool_access",
    input: { employee_id: "EMP-4521", tool_name: "Jira" }
  }
]

// Your code handles both
for (const toolCall of toolCalls) {
  const result = await executeTool(toolCall.name, toolCall.input);
  toolResults.push({ type: "tool_result", ... });
}
```

Both tools execute, both results feed back to LLM in one shot.
Fewer turns = faster resolution.

---

## The 5 Tools

```
1. search_knowledge_base  ← RAG — always first
2. check_vpn_status       ← read current state
3. enable_vpn             ← fix if expired
4. check_tool_access      ← read current state
5. enable_tool_access     ← fix if expired
```

Pattern: **search → check → fix**
Never fix without checking. Never check without reading policy.

---

## What's Different from Calendar Agent

| | Calendar Agent | IT Support Agent |
|---|---|---|
| RAG | ❌ | ✅ |
| Parallel tools | ❌ | ✅ |
| Tools | 2 | 5 |
| DB | None | None (JSON RAG) |
| Input | Date + time | Employee ID + issue |

Same agentic loop underneath.
Two new concepts layered on top.

---

## Key Lesson

The system prompt is the agent's personality.

```
"ALWAYS call search_knowledge_base first"
```

This one line changes the entire behavior.
The LLM follows it every single time.
System prompt design is as important as code.

*Next: why vocabulary mismatch is the real
reason to add vector embeddings to RAG.*
