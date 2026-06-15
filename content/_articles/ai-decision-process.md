---
title: "Code, Optimizer, or Model?"
subtitle: "A decision process for what AI should actually do — and why handing number problems to a language model burns tokens."
date: 2026-06-15
tags: [ai, architecture, decision-making, llm, optimization]
---

## The problem we're solving

Everybody wants to use the AI tool. Fewer people can say *what for*. The result is wasted
tokens: an LLM doing a job a `SELECT` statement would have done faster, cheaper, and without
ever making something up.

We need a way to look at any step in a real workflow and answer one question: **code, optimizer,
or model?** This piece builds that framework the way the work actually happens — a rough draft
that gets something wrong, then a second pass that fixes it.

That two-pass structure is the point, not an accident of editing. Nobody arrives at the right
answer on the first try. This is spec-driven development applied to your own thinking: write the
workflow down, look at what you wrote, and revise *before* you commit any of it to code. The
cheapest place to find a design flaw is in a markdown file.

---

## First pass (rough draft)

The starting frame: every step in a pipeline is either **deterministic** (same input → same
output, you can write the rule) or **non-deterministic** (the output varies, it takes judgment).
Deterministic steps are code. Non-deterministic steps are where AI comes in.

Two worked examples.

### Example 1 — Unit routing (receiving → technician's desk)

1. **Receipt**
2. **Route**
   - a — what kind of unit is this? *(deterministic)*
   - b — who is best able to repair this? *(non-deterministic — schedule, current availability,
     historical performance)*
   - c — how do we get it there? *(could go either way)*

### Example 2 — PR Review Agent

1. Receive the PR — *(D)*
2. Git clone — *(D)*
3. Run analyzers — *(D)*
4. Determine how the PR should be handled — *(N)*
5. Do the review — *(N)*
6. Submit the comments — *(D)*

This frame is useful. It already separates the plumbing from the thinking, and it correctly
flags steps 5 and "who repairs this" as the hard parts. If you stopped here you'd be ahead of
most teams.

But it has one flaw, and the flaw is the expensive kind: it quietly equates *non-deterministic*
with *AI's job*. That's the assumption that burns tokens.

---

## Numbers or meaning

There isn't one kind of non-deterministic. There are two that matter here, and only one of them belongs to an LLM.

- **Unspecifiable but structured.** No clean rule, but the decision is made over *quantifiable*
  signals — numbers. This is an optimization or ranking problem. The right tool is a scoring
  function, an optimizer (OR-Tools, Hungarian algorithm), or a small ML ranker.
- **Unspecifiable and unstructured.** The decision requires understanding *meaning* — language,
  code, intent, context you can't reduce to columns. This is the LLM's home.

So the real split isn't deterministic vs. non-deterministic. It's:

> **Can I write the rule?**
> If yes → code.
> If no → **can I distill it to numbers, or does it need context and meaning to resolve intent the data doesn't hold?**
> Numbers → optimizer. Meaning → model.

The token-waste epidemic is almost entirely people taking the *numbers* bucket and handing it to
a language model because it felt "non-deterministic."

---

## Second pass (refined draft)

### The corrected axis

Three buckets, not two:

| Bucket | Property | Right tool |
|---|---|---|
| **Specifiable** | You can write the algorithm; same input → same output; auditable. | Code / lookup / query |
| **Unspecifiable, structured** | No rule, but the decision is over quantifiable signals. | Optimizer / scoring fn / ML ranker |
| **Unspecifiable, unstructured** | Requires understanding language, code, or intent. | LLM |

A fourth, orthogonal consideration governs everything: **blast radius.** How bad is it if this
step is wrong? That doesn't change *which* tool does the work — it changes how much deterministic
checking wraps the work.

### Extraction — the bridge between buckets

One model move doesn't sit *in* a bucket so much as *connect* them: **extraction** — turning
unstructured input into structure. OCR on a label, vision on a photo, pulling fields out of a
freeform note, transcribing a call. It's a model task — it takes meaning to do it — but its
*output* is exactly the numbers and fields the other two buckets run on.

That makes extraction the highest-leverage thing a model does in most pipelines, because it
**moves the rest of the work left.** Extract once, and everything downstream collapses into a
query or an optimizer — cheap, testable, reproducible. The model touches the input at the
boundary and then gets out of the way. It is the opposite of the token-waste pattern: instead of
the model *making* the decision, it *manufactures the data* a cheaper tool decides on.

Three rules follow:

- **It feeds a step; it doesn't replace one.** A deterministic step stays deterministic — it
  just can't run until its input is in the form the rule reads. When a deterministic task lacks
  the data because the input arrived free-form (a smashed asset tag, a typed-in symptom), you
  *prepend* an extraction step that produces the key, and the lookup runs exactly as before. The
  step never became a judgment call; you fed it. This is the flip side of *determinism is a
  property of the input, not the step* — the same observation, read as an instruction.
- **Extract, don't decide.** When the deciding signal is trapped in unstructured text — a tech's
  first-time-fix history buried in free-form repair comments rather than a metrics column —
  extract *that* into a feature (a first-time-fix-rate number), then let the optimizer assign.
  Don't let the model make the live call; let it produce the column the optimizer was missing.
- **Extract once, then cache.** Extraction output is stable for a given input, so it memoizes.
  Re-running a vision model on the same photo every request is the same waste as re-deriving a
  value you already computed.

Extraction is one of three things the model does once you're in its bucket — and the only one
whose goal is to get you back *out* of it. The other two stay in the core: **judgment** — weigh a
situation and decide (PR risk, repair quality) — and **generation** — produce an artifact with no
single right form (the review write-up, a summary, a drafted reply). Which mode you're in changes
how you prompt and what "correct" means; all three still want the smallest possible core inside a
deterministic shell.

When a step is correctly in the model bucket and *still* underperforms, diagnosing why —
knowledge, behavior, or capability — is its own decision. That's the follow-up:
[Knowledge, Behavior, or Capability?]({{ '/articles/ai-model-gap-analysis/' | relative_url }}).

### Example 1, LLM was the wrong default choice

Re-labeling the routing pipeline with the sharper axis produces an uncomfortable result: **almost
none of it is an LLM task.**

| Step | First pass | Second pass | Why it moved |
|---|---|---|---|
| What kind of unit? | deterministic | **deterministic — *unless input is degraded*** | Asset-tag scan → type lookup. But a smashed label + a tech's scribbled note flips it to a vision/inference task. The determinism is a property of the *input*, not the step. |
| Who repairs it? | non-deterministic (→ implied AI) | **optimizer** | Schedule, availability, first-time-fix rate by unit type — all numbers. A weighted `ORDER BY` or an assignment optimizer. An LLM here is strictly worse: slower, costs money per call, non-reproducible, can't be unit-tested. |
| How to get it there? | could go either way | **deterministic or optimizer** | Inside a building: bin/shelf routing from a location map. Across sites: logistics optimization. It never really lands on the LLM. |

The lesson this example teaches *better than the PR one*: "non-deterministic" lured us toward AI,
and the correct answer was an optimizer the whole time. The one place the model earns its keep is
the degraded-input fallback for step a — reading a photo or a freeform note when the scan fails.
That's *extraction*, not a decision: the model rebuilds the unit-type the lookup needed and hands
it back, and the deterministic step runs as if the scan had worked.

Note that "who repairs it" *could* pull toward the LLM if the deciding signal were unstructured —
e.g., "Jim's gotten fast on the new Latitude boards" lives in Teams or the historicals live in free-form comments,
rather than in a metrics table. Even then, the right move is to *extract* that into a feature, not let the model do the
live assignment.

### Example 2, re-examined

The PR pipeline's D/N split was mostly right. Three refinements:

| Step | First pass | Second pass | Why |
|---|---|---|---|
| Run analyzers | (D) | **(D) — and a *principle*** | Run deterministic tools *first*, feed their output to the model as context. Never make the LLM rediscover a syntax error the compiler already found. Every such token is waste by definition. Analyzers exist partly to *shrink* the review job. |
| Determine handling | (N) | **mix** | Triage — docs-only? trivial diff size? touches `auth/` or a CODEOWNERS path? — is path globs and arithmetic, fully deterministic. The irreducibly-N part is judging *risk/complexity* of a real change. Don't spend model tokens deciding a README typo is low-risk. |
| Do the review | (N) | **(N) — the canonical LLM task** | Reasoning over code semantics, intent, idiom, subtle bugs. This is the part you cannot reduce to a rule, and the reason the whole agent exists. |
| Submit comments | (D) | **split** | The *mechanical posting* is a (D) API call. The *disposition* — approve / comment / block — is policy-gated and is where the deterministic shell clamps the model: never auto-approve a PR touching auth without a human; cap comment count; dedupe. |

### The pattern under both examples

**Deterministic shell, non-deterministic core — and make the core as small as you can.**

Both pipelines already show it: deterministic steps bookend the judgment steps. The discipline is
to push *everything you can* out of the core and into the shell, because the shell is cheaper,
faster, testable, auditable, and cannot hallucinate. The model should only ever be doing the part
that is irreducibly judgment-over-meaning. Path rules up front, analyzers next, model last — that
ordering isn't a convenience, it's the whole cost-control strategy.

### Don't try to make the model deterministic

A common reflex — "AI is non-deterministic, that's a problem" — leads people to try to nail the
LLM down. That defeats the reason you reached for it; its value *is* the unspecifiable.

You don't make the model deterministic. You make the **system** trustworthy by bounding the
non-deterministic core with deterministic checks and an evaluation harness. You can't prove the
output, but you can bound the *distribution* of outputs. For a regulated or governed context,
that's the determinism-substitute: evals are how you ship a non-deterministic component with a
straight face.

---

## TLDR - the AI decision process for software engineering

For any step in any workflow, ask three questions in order:

1. **Can I write the rule?** → If yes, it's code. Stop.
   - *Lacking the data because the input is free-form?* The step is still code — prepend an **extraction** step (model) to produce the key, then run the rule. You fed it; you didn't replace it.
2. **If not — numbers or meaning?** → Numbers go to an optimizer or ranker. Meaning goes to the model.
3. **What's the blast radius if it's wrong?** → That sets how thick the deterministic shell around it needs to be.

---

## Both drafts - nobody is perfect

The first frame wasn't wrong so much as *unfinished* — and you couldn't see what was unfinished
until it was written down and re-read. That's spec-driven development in miniature: the rough
draft surfaced a hidden assumption, the second pass named the missing distinction, and the
discovery cost a few paragraphs instead of a deployed agent routing repairs with a language model.

Think the workflow through before committing to code. The spec is where you're allowed to be
wrong cheaply.

---

*Steven Molen, Sr. Enterprise Architect*
*Co-authored with Claude Opus 4.8 — insight and judgment mine, articulation ours.*

*Part one of a two-part series. The companion piece:*

- *[Knowledge, Behavior, or Capability?]({{ '/articles/ai-model-gap-analysis/' | relative_url }}) — the model's correctly in the loop and still underperforming; diagnose the gap before you build the pipeline*
