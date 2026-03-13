---
title: "Building Sidekick: A Tiny Command-Line Coding Agent"
date: 2026-02-15T10:00:00-08:00
draft: false
category: "AI"
excerpt: "I built a command-line coding agent in very few lines of Python to understand what these systems actually are"
tags: ["AI", "Agents", "CLI", "Python", "Building"]
---

# Building a Tiny Command-Line Agent

I have been obsessed for a while with a certain kind of engineering move: taking a system that feels large and vaguely magical, and crushing it down until the whole thing fits in your head.

That instinct came in part from the kind of work Andrej Karpathy has done so well. A lot of people have had the experience of seeing a big model or a big system as this opaque industrial object. Then someone comes along, strips away the scaffolding, removes the abstraction layers, cuts the convenience wrappers, and shows the living mechanism directly. Suddenly the thing is smaller in conceptual distance. You can see it.

That was the feeling I wanted here.

I wanted to build a command-line coding agent in very few lines of Python. Not because I think the final system should stay tiny forever, but because compressing it was the fastest way for me to understand it honestly.

This project has been a labor of love in the attention sense. I kept pulling pieces out of the system until only the essential loop was left, and then sat with that loop until it finally clicked.

What clicked is that command-line coding agents are not mysterious at all once you strip them down.

They are loops.

The user asks for something. The system assembles context. A model decides whether to answer directly or call a tool. Tools run locally. Results come back. The model updates its view. The user approves risky actions. The system remembers enough to continue coherently. Then the next turn begins.

That is the organism.

Everything else is ergonomics, scale, safety, or productization.

## The inspiration: compress until the shape becomes obvious

One of the most powerful educational patterns in software is compression.

You take something that people usually encounter as a giant product:

- a training pipeline
- a transformer
- a compiler
- a database
- an agent

and you rewrite it so the structure is visible all at once.

That is what I was chasing here.

I kept thinking about the value of those compact projects that make you confront the essential moving parts directly. Once you remove orchestration layers, framework patterns, package boundaries, telemetry plumbing, and product scaffolding, you find out whether you actually understand the system or were just using it.

This project is my attempt to do that for coding agents. To understand the backbone that tools like Codex, Claude Code, OpenCode, and similar command-line agents all share, without trying to reproduce their production complexity.

## Why command-line agents are interesting

Command-line agents sit at a particularly interesting point in software.

They combine two worlds:

- language models, which are flexible and expressive
- Unix-style tools, which are concrete and executable

That combination is powerful because the terminal already contains the primitive actions of programming:

- list files
- read files
- search text
- create directories
- write code
- run code
- inspect outputs

The agent does not invent these powers from scratch. It learns how to choose among them in response to natural language goals.

That is why these systems feel so capable so quickly. They are standing on a substrate that programmers already use every day.

## What tools like Codex, Claude Code, and OpenCode are really teaching us

When you use a command-line agent seriously, you start to notice that the magic is in the shape of the loop around the model, not just in the model itself.

Different systems make different product choices, but many of them revolve around the same basic ideas:

- the terminal as the native interface
- tools as structured actions
- explicit or implicit permissions
- persistent working context
- local memory
- decomposition of bigger tasks into smaller ones

Once I started seeing those common pieces, the whole category became easier to reason about.

You can almost treat "coding agent" as a stack of concepts.

The model is one layer.
The loop is another.
The tool runtime is another.
The context model is another.
The permission model is another.

When people compare systems, they often compare only the model quality. The more interesting question, I think, is: what does the loop look like?

## The heart of it: the REPL loop

The first and most important concept is the REPL loop.

This is the beating heart of the agent. Without it, you just have one-shot completion.

The REPL does something deceptively simple:

1. take a user request
2. decide what to do
3. maybe call a tool
4. maybe ask the user first
5. observe the result
6. continue

This is the structure that turns static inference into an interactive system.

The reason the terminal is such a natural home for agents is that REPLs already have the right rhythm. They are conversational, stateful, iterative, and grounded in action. They let you watch the machine work.

When I built my own tiny version, this was the first thing that became clear: the loop is the product.

## Context management is the real discipline

The next big insight for me was that context management is one of the core design problems, not a side detail.

An agent can only reason over what it currently knows. So every turn has to answer a hidden question:

What does the model get to see right now?

In this little project, context includes things like:

- the user request
- the current working directory
- recent turns
- available skills
- the set of tools

That looks small, but it is enough to illustrate the real challenge.

Too little context makes the agent blind.
Too much context makes it noisy, expensive, and unfocused.

Once I understood that, a lot of command-line agent behavior started making more sense. Good systems are often just better at deciding what to include in the next turn.

## Tool use is where the agent becomes real

A lot of the seriousness of an agent comes from tool use.

Without tools, the model is only describing actions.
With tools, the system can actually perform them.

That changes everything.

Tool use means the model is choosing from a set of concrete affordances in the environment, not just generating text.

In this project, that includes simple command-line capabilities like:

- listing directories
- reading files
- grepping text
- creating directories
- changing directories
- writing files
- running Python

Those are enough to form a minimal local software workflow.

I know this should grow. Web search is a clear next tool to add. So are richer file operations, git awareness, and more robust execution controls. But even the minimal tool set already teaches the main lesson: an agent is a planner attached to actuators.

## Permissions changed how I thought about trust

Another thing that became much clearer while building this is that permissions shape the relationship between the human and the agent. They are not just a safety add-on.

A permission prompt says:

I understand what I want to do. Do you authorize it?

That makes the interaction legible.

You get a collaborator that proposes actions instead of a silent script runner. That distinction matters, especially in a coding environment where the difference between "read a file" and "overwrite a file" is huge.

Even in a tiny prototype, permission prompts make the architecture feel more honest.

## Memory

At first glance, memory sounds like a huge topic. At scale, it is.

But at the level of a minimal agent, memory is just the answer to:

What should survive into the next turn?

Building this made that question feel much more concrete. An agent does not need omniscience. It needs continuity.

That can start with very basic things:

- where am I working
- what did the user just ask me to do
- what tools were used recently

Even a small memory file gives the session a sense of shape. The agent no longer starts from zero every time.

Later, I want to expand this. Better summarization, more structured task state, maybe long-term memory. But the tiny version taught me that memory is just persistent state with good taste.

## Skills are a beautiful idea

One concept I like a lot in command-line agent design is skills.

A skill is basically a reusable local instruction. A way of saying: in this environment, here is how we tend to work.

That might include:

- preferred commands
- repo conventions
- coding style hints
- safety norms
- workflow shortcuts

I find this elegant because it separates general intelligence from local practice.

The model does not need every detail hard-coded into it. The environment can teach it how to behave here.

That feels like a very powerful pattern, and it is one I want to keep expanding.

## Hooks and plugins are where systems become ecosystems

I have not built a plugin system here, and I think that was the right choice for the first cut.

But while building the project, it became obvious where those extension points naturally live.

Hooks are the moments around the loop where other behavior can attach:

- before a request is planned
- before a tool is executed
- after a tool completes
- before memory is written
- after a turn ends

Once you can see those hook points, you can imagine all kinds of extensions:

- logging
- policy checks
- richer approval UIs
- repo-specific automation
- task tracking

Plugins are really just formalized ways of attaching to those seams.

I have not implemented them yet, but I now understand where they belong.

## Sub-agents are less magical than they sound

"Sub-agents" can sound much more complicated than it actually is.

In practice, the idea is straightforward:

- split a larger task into smaller tasks
- run the same core loop on each smaller task

That is it.

What changed for me while building this project is that I stopped thinking of sub-agents as some exotic higher-order structure. They are mostly recursion plus boundaries. A parent agent delegates a smaller scope of work, then integrates the result.

That is conceptually simple, and once you see it, a lot of agent orchestration starts to feel much less mystical.

## Worktrees and environment boundaries matter

One area I have not built out yet, but absolutely want to, is a stronger concept of workspace boundaries and worktrees.

This matters because real coding agents should be well-scoped, not just powerful.

Worktrees, sandboxing, and explicit workspace control are the kinds of features that let an agent operate in a serious engineering environment without feeling reckless. They define where the agent is allowed to act and how isolated those actions are.

That becomes more important the more useful the system gets.

## The future tool belt

A tiny agent teaches you two things at once:

- what the irreducible core is
- what is still missing

I already know I want to keep building this into something larger. Some things I want to add over time:

- web search
- more robust file editing
- better context selection
- git-aware workflows
- safer execution boundaries
- richer memory
- stronger hooks
- real sub-agent orchestration
- worktree support
- a larger skill system

These now feel like understandable extensions of a clear core, not random features added to a pile.

## What this project taught me

More than anything, this project taught me that command-line agents are understandable.

Not trivial. Not solved. Not easy to perfect.

But understandable.

That matters.

A lot of modern AI tooling is presented as hype or as opaque infrastructure. I wanted a different relationship to it. I wanted to build one with my own hands in a form small enough that I could see the whole organism at once.

And that worked.

I understand these systems much better now than I did before building this. I stripped one down to its control loop and lived inside the design decisions.

## Why I am going to keep going

This is the first honest version of the project, not the finished one.

That distinction matters to me.

I want this to grow into something more serious over time. A stronger command-line coding agent, with better tooling, better safety, better memory, better context handling, and a larger model of how local software work should be done.

But I want it to keep one property as it grows: legibility.

I want the core ideas to remain visible.

That is what I admire in compressed educational projects. They remind you not to hide the essence of the system from yourself.

That is what this project has been for me.
