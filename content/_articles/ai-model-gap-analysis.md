---
title: "Knowledge, Behavior, or Capability?"
subtitle: "Your model is underperforming. RAG vs. fine-tune is the wrong question — diagnose the gap before you build the pipeline."
date: 2026-06-15
tags: [ai, llm, rag, fine-tuning, evaluation, architecture]
---

> The follow-up to [Code, Optimizer, or Model?]({{ '/articles/ai-decision-process/' | relative_url }}). That piece gets a step
> *into* the model bucket. This one is what to do when it's there and still falling short.

---

## The problem we're solving

You did the first triage and the step genuinely landed in the model bucket — it needs meaning,
not a `SELECT` and not an optimizer. You shipped it. It's underperforming. Now two camps form in
the room: *"add RAG"* and *"fine-tune it."* Both cost real money and weeks of plumbing, and at
least one of them is usually solving a problem you don't have.

The waste here is subtler than handing a query to an LLM. It's standing up a retrieval pipeline
for a model that doesn't have a knowledge problem, or fine-tuning a model whose facts go stale a
month later. A fix that doesn't match the gap isn't just ineffective — it's expensive, and it
quietly convinces the room the problem is unsolvable when really it was just misdiagnosed.

One question again, asked *before* any infrastructure: **knowledge, behavior, or capability?**

---

## First pass (the common framing)

Most teams arrive with a binary: **RAG or fine-tune.**

- **RAG** when the model needs *facts* it doesn't have — your data, your docs, recent events.
- **Fine-tune** when the model needs to *act* a certain way — your format, your tone, your
  domain's idioms.

Both are real, and "inject knowledge vs. shape behavior" is a genuine split — it already saves you
from the most common blunder, fine-tuning to teach facts.

But the binary is incomplete. It assumes every model shortfall is one of these two, and it isn't:
there's a third the reflex can't see — **capability** — where the model simply doesn't have what
the task needs. That's the rest of this piece.

---

## Capability failure

There's a failure mode that isn't missing facts and isn't wrong behavior: the model **can't
reason through the task at all.** Perfect context, perfect format examples — and the logic still
comes out wrong. That's a **capability gap**, and it's invisible to the RAG/fine-tune binary
because both of those change what the model *has* and how it *responds*, never how well it
*thinks*.

So the real split is three, not two:

> **Where's the gap?**
> Doesn't *know* it → **knowledge** → retrieve.
> Doesn't *do* it right → **behavior** → prompt / tune.
> Can't *reason* it → **capability** → decompose and offload first; a stronger model only for what's left.

The expensive mistakes almost all come from forcing a capability gap into one of the first two
buckets: RAG-ing harder at a task the model is simply too weak to reason through, piling on
retrieval cost and latency without buying a single point of accuracy.

---

## Second pass (refined)

### The three gaps

| Gap | Symptom | What's actually missing | Fix |
|---|---|---|---|
| **Knowledge** | Confident, specific, and *wrong*; or "I don't know" about something that's in your data. | Information the model was never trained on and can't have. | Retrieval (RAG), tool calls to source systems, grounding. |
| **Behavior** | Right answer, wrong shape — format drift, wrong tone, ignores instructions, inconsistent. | A response pattern the base model doesn't default to. | Prompt + few-shot first; fine-tune when the pattern is stable and high-volume. |
| **Capability** | Wrong *logic* even with perfect context and perfect examples. | Reasoning depth the model doesn't have. | Decompose into smaller steps; offload the pieces that aren't meaning; a stronger model only for the residual step. |

### Diagnose in order — cheapest test first

You don't guess which gap you have. You run three tests, in order, and stop at the first one that
fixes it:

1. **Paste the ideal context in by hand.** Give the model exactly the facts it would need,
   inline. Fixed? → **knowledge gap.** Build the retrieval that delivers those facts
   automatically. (You just did RAG by hand — see the callback below.)
2. **Show it three good examples.** Still the wrong shape? Few-shot it with exemplars of the
   output you want. Fixed? → **behavior gap.** Keep the few-shot if the token cost is tolerable;
   fine-tune when volume makes the prompt overhead hurt.
3. **Hand it perfect context *and* examples.** Still wrong? The gap is **capability.** No amount
   of retrieval or tuning closes it. Probe with a frontier model to confirm the task is reachable,
   then *decompose* to deliver it cheaply — upgrade the standing model only for whatever step is
   still irreducibly hard.

This ordering *is* the discipline. The two cheap tests cost a few minutes in a playground; the
two expensive fixes — a retrieval pipeline, a fine-tune run — cost weeks. Never build the
infrastructure before the playground test tells you which infrastructure you need.

### Symptoms of misdiagnosis

- **Fine-tuning to inject knowledge.** The single most common waste. Weights are a terrible
  database: the facts go stale, you can't update one without a retrain, and the model still
  hallucinates around the edges. Knowledge → retrieval, never weights.
- **RAG to fix behavior.** Piling more context into the prompt does not make a model follow your
  format. You'll watch retrieval quality climb while format compliance stays flat. Behavior →
  examples / tuning.
- **Either, for capability.** Retrieval and tuning both assume the reasoning is already there and
  something else is in the way. When the reasoning isn't there, both just make a wrong answer
  more expensive.

### The escape hatch — behavior when you don't own the model

"Behavior → tune" quietly assumes you can change the weights. As an API consumer you can't, so the
fix collapses one rung: prompt + few-shot, and persist the exemplars as retrievable **memory** the
agent loads each session. Mechanically that's retrieval feeding behavior — the very thing the bullet
above calls a mistake. The difference is *why*: you're not hoping more context will coax out a
format the model already knows; you're injecting a **correction as a fact** because you have no way
to train it as a reflex. Same mechanism, opposite diagnosis.

It stays a workaround, not a graduation. It costs context on every call and never amortizes to free
the way a fine-tune eventually does — so it's the right move precisely *because* the proper fix
isn't yours to make, and only until it is.

### The capability fork — decompose before you upgrade

A capability gap has two ways out, and the order is the whole point, because the obvious move is
the wrong default.

- **Decompose — first.** Break the one hard judgment into smaller ones, and two things fall out.
  Each piece often drops back within reach of the *cheap* model you already had. And some pieces
  turn out not to be model work at all — arithmetic, a lookup, a constraint — which you **offload**
  to code or an optimizer (the *numbers* bucket you mis-filed as *meaning* — re-bucketing,
  straight out of the first framework). This is the *small core* discipline turned inward: shrink
  the model's job until a small model can hold it. It costs engineering once and then runs cheap
  forever.
- **A stronger model — to probe, then for the residue.** A frontier model is the fastest *test*
  you have: if it nails the prompt your weak model failed, you've confirmed both that it's a real
  capability gap and that the task is reachable. That makes it an excellent diagnostic and a poor
  default — shipping it means paying its premium on every call, forever, scaling with volume,
  where decomposition was a one-time cost. So use it to confirm the ceiling, then decompose
  underneath it, and keep it as the standing answer only for the one residual step that's still
  irreducibly beyond the cheap model. Then you're paying the premium on the smallest possible
  slice, not the whole task.

The reflex — "it's not smart enough, get a smarter one" — is the first framework's mistake one
level up: throwing *more model* at a problem instead of doing the work to make the problem
smaller. **The stronger model is your probe, not your product.**

### How this connects to the first framework

- **RAG is extraction at query time.** The knowledge-gap fix is the *bridge mode* from the first
  piece — extraction — pointed at a corpus instead of a single input. Retrieve, extract the
  relevant facts into the prompt, then let the model reason. Same move, same discipline.
- **Decompose and offload are "shrink the core."** Every capability fix is an instance of the
  deterministic-shell pattern: make the model's irreducible job as small as you can, and push
  everything else into cheaper, testable tools.
- **You can't diagnose without evals.** All three tests above are measurements. Without an eval
  harness you can't tell a knowledge gap from a capability gap, and you certainly can't tell
  whether the expensive fix worked. Evals are the determinism-substitute from the first piece —
  here they double as the *diagnostic instrument*.

### A worked example — the PR review agent's "do the review"

Same agent as the first framework. Step 5 is correctly in the model bucket — reasoning over code
semantics is the reason the agent exists — but its reviews are weak. Three different weaknesses,
three different fixes:

| Symptom | Gap | Fix |
|---|---|---|
| Flags code that's fine and misses that a helper is deprecated *in our codebase* — it doesn't know our conventions. | **Knowledge** | Retrieve our standards doc, the file's neighbors, and prior review comments into the prompt. Don't fine-tune house style into the weights — it'll drift the next time the standard changes. |
| Catches the right issues but the write-up rambles, skips our severity tags, and varies run to run. | **Behavior** | Few-shot it with three past reviews in our format. If the agent runs thousands of times a day and the prompt overhead bites, *then* fine-tune the format. |
| With the standards *and* good examples in front of it, it still can't trace a cross-file race condition. | **Capability** | Decompose: map the call graph deterministically, then ask the model about specific edges — a smaller question the cheap model can answer. Use a frontier model to confirm the trace is even possible; upgrade the standing model only if that one edge-judgment still beats it. |

Note that only one row is a fine-tune, and only above a volume threshold; and the capability row
*decomposes before it upgrades*. Both follow the same rule — the smarter model and the trained
model are recurring costs, so they come last and only for the slice that's left.

---

## TLDR - tuning how your model performs

The step is in the right bucket but underperforming. Ask, in order:

1. **Does it work if I paste the facts in?** → Yes: **knowledge gap** → build retrieval (RAG =
   extraction at query time).
2. **Does it work if I add examples?** → Yes: **behavior gap** → few-shot, then fine-tune at
   volume.
3. **Still wrong with perfect facts + examples?** → **capability gap** → decompose, and offload
   the pieces that aren't meaning → a stronger model only for the residual step. Probe with a big
   model to confirm it's reachable; don't ship it as the default.

And the rule under all three: **diagnose in the playground before you build in the pipeline.**

---

## The expensive habit

Every wrong fix in this piece has the same tell: it's *standing* infrastructure bought to skip a
*one-time* diagnosis. Retrieval for a behavior problem, a fine-tune for a knowledge problem, a
frontier model for a problem decomposition would have solved — each one bills you again on every
call, for a question you could have settled in a playground in an afternoon. The smartest model,
the biggest pipeline, and the trained weights are the three costs that never stop. Reach for them
last, and only for the slice still standing after everything cheaper has done its part.

---

*Steven Molen, Sr. Enterprise Architect*
*Co-authored with Claude Opus 4.8 — insight and judgment mine, articulation ours.*

*Part two of a two-part series. Start here:*

- *[Code, Optimizer, or Model?]({{ '/articles/ai-decision-process/' | relative_url }}) — the decision process for what AI should do in the first place, before you ever ask whether it's underperforming*
