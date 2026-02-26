# AI Context Is Everything

*On the one constraint that explains every other technique in AI-assisted development, and why ignoring it is the fastest way to produce expensive garbage.*

---

## The Elephant in the Room

Every article I write about AI-assisted development circles the same drain. Spikes, planning files, working agreements, timeboxing, kanban boards -- they're all useful techniques. But none of them explain *why* they're necessary. None of them name the constraint they exist to manage.

So let me name it: **the AI context window is finite, and everything else follows from that fact.**

This isn't a temporary limitation. It isn't a problem that next year's model will solve. Context windows will get larger, yes, but so will the projects we throw at them. The constraint is structural. Every decision you make about how to work with AI -- every prompt you write, every file you organize, every session you plan -- is, whether you realize it or not, a decision about how to manage a scarce resource: the AI's ability to hold and reason about information at one time.

If you're using AI to build software and you don't understand this, you're driving blind. Everything else I've written assumes you understand this. So this is where we start.

---

## What a Context Window Actually Is

Here's the minimum you need to know. An AI model doesn't see characters or words. It sees *tokens* -- chunks of text, roughly three-quarters of a word each. Every model has a maximum number of tokens it can process in a single session: the context window. Claude's is around 200,000 tokens. GPT-4's varies by version. Copilot's is smaller still, and it manages it aggressively behind the scenes.

Everything -- your prompt, the files you reference, the AI's own responses, the conversation history, the system instructions, the skills that load -- all of it competes for space in that window. When the window fills, the model doesn't gracefully summarize and continue. It starts losing information. Earlier parts of the conversation degrade. The AI's responses become less coherent, less grounded, more prone to the kind of confident nonsense that looks right but isn't.

The context window is not "memory." It's a desk. A finite surface where the AI can spread out its work. When the desk is full, papers start falling on the floor. The AI doesn't know they fell. It just keeps working with whatever's still on the surface.

Every technique I've developed for myself and my own development working with AI is a strategy for managing that desk. Hopefully this understanding and these concepts will help other craft their own experience and gain their own success stories enabled by AI.

---

## Pillar 1: Limit What the AI Consumes Per Session

The most direct response to finite context is discipline about what goes into it.

Every token the AI spends processing the wrong thing is a token it can't spend on the right thing. Dump your entire codebase into a prompt and the AI won't understand it better -- it'll understand everything *worse*, because the signal is buried in noise. The desk is covered in papers and the AI can't find the one it needs.

This is why clear requirements matter more with AI than with human developers. A human developer who gets a vague requirement will waste *their* time wandering. An AI that gets a vague requirement wastes *context* wandering. Every wrong guess, every abandoned approach, every "let me try something else" consumes window space that doesn't come back. Three failed attempts at the wrong solution mean the fourth attempt -- the right one -- has less context to work with.

Timeboxing is context hygiene. When you tell the AI "try three times, then stop and report," you're not just preventing wasted effort. You're preventing context pollution. Twenty failed approaches stacked up in a session don't just waste time -- they actively degrade the AI's ability to reason about the problem, because they're all sitting on the desk, crowding out the information that actually matters.

Spikes are the same principle applied to research. Separating investigation from implementation -- "produce a research document, not code" -- keeps two different context-consuming activities in separate sessions. The research spike fills a context window with documentation and synthesis. The implementation session fills a different window with architecture and code. Neither contaminates the other.

The discipline is simple: every session should have a clear purpose, a bounded scope, and only the information needed for that purpose loaded into context. Anything else is waste.

---

## Pillar 2: Help It Remember Across Context Boundaries

AI agents have no persistent memory across sessions. Every conversation starts from zero. This is not a bug. It's the architecture. The context window is the *only* memory the AI has, and when the session ends, the window closes.

This is the problem that blindsides most developers. You spend two hours in a productive session. The AI understands your architecture, your patterns, your constraints. It's producing clean, consistent output. Then the session ends -- a crash, a token limit, you close the laptop -- and everything the AI learned is gone. The next session starts with a blank context window and no idea what happened before.

The solution is external memory. Not AI memory -- *your* memory, encoded as files the next session can read.

Planning files in a `planning/` folder. Research documents in `research/`. Completed task records with execution notes. A kanban board tracked as markdown files. These are not project management overhead. They are the external memory system that compensates for the fact that your team member gets amnesia at every context boundary.

The pattern is consistent: capture knowledge as artifacts that outlive sessions. Not as conversation context that dies when the window closes. Not as "the AI will remember" -- it won't. As files. On disk. That the next session reads on startup.

Every minute you spend writing a plan file or a research summary is a minute you *don't* spend re-explaining the same context to a fresh session. That's not overhead. That's the highest-leverage work you can do.

---

## Pillar 3: Bridge the Gaps for Holistic Solutions

Here's where it gets architectural. Pillars one and two handle individual sessions: keep them focused, capture their output. But software isn't built in one session. A real project spans dozens or hundreds of sessions, each with its own context window, none of which can see the whole.

So how do you build something cohesive when no single session can hold all of it?

Four  mechanisms, working together.

### Architecture Files: What NOT to Do

Your instruction files -- `CLAUDE.md`, `.copilot-instructions.md`, whatever your tool reads -- set the boundaries that every session inherits. "Use TypeScript interfaces for all message types." "Handlers go in `handlers/`. Services go in `services/`." "Do not create god objects."

These constraints are small enough to fit in every context window and powerful enough to keep independent sessions building within the same guardrails. Session twelve doesn't need to see what sessions one through eleven produced. It just needs the same architectural rules, and the code it produces will be compatible with everything else.

This is why instruction files should be treated as working agreements, not configuration. They're the connective tissue between sessions that will never talk to each other.

### Planning Files: What TO Do Next

Architecture files tell each session how to behave. Planning files tell each session what to build and where the project is headed. The plan is the thread that keeps independent sessions building one coherent thing instead of twenty disconnected pieces.

When a session reads an in-progress plan with steps one through three checked off and step four next, it doesn't need the full project history. It needs the plan, the relevant code, and the architectural rules. That's enough context to contribute meaningfully to the whole.

### Pluggable Skills: The Right HOW, On Demand

This is the piece most people miss. If architecture files and planning files are the static context -- loaded every session, always present -- then skills are the dynamic context. They load only when relevant and bring only what's needed for *this* task to the context window.

A test-driven development skill loads when you're writing tests. A debugging skill loads when something breaks. A research skill loads when you're spiking. Each one provides targeted capability without bloating every session with everything the AI might eventually need.

This is context efficiency by design. Instead of cramming every possible instruction into the context window upfront, skills deliver the right methodology at the right moment. The context window stays focused on the actual work.

Skills are also portable -- they're markdown files, not conversation state. When your tokens run dry on one tool and you switch to another, the skills come with you. The context strategy survives the tool switch because it was never dependent on the tool in the first place.

### Workflow: Stitch It Together Yourself

Architecture files, planning files, skills -- these are tools. They don't assemble themselves into a workflow. You have to do that part, and nobody else's workflow is going to fit you perfectly.

Everyone in this space wants to sell you theirs. "Here's my ten-step AI development process." "Here's the framework that changed everything." "Here's the silver bullet." There is no silver bullet. There never has been in software engineering, and AI doesn't change that.

What works is personal. It depends on how you think, how your team operates, what you're building, and what stage the project is in. My workflow -- spikes, kanban boards, timeboxing, structured plans -- works for me because I've run Scrum teams for decades and those patterns are already in my muscle memory. Yours might look completely different and produce equally good results.

The recommendation is simple: fill your toolbox. Read what others are doing. Steal the ideas that click and ditch the rest. Don't be afraid to change your workflow next month when you learn something new. The developers who get stuck are the ones who adopt someone else's process wholesale and treat it as doctrine instead of a starting point.

This is the same lesson every new Scrum team learns. You don't copy another team's working agreement. You build your own, informed by experience and adapted to your context. Then you iterate. The retrospective exists because no process is right on the first try.

Your AI workflow is no different. The tools in this article are building blocks. How you assemble them is up to you.

---

## The Realization

The pattern is quite obvious once you step away from the trees to see the forest.

Scrum patterns for AI? Context management. Structured prompts? Context management. Spikes? Context management. The research library? Context management -- pre-loading the desk with validated information so the AI doesn't fill it with hallucinations. Timeboxing? Context hygiene. Vendor portability? Context management -- your skills and artifacts survive tool switches because they're files, not conversation state.

Every practice I've developed, every technique that's actually worked -- all of them are strategies for managing a finite context window. The ones that seem like "good engineering practices" are. They just happen to be *especially* critical when your team member can only hold 200,000 tokens of information at a time and starts from zero every session.

This isn't a coincidence. It's the physics of the medium.

The developers who thrive with AI are the ones who design for the constraint. Every prompt, every file, every workflow decision is an answer to the same question: "How do I make the best use of the context window I have?"

---

## What I Learned

**Context is the constraint. Everything else is a coping strategy.** Every technique that works with AI -- requirements, planning, spikes, timeboxing, working agreements, skills, kanban boards -- is a response to finite context. Understand the constraint and the techniques make sense. Ignore it and nothing works consistently and the applications the AI helps you write fall apart almost as soon as they hit the market.

**Tokens spent on the wrong thing are tokens you don't get back.** Every vague requirement, every failed attempt, every unnecessary file loaded into context reduces the AI's ability to do the actual work. Be deliberate about what goes into the window.

**External memory isn't overhead. It's the system.** Plan files, research documents, completed task records -- these aren't project management bureaucracy. They're the memory your AI doesn't have. Skip them and you'll spend more time re-explaining than building.

**Architecture files are the cheapest bridge between sessions.** A few lines of constraints in your instruction file cost almost nothing in context and prevent sessions from diverging. They're the highest-leverage artifact you can write.

**Skills are context on demand.** Static rules load every session. Skills load when needed. This distinction matters when every token counts.

**The window will get bigger. The constraint won't go away.** Next year's model will have a larger context window. Your projects will also be larger. The discipline of managing context isn't a workaround for today's limitations -- it's a permanent feature of working with AI.

---

*Steven Molen, Sr. Enterprise Architect*
*GIST co-authored with Claude Opus 4.6 -- because the best thing you can do for your AI is understand the box it's working inside.*

*This is part of a series on building VS Code extensions with AI. See also:*
- *[AI-DEVELOPMENT-IS-SCRUM-WITH-SMALLER-ENGINEERS.md](AI-DEVELOPMENT-IS-SCRUM-WITH-SMALLER-ENGINEERS.md) -- On managing non-deterministic junior developers*
- *[AI-SPIKES-EXPLAINED.md](AI-SPIKES-EXPLAINED.md) -- Two techniques that save hours of thrashing*
- *[AI-NO-VENDOR-LOCK-IN.md](AI-NO-VENDOR-LOCK-IN.md) -- On switching AI tools without missing a beat*
- *[THE-AI-JOURNEY.md](THE-AI-JOURNEY.md) -- How AI is both a 10x multiplier and a 10x liability*
- *[VSCODE-EXTENSIONS-ARE-CLIENT-SERVER.md](VSCODE-EXTENSIONS-ARE-CLIENT-SERVER.md) -- The mental model nobody gives you*
- *[MARKDOWN-IS-THE-LANGUAGE-OF-AI.md](MARKDOWN-IS-THE-LANGUAGE-OF-AI.md) -- On the millions being left on the table*

---
