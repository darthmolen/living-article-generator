# AI-Generated Plan: Direct Repository Injection (Less Than Optimal)

## What the AI Proposed

When asked to add Postgres persistence to a multi-agent swarm orchestrator, the AI designed a **direct repository pattern** where:

1. A `SwarmRepository` class wraps all DB operations
2. The repository is injected into the `SwarmOrchestrator` as an optional dependency
3. Every state mutation in the orchestrator gets an `if self.repository:` guard that writes through to the database
4. Existing in-memory stores (TaskBoard, InboxSystem, TeamRegistry) remain as-is
5. The orchestrator becomes responsible for coordinating both cache and persistence

## The Architecture

```
SwarmOrchestrator
  ├── self.task_board = TaskBoard()           # in-memory
  ├── self.inbox = InboxSystem()              # in-memory
  ├── self.registry = TeamRegistry()          # in-memory
  ├── self.repository = SwarmRepository()     # optional DB
  │
  ├── _plan():
  │     await self.task_board.add_task(...)
  │     if self.repository:                   # scattered guards
  │         await self.repository.create_task(...)
  │
  ├── _execute():
  │     await self.task_board.update_status(...)
  │     if self.repository:                   # more guards
  │         await self.repository.update_task_status(...)
```

## Why This Seems Reasonable

- Existing 299 tests pass unchanged (repository is None by default)
- Additive changes only — no modifications to TaskBoard/InboxSystem/TeamRegistry
- Clear separation: in-memory stores for speed, repo for persistence
- Write-through pattern is well-understood

## Where It Falls Apart

### Problem 1: Scattered `if self.repository` Guards

Every mutation point in the orchestrator needs a guard. The orchestrator has 7+ mutation sites across 5 phases. Each one is a place where cache and DB can drift if someone forgets the guard or adds a new mutation path.

### Problem 2: Orchestrator Knows Too Much

The orchestrator is already responsible for QA, planning, spawning, executing, and synthesizing. Adding "also coordinate persistence" makes it a god object. The SRP violation compounds with every feature.

### Problem 3: No Abstraction Over the Cache

TaskBoard, InboxSystem, and TeamRegistry are direct dependencies. If you later want to swap the cache (Redis for horizontal scaling, for example), you'd need to modify every call site in the orchestrator again.

### Problem 4: Recovery is Bolt-On

Loading swarm state from DB to reconstruct in-memory stores requires a separate code path. The orchestrator needs a `_hydrate_from_db()` method that manually populates each in-memory store — duplicating the knowledge of how state is structured.

### Problem 5: Two Sources of Truth During Writes

During a write, the in-memory store is updated first, then the DB. If the DB write fails, the cache is ahead of persistence. There's no transaction boundary that spans both. Retry logic would need to reconcile this, adding more complexity to the orchestrator.

## The AI's Blind Spot

The AI designed for the immediate task (persist state) without considering the system's future evolution. It optimized for "minimal changes to existing code" rather than "sustainable architecture." The result works today but creates coupling that makes every subsequent feature harder:

- Want Redis cache? Rewrite all orchestrator call sites.
- Want event sourcing? The dual-write pattern conflicts.
- Want to test persistence independently? Can't — it's interleaved with orchestration logic.
- Want another consumer of swarm state? They need to know about both the cache and the repo.
