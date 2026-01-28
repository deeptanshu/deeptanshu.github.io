---
title: "Carryover: Portable state handoff for LLMs"
date: 2026-01-28T10:00:57-08:00
draft: false
category: ""
excerpt: ""
tags: ["LLMs", "AI", "Building"]
---

# Carryover: When you hit the rate limit wall

There's a particular kind of frustration that comes from being deep in flow, making real progress, and then seeing: "You've reached your usage limit. Try again in 4 hours and 37 minutes."

This kept happening to me with Claude Code. I'd be mid-refactor on some gnarly module, context loaded with file diffs, agent decisions, skill configurations, and suddenly I'd need to switch to ChatGPT or Gemini to keep going.

The manual process was miserable. Copy conversation logs. Paste into new chat. Type out what was happening. Wait for the new model to catch up. Try to get productive again while your brain screams that you were *just there* five minutes ago.

So I built Carryover.

## Context is more than chat history

From [my deep dive into Claude Code token usage]({{< relref "essays/claude-code-rate-limits-token-budgets.md" >}}): I was burning through 41+ million tokens, mostly on context re-reading. Every time I hit a rate limit and tried to manually transfer work to another LLM, I was recreating my entire work state in a new environment.

But work state isn't just the conversation transcript. It's file diffs showing what code changed. It's skills I'd configured. It's the reasoning chain of what approaches worked and didn't. It's why we made certain choices, weighted by recency.

Without all that, you're starting from scratch.

## Local models can compress

I've been curious about [what smaller models do well]({{< relref "essays/when-small-models-are-better-geometry-of-intelligence.md" >}}). Turns out 7B parameter models running on Ollama are good at compression, selectively attending to fewer tokens without adding new information. They run fine even on my wife's 8GB M1 MacBook Air.

I didn't need a massive model to create a handoff document. I needed something that could understand the structure of my work session and preserve what mattered.

After three rate-limited sessions with Claude Code and several more with ChatGPT, dusting off my GitHub account that hadn't been touched in 12+ years (my last commit was a [text summarizer](https://github.com/lekhakpadmanabh/Summarizer)), Carryover is live.

## How it works

There are attempts to standardize universal context protocols. This isn't one of them. This is me solving a specific problem and sharing it because others might find it useful.

Here's what's interesting: you can clone more than just the repository. Clone my `SPECS.md`, tweak it, have an AI build your version. Your implementation would be different, but you'd benefit from the thinking in the structure.

## What I learned

**Context is work state.** Not obvious until you've lost it. Context is file diffs, skills, the hierarchy of agent decisions, choices ranked by recency. That's what you need to transfer to be productive immediately somewhere else.

**Models get confused about who they are.** This was hilarious. I gave a handoff file to Claude's web UI. It immediately started claiming it had made changes to my filesystem.

"Wait," I said. "You're not Claude Code. You're the web UI. You can't touch my filesystem."

"You're absolutely right," it responded.

The handoff document needs to clarify what the new environment can and can't do.

**7B is the sweet spot.** I used local models because I hit this need exactly when Claude Code was unavailable. Larger models would compress better, sure. But 7B did the job. The 1B-3B models couldn't handle long conversations with multiple threads. 7B was small enough to run locally, capable enough to understand technical context.

**It's not just about rate limits.** I frequently switch between LLMs to get different perspectives on the same problem. I'd previously built an "LLM Prism" (my version of Karpathy's LLM Council, showing questions to multiple models). But I barely used it because most of my sessions were deep in Claude Code with all that rich work state. I couldn't port the question; I needed to port the context.

Now I can get ChatGPT's take on a refactoring decision without losing where I was.

## Forking specs, not just code

Traditional open source: clone the repo, run the code. But you can also clone the specs, customize them, and have AI build your version.

That's what I care about here. If you hit rate limits or want to switch between AI tools without losing your place, take the concept and make it yours.

The code is meant to be forked in intent, not just in git.

## Try it

Carryover is at [GitHub](https://github.com/deeptanshu/carryover). The token usage analysis that led to this is [here]({{< relref "essays/claude-code-rate-limits-token-budgets.md" >}}).

If you build your own version, I'd like to hear about it.

---

*Built because I got tired of copying chat logs. Runs on 7B models on a MacBook Air.*