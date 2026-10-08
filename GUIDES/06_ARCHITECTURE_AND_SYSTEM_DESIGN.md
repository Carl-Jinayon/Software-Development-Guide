# Architecture and System Design — Deep Dive

Companion to Part 4 of `01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md`.

---

## Table of Contents

- [1. Core principles](#1-core-principles)
- [2. Layers, boundaries, and cohesion](#2-layers-boundaries-and-cohesion)
- [3. Clean architecture and dependency direction](#3-clean-architecture-and-dependency-direction)
- [4. Monolith or microservices](#4-monolith-or-microservices)
- [5. Monorepo versus polyrepo](#5-monorepo-versus-polyrepo)
- [6. Event-driven architecture](#6-event-driven-architecture)
- [7. Caching](#7-caching)
- [8. Database patterns](#8-database-patterns)
- [9. API architecture](#9-api-architecture)
- [10. Scalability](#10-scalability)
- [11. Reliability patterns](#11-reliability-patterns)
- [12. Observability architecture](#12-observability-architecture)
- [13. Message queues](#13-message-queues)
- [14. Distributed system problems](#14-distributed-system-problems)
- [15. System design interviews](#15-system-design-interviews)
- [16. Worked designs](#16-worked-designs)
- [17. Anti-patterns](#17-anti-patterns)
- [18. Learning resources](#18-learning-resources)

---

# 1. Core principles

## 1.1 The twelve

| Principle | Meaning | Violation symptom |
|---|---|---|
| **Separation of concerns** | One reason to change per unit | Changing one thing requires editing unrelated files |
| **Single responsibility** | One unit does one thing | A "utils" module with 400 functions |
| **DRY** | One authoritative copy of knowledge | Validation logic duplicated in three places |
| **YAGNI** | Do not build what was not asked for | Configurable plugin system with zero plugins |
| **KISS** | Prefer the obvious solution | A custom message bus when `asyncio.Queue` would do |
| **Principle of least privilege** | Minimum access needed | The app connects as `root` |
| **Fail fast** | Crash on programmer error, degrade on expected failure | A `None` propagating six frames deep |
| **Explicit over implicit** | Make behaviour visible | Magic config values in three places |
| **Composition over inheritance** | Prefer small collaborating parts | A five-level class hierarchy |
| **Don't repeat yourself, but do repeat yourself** | DRY for knowledge, not for text | Two identical functions that encode different rules |
| **Push left** | Fix problems as early as possible | A production incident from a type error |
| **Boring technology** | Proven tools over novel ones | A database nobody has used in production |

## 1.2 Coupling and cohesion

**High cohesion** — things that change together live together.
**Low coupling** — a change in one part does not force changes in another.

```python
# ✗ LOW cohesion, HIGH coupling — one class doing everything
class Application:
    def connect_database(self): ...
    def validate_user(self, user): ...
    def render_html(self, data): ...
    def send_email(self, to, body): ...
    def process_payment(self, amount): ...


# ✓ HIGH cohesion, LOW coupling — each concern is independent
class Database:
    def connect(self): ...

class UserValidator:
    def validate(self, user) -> list[str]: ...

class HtmlRenderer:
    def render(self, data) -> str: ...

class EmailSender:
    def send(self, to: str, body: str) -> None: ...

class PaymentProcessor:
    def charge(self, amount: Decimal) -> Receipt: ...
```

**The test:** if you delete the email feature, how many files change? If the answer is more than "the email sender and its tests," your coupling is too high.

## 1.3 The dependency rule

> **High-level policy must not depend on low-level detail. Both must depend on abstractions.**

The domain layer must not import the database. The database must not import the domain. This is what makes the code testable and portable.

---

# 2. Layers, boundaries, and cohesion

## 2.1 The three layers

```
┌───────────────────────────────────────────────────────────┐
│  PRESENTATION                                              │
│  HTTP routes, request parsing, response formatting         │
│  Depends on: domain, data                                  │
│  Knows about: HTTP                                         │
└──────────────────────────┬────────────────────────────────┘
                           │
┌──────────────────────────▼────────────────────────────────┐
│  DOMAIN                                                    │
│  Business rules. Pure logic. Entities, value objects.      │
│  Depends on: nothing                                      │
│  Knows about: the problem, not the technology              │
└──────────────────────────┬────────────────────────────────┘
                           │
┌──────────────────────────▼────────────────────────────────┐
│  INFRASTRUCTURE                                            │
│  Database, HTTP client, email, filesystem, queues          │
│  Depends on: domain interfaces                             │
│  Contains: all the technology-specific code               │
└───────────────────────────────────────────────────────────┘
```

## 2.2 Why this matters, concretely

```python
# domain/scheduling.py — ZERO technology imports
from dataclasses import dataclass
from datetime import datetime, timedelta

MIN_INTERVAL = timedelta(days=1)
MAX_INTERVAL = timedelta(days=365)


@dataclass(frozen=True)
class ReviewOutcome:
    quality: int          # 0-5, how well the user recalled
    reviewed_at: datetime


@dataclass(frozen=True)
class Schedule:
    next_review: datetime
    interval: timedelta
    ease: float
    repetitions: int


def schedule_next(current: Schedule, outcome: ReviewOutcome) -> Schedule:
    """
    Pure function. No database, no HTTP, no clock.
    This is why it can be tested in microseconds with no fixtures.
    """
    if outcome.quality < 3:
        return Schedule(
            next_review=outcome.reviewed_at + MIN_INTERVAL,
            interval=MIN_INTERVAL,
            ease=max(1.3, current.ease - 0.2),
            repetitions=0,
        )

    repetitions = current.repetitions + 1
    if repetitions == 1:
        interval = timedelta(days=1)
    elif repetitions == 2:
        interval = timedelta(days=6)
    else:
        interval = timedelta(days=min(round(current.ease * interval_days(current.interval)),
                                      MAX_INTERVAL.days))

    return Schedule(
        next_review=outcome.reviewed_at + interval,
        interval=interval,
        ease=current.ease + (0.1 - (5 - outcome.quality) * (0.08 + (5 - outcome.quality) * 0.02)),
        repetitions=repetitions,
    )


def interval_days(interval: timedelta) -> int:
    return max(1, interval.days)
```

```python
# tests/domain/test_scheduling.py — instant, no infrastructure
from datetime import datetime, timedelta
from app.domain.scheduling import ReviewOutcome, Schedule, schedule_next


def test_failed_recall_resets_the_interval():
    current = Schedule(next_review=datetime(2026, 1, 1),
                       interval=timedelta(days=30), ease=2.5, repetitions=4)

    result = schedule_next(current, ReviewOutcome(quality=1, reviewed_at=datetime(2026, 1, 1)))

    assert result.interval == timedelta(days=1)
    assert result.repetitions == 0
    assert result.ease == pytest.approx(2.3)


def test_second_successful_review_is_six_days():
    current = Schedule(datetime(2026, 1, 1), timedelta(days=1), 2.5, 0)

    result = schedule_next(current, ReviewOutcome(quality=5, reviewed_at=datetime(2026, 1, 1)))

    assert result.interval == timedelta(days=6)
```

**This runs in under a millisecond with zero setup.** If this logic lived inside a route handler wired to SQLAlchemy, every test would need a database.

## 2.3 Boundary smells

| Smell | Why it is a problem |
|---|---|
| Domain imports SQLAlchemy | Domain can no longer be tested or reused without a database |
| A route handler contains business rules | Logic cannot be reused by a CLI, a queue worker, or another route |
| Repository returns ORM objects to the domain | The domain is coupled to the schema |
| A "helpers" or "utils" module | No cohesion — a dumping ground |
| Circular imports | Signals the boundaries are wrong |

```python
# ✗ Domain coupled to the database
class Card:
    def save(self):
        db.session.add(CardModel(front=self.front, back=self.back))   # domain knows about ORM


# ✓ Domain is a plain object; the repository knows about persistence
@dataclass(frozen=True)
class Card:
    front: str
    back: str


class CardRepository:
    def add(self, card: Card) -> Card:
        row = CardModel(front=card.front, back=card.back)
        self._session.add(row)
        self._session.flush()
        return Card(front=row.front, back=row.back)
```

---

# 3. Clean architecture and dependency direction

## 3.1 The layers, inward

```
┌───────────────────────────────────────┐
│  Frameworks & Drivers (outermost)     │  FastAPI, SQLAlchemy, Redis
├───────────────────────────────────────┤
│  Interface Adapters                   │  Route handlers, repository impls
├───────────────────────────────────────┤
│  Application Use Cases                │  Orchestration; calls domain
├───────────────────────────────────────┤
│  Entities (domain, innermost)         │  Pure. Depends on nothing.
└───────────────────────────────────────┘

Dependencies point INWARD only.
```

## 3.2 The port and adapter pattern

A **port** is an interface the domain defines. An **adapter** is the technology-specific implementation.

```python
# ── Port (defined by the domain, no technology) ──────────────
class CardRepository(Protocol):
    def add(self, card: Card) -> Card: ...
    def get(self, card_id: int) -> Card | None: ...
    def get_for_user(self, card_id: int, user_id: int) -> Card | None: ...
    def list_due(self, user_id: int, now: datetime) -> list[Card]: ...


class Clock(Protocol):
    def now(self) -> datetime: ...


class Notifier(Protocol):
    def notify_review_due(self, user_id: int, count: int) -> None: ...


# ── Use case (depends only on ports) ─────────────────────────
class ReviewCard:
    def __init__(self, cards: CardRepository, clock: Clock):
        self._cards = cards
        self._clock = clock

    def execute(self, card_id: int, user_id: int, quality: int) -> Schedule:
        card = self._cards.get_for_user(card_id, user_id)   # ownership baked in
        if card is None:
            raise CardNotFound(card_id)

        outcome = ReviewOutcome(quality=quality, reviewed_at=self._clock.now())
        schedule = schedule_next(card.schedule, outcome)
        self._cards.update_schedule(card_id, schedule)
        return schedule


# ── Adapters (implementations, depend on domain) ─────────────
class SystemClock:
    def now(self) -> datetime:
        return datetime.now(timezone.utc)


class SqlAlchemyCardRepository:
    def __init__(self, session: Session):
        self._session = session

    def get_for_user(self, card_id: int, user_id: int) -> Card | None:
        row = self._session.execute(
            select(CardModel).where(CardModel.id == card_id,
                                    CardModel.user_id == user_id)
        ).scalar_one_or_none()
        return to_domain(row) if row else None


# ── Wiring (the composition root — the only place both meet) ─
def build_review_card(session: Session) -> ReviewCard:
    return ReviewCard(cards=SqlAlchemyCardRepository(session), clock=SystemClock())
```

**The payoff:** the use case can be tested with fakes in microseconds, and the database can be swapped for an API without touching business logic.

```python
class InMemoryCardRepository:
    """Test double implementing the port."""
    def __init__(self, cards: list[Card] | None = None):
        self._cards = {c.id: c for c in cards or []}

    def get_for_user(self, card_id: int, user_id: int) -> Card | None:
        card = self._cards.get(card_id)
        return card if card and card.user_id == user_id else None

    def update_schedule(self, card_id: int, schedule: Schedule) -> None:
        card = self._cards[card_id]
        self._cards[card_id] = replace(card, schedule=schedule)


class FixedClock:
    def __init__(self, moment: datetime):
        self._moment = moment

    def now(self) -> datetime:
        return self._moment


def test_review_schedules_next_review():
    now = datetime(2026, 1, 1, 12, 0, tzinfo=timezone.utc)
    card = Card(id=1, user_id=42, front="Q", back="A",
                schedule=Schedule(now, timedelta(days=1), 2.5, 0))
    repo = InMemoryCardRepository([card])
    use_case = ReviewCard(repo, FixedClock(now))

    result = use_case.execute(card_id=1, user_id=42, quality=5)

    assert result.interval == timedelta(days=1)
    assert result.repetitions == 1


def test_cannot_review_another_users_card():
    use_case = ReviewCard(InMemoryCardRepository([card]), FixedClock(now))

    with pytest.raises(CardNotFound):
        use_case.execute(card_id=1, user_id=999, quality=5)     # ← authorisation in the port
```

## 3.3 The composition root

One place where all implementations are wired together. Kept at the edge, never imported by domain code.

```python
# app/container.py
class Container:
    """The composition root. The only module that knows every concrete class."""
    def __init__(self, session: Session):
        self.session = session
        self.clock: Clock = SystemClock()

    def review_card(self) -> ReviewCard:
        return ReviewCard(SqlAlchemyCardRepository(self.session), self.clock)

    def create_deck(self) -> CreateDeck:
        return CreateDeck(SqlAlchemyDeckRepository(self.session),
                          SqlAlchemyCardRepository(self.session))


def get_container(session: Session = Depends(get_db)) -> Container:
    return Container(session)
```

---

# 4. Monolith or microservices

## 4.1 The honest comparison

| | Monolith | Microservices |
|---|---|---|
| **Deploy** | One unit | Many independent units |
| **Debug** | One stack trace | Distributed tracing across services |
| **Local dev** | `docker compose up` | Run everything, or stub services |
| **Cross-cutting change** | One commit | Coordinate N services, N releases, N contracts |
| **Network** | None | Failure mode #1 |
| **Data consistency** | ACID transaction | Eventual consistency, compensation |
| **Testing** | Straightforward | Testcontainers, contract tests, service mocks |
| **Cost** | Low | Enormous operational overhead |
| **Team size needed** | 1+ | Realistically 10+ |

## 4.2 When microservices are actually justified

Not "when the team is large." Specifically:

1. **A component needs independent scaling** and you have measured that it dominates cost
2. **Different release cadences** — the ML pipeline needs daily releases, the API needs monthly
3. **Hard team boundaries** with genuine ownership
4. **Hard technology boundaries** — a GPU service, a streaming service
5. **You have the observability to debug across service boundaries**

## 4.3 What to build instead

**A modular monolith.** Clear module boundaries, one deployable, internal interfaces:

```
app/
├── modules/
│   ├── users/
│   │   ├── domain.py          # pure entities and rules
│   │   ├── service.py         # use cases
│   │   ├── repository.py      # port + adapter
│   │   └── routes.py          # HTTP
│   ├── decks/
│   │   ├── domain.py
│   │   ├── service.py
│   │   ├── repository.py
│   │   └── routes.py
│   └── reviews/
└── shared/                     # cross-cutting only: config, errors, auth deps
```

Modules communicate through explicit service interfaces, not by reaching into each other's tables. **This gives you most of microservices' modularity with none of the distributed-systems cost** — and when you later need to split a module out, the seam already exists.

## 4.4 The seam test

> Can you describe, for a candidate module, exactly which files would move and which interfaces would need a network adapter? If yes, the boundary is real. If not, you have not found a module.

---

# 5. Monorepo versus polyrepo

| | Monorepo | Polyrepo |
|---|---|---|
| **Atomic changes** | Yes — one commit across services | No — N PRs, N releases, order matters |
| **Refactoring across modules** | One change | Impossible without coordination |
| **CI cost** | One pipeline, can cache | N pipelines |
| **Local setup** | Clone once | Clone N repositories |
| **Permission isolation** | Everyone sees everything | True isolation |
| **Release independence** | Possible but harder | Native |

## 5.1 Tooling for a monorepo

| Tool | Purpose |
|---|---|
| **Nx / Turborepo** | Task orchestration, affected-only builds |
| **Bazel** | Correct incremental builds at scale |
| **Changesets** | Coordinated versioning and release notes |
| **pnpm workspaces / uv** | Dependency management |

**Affected-only builds matter enormously:** running the full test suite on every commit is the main reason monorepos become slow.

---

# 6. Event-driven architecture

## 6.1 The pattern

```
        ┌──────────┐
        │  User    │──register──▶ ┌──────────┐
        └──────────┘              │  API     │
                                  └────┬─────┘
                                       │ emit UserRegistered
                                       ▼
                        ┌──────────────┬──────────────┐
                        ▼              ▼              ▼
                  ┌──────────┐  ┌──────────┐  ┌──────────────┐
                  │  Email   │  │ Analytics│  │ Initial deck │
                  │ service  │  │ service  │  │   creation   │
                  └──────────┘  └──────────┘  └──────────────┘

The API does not wait for any of them. It returns immediately.
```

## 6.2 When it helps

| Benefit | Explanation |
|---|---|
| **Decoupling** | Producers do not know consumers |
| **Independent scaling** | Scale the slow consumer, not everything |
| **Failure isolation** | A slow email service does not block registration |
| **Extensibility** | New consumers without changing the producer |
| **Auditability** | An event log is a durable history |

## 6.3 When it hurts

| Problem | Consequence |
|---|---|
| **No transaction** | A failure between emit and consume leaves inconsistent state |
| **Debugging** | Control flow is invisible across services |
| **Ordering** | Events are usually not ordered |
| **Duplicates** | At-least-once delivery means consumers must be idempotent |
| **Testing** | Harder; you need contract tests |
| **Operational cost** | Queues, consumers, dead-letter queues |

## 6.4 Designing events

```python
from pydantic import BaseModel
from datetime import datetime


class DomainEvent(BaseModel):
    """Immutable, versioned, minimal. Contains facts, not commands."""
    event_id: UUID
    occurred_at: datetime
    version: int = 1


class UserRegistered(DomainEvent):
    user_id: int
    email: str


class DeckShared(DomainEvent):
    deck_id: int
    owner_id: int
    shared_with_user_id: int
    permission: Literal["read", "write"]
```

**Event naming:** past tense facts (`UserRegistered`), not commands (`RegisterUser`). The distinction matters: a command has exactly one handler and may be rejected; an event has zero or more handlers and is a statement of something that already happened.

## 6.5 Idempotent consumers

```python
class DeckShareHandler:
    """
    At-least-once delivery means this will run more than once.
    Deduplicate on event_id.
    """
    def __init__(self, repo: DeckRepository, processed: ProcessedEventStore):
        self._repo = repo
        self._processed = processed

    def handle(self, event: DeckShared) -> None:
        if self._processed.already_handled(event.event_id):
            return                       # already done — no-op

        deck = self._repo.get(event.deck_id)
        deck.grant_access(event.shared_with_user_id, event.permission)

        self._processed.mark_handled(event.event_id)     # same transaction
        self._repo.commit()
```

**Mark-as-processed and the state change must be in the same transaction.** Otherwise a crash between them either loses the change or reprocesses it.

---

# 7. Caching

## 7.1 The levels

| Level | Latency | Scope | Invalidated by |
|---|---|---|---|
| **In-process (L1)** | ns | One process | TTL or manual |
| **Redis (L2)** | ~1ms | All instances | TTL or manual |
| **CDN** | ~50ms | Global | TTL or purge |
| **Database buffer** | varies | Database | Internal |
| **Browser** | 0 | One user | Cache-Control headers |

## 7.2 The three hard problems

```
1. What invalidates this?
2. What happens on a cache miss?
3. What happens when the cache itself is down?
```

## 7.3 Cache-aside

```python
def get_deck(deck_id: int, user_id: int) -> Deck:
    key = f"deck:{deck_id}"

    try:
        if (cached := redis.get(key)) is not None:
            return Deck.model_validate_json(cached)
    except redis.RedisError:
        logger.warning("cache_read_failed", key=key)
        # Fall through to the database. A cache outage must not be an outage.

    deck = repo.get_for_user(deck_id, user_id)       # miss → source of truth
    if deck is not None:
        try:
            redis.setex(key, 300, deck.model_dump_json())
        except redis.RedisError:
            pass                                      # never fail on cache write

    return deck
```

## 7.4 Invalidation strategies

| Strategy | How | When |
|---|---|---|
| **TTL** | Expire after N seconds | Simple; eventual consistency is acceptable |
| **Write-through** | Update the cache on every write | Strong consistency needed |
| **Cache invalidation on write** | Delete the key when the source changes | Most common correct choice |
| **Event-driven** | Publish a change event; consumers invalidate | Many caches, decoupled |

```python
def update_deck(deck_id: int, data: UpdateDeck, session: Session) -> Deck:
    deck = repo.update(deck_id, data)
    session.commit()

    try:
        redis.delete(f"deck:{deck_id}")            # same transaction boundary is
    except redis.RedisError:                        # NOT possible — accept the window
        logger.warning("cache_invalidation_failed", deck_id=deck_id)

    return deck
```

## 7.5 Cache stampede

When a hot key expires, every concurrent request hits the database at once.

```python
import random

def get_deck_with_stampede_protection(deck_id: int, user_id: int) -> Deck:
    key = f"deck:{deck_id}"

    if cached := _try_cache(key):
        return cached

    # Randomised TTL extension: only one request wins the race and does the work
    lock_key = f"lock:{key}"
    if redis.set(lock_key, "1", nx=True, ex=10):
        try:
            deck = repo.get_for_user(deck_id, user_id)
            if deck:
                _set_cache(key, deck, ttl=300 + random.randint(0, 30))   # jitter
            return deck
        finally:
            redis.delete(lock_key)
    else:
        time.sleep(0.05)      # another request is rebuilding; brief wait
        return _try_cache(key) or repo.get_for_user(deck_id, user_id)
```

**Jittered TTLs** are simpler and nearly as effective: if a thousand keys all expire at once, they will not all expire at once.

---

# 8. Database patterns

## 8.1 The N+1 problem

The most common performance bug in application code.

```python
# ✗ One query for the list, then one per row
decks = session.query(Deck).filter_by(owner_id=user_id).all()
return [{"title": d.title, "cards": len(session.query(Card).filter_by(deck_id=d.id).all())}
        for d in decks]

# 1 + N queries. At 100 decks that is 101 round trips.


# ✓ Eager loading — 2 queries regardless of N
from sqlalchemy.orm import selectinload

decks = session.execute(
    select(Deck).options(selectinload(Deck.cards)).filter(Deck.owner_id == user_id)
).scalars().all()
```

**Detect it:** SQLAlchemy logs `N+1 selects were detected` when `echo=True`. Always check the query count in development.

## 8.2 Pagination

| Method | Good for | Problem |
|---|---|---|
| **Offset** (`LIMIT 50 OFFSET 200`) | Pages you can jump to | Slow at deep offsets; rows can shift under you |
| **Keyset / cursor** (`WHERE id > 100 ORDER BY id LIMIT 50`) | Infinite scroll, feeds | No random page jumps |
| **Seek** (`WHERE (created_at, id) > ($1, $2)`) | Sorted feeds | Requires a stable sort |

```python
# Offset degrades: OFFSET 1,000,000 makes the database scan a million rows
def list_cards_offset(deck_id: int, page: int = 0, limit: int = 50):
    return session.query(Card).filter_by(deck_id=deck_id) \
                  .order_by(Card.id).limit(limit).offset(page * limit)


# Keyset is O(limit) regardless of depth
def list_cards_keyset(deck_id: int, after_id: int | None = None, limit: int = 50):
    query = session.query(Card).filter_by(deck_id=deck_id).order_by(Card.id).limit(limit)
    if after_id is not None:
        query = query.filter(Card.id > after_id)
    return query.all()
```

## 8.3 Indexing

```sql
-- B-tree: equality and range lookups, sorting
CREATE INDEX idx_cards_deck_due ON cards (deck_id, due_at);

-- Composite index column order
--   equality columns first, range column last
--   WHERE deck_id = ? AND due_at <= ?    →  (deck_id, due_at)  ✓
--   WHERE due_at <= ? AND deck_id = ?    →  (deck_id, due_at)  ✓ (query planner reorders)

-- Partial index: only index the rows you query
CREATE INDEX idx_due_cards ON cards (due_at) WHERE deleted_at IS NULL;

-- Covering index: include columns to avoid table lookups
CREATE INDEX idx_cards_covering ON cards (deck_id, due_at) INCLUDE (front, back);

-- Full-text
CREATE INDEX idx_cards_search ON cards USING gin (to_tsvector('english', front));
```

```sql
EXPLAIN ANALYZE SELECT * FROM cards WHERE deck_id = 42 AND due_at <= NOW();
-- Seq Scan on cards  (cost=0.00..45231.00)     ← needs an index
-- Index Scan using idx_cards_deck_due  (cost=0.42..8.1)   ← good
```

## 8.4 Soft deletes

```python
class Card(Base):
    __tablename__ = "cards"

    id: Mapped[int] = mapped_column(primary_key=True)
    deleted_at: Mapped[datetime | None]

    @property
    def is_deleted(self) -> bool:
        return self.deleted_at is not None
```

**Every query must filter.** A forgotten filter leaks deleted data. Consider a global query filter, or hard deletes with an archive table.

## 8.5 Read replicas and connection pooling

```python
def get_read_session() -> Session:
    """Read traffic can go to a replica. Writes cannot."""
    return Session(bind=read_replica_engine)


def get_write_session() -> Session:
    return Session(bind=primary_engine)
```

**The hazard: replication lag.** A user creates a deck, is redirected, and the replica has not caught up — so the deck appears to be missing. Always read-after-write from the primary, or accept eventual consistency deliberately.

---

# 9. API architecture

## 9.1 REST

| Method | Meaning | Idempotent | Safe |
|---|---|---|---|
| `GET` | Read | Yes | Yes |
| `POST` | Create / non-idempotent action | No | No |
| `PUT` | Full replace | Yes | Yes |
| `PATCH` | Partial update | **No** (usually) | Yes |
| `DELETE` | Remove | Yes | Yes |

**Never use GET for state changes.** It is prefetched by browsers, cached by proxies, and logged with URLs.

## 9.2 Versioning

```
/api/v1/cards     ← stable
/api/v2/cards     ← breaking changes only
```

**Rules:** never repurpose a field. Never remove a field. Never change a type. Add new fields, mark old ones deprecated with a sunset date.

## 9.3 Error format

```python
class ProblemDetail(BaseModel):
    """RFC 9457 problem details — a standard error format clients can parse."""
    type: str = Field(validation_alias="$type")
    title: str
    status: int
    detail: str
    instance: str | None = None
    errors: list[dict] | None = None
    trace_id: str | None = None


@app.exception_handler(RequestValidationError)
async def validation_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content=ProblemDetail(
            type="https://myapp.com/errors/validation",
            title="Validation failed",
            status=422,
            detail="The request body failed validation",
            errors=[
                {"field": ".".join(str(p) for p in e["loc"]), "message": e["msg"]}
                for e in exc.errors()
            ],
            trace_id=request.state.trace_id,
        ).model_dump(by_alias=True),
    )
```

## 9.4 Rate limiting headers

```python
@app.middleware("http")
async def rate_limit_headers(request: Request, call_next):
    response = await call_next(request)
    limit = 100
    remaining = rate_limiter.remaining(request.client.host)
    response.headers["X-RateLimit-Limit"] = str(limit)
    response.headers["X-RateLimit-Remaining"] = str(remaining)
    response.headers["X-RateLimit-Reset"] = str(int(time.time()) + 3600)
    if remaining == 0:
        response.headers["Retry-After"] = "3600"
    return response
```

## 9.5 GraphQL when it earns its place

GraphQL lets clients request exactly the fields they need. It is worth it when clients genuinely differ and over-fetching is measurable. It is not worth it as a default — it adds schema design, N+1 risk at the resolver level, query-depth attacks, and a harder caching story.

---

# 10. Scalability

## 10.1 The ladder

Apply in order. **Each rung is much cheaper than the next.**

```
1. Add an index                        ← minutes, often 100×
2. Fix the N+1 queries                 ← hours, often 10×
3. Cache the expensive reads           ← hours
4. Add pagination and limits           ← hours
5. Optimise the slow queries           ← days
6. Increase the database instance      ← minutes, more money
7. Add read replicas                   ← days
8. Vertical split of hot data          ← weeks
9. Horizontal sharding                 ← months
10. Extract a service                  ← months
```

Most teams that jump to step 9 should have been on step 3.

## 10.2 Stateless services

```python
# Stateful: sessions in process memory — breaks with more than one instance
SESSIONS: dict[str, dict] = {}

# Stateless: everything in the token or in shared storage
def create_session(user_id: int) -> str:
    session_id = secrets.token_urlsafe(32)
    redis.setex(f"session:{session_id}", 86400, json.dumps({"user_id": user_id}))
    return session_id
```

**Stateless means any instance can serve any request**, which is what makes horizontal scaling and zero-downtime deployment possible.

## 10.3 Load balancing and health

```python
# Readiness: can it serve traffic? Remove from the pool if not.
# Liveness: is the process alive? Restart if not.

# Distinguish them, or a database blip becomes a crash loop:
#   readiness fails → removed from load balancer, still running, recovers
#   liveness fails   → killed and restarted, loses in-flight work
```

## 10.4 The CAP theorem, practically

When a network partition occurs, you must choose:

| Choice | You get | You lose |
|---|---|---|
| **CP** — refuse to respond rather than risk stale reads | Consistency | Availability |
| **AP** — serve possibly-stale data | Availability | Immediate consistency |
| **PA** — prefer consistency, accept partition failure | Most systems' real behaviour | — |

**Most applications want CP for writes and AP for reads.** Serving a stale feed is fine; double-charging a card is not.

---

# 11. Reliability patterns

## 11.1 Timeouts

```python
import httpx

DEFAULT_TIMEOUT = httpx.Timeout(5.0, connect=2.0)

async with httpx.AsyncClient(timeout=DEFAULT_TIMEOUT) as client:
    response = await client.get(url)
```

**No timeout means an indefinite hang.** Every network call needs one — connections, reads, writes, and the total.

## 11.2 Retries with exponential backoff

```python
import random

async def fetch_with_retry(url: str, attempts: int = 3, base_delay: float = 0.5):
    async with httpx.AsyncClient(timeout=DEFAULT_TIMEOUT) as client:
        for attempt in range(attempts):
            try:
                response = await client.get(url)
                if response.status_code < 500:      # 4xx will not improve on retry
                    return response
            except httpx.TimeoutException:
                pass

            if attempt < attempts - 1:
                delay = base_delay * (2 ** attempt)
                delay *= 0.5 + random.random()      # jitter — prevents thundering herd
                await asyncio.sleep(delay)

    raise UpstreamUnavailable(url)
```

**Jitter is not optional.** Without it, every client retries in lockstep and the recovering service is immediately overwhelmed again.

## 11.3 Circuit breaker

```python
class CircuitBreaker:
    """Stop calling a failing dependency so it can recover."""

    CLOSED, OPEN, HALF_OPEN = "closed", "open", "half_open"

    def __init__(self, failure_threshold: int = 5, timeout: float = 30.0):
        self.state = self.CLOSED
        self.failures = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.opened_at: float = 0

    def record_success(self) -> None:
        self.failures = 0
        self.state = self.CLOSED

    def record_failure(self) -> None:
        self.failures += 1
        if self.failures >= self.failure_threshold:
            self.state = self.OPEN
            self.opened_at = time.monotonic()

    def allow_request(self) -> bool:
        if self.state == self.CLOSED:
            return True
        if self.state == self.OPEN:
            if time.monotonic() - self.opened_at > self.timeout:
                self.state = self.HALF_OPEN     # probe with one request
                return True
            return False                        # fail fast — do not hammer a dead service
        return True
```

## 11.4 Idempotency

```python
def process_payment(user_id: int, amount: Decimal,
                    idempotency_key: str, session: Session) -> Payment:
    """
    Any operation with a side effect must be safe to repeat.
    Network failures make the client retry; without this, they are charged twice.
    """
    existing = session.execute(
        select(Payment).where(Payment.idempotency_key == idempotency_key)
    ).scalar_one_or_none()

    if existing:
        return existing                          # return the ORIGINAL result

    payment = Payment(user_id=user_id, amount=amount, idempotency_key=idempotency_key)
    session.add(payment)
    session.commit()
    return payment
```

## 11.5 Graceful degradation

```python
def get_recommendations(user_id: int) -> list[Card]:
    """If the recommendation service is down, fall back to something useful."""
    try:
        return recommendation_service.fetch(user_id)
    except (ServiceUnavailable, TimeoutError):
        logger.warning("recommendations_unavailable", user_id=user_id)
        return due_cards(user_id)[:20]          # a degraded but functional answer
```

---

# 12. Observability architecture

| Signal | Question | Retention | Cost |
|---|---|---|---|
| **Logs** | What happened in detail? | 7–30 days | Low at low volume, explodes at high volume |
| **Metrics** | How is it trending? | 1–2 years | Cheap (time series) |
| **Traces** | Where did the latency go? | 7–14 days | Expensive — sample |
| **Profiles** | Where is the CPU going? | Short | Expensive |
| **Errors** | What is broken? | Long | Cheap, essential |

**Distributed tracing** requires propagating a trace ID through every service:

```python
@app.middleware("http")
async def tracing(request: Request, call_next):
    trace_id = request.headers.get("x-trace-id") or secrets.token_hex(16)
    request.state.trace_id = trace_id
    structlog.contextvars.bind_contextvars(trace_id=trace_id)

    response = await call_next(request)
    response.headers["x-trace-id"] = trace_id      # so the client can quote it
    return response
```

---

# 13. Message queues

## 13.1 When

| Use a queue when | Not when |
|---|---|
| The work takes longer than the user should wait | The user needs the result immediately |
| The work can be retried independently | The operation must be atomic with the request |
| Load is bursty | Throughput is steady |
| You need fan-out to several consumers | There is only one consumer |

## 13.2 Patterns

| Pattern | How | Use |
|---|---|---|
| **Work queue** | One message, one consumer | Background jobs |
| **Pub/sub** | One message, many consumers | Notifications, cache invalidation |
| **Request-reply** | Caller waits for a response | Async RPC |
| **Saga** | Distributed transaction via compensating actions | Multi-service workflows |

## 13.3 Exactly-once is not achievable

Delivers are **at-least-once**. Build consumers that are idempotent and assume duplicates.

```python
class ReviewProcessor:
    """
    Idempotent consumer: deduplicates on message id, and its state change
    commits in the same transaction as the dedup record.
    """
    def process(self, message: dict) -> None:
        if self._processed.already_handled(message["id"]):
            return

        card = self._cards.get(message["card_id"])
        if card:
            self._cards.update_schedule(card.id, schedule_next(card.schedule,
                                                                ReviewOutcome(**message)))

        self._processed.mark_handled(message["id"])
        self._session.commit()        # atomic with the change above
```

## 13.4 Poison messages

```python
def process_with_dead_letter_queue(message: dict, retries: int = 3) -> None:
    try:
        handle(message)
    except (ValidationError, ValueError) as exc:
        # Malformed and will never succeed — do not retry
        dead_letter.publish(message, reason=str(exc))
    except Exception as exc:
        if retries >= MAX_RETRIES:
            dead_letter.publish(message, reason=f"{type(exc).__name__}: {exc}",
                                original_retries=retries)
        else:
            queue.republish(message, delay=retry_delay(retries))
```

**A dead-letter queue is essential.** Without one, one unparseable message blocks the queue forever.

---

# 14. Distributed system problems

These are the things that are genuinely hard, and no amount of care eliminates them.

| Problem | What happens | Mitigation |
|---|---|---|
| **Partial failure** | Some parts succeed, some fail | Compensating actions (saga), idempotency |
| **No global clock** | "Which happened first?" is unanswerable | Lamport clocks, event ordering by ID |
| **Eventual consistency** | Reads may miss recent writes | Accept it, or read from primary |
| **Network partitions** | Nodes cannot communicate | CAP choices; circuit breakers |
| **Split brain** | Two nodes both believe they lead | Consensus (etcd, ZooKeeper), leader election |
| **Unreliable networks** | Messages lost, duplicated, reordered | At-least-once + idempotent consumers |
| **No global transaction** | ACID does not span systems | Saga with compensating actions |

## 14.1 Saga pattern

```python
class CheckoutSaga:
    """
    Distributed transaction as a sequence of local transactions,
    each with a compensating action to undo it if a later step fails.
    """
    def execute(self, order: Order) -> None:
        try:
            self.inventory.reserve(order)              # T1
            try:
                self.payment.charge(order)            # T2
                try:
                    self.shipping.create(order)       # T3
                except Exception:
                    self.payment.refund(order)        # C3
                    raise
            except Exception:
                self.inventory.release(order)          # C2
                raise
        except Exception:
            self.payment.cancel_if_authorized(order)  # C1
            raise
```

**Compensation is not rollback.** You cannot un-charge a card; you issue a refund, which may itself fail. Every step needs its own retry and reconciliation strategy.

---

# 15. System design interviews

## 15.1 The framework

```
1. CLARIFY REQUIREMENTS        5 min
   - Users? Scale? Latency target? Read/write ratio?
   - What must it do vs. what is nice to have?

2. ESTIMATE                    2 min
   - 100k DAU × 20 requests = 2M requests/day ≈ 25 rps
   - 500 bytes/request = 15 GB/day ≈ 10 TB/year
   - Do not over-engineer; show the arithmetic

3. HIGH-LEVEL DESIGN          10 min
   - Draw the boxes: client, API, services, data stores, external
   - Explain the data flow for ONE request

4. DEEP DIVE                  10 min
   - Pick the hardest component and drill in
   - Ask what interviewer cares about

5. OPERATIONS                 5 min
   - Monitoring, scaling, failure modes, security
```

## 15.2 Clarifying questions worth asking

- How many users? How fast growing?
- Read-heavy or write-heavy?
- What is the latency requirement? (Users cannot tell 100ms from 300ms.)
- Strong or eventual consistency acceptable?
- Multi-region, or single region?
- What is the budget?
- Compliance constraints?

## 15.3 What interviewers score

| Signal | Good | Bad |
|---|---|---|
| **Clarification** | Asks before answering | Assumes and starts |
| **Trade-offs** | "Postgres because the data is relational; I would shard at ~10M rows" | "NoSQL for scalability" |
| **Reasoning aloud** | Explains the decision as it is made | Silent then abrupt |
| **Specificity** | Cites real numbers | "It would be fast" |
| **Failure** | "If the cache is down, we fall back to the database" | Never mentions failure |
| **Simplicity** | "A monolith, because there is one team" | Microservices unprompted |

**The most common failure is over-engineering.** Interviewers are listening for whether you add complexity only when a stated requirement demands it.

---

# 16. Worked designs

## 16.1 URL shortener

**Requirements:** 100M URLs created/month, 10:1 read ratio, sub-50ms reads, custom aliases optional.

```
CLIENT
  │
  ▼
API ─────────────────────────► Redis (hot URLs, 24h TTL)
  │
  ├──► PostgreSQL (source of truth, partitioned by hash(url_code))
  └──► Kafka (analytics events)
```

| Concern | Decision |
|---|---|
| Code generation | Base62 of 7 random chars; retry on collision |
| Read path | Redis first; fall back to Postgres; negative-cache misses |
| Write path | Postgres, then populate the cache |
| Analytics | Async via queue — never in the request path |
| Custom aliases | Store, but count and rate-limit them |
| Abuse | Rate limit, block known malicious destinations |

## 16.2 Real-time chat

```
CLIENT (WebSocket)
   │
   ▼
WebSocket gateway ───► Redis Pub/Sub (fan-out to connected nodes)
   │
   └──► Message store (Postgres) + Queue (offline delivery)
```

| Concern | Decision |
|---|---|
| Connection state | Redis; any node can route to any user's node |
| Ordering | Per-conversation sequence numbers |
| Offline users | Persist, deliver on reconnect |
| Scaling | Stateless gateways behind a load balancer with sticky sessions on user ID |
| Presence | Redis with TTL expiry — heartbeat keys that expire |

## 16.3 File upload service

```
CLIENT ── presigned URL ──▶ API (authorises, signs)
   │                              │
   │ direct PUT                   └──▶ Object storage (S3)
   │                              └──▶ Queue (virus scan, thumbnail)
   ▼                                        │
Object storage ◀────────────────────────────┘
```

| Concern | Decision |
|---|---|
| Large files | Client uploads directly to storage; the API never proxies bytes |
| Validation | Authorise before signing; scan after upload |
| Progress | Multipart upload with presigned part URLs |
| Availability | Storage is the source of truth; metadata in Postgres |

---

# 17. Anti-patterns

| Anti-pattern | Why it is bad | Instead |
|---|---|---|
| **Distributed monolith** | Microservices that must deploy together — all cost, none of the benefit | Modular monolith |
| **Shared database across services** | Any schema change breaks every consumer | Own your data |
| **Premature microservices** | Distributed systems problems with none of the benefits | One deployable until proven otherwise |
| **God object / God service** | Every change touches it; nothing else can change in parallel | Split by responsibility |
| **Chatty interfaces** | 50 network calls per user action | Coarse-grained APIs |
| **Missing timeouts** | One slow dependency stalls everything | Timeouts everywhere |
| **Retries without backoff** | Thundering herd | Exponential backoff with jitter |
| **Ignoring eventual consistency** | Surprising bugs under load | Design for it explicitly |
| **Optimising before measuring** | Wasted effort on the wrong thing | Measure first |
| **Horizontal scaling a stateful app** | Sessions and uploads break | Make it stateless first |
| **Secrets in source** | Breach | Environment variables, secret manager |
| **No migration plan** | Every release is risky | Expand and contract |

---

# 18. Learning resources

| Topic | Resource |
|---|---|
| **Designing Data-Intensive Applications** | Kleppmann — the single best book on this topic |
| **Microservices patterns** | Richardson — the catalogue of patterns and their trade-offs |
| **System design** | *System Design Interview* (Alex Xu); the "Gaurav Sen" YouTube series |
| **Real-world architecture** | Real production engineering blogs: Netflix, Uber, Discord, Cloudflare, Shopify |
| **Domain-driven design** | Evans — where the "domain layer" language comes from |
| **Clean architecture** | *Clean Architecture* (Martin) |
| **Databases** | *Use the Index, Luke* (free); *Database Internals* (Kleppmann, free) |
| **Reliability** | *Site Reliability Engineering* (Google, free) |

---

## Closing

Architecture is not a diagram you draw at the start. It is a series of decisions about boundaries and trade-offs, made continuously, and revised when reality disagrees.

The strongest architectural instinct is knowing what **not** to build. Every boundary you draw is a cost. Draw them where the coupling actually is, keep the logic testable, and let the measurement tell you what to change next.
