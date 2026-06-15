# Human-Corrected Plan: Service Layer with Real Infrastructure Testing

## Two Course Corrections

A senior engineer reviewed the AI's persistence plan twice. The first correction introduced the service layer pattern. The second pushed even further: eliminate SQLite test faking entirely and build real infrastructure from the start.

## Correction 1: Service Layer (Not Direct Repo Injection)

The AI proposed injecting a `SwarmRepository` directly into the orchestrator, guarded by `if self.repository:` at every mutation site. The engineer recognized this as a coupling anti-pattern and introduced `SwarmService` — a single abstraction that owns both cache (in-memory stores) and persistence (optional repo).

### Before (AI's Plan)
```python
# Orchestrator has 7+ sites like this:
await self.task_board.update_status(task_id, "completed", result)
if self.repository:
    await self.repository.update_task_status(swarm_id, task_id, "completed", result)
```

### After (Service Pattern)
```python
# One call. Service handles cache + persist internally.
await self.service.update_task_status(task_id, "completed", result)
```

The service wraps the existing in-memory stores (TaskBoard, InboxSystem, TeamRegistry) as its cache layer. When a repo is provided, writes go to both. When not, it's cache-only — identical to today's behavior. The orchestrator gets simpler. The cache implementation is swappable (Redis later). Recovery is natural: `service.load(swarm_id)` hydrates cache from DB.

## Correction 2: Real Postgres Tests from Day One (Not SQLite)

The AI's original plan used SQLite in-memory for repository tests. The engineer pushed back: SQLite hides real Postgres behavior — JSONB types, UUID handling, constraint enforcement, migration correctness. If the migration is wrong, SQLite won't catch it.

The corrected approach:

1. **Phase 0 builds real infrastructure first** — Docker Compose with Postgres, Alembic migrations, and a per-test database isolation fixture
2. **Every test from Phase 1 onward runs against real Postgres** — no SQLite anywhere
3. **Tests validate both code AND migrations** — if a migration is broken, the test fails
4. **Arrange steps use the repository** — no fixtures or faking. Need data? Insert it with the repo.

### Per-Test Database Isolation Pattern

Each integration test gets its own Postgres database:

```python
@pytest.fixture
async def db_engine():
    db_name = f"test_{uuid4().hex[:12]}"
    # Create fresh database
    # Run alembic upgrade head (real migrations)
    # Yield engine
    # Drop database on teardown
```

This pattern is reusable across any project with Postgres. The same `alembic upgrade head` command runs in tests, Docker Compose, and CI pipelines — one migration path, validated everywhere.

### Why This Matters

The AI would have shipped with SQLite tests that pass but hide real-world failures:
- JSONB columns work differently in SQLite vs Postgres
- UUID types don't exist in SQLite
- Foreign key constraints behave differently
- The migration script itself is never tested until production

By investing in the fixture infrastructure upfront, every subsequent test is a real integration test. The marginal cost of each new test is zero — just write the test, the fixture handles database lifecycle.

## The Compounding Effect

Both corrections compound:

1. **Service layer** means the orchestrator has one dependency instead of two (no cache + repo coordination)
2. **Real Postgres tests** mean the service's write-through actually works against the real schema
3. **Per-test isolation** means tests are independent, parallelizable, and deterministic
4. **Reusable fixture** means every future feature (recovery, MCP server, event replay) gets tested against real Postgres from the start

The AI optimized for "get something working quickly." The engineer optimized for "build something that stays working as the system grows." The difference is about 50 lines of fixture code — a trivial investment that prevents an entire class of integration bugs.

## The Final Architecture

```
SwarmOrchestrator
  └── SwarmService(repo?)
        ├── cache: TaskBoard, InboxSystem, TeamRegistry (hot path)
        ├── repo: SwarmRepository | None (Postgres, optional)
        ├── get_*() → reads cache
        ├── update_*() → cache + repo
        └── load(swarm_id) → hydrates cache from repo

Tests:
  unit/     → SwarmService cache-only (no DB, fast)
  integration/ → Everything against real Postgres (per-test DB, real migrations)
```
