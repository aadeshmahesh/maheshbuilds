---
title: "Tool Use — The LLM Decides, Your Code Executes"
description: "A deep dive into how tool calling actually works — the request shape, response shape, and the critical insight most tutorials miss."
date: 2026-09-23
tags: [tool-use, agentic-ai, anthropic, ollama, nodejs]
---

The most important insight about tool use in AI agents:

**The LLM never calls APIs directly.**

It tells YOU what to call.
YOU call it.
YOU report back.

That's it. Everything else follows from this.

---

## What Tools Actually Are

Tools are function definitions you send to the LLM.
They describe what's available — name, description, and input shape.

```js
// Anthropic format — source of truth
const tools = [
  {
    name:        "check_availability",
    description: "Check if a time slot is available on the calendar",
    input_schema: {
      type: "object",
      properties: {
        date:       { type: "string", description: "Date in YYYY-MM-DD" },
        start_time: { type: "string", description: "Start time HH:MM" },
        end_time:   { type: "string", description: "End time HH:MM" },
      },
      required: ["date", "start_time", "end_time"],
    },
  }
];
```

The LLM reads these definitions and decides when to use them.
You decide what they actually DO.

---

## The Request Shape (What You Send)

Every LLM turn — you send this:

```json
{
  "model":   "claude-sonnet-4-6",
  "system":  "You are a calendar assistant...",
  "tools":   [...tool definitions...],
  "messages": [
    { "role": "user", "content": "Schedule standup tomorrow 10am" }
  ]
}
```

System prompt + tools = sent every single turn.
Messages = grows with every turn.

---

## The Response Shape (What You Get Back)

When LLM wants to call a tool:

```json
{
  "stop_reason": "tool_use",
  "content": [
    {
      "type":  "text",
      "text":  "I'll check if that time is available."
    },
    {
      "type":  "tool_use",
      "id":    "tool_abc123",
      "name":  "check_availability",
      "input": {
        "date":       "2026-07-26",
        "start_time": "10:00",
        "end_time":   "10:30"
      }
    }
  ]
}
```

`stop_reason: "tool_use"` → loop continues.
`stop_reason: "end_turn"` → loop exits.

---

## What You Do With It

```js
// 1. Extract tool calls
const toolCalls = response.content
  .filter(b => b.type === "tool_use");

// 2. Execute each one (YOUR code, not LLM)
for (const toolCall of toolCalls) {
  const result = await executeTool(toolCall.name, toolCall.input);

  // 3. Feed result back to LLM
  toolResults.push({
    type:        "tool_result",
    tool_use_id: toolCall.id,   // must match the id above
    content:     JSON.stringify(result)
  });
}

// 4. Send back for next turn
messages.push({ role: "user", content: toolResults });
```

---

## Parallel Tool Calls

The LLM can request multiple tools in one turn.
Your for loop handles them all.

```json
{
  "content": [
    { "type": "tool_use", "name": "check_vpn_status",  "input": {...} },
    { "type": "tool_use", "name": "check_tool_access", "input": {...} }
  ]
}
```

Both execute. Both results go back together.
Fewer turns. Faster resolution.

---

## Ollama vs Anthropic — Different Shapes

This caught me early. Ollama and Anthropic return
tool calls in different shapes:

```
Anthropic:
  response.content → filter type === "tool_use"
  tool.input → already an object

Ollama:
  response.message.tool_calls → array
  tc.function.arguments → sometimes a JSON string

qwen2.5-coder:7b (special case):
  puts tool call as JSON string inside message.content
  no tool_calls field at all
```

My solution — a normalizeResponse() function that
converts any provider response to the same internal shape.
Swap providers by changing one env variable.

---

## The Tool_use_id Is Critical

```json
LLM response:
  { "id": "tool_abc123", "name": "check_availability" }

Your tool result must reference same id:
  { "tool_use_id": "tool_abc123", "content": "..." }
```

If IDs don't match — LLM gets confused.
Always use the exact ID from the tool_use block.

---

## Tool Description Quality Matters

The LLM uses the description to decide WHEN to call the tool.
Bad description = wrong tool called at wrong time.

```js
// Bad
description: "Check VPN"

// Good
description: "Check if employee VPN certificate is active or expired.
              Call this AFTER searching knowledge base.
              Returns status, expiry date, and certificate ID."
```

The description is a contract with the LLM.
Write it like documentation for a junior developer.

---

## Key Takeaway

> Tools = function definitions you send to LLM
> LLM decides when + what to call
> YOUR code executes the actual logic
> Results feed back into messages[]
> Loop continues until stop_reason === "end_turn"
>
> The LLM is the brain. Your code is the hands.
> Never forget which is which.

*Next: putting it all together — the site search agent with real vector embeddings.*
