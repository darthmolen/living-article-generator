# AI Spikes: Two Techniques That Save Hours of Thrashing

*On forcing your AI to stop generating code and start understanding the problem, before you both waste an afternoon.*

---

## The Problem Spikes Solve

AI agents have a bias toward action. Give one a problem and it will start building immediately -- writing code, generating files, producing output. It does not stop to ask whether it understands the problem. It does not research first. It does not consider whether the approach is sound. It just goes.

You've seen this with junior developers. The ones who start typing before they finish reading the ticket. The ones who produce a pull request in two hours that takes four hours to review and another three to fix. Speed without understanding is just rework with extra steps.

Spikes exist to interrupt that cycle. In Scrum, a spike is a timeboxed investigation where the deliverable is knowledge, not code. The concept maps directly to AI-assisted development, but I've found that AI spikes come in two distinct flavors, each triggered by a different situation and each producing a different kind of output.

---

## The Research Spike

### When to Use It

You don't know what you don't know. A new library, an unfamiliar API, an architectural pattern you've heard of but never implemented. You could tell the AI to "just build it" and watch it hallucinate method signatures and invent APIs that don't exist. Or you could spike first.

### The Process

The prompt is explicit: "This is a spike. Research the [thing]. Produce a summary document, not code."

I specify what I want to understand. Not "tell me everything about the Copilot SDK" -- that produces a generic overview that helps nobody. Instead: "Explore the session management API. What events are available? What does the lifecycle look like? Can we hook into session termination? What are the edge cases around multi-window sessions?"

Focused questions produce focused research. The same principle that makes good user stories makes good spike prompts.

### The Deliverable

The output goes into a `research/` or `documentation/` folder -- depending on whether you want it versioned with your source code or kept as a local reference. It's a markdown document. Not code. Not a prototype. A document that a human reads and makes decisions from.

This is the critical separation. AI is excellent at reading documentation and synthesizing findings. It's terrible at knowing what to do with those findings. And it's even worse at remembering that knowledge when it shifts into implementation mode. The spike captures the research as an artifact that survives the context boundary.

### Why This Works

The research document becomes a reusable asset. When you're ready to implement -- maybe in this session, maybe next week -- you point the AI at the document and say "implement based on these findings." The AI isn't guessing anymore. It's working from curated, validated knowledge that you've already reviewed.

You also get something subtle but valuable: you've forced yourself to read the research before writing code. That five-minute read often changes the implementation plan entirely. Libraries don't work the way you assumed. APIs have constraints you didn't expect. The spike surfaces all of that before you've written a line of code that assumes otherwise.

---

## The Bare Bones Code Spike

### When to Use It

This one starts differently. You're already building. The AI is already writing code. And something isn't working.

A bug has surfaced when consuming a component. Or the way you're using a library doesn't feel right -- the code works but it's fighting the component's design. The AI tries to fix it. Then tries again. Each workaround gets more creative and less correct. You're watching it chase its tail.

That's the trigger. The AI is struggling because it's trying to solve a complex problem through layers of your application's abstraction, and it can't see the actual issue through all that noise.

### The "Stop." Moment

This is the intervention: "Stop. Let's spike this with a simpler test."

The word "stop" matters. AI agents don't naturally pause and reconsider. They push forward. Left alone, the AI will attempt twenty increasingly baroque workarounds before it occurs to it (if it ever does) that the problem might be elsewhere. You have to be the one who calls the halt.

### The Process

The prompt I use follows a specific sequence:

1. **Pull the actual component code or documentation.** Not our wrapper, not our abstraction layer. The real thing. Download the source, copy the relevant docs, get the actual API reference into the AI's context.

2. **Research how the component's own samples handle the use case.** Every well-maintained library has examples. How do *they* do it? What patterns do the maintainers use? If the samples don't cover our use case, read the source and understand how the internals work.

3. **Write an integration test that strips the problem down to its absolute core, using the component directly.** No application code. No wrappers. No services. Just the component, a test, and the minimal setup to reproduce the issue or demonstrate the concept.

This is the key step. By removing every layer of your application's abstraction, you force the AI to work with the component as it was designed to be used. The test either passes or it doesn't, and either outcome tells you something definitive.

### The Verdict

The goal is two-fold:

1. **Is this a component issue or our code's issue?** If the integration test fails with just the bare component, you've found a library bug or a legitimate limitation. If it passes, your application code is doing something wrong.

2. **What is the right way to use this third-party code?** Even when there's no bug, the bare bones test reveals how the component is meant to be consumed. Half the time, the "bug" turns out to be your code fighting the component's intended API.

Either way, you now have a clean, minimal test that demonstrates the correct behavior. That test becomes documentation. It stays in your codebase as a reference for how the component actually works, separate from how your application wraps it.

---

## When to Spike vs. When to Just Build

Not everything needs a spike. Here's the decision:

**Spike when:**
- You're working with a library or API you haven't used before
- The AI has tried and failed twice on the same problem
- The code works but feels like it's fighting the framework
- You're about to build on an assumption you haven't verified
- The AI starts hallucinating method signatures or inventing APIs

**Just build when:**
- You've used this component before and understand its patterns
- The task is straightforward implementation with well-known tools
- The AI's first attempt is clean and correct
- You're making changes within code you already own and understand

The heuristic is simple: if the AI is struggling, or if you're uncertain, spike. Five minutes of investigation saves an hour of rework.

---

## The Anti-Pattern: Skipping the Spike

I've watched this play out dozens of times. The AI encounters a problem with a third-party component. Instead of stepping back to understand the component, it starts wrapping the problem in try-catch blocks, adding retries, building adapter layers, creating increasingly complex workarounds that treat the symptom while ignoring the cause.

The result is AI slop at its worst: code that technically works but is unmaintainable, fragile, and built on a misunderstanding of how the underlying component operates. Six months later, someone updates the dependency and the whole house of cards collapses because nobody understood the foundation it was built on.

The spike takes ten minutes. The workaround debt takes weeks to unwind. This is the same trade-off that Scrum has always been about: a small investment in understanding up front prevents a large investment in rework later.

---

## What I Learned

**AI doesn't know when to stop.** You do. The spike is the mechanism for exercising that judgment.

**Research and implementation are different activities.** Mixing them produces worse results for both. Separate them.

**The simplest test is the most revealing.** When you strip a problem down to its core, the answer is usually obvious. The complexity was hiding it.

**Spike artifacts outlive sessions.** The research document, the bare bones test -- these survive context boundaries, session crashes, and tool switches. The code the AI thrashed on for twenty minutes doesn't.

**You already know how to do this.** If you've ever told a junior dev "stop coding and go read the docs first," you've already run a spike. The discipline is the same. The team member is different.

---

*Steven Molen, Sr. Enterprise Architect*
*GIST co-authored with Claude Opus 4.6 -- because sometimes the best code you write is the code that tells the AI to stop writing code.*

*This is part of a series on building VS Code extensions with AI. See also:*
- *[AI-DEVELOPMENT-IS-SCRUM-WITH-SMALLER-ENGINEERS.md](AI-DEVELOPMENT-IS-SCRUM-WITH-SMALLER-ENGINEERS.md) -- On managing non-deterministic junior developers*
- *[THE-AI-JOURNEY.md](THE-AI-JOURNEY.md) -- How AI is both a 10x multiplier and a 10x liability*
- *[VSCODE-EXTENSIONS-ARE-CLIENT-SERVER.md](VSCODE-EXTENSIONS-ARE-CLIENT-SERVER.md) -- The mental model nobody gives you*
- *[MARKDOWN-IS-THE-LANGUAGE-OF-AI.md](MARKDOWN-IS-THE-LANGUAGE-OF-AI.md) -- On the millions being left on the table*

---
