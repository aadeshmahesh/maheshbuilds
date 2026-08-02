---
title: "MCP — The USB-C Standard for AI Tools"
description: "Model Context Protocol is how AI agents connect to the world. Anthropic created it, OpenAI and Google adopted it. Here's what it means for engineers."
date: 2026-08-12
tags: [mcp, agentic-ai, anthropic, tools, protocol]
---

When I first built AI agents, every tool integration
was custom code I wrote myself.

```js
// I wrote this for every single integration
async function executeTool(name, input) {
  if (name === "check_vpn_status") {
    return await fetch("https://vpn-api.acme.com/...");
  }
  if (name === "check_jira_access") {
    return await fetch("https://jira.acme.com/...");
  }
  // repeat for every tool...
}
```

Every app needed its own version.
Every team duplicated the work.

MCP solves this.

---

## What MCP Is

**MCP = Model Context Protocol**

Created by Anthropic in 2024.
Now adopted by OpenAI, Google, Microsoft, and Cursor.

It's a standard protocol for connecting LLMs to tools —
like USB-C, but for AI.

```
Before MCP:
  Claude → custom code → GitHub
  Claude → custom code → Slack
  Claude → custom code → Postgres
  (every connection different, repeated everywhere)

After MCP:
  Claude → MCP → GitHub MCP Server
  Claude → MCP → Slack MCP Server
  Claude → MCP → Postgres MCP Server
  (one standard, plug and play)
```

---

## What MCP Servers Expose

MCP servers expose three things:

**Tools** — actions the LLM can call
```
create_github_issue, send_slack_message, query_database
```

**Resources** — data the LLM can read
```
files, docs, database rows, emails
```

**Prompts** — reusable prompt templates
```
pre-built instructions for common tasks
```

---

## Manual Tools vs MCP

```js
// Before MCP — you write everything
const tools = [{
  name: "create_github_issue",
  description: "...",
  input_schema: { ... }
}];

async function executeTool(name, input) {
  if (name === "create_github_issue") {
    await fetch("https://api.github.com/...", { ... });
  }
}

// After MCP — just connect to the server
mcp_servers: [{
  type: "url",
  url:  "https://github-mcp-server.com/sse",
  name: "github"
}]
// Tools available instantly — no executeTool() needed
```

---

## Building Your Own MCP Server

```js
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

const server = new McpServer({ name: "it-support", version: "1.0.0" });

server.tool(
  "check_vpn_status",
  "Check if employee VPN is active or expired",
  { employee_id: z.string() },
  async ({ employee_id }) => {
    const result = await checkVPN(employee_id);
    return { content: [{ type: "text", text: JSON.stringify(result) }] };
  }
);

server.listen();
```

Now any MCP-compatible AI can use your VPN tool.
Write once. Use everywhere.

---

## When to Use MCP vs Manual Tools

| | Manual Tools | MCP Server |
|---|---|---|
| Learning / prototyping | ✅ | Overkill |
| One app uses tools | ✅ | Overkill |
| Multiple apps need same tools | ❌ Duplicate code | ✅ |
| Team sharing integrations | ❌ | ✅ |
| Claude Desktop / Cursor access | ❌ | ✅ |

I built manual tools first to understand the internals.
Then MCP made complete sense — it's the same concept, standardized.

---

## Popular MCP Servers Available Now

```
Official (Anthropic): Filesystem, Postgres, GitHub, Slack, Google Drive
Community: Jira, Notion, Linear, Stripe, Cloudflare
```

Claude Desktop, Cursor, and Zed Editor all support MCP.
Your MCP server works with all of them out of the box.

---

## Key Takeaway

> MCP = standard protocol for connecting LLMs to the world
>
> Manual tools taught me WHY tool calling works.
> MCP is the industry's HOW at scale.

Think of it like REST APIs — you understood HTTP first,
then frameworks made sense. Same pattern here.

*Next: embeddings — how text becomes numbers that capture meaning.*
