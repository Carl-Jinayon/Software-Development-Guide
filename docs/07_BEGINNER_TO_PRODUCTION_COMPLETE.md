# Complete Beginner-to-Production Guide — Konek'ta (Everything in One File)

**Stack:** Python 3.12 + FastAPI + PostgreSQL + Docker + GitHub Actions (backend) · Flutter (Dart) client (Android first) · Figma (design)  
**Team:** 2 people · ~15–20 hrs/week · ~6 months  
**Goal:** you (a beginner) can read this once and know what to do each week, what to learn, and why.

> Professional rule this whole guide rests on: **software is 30–40% coding**. The rest is deciding what to build, placing things, proving they work, and shipping. Beginners fail by skipping that 60% and drowning in code.

---

## 0. How a professional actually builds (mental model)

```
Idea → 1 sentence → 4–6 MVP features → scope locked
     → ERD + ADRs + API contract → threat model + tests planned
     → foundation (CI green) → vertical slices (weekly) → harden → deploy → learn from real users → iterate
```

**Vertical slice** = one feature done from database → backend → client → tests → deployed. Never "backend 90% done, client 0%."

**Three habits to install today:**
1. Every day ends with a commit + 3-line note: *what broke, what I learned*.
2. Never push to `main` directly; always `feature/x → PR → review → merge`.
3. Read a guide **only when stuck** on the thing in front of you. Close it otherwise.

---

## 1. Environment setup (Week 0, once)

### 1.1 Backend
```bash
# Install uv (faster than pip)
curl -LsSf https://astral.sh/uv/install.sh | sh   # Windows: winget install astral-sh.uv

mkdir konekta && cd konekta
mkdir backend mobile docs
cd backend
uv init
uv add fastapi uvicorn[standard] sqlalchemy[asyncio] asyncpg alembic pydantic-settings argon2-cffi pyjwt python-dotenv
uv add --dev ruff mypy pytest pytest-asyncio httpx
```

Create `backend/pyproject.toml` (ruff + mypy + pytest). Copy from `TEMPLATES/02_PYTHON_CODE_PATTERNS.md` instead of inventing.

### 1.2 Docker Compose
```yaml
# compose.yaml
services:
  api:
    build: ./backend
    env_file: .env
    ports: ["8000:8000"]
    depends_on: [db]
  db:
    image: postgres:16
    environment: { POSTGRES_USER: app, POSTGRES_PASSWORD: app, POSTGRES_DB: app }
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]
volumes:
  pgdata:
```

### 1.3 Flutter
```bash
# Install Flutter SDK (flutter.dev), then:
cd mobile
flutter create . --platforms=android
flutter pub add http dio flutter_secure_storage provider
```

### 1.4 Git hygiene from commit one
```bash
cat > .gitignore <<'EOF'
.env
__pycache__/
*.pyc
.dart_tool/
build/
.idea/
.vscode/
EOF
git init -b main
```

**Verify:** `docker compose up` shows both containers healthy; `flutter doctor` shows green.

---

## 2. Design & scope before code (Weeks 1–4)

### 2.1 The four questions you must answer
1. One-sentence pitch.
2. Core loop (the one thing users repeat).
3. MVP list (4–6 features) + defended "Won't have" list.
4. Top 3–5 risks, each with a mitigation and a measurable trigger.

**Professional technique — MVP by subtraction:**
Write the full dream list → define the core loop → for each feature ask *"if deleted, could the user still complete the loop and come back?"* → keep only the "no" answers → 60-second test (new user doing the core loop in under a minute).

### 2.2 ERD (write before any migration)
The five that matter for Konek'ta: `USER`, `CATEGORY`, `USER_CATEGORY`, `POST`, `POST_LINK` (a post's parent). Sketch in DBML/mermaid. Decide cascades now:
- `POST.parent_post_id` → `ON DELETE SET NULL` (never CASCADE — one deletion wiping a thread).
- Reports → `RESTRICT` (evidence survives).
- Everything else → CASCADE.

### 2.3 ADRs (min 3)
Write one per decision: **FastAPI vs Django**, **Postgres**, **monolith vs microservices**, plus app-specific ones (keyset pagination, admin-only categories, no comment table, reveal mechanic cut).

### 2.4 API contract (the most important doc)
Before either side builds, both agree on endpoints, request/response shapes, status codes, pagination cursor, and the Android-side error-handling table. Rule: **nobody changes it alone**.

### 2.5 Figma (parallel)
Wireframes → hi-fi → tokens. Export tokens once into `mobile/lib/theme.dart`. Empty/error/loading states designed **now**, not later.

---

## 3. Security design (Week 5, before any endpoint)

**Learn:** STRIDE (Spoofing/Tampering/Repudiation/Info disclosure/Denial of service/Elevation).  
**Do:** threat model doc; NFRs with numbers (latency, rate limits, PII never returned); test plan.

**Non-negotiable security checklist (build in, not later):**
- Argon2id password hashing.
- JWT access (15 min) + refresh rotation; reuse → revoke session.
- Ownership **in the query** (`WHERE owner_id = :user_id`).
- Parameterised queries only — no f-string SQL.
- Pydantic `extra="forbid"` (no mass assignment).
- Rate limits: login 5/min/IP, 10/hr/account.
- PII (`birthday`, `phone`) **never** returned by any endpoint.
- `.env` gitignored; pip-audit + Bandit in CI; secrets in env or a vault.

**The 4 security tests (write as named stubs before impl):**
```python
def test_cannot_read_another_users_profile(client, other_user): ...
def test_cannot_modify_another_users_profile(client, other_user): ...
def test_mass_assignment_cannot_escalate(client): ...
def test_responses_never_contain_secrets(client): ...
```

---

## 4. Milestone 0 — "Hello, service" (Week 6)

**Learn:** FastAPI app factory, Docker Compose, env config, Flutter http.  
**Goal:** one request crosses every layer once.

**Backend files:**
```python
# backend/app/main.py
from fastapi import FastAPI

def create_app() -> FastAPI:
    app = FastAPI(title="Konek'ta API")
    @app.get("/health")
    async def health() -> dict[str, str]:
        return {"status": "ok"}
    return app

app = create_app()
```
```python
# backend/app/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    jwt_secret: str
    class Config:
        env_file = ".env"
```

**Client:**
```dart
// mobile/lib/api.dart
import 'package:http/http.dart' as http;

Future<String> fetchHealth(String baseUrl) async {
  final res = await http.get(Uri.parse('$baseUrl/health'));
  if (res.statusCode != 200) throw Exception('HTTP ${res.statusCode}');
  return res.body; // {"status":"ok"}
}
```

**Verify:** `docker compose up`, `curl http://localhost:8000/health` → 200, emulator shows status, CI green with 0 tests, a friend clones and runs it.

---

## 5. Milestone 1 — Auth (Week 7–8)

**Learn:** Argon2id, JWT, refresh rotation, enumeration safety, rate limiting.

**Backend core (illustrative):**
```python
# domain/passwords.py  (pure — no framework imports)
from argon2 import PasswordHasher
_ph = PasswordHasher(time_cost=3, memory_cost=65536, parallelism=4)

def hash_password(plain: str) -> str: return _ph.hash(plain)
def verify_password(plain: str, hashed: str) -> bool:
    try:
        return _ph.verify(hashed, plain)
    except Exception:
        return False
```
```python
# api/auth.py (excerpt)
@router.post("/auth/register", status_code=201)
async def register(payload: RegisterIn, db: AsyncSession = Depends(get_session)):
    # age gate, breached-password check, uniqueness
    user = User(email=payload.email.lower(), username=payload.username,
                password_hash=hash_password(payload.password), ...)
    db.add(user); await db.commit(); await db.refresh(user)
    return UserOut.model_validate(user)  # never exposes hash/birthday/phone
```

**Enumeration-safe login:** wrong password and unknown account both return the **same** 401 body. Recovery flow returns `202` with an identical body whether or not the account exists.

**Refresh rotation:** issue a refresh token as JWT with a `jti`; store only its hash in `sessions`. On redeem, mark it used and issue a new one. **Reuse of a used token revokes the session.** Test this explicitly.

**Verify:** curl register/login/refresh; second refresh with same token → 401; mass-assignment of `role` ignored; no PII in any response.

---

## 6. Milestone 2 — Onboarding (Week 9)

**Learn:** FKs, seed data, transactional writes, hard UX blocks.

**Build:**
- `categories` table + seed 60–80 rows (admin-only creation in v1).
- `user_categories` PK `(user_id, category_id)`.
- Endpoint requires ≥3 interests; writes all rows **before** 200.
- Custom suggestions: status `pending`; never auto-publish.
- Feed returns `428 interests_required` for a user with 0 interests (never empty list as "fine").

**Flutter:** multi-select picker, disabled Continue under 3, visible selection state.

---

## 7. Milestone 3 — For You feed (Week 10) ⭐ hardest query

**Learn:** SQL JOINs, subqueries, keyset pagination, N+1.

**Discipline:** write the SQL in `psql` **by hand first**, see correct rows, then port to SQLAlchemy.

**The query (from your spec):**
```sql
SELECT p.id, p.body, p.category_id, p.is_anonymous,
       p.heart_count, p.linked_count, p.created_at, p.parent_post_id,
       u.username, c.name AS category_name
  FROM posts p
  JOIN users u ON u.id = p.author_id AND u.deleted_at IS NULL
  JOIN categories c ON c.id = p.category_id AND c.is_active
 WHERE p.deleted_at IS NULL
   AND p.category_id IN (SELECT category_id FROM user_categories WHERE user_id = :reader_id)
   AND p.author_id <> :reader_id
   AND p.id NOT IN (SELECT post_id FROM blocks WHERE blocker_id = :reader_id)
   AND p.author_id NOT IN (SELECT blocked_id FROM blocks WHERE blocker_id = :reader_id)
   AND (:cursor_ts IS NULL OR (p.created_at, p.id) < (:cursor_ts, :cursor_id))
 ORDER BY p.created_at DESC, p.id DESC
 LIMIT 20;
```

**Key decisions to defend:**
- `p.id NOT IN (SELECT parent_post_id...)` → a post in two of the reader's categories appears once.
- Exclude own posts (your posts live on your profile).
- Keyset, not `OFFSET` (constant cost per page).
- No joins to `hearts`/`bookmarks` per row — batch them for the 20 returned ids.

**Index:**
```sql
CREATE INDEX posts_feed_idx ON posts (category_id, created_at DESC, id DESC);
```
(Equality column first, range column last. Reversed = useless.)

**Cursor:** base64-encode `timestamp|id`; reject tampering → 400; final page `next_cursor = null`.

**Flutter:** `ListView` + pull-to-refresh + load-more-on-scroll; explicit empty state; show `matched_category` per row.

**Verify:** For You only returns interest categories; appears once; no own posts; soft-deleted excluded; cursor round-trips with no gaps/dups.

---

## 8. Milestone 4 — Posting + links (Week 11)

**Learn:** self-referential FK, depth check, constraints, cascade semantics.

**Model rules:**
```sql
CREATE TABLE posts (
  id bigint PRIMARY KEY,
  author_id bigint NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  category_id bigint NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,
  parent_post_id bigint REFERENCES posts(id) ON DELETE SET NULL,
  body text NOT NULL CHECK (char_length(body) BETWEEN 1 AND 2000),
  is_anonymous bool NOT NULL DEFAULT true,
  heart_count int NOT NULL DEFAULT 0 CHECK (heart_count >= 0),
  linked_count int NOT NULL DEFAULT 0 CHECK (linked_count >= 0),
  created_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz,
  CHECK (parent_post_id IS NULL OR parent_post_id <> id)
);
```

**Service (pure):**
```python
def depth_of(parent_id: int, lookup) -> int:
    n, cur = 0, parent_id
    while cur is not None and n < 5:
        n += 1
        cur = lookup(cur)   # parent_post_id or None
    return n

# reject if depth > 3
```

**Note:** no title, no editing (editing breaks linked posts' meaning). `is_anonymous` copied from user setting at creation but overridable per post.

**Verify:** self-link → 422; depth 4 → 422; deleting parent leaves child as root; category required → 422.

---

## 9. Milestone 5 — Engagement (Week 12)

**Learn:** transactions, toggles, idempotency.

**Heart toggle (transactional):**
```python
async with db.begin():
    existing = await db.get(Heart, (user_id, post_id))
    if existing:
        await db.delete(existing)
        await db.execute(update(Post).where(Post.id == post_id).values(heart_count=Post.heart_count - 1))
    else:
        db.add(Heart(user_id=user_id, post_id=post_id))
        await db.execute(update(Post).where(Post.id == post_id).values(heart_count=Post.heart_count + 1))
```
PK `(user_id, post_id)` prevents double despite races.

**Audit test:** recompute `heart_count` from `hearts` and assert equality — this catches silent drift.

**Follows:** `CHECK (follower_id <> followee_id)` + PK `(follower_id, followee_id)`. **Following overrides interest filtering** (explicit subscription).

---

## 10. Milestone 6 — Profile & settings (Week 13)

- Own posts (paged).
- Username change → 409 with generic body (no enumeration).
- Password change → revoke **all other sessions**.
- Number / bio change (PII rules).
- Anonymity toggle (FR-034) defaults from user setting, overridable per post.
- Cross-user access test: another user's token → 403/404.

---

## 11. Hardening (Week 14)

- Concurrency tests on feed: idempotency, cursor stability under inserts/deletes.
- Load smoke: For You query at 10k posts; document p95.
- Accessibility: screen readers, tap targets, contrast (Flutter).
- Reconciliation job: recompute denorm counters; compare.
- OWASP Top 10 pass against API (auth, injection, SSRF, etc.).
- Privacy review: no PII responses; erasure path tested.

---

## 12. Release (Weeks 15–16)

- README a stranger can follow + architecture diagram.
- Deployment runbook: deploy, migrate, rollback, on-call triage.
- Pre-flight checklist (every box ticked or explicitly waived).
- Tag `v1.0.0` + changelog.
- Beta invites: 30–50, then +5/week.
- Seed 200–500 posts across top 20 categories from founding accounts.
- Monitoring live: logs, metrics, error tracking, empty-feed-rate alert.

**The real gate:** a person not on the team completes register → interests → read → write linked post, unaided, developer out of the room.

---

## 13. Stabilise & iterate (Weeks 17–24+)

- Weekly bug triage: P0 within 24h, rest weekly.
- Iterate from **observed behaviour**, not guesses. Empty-state friction → biggest wins.
- Monthly: answer 4 checkpoint questions honestly. If #1 or #2 is "no", delay v1.1 and fix fundamentals.
- v1.1: pick **one** deferred feature by real-user evidence, run the whole cycle again (no skipping design).
- Maintenance ongoing: dependency/security scans weekly, DB restore drill monthly, threat-model refresh quarterly, explicit sunset decision when the time comes.

---

## 14. Daily & weekly rhythm (what your calendar looks like)

**Daily (2–4 hrs):**
1. 60–90 min: build one unit of the current slice.
2. 45–90 min: run end-to-end; hit with curl/emulator.
3. 15 min: tests for what you built.
4. 15 min: commit + 3-line note.

**Weekly (Sunday 30-min sync):** pick the slice, assign ownership, confirm the gate. **Saturday:** ship + "Week N" note (shipped / bugs / blocked / next).  
**Monthly:** checkpoint questions; if fundamentals failing, stop new features.

**When exams/life hit:** freeze at the current **phase boundary**, never mid-slice. Resume after.

---

## 15. Learning map (what to learn, when, and where)

| When | Learn | Where |
|---|---|---|
| W0 | Git, Compose, HTTP basics, tooling | `GUIDES/01` Part 0–1 |
| W1–2 | MVP scoping, requirements thinking | `GUIDES/01` Part 2–3 |
| W3–4 | ERD, ADRs, API contracts | `GUIDES/01` Part 4 + `TEMPLATES/01` T03/T04/T06/T07 |
| W5 | Threat modelling, OWASP basics | `GUIDES/03` while implementing auth |
| W6–7 | Python typing, FastAPI DI, Flutter HTTP, Docker | `TEMPLATES/02` code shape |
| W8–9 | Auth crypto, FK/seed | `GUIDES/03` §4/§10 |
| W10 | SQL/JOINs, indexes, pagination | `GUIDES/06` (cache/pagination sections) |
| W12 | Transactions, idempotency | `GUIDES/04` (testing), `GUIDES/06` |
| W14 | Load, a11y, OWASP | `GUIDES/03`, `GUIDES/05` |
| ongoing | Only read sections when stuck on that problem | README's one rule |

Rule: **never** read a guide you aren't currently stuck on.

---

## 16. If you get stuck (triage order)

1. Reproduce in 5 lines with no framework. Logic bug? Fix it there — not in the route.
2. DB? Run SQL by hand; check index; `EXPLAIN`.
3. Interface mismatch? Open API contract — 90% of "broken" is a shape mismatch.
4. Config? base URL, ports, `.env` typo.
5. Read only the matching guide section.
6. Stop after 45 min; write down what you tried; ask partner / open issue. Don't spiral.

---

## 17. Quick commands you'll use daily

```bash
# backend
uv run uvicorn app.main:app --reload
uv run alembic upgrade head
uv run alembic downgrade -1
uv run pytest -q
uv run ruff check . && uv run mypy .
docker compose up -d && docker compose logs -f api

# DB shell
docker compose exec db psql -U app -d app

# Flutter
cd mobile && flutter pub get
flutter run
flutter test
flutter analyze
```

---

## 18. Final truth

The hardest part of this app isn't coding. It's deciding what **not** to build, writing that down with your partner before either of you types, and refusing to reopen scope at 2am. Every template, gate, and test in this guide exists to protect that one habit:

> Write it down. Agree. Then code.
