# Granular Day-by-Day Breakdown — Konek'ta

**Pair with:** `docs/07_BEGINNER_TO_PRODUCTION_COMPLETE.md` (the technical reference) and `docs/03_PROJECT_LIFECYCLE_TIMELINE.md` (weeks/gates).  
**Format:** each block = ~1 focused session (60–120 min). Stop when the "Done when" checkbox is true; do not move on.

> Professional rule: a session that ships one verified unit beats a session that "worked on the backend." If you didn't run it, it isn't progress.

---

## Week 0 — Tooling and plumbing (no product code)

### W0.1 — Check your toolchain
- Install Python 3.12, `uv`, Docker Desktop (or engine), Flutter SDK, VS Code + extensions (Python, Ruff, Flutter, Docker).
- **Learn:** what an interpreter, a package manager, and a container are (15 min max, YouTube, then close).
- **Done when:** `python --version`, `uv --version`, `docker --version`, `flutter doctor` all green.

### W0.2 — Create repos + hygiene
- Create `backend/`, `mobile/`, `docs/`. Init git on `main` with branch protection.
- Add `.gitignore` **first** (`.env`, `__pycache__`, `build/`, `.dart_tool/`).
- `git status` must show no `.env` ever.
- **Done when:** `git log` shows one initial commit; `.env` is ignored.

### W0.3 — Compose runs
- Write `compose.yaml` (api + db). Write a `Dockerfile` for FastAPI.
- `docker compose up` → both healthy. `curl http://localhost:8000/health` → `{"status":"ok"}`.
- **Learn:** what a service and a volume are.
- **Done when:** you can (and did) destroy the volume and rebuild from scratch.

### W0.4 — Flutter scaffold
- `flutter create mobile --platforms=android`; `flutter run` on emulator.
- One screen calling `/health` with `http`, showing status or error text.
- **Done when:** emulator shows the status string from the backend.

### W0.5 — CI skeleton
- GitHub Actions: `lint → type → test → security scan`. It will be green with 0 tests — that's the point.
- Flutter job: `flutter analyze` + `flutter test`.
- **Learn:** CI is a gate, not a report.
- **Done when:** a PR runs green automatically.

---

## Milestone 0 — Hello service (sessions)

### M0.1 — Config from environment
- `app/core/config.py` with Pydantic `BaseSettings` (database_url, jwt_secret).
- **Learn:** 12-factor config; why secrets never live in code.
- **Done when:** changing `.env` changes behaviour without code edits.

### M0.2 — Health endpoint (no logic in main)
- `app/main.py` is an app factory only. Route lives in `api/routes.py`.
- **Learn:** small modules, one responsibility.
- **Done when:** `curl /health` returns 200 and main.py < 20 lines.

### M0.3 — Alembic baseline
- `alembic init`, point at `database_url`, create empty first migration, `upgrade` and `downgrade`.
- **Learn:** migrations are code; always test rollback.
- **Done when:** you've migrated up and down on a fresh DB successfully.

### M0.4 — Manual SQL exploration
- Create `users` manually in psql; insert a row; `SELECT`. Drop and recreate.
- **Learn:** what `UNIQUE`, `NOT NULL`, `CHECK` do.
- **Done when:** you can describe each constraint in your own words.

---

## Milestone 1 — Auth

### M1.1 — Users migration (first real table)
```sql
CREATE TABLE users (
  id bigint PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  email citext NOT NULL UNIQUE,
  username citext NOT NULL UNIQUE,
  password_hash text NOT NULL,
  birthday date NOT NULL,
  phone_e164 text,
  email_verified_at timestamptz,
  is_anonymous_default boolean NOT NULL DEFAULT true,
  bio text CHECK (bio IS NULL OR char_length(bio) <= 280),
  avatar_url text,
  created_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz
);
```
- **Learn:** data types; citext; soft delete.
- **Done when:** migration runs and rolls back cleanly.

### M1.2 — Password hashing (pure function, tested alone)
- Implement in `domain/passwords.py` with Argon2id. **No framework imports.**
- Write 3 tests: hash ≠ input; verify round-trips; wrong password fails.
- **Learn:** hashing ≠ encryption; why SHA-256 is wrong for passwords.
- **Done when:** tests pass and you can explain the parameters (time/memory/parallelism).

### M1.3 — Register endpoint
- Pydantic `RegisterIn` with `extra="forbid"`; age gate; breached-password check; store lowercase email.
- Response is `UserOut` — **no** hash, birthday, phone.
- **Learn:** validation at the boundary; contract security.
- **Done when:** curl register returns 201 with no secrets; DB row exists.

### M1.4 — The 4 security tests, wired as red
- Write them before implementing login/ownership. Watch them fail in CI.
- **Learn:** TDD as a design tool.
- **Done when:** tests exist, fail for the right reason, are in CI.

### M1.5 — Login (enumeration-safe)
- Generic 401 for wrong password AND unknown account — byte-identical body.
- Issue access (15 min) + refresh (days); store **hash** of refresh in `sessions`.
- **Learn:** JWT claims, expiry; account enumeration as a real attack.
- **Done when:** both failure paths return identical bodies; you diff them.

### M1.6 — Refresh rotation
- On redeem: mark used, issue new. Reuse of a used token → revoke session → 401.
- **Test it:** same refresh twice → second fails.
- **Done when:** a replayed refresh is rejected.

### M1.7 — Password recovery
- Reset code to email/phone; 5 wrong → invalidate flow; successful reset revokes **all** sessions.
- **Done when:** reset leaves old sessions dead (test by logging in elsewhere).

### M1.8 — Rate limiting
- Login 5/min/IP, 10/hr/account. Implementation: slowapi or manual counter.
- **Done when:** 6th attempt in a minute returns 429 with `Retry-After`.

### M1.9 — Flutter auth screens
- Register + login forms; token in secure storage; Dio interceptor.
- **Done when:** a user can register, log in, and the next request carries the bearer.

---

## Milestone 2 — Onboarding

### M2.1 — Categories migration + seed
```sql
CREATE TABLE categories (id bigserial PRIMARY KEY, name citext UNIQUE NOT NULL, ...);
INSERT INTO categories (name) VALUES ('Gaming'), ...;  -- 60-80 rows
```
- **Learn:** seed data; why ~60–80 (fewer → everything matches).

### M2.2 — user_categories
- PK `(user_id, category_id)`; CASCADE from user, RESTRICT from category.

### M2.3 — Interest selection endpoint
- Accept list; validate ≥3; write rows in `db.begin()`; 200 only after commit.
- **Test:** 2 interests → rejected; 3 → ok; re-submit preserves.

### M2.4 — Custom suggestion queue
- Store with `proposing_user`, status `pending`; admin merges/creates; notify proposer.
- **Rule:** a suggestion never becomes a live category automatically.

### M2.5 — Flutter picker
- Multi-select, visible checkmarks, Continue disabled until ≥3.

### M2.6 — Gate to feed
- `428 interests_required` if a feed is requested with no interests.
- **Done when:** one cannot reach the feed without ≥3 interests, even by direct API call.

---

## Milestone 3 — For You feed (the hardest one)

### M3.1 — posts migration
- See Section 8 of master guide for the exact DDL.

### M3.2 — Write SQL by hand first (do this!)
- Open `psql`, paste the query from spec §3.7, run against seed data.
- Add filters one at a time; inspect each with `EXPLAIN`.
- **Learn:** reading `EXPLAIN`; spotting seq scans.

### M3.3 — Port to SQLAlchemy (thin repository)
- One method `feed_for(reader_id, cursor)`; include interests join, own-post exclusion, blocks both ways, dedup, cursor.
- **Learn:** repository pattern; ORM vs raw SQL.

### M3.4 — Indexes (verify with EXPLAIN)
```sql
CREATE INDEX posts_feed_idx ON posts (category_id, created_at DESC, id DESC);
```
- **Verify:** the plan uses the index (no seq scan at 10k rows).

### M3.5 — Cursor
- Encode `(created_at, id)` as base64; decode; reject tamper → 400; final page `null`.
- **Test:** tampered cursor → 400; round-trip returns 20 then next 20, no gaps/dups.

### M3.6 — matched_category + dedup
- Include the category that caused inclusion; dedup posts appearing under two interest categories.

### M3.7 — Batched heart/bookmark/author state
- One query for the 20 returned ids; never N+1.

### M3.8 — Flutter feed
- `ListView` + pull-to-refresh + load-more on scroll; per-row matched-category label; empty state with "choose interests" link.

### M3.9 — Tests
- For You only returns interest categories.
- A post appears once.
- No own posts, no soft-deleted, no blocked-author posts.
- Cursor round-trips.

### M3.10 — Concurrency smoke
- Insert a post between page 1 and page 2; verify no gap/duplicate for the reader already mid-scroll.
- **Done when:** all M3 tests green; `EXPLAIN` shows index use at 10k rows.

---

## Milestone 4 — Posting + links

### M4.1 — Create post endpoint
- Body 1–2000, exactly one category, category must be active.
- 429 if > 10 posts/hr.

### M4.2 — Linking rules (service, pure)
- At most one parent; no self-link (DB CHECK too); depth ≤3; `ON DELETE SET NULL` so children survive.

### M4.3 — Anonymous flag
- Copied from user setting at creation; overridable per post.

### M4.4 — Flutter composer
- Category picker (blocked submit if unset), char counter, "replying to" preview for links, draft retained on back, rate-limit handling.

### M4.5 — Tests
- Self-link → 422; depth 4 → 422; deleted parent → child becomes root; no category → 422; body > 2000 → 422.

---

## Milestone 5 — Engagement

### M5.1 — Heart toggle (transactional)
- PK `(user_id, post_id)`; increment/decrement `heart_count` in one transaction.
- **Test:** double heart → one row; concurrent toggle doesn't corrupt counter.

### M5.2 — Audit test (recompute)
- Recompute counters from `hearts`, assert equality. Catches drift.

### M5.3 — Bookmarks
- Per user; list paged by `created_at DESC`.

### M5.4 — Follows
- PK `(follower, followee)`; self-check; Following feed overrides interest filter.

### M5.5 — Flutter
- Optimistic heart toggle with rollback on failure; follow button reflects server state after refresh.

---

## Milestone 6 — Profile & settings

### M6.1 — Own posts feed (paged)
- Author-scoped; soft-delete filtered.

### M6.2 — Username change
- 409 collision, generic body (no enumeration).

### M6.3 — Password change
- Revoke all other sessions; keep current session.

### M6.4 — Anonymity toggle (FR-034)
- Default from setting; overridable per post.

### M6.5 — Cross-user access tests
- Other user's token → 403/404; mass-assignment ignored.

### M6.6 — Flutter screens
- Settings, own profile, change-username/password flows; loading/error states.

---

## Hardening (Week 14)

### H1 — Concurrency & idempotency
- Same operation twice → same outcome; no double rows.
- Cursor stability under concurrent inserts.

### H2 — Load smoke
- 10k posts; document p95; if bad, check index then query plan.

### H3 — Accessibility (Flutter)
- Screen readers, tap targets ≥48, contrast.

### H4 — Reconciliation job
- Recompute denorm counts; run nightly in CI.

### H5 — OWASP pass
- Walk Top 10 against the API; document findings + fixes.

### H6 — Privacy/erasure
- Soft-delete + purge path; verify PII never returned; test erasure end-to-end.

---

## Release (Weeks 15–16)

### R1 — README + diagram
- Setup, architecture, API overview. A stranger can run it.

### R2 — Runbook
- Deploy, migrate, rollback, on-call triage.

### R3 — Pre-flight
- Tick every box (or waive with reason).

### R4 — Deploy
- PaaS or Compose; env + secrets; health check wired to load balancer.

### R5 — Tag + changelog
- `v1.0.0`.

### R6 — Seed content
- 200–500 posts across top 20 categories from founding accounts.

### R7 — Monitoring
- Logs, metrics, error tracking, empty-feed-rate alert.

### R8 — Real-user gate
- Outside person completes the loop unaided. If not, fix the friction and re-test.

---

## Daily rhythm (copy this into your calendar)

- **Block 1 (60–90 min):** build one unit of the current sub-step.
- **Block 2 (45–90 min):** run end-to-end (curl / emulator).
- **Block 3 (15 min):** tests for what you built.
- **Block 4 (15 min):** commit + 3-line note.

Tired → do block 4 only. Never skip two days in a row.

---

## Learning rules (repeat until boring)

1. 10 minutes reading → build something → commit.
2. Guides are looked up **when stuck**, not read ahead.
3. Every session ends with a running thing, even tiny.
4. Freeze at a phase boundary, never mid-slice.

## Triage when stuck

1. Reproduce in 5 lines without the framework.
2. DB: run SQL by hand; check `EXPLAIN`.
3. Interface: open API contract.
4. Config: base URL, ports, `.env`.
5. Read only the matching guide section.
6. Stop after 45 min; write what you tried; ask.

---

## Suggested project file tree (final shape)

```
konekta/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/ (routes.py, deps.py)
│   │   ├── core/ (config.py)
│   │   ├── domain/ (pure logic)
│   │   ├── services/
│   │   ├── repositories/
│   │   └── models/ (SQLAlchemy)
│   ├── alembic/
│   ├── tests/
│   ├── pyproject.toml
│   └── Dockerfile
├── mobile/ (Flutter)
│   ├── lib/
│   │   ├── api/
│   │   ├── models/
│   │   ├── screens/
│   │   ├── state/
│   │   └── theme.dart
│   └── test/
├── docs/
│   ├── 01_KONEK_TA_PRODUCT_SPEC.md
│   ├── 02_KONEK_TA_APP_FLOW.md
│   ├── design/ (Figma handoff)
│   └── ...timeline, master guide, etc.
└── compose.yaml
```

---

## The 12 professional habits to install this week

1. Define Done in writing.
2. Split by layer, not feature.
3. Write threat model before first endpoint.
4. ERD before migration.
5. API contract before coding either side.
6. Domain logic pure (no framework imports).
7. Ownership in the query.
8. Migrations tested both ways.
9. Tests before "it works" claim.
10. PR review every change.
11. Ship weekly, even broken.
12. Write down decisions with reasons.

**If you do only one thing today:** write the one-sentence pitch and list 4–6 MVP features. That single step moves you from idea to project. Everything else depends on it.
