---
title: "Claude Code Rate Limits: Diagnosing my token usage "
date: 2026-01-26T21:04:07-08:00
draft: false
category: "Claude Code"
excerpt: "I'm running into rate limits every hour or so into using claude code. I analyzed on session < 3% of my AI bill went to intelligence. The rest- regurgitating stale and bloated context"
tags: []
---

Every few hours using Claude Code, I hit a wall. It is incredibly frustrating to be mid-flow and get locked out by a rate limit. After burning through roughly **44 million tokens** and a **$67.58 bill**, I dug into the logs to find out why.

The data shows I wasn't paying for intelligence. I was paying a "memory tax" for conversations that never ended.

---

## The $68 breakdown

In about 17 days, I made nearly 500 requests. Here is where the money actually went:

| Category | Cost | % of Total |
| --- | --- | --- |
| Building context (Cache Write) | $39.95 | 67.1% |
| Re-reading history (Cache Read) | $17.30 | 29.1% |
| **Actual AI answers (Output)** | **$2.14** | **3.2%** |
| New prompts | $0.15 | 0.3% |

**Only 3% of my bill went toward actual answers.**  
The rest was spent reminding the AI what we had already discussed.

---

## The $0.77 "yes"

The most expensive word I typed was "yes."

In one marathon session, I hit 361 requests without ever clearing the history. By the end, the context had ballooned to over **153,000 tokens**. Because Claude Code re-processes the entire conversation history with every new message, a simple confirmation cost $0.77. At the start of that same session, it would have cost less than a dime.

---

## The 5-hour rolling window

I used to think limits reset daily. They don't.

Claude subscriptions operate on a **5-hour rolling window** starting from your first use. If you exhaust your budget in a heavy two-hour burst of context-heavy work, you are stuck until that window slides forward.

---

## Why the bill was so high

My analysis identified three habits that caused the bloat:

* **Marathon sessions:** I kept sessions active for days. Every tool call, shell output, and file read became permanent debt that I paid for in every subsequent message.
* **Task mixing:** I used one session for everything—coding, stock analysis, and job searching. Claude doesn't need to know my tax-loss harvesting strategy to fix a UI bug, but I paid for it to remember both.
* **The model mismatch:** I used **Opus 4.5** for every single command. Analysis showed that **66% of my queries**—simple things like `ls`, `cd`, or moving files—could have been handled by Haiku for 80% less money.

---

## My new workflow

I'm changing how I work to stop the waste:

1. **Clear context often:** I now use `/clear` or start fresh sessions every 50 requests.
2. **Match the model to the task:** I use Haiku for navigation and confirmations, saving Opus only for architecture and deep logic.
3. **Local routing:** I am building a tool to route basic shell commands to a local model (`llama3.2:1b`). This handles the boring stuff for $0.00.

AI is only expensive if you force it to have a perfect, infinite memory for things that don't matter.

---

Would you like me to help you set up a simple shell alias that uses a cheaper model for basic file navigation?

---


*If this saved you from the meta-analysis death spiral, share it.*