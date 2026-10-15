# Beginner Coding Breakdown for Konek'ta

**Audience:** a Python+backend beginner (and a partner new to Flutter) starting the first production code.  
**Stack:** FastAPI + PostgreSQL (backend) · Flutter (Dart) client (Android first) · Figma for design.  
**Companion:** `docs/03_PROJECT_LIFECYCLE_TIMELINE.md` (weeks) and `MY_PLAN/02_MY_APP_PROJECT_SPEC.md` (API contract once written).

> **The rule for newbies:** do **not** finish reading all the guides before coding. Read the 10 minutes needed for the next thing, build it, commit. Depth comes from a running program that fails, not from 2,000 lines read about a program you haven't started.

---

## 0. Non-negotiables (print these)

1. **Every day ends with a commit.** Even 5 lines. Evidence > effort.
2. **One running thing before the next concept.** Green app → next concept.
3. **Never push directly to `main`.** `feature/x` → PR → review → merge.
4. **Read guide sections only when stuck** on the thing you're doing right now.
5. **Test the logic as pure Python first** (no DB, no framework) — that's how you learn what the code does.
6. **One `no new topics` day per week** to consolidate and refactor.

---

## 1. The mental model before you write anything (1–2 hours, once)

Build this in your head, then start:

```
Flutter client     --HTTP/JSON-->  FastAPI  --SQL-->  PostgreSQL
   Dio/http                      routes.py         repositories.py
   ViewModels                  services.py       models (SQLAlchemy)
                               domain/ (pure)
```

**Four ideas to hold on to:**
- `domain/` imports **nothing** from FastAPI or SQLAlchemy. If you can't test it with plain `python -c`, it's in the wrong place.
- `repositories/` are the only place that runs SQL. Routes never query the DB directly.
- **Authorisation is part of the query** (`WHERE owner_id = :user_id`), not a check you remember to do.
- The database enforces what it can (unique, FK, check). App code checks what the DB can't (thread depth 3).

---

## 2. Milestones (vertical slices)

Each milestone = one shippable thing that works end-to-end. **Do not start milestone N+1 until milestone N's tests pass.**

---

### Milestone 0 — "Hello, service" (Week 5–6, ~1 week)

**Goal:** prove the plumbing runs. You understand a request from phone to DB and back.

**Learn (just-in-time):**
- Python: virtual env with `uv`, type hints, dataclasses/Pydantic.
- HTTP: what a path, body, status code are.
- Docker: what `Flutter` does.

**Build:**
```
backend/
  pyproject.toml          # ruff + mypy + pytest configured (copy TEMPLATE 02)
  app/
    main.py               # FastAPI app factory + /health
    api/routes.py         # GET /health -> {"status":"ok"}
    core/config.py        # settings from env (typed)
docker-Flutter.yml        # fastapi + postgres
.env.example              # never commit real .env
```

**Client (Flutter):**
- New Flutter project; one screen showing "status from backend".
- http/Dio GET `/health`, display it.

**How to know it works:**
- `docker compose up` → `curl http://localhost:8000/health` returns 200.
- Android emulator shows the status string.
- `pytest` runs with 0 tests passing (still green CI).

**Common newbie mistakes:**
- Editing `.env` then committing it. (Set `.gitignore` first, commit second.)
- Putting business logic in `main.py`. Keep it a factory only.
- Skipping types because "it's small". Types are cheapest now.

---

### Milestone 1 — Auth slice (Week 8)

**Goal:** a real user can register, log in, stay logged in, and recover a password.

**Learn (just-in-time):**
- Argon2id hashing (why not SHA-256).
- JWT: access vs refresh, expiry, signing.
- Why passwords must never be returned.
- HTTP 401 vs 403 vs 409 vs 422.

**Build (backend):**
| Step | What | Verify |
|---|---|---|
| 1 | `users` table: id, email (citext unique), username (citext unique), password_hash, birthday (date), phone_e164, is_anonymous_default, created_at, deleted_at | Alembic migration runs + rolls back |
| 2 | `register`: validate body, age gate, breached-password check, Argon2id hash, return no secrets | pytest: happy path + every AC in FR-002 |
| 3 | `login`: generic error for wrong password AND unknown account (no enumeration) | pytest: both return identical 401 body |
| 4 | JWT access (15 min) + refresh token; store **hash** of refresh in `sessions` | Inspect JWT at jwt.io; refresh redeems once |
| 5 | Refresh rotation: reuse → revoke session | pytest reuse test passes |
| 6 | Password recovery: reset code, 5-wrong invalidation, revoke all sessions | pytest |
| 7 | Rate limits: 5/min/IP, 10/hr/account | Integration test |

**The 4 security tests (write these even before impl):**
```
def test_cannot_read_another_users_user(client, other_user): ...
def test_cannot_modify_another_users_profile(client, other_user): ...
def test_mass_assignment_cannot_escalate(client): ...            # send role/admin
def test_responses_never_contain_secrets(client): ...            # no hash, birthday, phone
```

**Android:** login + register screens + token storage (DataStore) + retrofit auth interceptor.

**How to know it works:**
- Register → row in DB → no PII in response.
- Login → access + refresh.
- Refresh with the same token twice → 401 the second time.
- Postman/curl replay of a stolen refresh fails.

---

### Milestone 2 — Onboarding + categories (Week 9)

**Goal:** after login you cannot reach the feed until you pick ≥3 interests.

**Learn:** FK constraints, seed data, citext, transaction wrap of write-then-read.

**Build:**
- `categories` table + seed 60–80 categories (admin-only in v1).
- `user_categories` with PK `(user_id, category_id)`.
- `interest selection` endpoint: write all selected rows before returning 200.
- Custom suggestion endpoint: store as `pending`, admin-only creation.
- Android: multi-select picker with visible selection, "need ≥3" block.

**Tests:** cannot reach feed with 0 interests → 428 `interests_required`.

---

### Milestone 3 — For You feed (Week 10) ⭐ the hardest query

**Goal:** paginated feed filtered by interests, explainable, deduplicated.

**Learn:** SQL `JOIN`, `WHERE ... IN (...)`, keyset pagination (cursor = `(created_at, id)`), N+1 avoidance, `EXPLAIN`.

**Build step-by-step (each step its own commit):**
1. `posts` table with constraints + counters starting 0.
2. Seed 50 posts across categories.
3. Write the SQL from spec §3.7 by hand (in `psql` or DB shell) and make it return 20 rows. **Do this before touching ORM.**
4. Port to SQLAlchemy; wrap in repository method `feed_for(reader_id, cursor)`.
5. Add cursor encode/decode (base64 of `timestamp|id`), validate tampering → 400.
6. Add `$matched_category$` to response.
7. Add block filtering (join `blocks` both directions).
8. Client: `ListView`/`CustomScrollView`, pull-to-refresh, load-more on scroll.

**Tests:**
- For You returns only categories in user's interests (property-ish: try every subset).
- A post appears once even if reader follows two of its categories.
- Reader's own posts never appear.
- Soft-deleted posts never appear.
- Keyset returns 20 then next 20 with no gaps/dups.

**Common mistakes:**
- Using `OFFSET` because it's easier. Don't — change to cursor when you hit ~100 rows.
- Querying hearts/bookmarks per row → N+1. Batch them in a second query for the 20 ids.
- Forgetting to exclude blocked users in *both* directions.

---

### Milestone 4 — Posting + links (Week 11)

**Goal:** create a post, link to a parent, enforce 3-level depth.

**Learn:** self-referential FK, recursive-ish depth check, Pydantic `extra="forbid"`, CHECK constraints.

**Build:**
- `POST /posts` with `body` (1–2000), `category_id`, optional `parent_post_id`, optional `is_anonymous`.
- Depth check in service: walk parent chain counting; reject >3 with 422.
- Self-link check at DB (`CHECK (parent_post_id <> id)`).
- `ON DELETE SET NULL` on parent so deleting a parent doesn't nuke a thread.
- Android: Flutterr with category picker and "replying to" preview.

**Tests:** self-link rejected, depth 4 rejected, deleted parent leaves child as root, category required.

---

### Milestone 5 — Engagement (Week 12)

**Goal:** heart/bookmark/follow, transactional counters.

**Learn:** upsert (toggle), transactions, idempotency.

- `hearts` PK `(user_id, post_id)` → toggle. On insert increment `heart_count`; on delete decrement; wrap in transaction.
- `bookmarks` per user.
- `follows` with `CHECK (follower <> followee)`.
- Android: heart toggles optimistically with rollback on failure.

**Reconciliation test:** recompute `heart_count` from `hearts` and assert equality (the audit that catches drift).

---

### Milestone 6 — Profile & settings (Week 13)

- Own posts feed.
- Change username (409 on collision, generic body).
- Change password (revokes other sessions).
- Change number.
- Anonymity toggle defaulting from user setting at creation, overridable per post.

---

## 3. If you get stuck (professional triage order)

1. **Can I reproduce it in 5 lines without the framework?** → `python -c` or a pytest test. If yes, the bug is in your logic; fix it.
2. **Is it a DB issue?** → check `pgAdmin`/`psql`, run the SQL by hand, read `EXPLAIN`.
3. **Is it an interface issue?** → open the API contract; 90% of "it doesn't work" is shape mismatch.
4. **Is it config/env?** → `.env`, ports, base URL. Look for a typo before a re-architecture.
5. **Read the matching guide section only.** Not the whole file.
6. **Stop after 45 minutes of spinning.** Write down what you tried, ask partner or open an issue. Don't spiral.

---

## 4. Daily routine (newbie-sustainable, ~2–4 hours)

| Block | Time | Do |
|---|---|---|
| 1 | 60–90 min | Code the current slice's next unit (one endpoint, one screen, one test). |
| 2 | 45–90 min | Make it run end-to-end; hit it with curl/Postman/emulator. |
| 3 | 15 min | Write/adjust tests for what you just built. |
| 4 | 15 min | Commit + brief write-up ("what broke, what I learned"). |

If you're tired, do block 4 only. Two days never skip in a row (roadmap rule).

---

## 5. Language cheat sheet for your stack

**Python (90% of what you'll use):** `list`, `dict`, `set`, comprehensions, `dataclass`, `Protocol`/typing, exceptions, `contextmanager`, async basics (`async def`, `await`), `pathlib`, `hashlib`.  
**SQL you must be comfortable typing:** `JOIN ... ON`, `WHERE IN (subquery)`, `ORDER BY ... LIMIT`, `INSERT ... ON CONFLICT`, indexes, migrations.  
**FastAPI:** path/query/body params, dependency injection (`Depends`), lifespan, Pydantic models with `extra="forbid"`, status codes, exception handlers.  
**Flutter:** widgets, `StatelessWidget`/`StatefulWidget`, lists (`ListView`/`CustomScrollView`), state mgmt (Provider/Riverpod/BLoC), `FutureBuilder`, `Navigator`.  
**http/Dio:** typed models via `fromJson`, interceptors for bearer token, mapped error classes.

---

## 6. When are you "done learning to code"? (real answer)

Never. The useful milestone is earlier: **you are a working developer when you can take one FR from the spec, implement it backend-to-client, write its tests, and deploy it — in a week, alone, without tutorials open for the parts you already know.** That's Week 13's definition of done for this project; not mastery, just competence.

---

## 7. Where to find things fast

- Backend code shape → `TEMPLATES/02_PYTHON_CODE_PATTERNS.md` (copy, don't invent).
- Tests → `GUIDES/04_TESTING_AND_QA_GUIDE.md` only when you write your first real test.
- Auth/security details → `GUIDES/03_SECURITY_ENGINEERING_GUIDE.md` while implementing auth.
- DevOps/CI/Docker → `GUIDES/05` only when setting up CI/Docker.
- Architecture/why-a-monolith → `GUIDES/06` §§1–4 only.

If a guide topic doesn't match the problem in front of you today, close the file.
