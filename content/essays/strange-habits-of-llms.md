---
title: "Strange Habits of LLMs: Oops I Did It Again"
date: 2026-01-25T22:48:41-08:00
draft: false
category: ""
excerpt: "I was fixing a bug for my newborn's app and ChatGPT had a complete brain-body disconnect: It kept reasoning 'don't do the thing' → did it → 'oops I did this' → redo → 'oops I did it again'"
tags: [LLMs, Autoregressive, Psychology,]
---

I was fixing a bug for my newborn's app and ChatGPT had a complete brain-body disconnect:

It kept reasoning "don't do the thing" → did it → "oops I did this" → redo → "oops I did it again"

In the SAME response. Five times. (See pic)

**Five attempts. Same mistake. Every single time.**

I couldn't resist trying to anthropomorphize this: if this were a human, why would they do this?

---

**It's the AI equivalent of saying "um" then apologizing for saying "um," and doing it again.**

This is familiar. A manager once told me I say "um" too much. So I'd get into a presentation, hear myself drop an "um," pause and go, "Sorry for all the ums…" 

And then—two sentences later—"um, the key point is…" 

Again. 

Or like a tennis player with a wrong serve toss for 15 years. Coach explains the fix. They understand completely. But mid-serve muscle memory takes over. Ball's left their hand. Can't rewind, only finish and try again.

---

This fascinated me and took me down a rabbit hole. 

**Turns out it's because of three competing factors:**

**1. Autoregressive Generation**
LLMs generate one token at a time. Once generated, can't be deleted. Like saying "um" with no backspace—has to finish the sentence before apologizing and trying again.

**2. Statistical Momentum vs. Instructions**
Highest probability next token is $ because thousands of training examples have it. But instruction says "don't use $." 

Reasoning is right (model says not to use $) but next token is still $. 

Training data wins. Nature over nurture?

**3. Context Contamination**
Wrong token becomes part of context. Each failed attempt makes the wrong pattern more likely. Loop feeds itself.

---

**Takeaway: Don't Fight the Habit Mid-Action—Change the Environment**

Telling an LLM "don't do X" after it's already doing X is like telling yourself "don't say 'um'" while on stage. 

Don't keep pushing willpower. Change the setup:

→ Reset early - New message/new chat
→ Constrain the response - Diff-only or final snippet only
→ Add a verification step - Confirm the thing is not in the output

Same rule for humans and models: when stuck in a loop, step out and redesign the conditions.

---

**Spotted an LLM in an apology loop? Drop your favorite examples in the comments.**

#AI #LLM #MachineLearning #ChatGPT #ProductManagement #TechInsights #ArtificialIntelligence #PromptEngineering #AIProduct #TechLeadership