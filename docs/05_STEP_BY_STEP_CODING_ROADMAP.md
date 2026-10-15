# Step-by-Step Coding Roadmap (Beginner, Flutter + FastAPI)

**Project:** Konek'ta — FastAPI + PostgreSQL backend · Flutter (Dart) client (Android first) · Figma for design.  
**Typical cadence:** ~2–4 hours/day, 6 days/week. Every day ends with a commit.  
**Companion files:** `docs/03_PROJECT_LIFECYCLE_TIMELINE.md` (weeks/gates), `docs/01_KONEK_TA_PRODUCT_SPEC.md` (requirements).

> **How professionals learn while building:** do not finish a tutorial first. Read 10 minutes, build one running thing, commit. Repeat. This roadmap is deliberately sliced vertically: each milestone is a tiny app that a user could use.

---

## 0. Before you write code (once)

- [ ] Repos created (`backend/`, `mobile/` if separate, or one monorepo).
- [ ] Branch protection on `main`. CI runs lint + tests.
- [ ] `.gitignore` in place **before** first commit (`.env`, `*.pyc`, `build/`, `.dart_tool/`).
- [ ] `docs/` holds: decision log, ERD, API contract, threat model placeholders.
- [ ] Agreed Definition of Done (6-point) in `MY_PLAN/02`.

**Mental model (hold this the whole time):**
```
Flutter (widgets)  --HTTP/JSON-->  FastAPI  --SQL-->  PostgreSQL
  http/Dio                     routes.py        repositories.py
  state mgmt                   services.py      SQLAlchemy models
                               domain/ (pure Python, no framework)
```
**The one rule that makes learning possible:** `domain/` imports **nothing** from FastAPI or SQLAlchemy. If you can't test it with `pytest` and no DB, it's in the wrong place.

---

## Milestone 0 — "Hello, service" (target: 1 week)

**Goal:** one request travels Flutter → FastAPI → Postgres → back. That proves the plumbing before any feature.

**Learn just-in-time:**
- Python: venv with `uv`, type hints, Pydantic.
- HTTP: path, body, status code.
- Docker Compose (what it does, not every option).
- Flutter: `StatelessWidget`, `http.get`, `FutureBuilder`.

**Build backend (commit-by-commit):**
1. `pyproject.toml` (copy `TEMPLATES/02` config) + `uv sync`.
2. `app/main.py`: app factory only; `GET /health → {"status":"ok"}`. No logic here.
3. `compose.yaml`: FastAPI + Postgres + healthcheck; `.env.example` committed.
4. Alembic init; one empty migration so you've practised migrate + rollback.

**Build client:**
1. New Flutter project; `flutter run` launches an emulator.
2. One screen that calls `GET /health` with `http` (or Dio), shows status or error.
3. Base URL from a config, not hard-coded everywhere; map 401/404/network to messages.

**Verify:**
- `docker compose up` running; `curl http://localhost:8000/health` → 200.
- Emulator screen shows status string.
- CI green with **0 tests** (that's acceptable at Milestone 0).
- Someone else clones, installs, runs both. If they can't, fix the README, not "they did it wrong".

**Common newbie mistakes to avoid:**
- Committing `.env`. (Set `.gitignore` first, verify with `git status`.)
- Putting logic in `main.py`. Keep it a factory.
- Skipping types because "it's tiny". Types cost least today.

---

## Milestone 1 — Auth (target: 1 week)

**Goal:** register, log in, stay logged in, recover password — with the 4 security tests green.

**Learn just-in-time:** Argon2id (never SHA-256), JWT access vs refresh, why store a **hash** of the refresh token, refresh rotation + reuse detection, HTTP 401/403/409/422, account enumeration.

**Build backend in this order:**
1. `users` migration: id, email citext **unique**, username citext **unique**, password_hash, birthday date, phone_e164, is_anonymous_default, created_at, deleted_at. *No real_name column.*
2. `register`: Pydantic `extra="forbid"`, age gate (FR-006), breached-password check, Argon2id hash, response with **no** password_hash/birthday/phone.
3. `login`: generic 401 **identical** for wrong password and unknown account.
4. JWT access (15 min) + refresh; store **hash** of refresh in `sessions`.
5. Refresh rotation: reuse → revoke session.
6. Password recovery: code, 5-wrong invalidation, revoke **all** sessions on success.
7. Rate limits: 5/min per IP, 10/hr per account.

**The 4 security tests (write these as named stubs before implementing):**
- `test_cannot_read_another_users_profile`
- `test_cannot_modify_another_users_profile`
- `test_mass_assignment_cannot_escalate` (send `role`/`id`)
- `test_responses_never_contain_secrets`

**Build client:** register + login screens, token storage (`flutter_secure_storage` or DataStore equivalent), Dio interceptor attaching bearer, logout.

**Verify:** register → row in DB, no PII in response; refresh **twice with the same token → 401 the second time**; mass-assignment ignored.

---

## Milestone 2 — Onboarding (target: 1 week)

**Goal:** a user cannot reach the feed until they pick ≥3 interests.

**Build:**
1. `categories` table + seed 60–80 (admin-only in v1).
2. `user_categories` with PK `(user_id, category_id)`.
3. `PUT/POST interests`: write all rows before 200.
4. Custom-suggestion queue (status `pending`, admin merges).
5. Feed returns `428 interests_required` for 0 interests.
6. Flutter: multi-select picker, visible selection, block-continue under 3.

**Verify:** register → lands in picker, **cannot bypass to feed** with <3 interests.

---

## Milestone 3 — For You feed (target: 1 week) ⭐ hardest query

**Goal:** paginated, interest-filtered, deduped, explainable feed with keyset cursor.

**Discipline:** write the SQL in `psql` **by hand first**, make it return correct 20 rows, *then* port to SQLAlchemy. This single habit removes most feed bugs.

**Build:**
1. `posts` migration with constraints + counters start 0; seed 50 posts across categories.
2. Repository method `feed_for(reader_id, cursor)` with the four filters: interests IN, exclude own posts, block both ways, dedup by parent.
3. Keyset cursor on `(created_at, id)` — base64-encode; reject tampering → 400.
4. `matched_category` in every response.
5. Flutter: list with pull-to-refresh + load-more on scroll; explicit empty state.

**Verify (tests):**
- Every returned post's category ∈ reader's interests.
- A post in two followed categories appears **once**.
- Own posts never appear; soft-deleted never appear.
- Cursor round-trips: 20 then 20, no gaps, no duplicates.
- If slow: run `EXPLAIN` first. Almost always the missing index `(category_id, created_at DESC, id DESC)`.

---

## Milestone 4 — Posting + links (target: 1 week)

**Goal:** create a post, link to a parent, enforce 3-level depth.

**Build:**
- `POST /posts`: body 1–2000 (CHECK), exactly one category, optional `parent_post_id`.
- Depth check in service: walk parent chain; reject >3 → 422.
- DB: `CHECK (parent_post_id <> id)`; `ON DELETE SET NULL` on parent (BR-05).
- Anonymous flag copied from user setting at creation, overridable per post.
- Flutter: composer with category picker + "replying to" preview + char counter (blocked if no category).

**Verify:** self-link → 422; depth 4 → 422; deleting a parent leaves children as roots.

---

## Milestone 5 — Engagement (target: 1 week)

**Goals (backend transactions):**
- Hearts: toggle with `heart_count` increment/decrement in **one transaction**; PK `(user_id, post_id)` blocks doubles.
- Bookmarks per user.
- Follows with `CHECK (follower <> followee)`.
- Add an **audit test**: recompute counts from base tables, assert equals. Catches drift.

**Client:** optimistic heart toggle with rollback on failure; follow buttons reflect state after refresh.

---

## Milestone 6 — Profile & settings (target: 1 week)

- Own posts feed.
- Username change → 409 (generic body) on collision.
- Password change → revoke other sessions.
- Number change, bio, avatar URL (optional).
- Anonymity toggle (FR-034): posting uses setting, overridable per post.
- Verify cross-user access: someone else's token can't read or edit your data.

---

## 1. Design with Figma (parallel track, Week 3)

Professionals **draw before they type**. Keep Figma as the source of truth for UI:

- Low-fi wireframes first (boxes + labels), then high-fi only after flow is locked.
- One frame per screen in the spec's screen map; include loading/empty/error states.
- Tokens: colors, type scale, spacing → export into `mobile/lib/theme.dart` (or equivalent) **once**, then reuse.
- Handoff folder: `docs/design/` with exported specs + a short "how to read this file" note.

**Rule:** design tokens change only through Figma + a PR. No color literals in client code.

---

## 2. Branching & quality (every week, non-negotiable)

- `main` always green and deployable.
- Branch: `feature/<slice>` → PR → review → merge.
- CI: lint → type check (`mypy` for backend) → tests → `pip-audit` + Bandit → (mobile) `flutter analyze` + tests.
- Definition of Done: runs on clean clone · ≥1 test for core logic · 90%+ domain coverage · 4 security tests where data is touched · reviewed · documented · merged.
- Ship every Saturday with a "Week N" note: shipped / bugs / blocked / next.
- One "no new topics" day per week to consolidate.

---

## 3. If you get stuck (professional triage order)

1. **Reproduce in 5 lines without the framework.** `python -c` or a pytest/Flutter test. Logic bug? Fix here — not in the route.
2. **DB?** Run the SQL by hand in `psql`; inspect indexes; `EXPLAIN`.
3. **Interface mismatch?** Open the API contract doc — 90% of "broken" is a shape mismatch.
4. **Config/env?** Base URL, ports, `.env`. Typos before re-architecture.
5. **Read only the matching guide section.** Not the whole file.
6. **Stop after 45 min of spinning.** Write down what you tried; open an issue / ask partner. Don't spiral.

---

## 4. Daily routine (~2–4 hrs, sustainable)

| Block | Time | Do |
|---|---|---|
| 1 | 60–90 min | Build one unit of the current slice (endpoint / screen / test). |
| 2 | 45–90 min | Run end-to-end; hit it with curl/emulator. |
| 3 | 15 min | Tests for what you just built. |
| 4 | 15 min | Commit + 3-line write-up ("what broke, what I learned"). |

Tired? Do block 4 only. **Never skip two days in a row** (momentum beats intensity).

---

## 5. "Done learning to code" in this project

Not mastery. Competence = taking one FR from the spec, implementing backend → client, writing its tests, deploying it — in a week, alone, with tutorials closed for the parts you already know. That's the Week 13 bar here.

---

## 6. Quick reference map

- Architecture & why monolith → `GUIDES/06` §§1–4 only.
- Testing → `GUIDES/04` when you write the first real test.
- Auth/security → `GUIDES/03` while implementing auth.
- DevOps/CI/Docker → `GUIDES/05` when setting up CI.
- Backend code shape → `TEMPLATES/02` (copy, don't invent).
- Templates for every artifact → `TEMPLATES/01` (T01–T21), named where used in the timeline.

*If a guide topic doesn't match the problem in front of you today, close the file.*
