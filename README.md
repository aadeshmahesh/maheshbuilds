# maheshbuilds.dev

Personal AI Engineering blog — built with Astro, hosted on Cloudflare Pages.

## Stack
- **Astro** — static site generator
- **Cloudflare Pages** — hosting + CDN
- **Markdown** — one .md file per blog post

## Local Dev

```bash
npm install
npm run dev
# → http://localhost:4321
```

## Deploy

```bash
# Push to GitHub → Cloudflare Pages auto-deploys
git add .
git commit -m "post: your post title"
git push
```

## Writing a New Post

1. Create a new file: `src/content/blog/your-post-slug.md`
2. Add frontmatter:

```markdown
---
title: "Your Post Title"
description: "One line summary"
date: 2026-07-22
tags: [agentic-ai, nodejs, react]
---

Your content here...
```

3. Push to GitHub → live in 60 seconds ✅

## Blog Post Template

```markdown
---
title: ""
description: ""
date: 2026-07-22
tags: []
---

## The Problem

## The Pattern / Concept

## In Code

```js
// real code here
```

## What I Built

## Key Takeaway

## What's Next
```

## Planned Posts

- [x] Week 1: The Agentic Loop
- [ ] Week 2: RAG — No Vectors Needed
- [ ] Week 3: IT Support Agent
- [ ] Week 4: Agentic AI vs AI Agent
- [ ] Week 5: Human-in-the-Loop Pattern
- [ ] Week 6: Webhook Resume Pattern
- [ ] Week 7: Job Queues for AI Agents
- [ ] Week 8: Token Cost Control
