# Project Lifecycle Timeline

**Project:** Konek'ta — relevance-first social feed (FastAPI + PostgreSQL backend · Flutter/Dart client (Android first) · Figma for design)  
**Team:** 2 (backend/security + Android client)  
**Owner:** both partners share ownership; each section names a single accountable person (RACI).  
**Method:** iterative, gate-driven. Weekly shipped builds. No code starts before the Phase 2 gate.

> **How professionals use this file:** this is the *shared contract of schedule*, not a wish list. A task is not "done" because someone worked on it; it is done when the phase gate passes. If a gate fails, the next phase does not start — you fix and re-gate.

---

## 0. Strategic framing

| Principle | Rule |
|---|---|
| Ship something that runs every week | Even a broken deployed build teaches more than perfect code never released |
| Plan in weeks, deliver in slices | A "release" is a vertical slice users can complete, not a layer that is "90% done" |
| Gate before transition | Each phase ends at a gate. Failing a gate means the phase restarts, not that you push forward |
| Change control after coding starts | Scope changes require updating this doc + the spec + agreed decision log entries |

**Rough effort:** ~6 months, ~15–20 hrs/week total across both people. Roughly 30–40% planning/design, 60–70% build/test/ship.

---

## 0.5 Learning while building (newbie protocol)

**How professionals actually learn a new stack:** they don't finish a 40-hour tutorial first. They build one running slice, learn exactly the 20% of the stack that slice needs, then repeat. Depth accrues from failure in real code, not from notes.

Rules (non-negotiable):
1. **Read 10 minutes, build something.** Never read a guide cover to cover before coding.
2. **Every day ends with a commit and a 3-line write-up** ("what broke, what I learned").
3. **One concept → one test → one commit.** A concept you can't demonstrate in 5 lines isn't learned yet.
4. **Read the matching guide section only when you hit the wall it describes.** If you aren't stuck, close the file.
5. **No dual-track "study" days.** If you're tired, commit 5 lines or skip entirely — don't pseudo-learn.
6. **Freeze at a phase boundary, never mid-slice** when exams or life intervene.

Mental model to hold before the first line:
```
Android (Compose)  --HTTP/JSON-->  FastAPI  --SQL-->  PostgreSQL
  Retrofit                     routes.py        repositories.py
  ViewModels                  services.py       SQLAlchemy models
                              domain/        (pure Python, no framework)
```
Keep `domain/` importing nothing from FastAPI/SQLAlchemy. That single rule is what makes learning (and testing) possible.

---

## 1. Timeline at a glance

| Phase | Window | Duration | Output |
|---|---:|---:|---|
| 0 · Kickoff & baseline | Week 0 | 1 week | Tooling, repo, shared vocabulary |
| 1 · Discovery | Week 1 | 1 week | One-sentence pitch, core loop, risk scan |
| 2 · Scope | Week 2 | 1 week | MVP list (4–6) + defended "Won't have" list |
| 3 · Design | Weeks 3–4 | 2 weeks | Charter, ERD, use cases, ADRs, API contract |
| 4 · Security design | Week 5 | 1 week | Threat model, NFRs, test plan, Definition of Done |
| 5 · Foundation build | Weeks 6–7 | 2 weeks | Repo scaffold, CI green, Docker, first migrations |
| 6 · MVP build (sprints) | Weeks 8–13 | 6 weeks | 6 vertical slices, each shippable |
| 7 · Hardening | Week 14 | 1 week | Security tests, load smoke, reliability |
| 8 · Release readiness | Week 15 | 1 week | Docs, runbook, pre-flight checklist |
| 9 · Deploy & launch | Week 16 | 1 week | Live URL, beta invites, monitoring |
| 10 · Stabilisation & feedback | Weeks 17–20 | 4 weeks | Fix real bugs from real users |
| 11 · v1.1 increment | Weeks 21–24 | 4 weeks | First deferred features, by demand |
| 12 · Maintenance & retirement decision | Ongoing | — | Operate, patch, plan v2 or sunset |

**Total to call "MVP done, in production, used by real users": ~16–20 weeks.**

---

## 2. Roles (RACI summary)

| Area | Accountable | Consulted |
|---|---|---|
| FastAPI service, Postgres, migrations, business logic | Backend | — |
| Flutter client, screens, navigation, state | Client | — |
| Auth, authorisation, rate limiting, threat model | Backend | Android (UX impact) |
| CI/CD, Docker, deployment runbook | Backend | Android (build/release) |
| API contract changes | **Both (must agree)** | — |
| Scope changes / MVP list | **Both (must agree)** | — |
| Security review of PRs | Backend | — |
| User-facing copy, empty-state design | Both | — |

**Rule:** one accountable person per deliverable. "Both" only where the artifact *is* the interface between the two.

---

## 3. Phase details

### Phase 0 — Kickoff & baseline (Week 0)

**Goal:** tooling, repository, shared vocabulary. No product decisions yet beyond stack.

| Owner | Deliverables |
|---|---|
| Both | Read `GUIDES/01` Part 0–1; agree on the 25 vocabulary terms |
| Backend | GitHub repo, branch protection on `main`, CI skeleton, issue labels |
| Android | Flutter project scaffold, state mgmt + http/Dio stub, Figma-linked component plan |
| Both | `docs/` folder created; `.env` gitignored from commit one |

**Gate 0:** A stranger can clone both repos, run `docker compose up` / open the app, and see a running (empty) thing.

---

### Phase 1 — Discovery (Week 1)

**Goal:** move from a list of ideas to a shared picture of the product.

| Deliverable | Owner | Source |
|---|---|---|
| One-sentence pitch (P1–P6 principles supported) | Both | Product spec Part 2 |
| Core loop documented | Both | Product spec Part 2.5 |
| User classes named (target + explicitly-not-target) | Both | Product spec Part 2.4 |
| Risk scan: top 5 risks + mitigations + triggers | Both | Product spec Part 12 |

**Gate 1:** Both can state the pitch in one sentence without reading; top risks include measurable triggers.

---

### Phase 2 — Scope (Week 2)

**Goal:** decide what ships and — more importantly — what does not.

| Deliverable | Owner |
|---|---|
| MVP list, 4–6 features, by subtraction (survival test + 60-second test applied) | Both |
| Written "Won't have" list with reasons | Both |
| Backlog of deferred items tagged `later` with one-line justification | Both |
| Project charter (scope, timeline, roles, definition of success) | Both |

**Gate 2:** Both answer "what are we building?" and "what are we not building?" with identical lists.

---

### Phase 3 — Design (Weeks 3–4)

**Goal:** lock the structure both sides will code against.

| Week | Focus | Deliverables | Owner |
|---|---|---|---|
| 3 | Data | ERD with cascade rules, business rules, indexes (formalise the domain model) | Backend (Android reviews) |
| 3 | UX flow (Figma) | Wireframes → hi-fi in Figma; screen map; navigation graph; error/empty; Design tokens (colors/type/spacing) | Client (Backend reviews) |
| 3 | Design handoff | Figma component spec + asset export checklist; handoff doc into `docs/design/` | Both |
| 4 | Decisions | 3+ ADRs: framework, DB, monolith-vs-microservices, plus app-specific calls (e.g. keyset pagination, `SET NULL` on parent link, admin-only categories) | Both |
| 4 | Interface | Full API contract: endpoints, request/response shapes, status codes, pagination, Android-side error handling table | Both |

**Gate 3:** Partner can describe every endpoint and the primary ERD entities without opening code.

---

### Phase 4 — Security design (Week 5)

**Goal:** security and quality constraints exist *before* there is endpoint code.

| Deliverable | Owner | Source |
|---|---|---|
| Threat model (STRIDE, assets, mitigations) | Backend (Both review) | Spec Part 10 + security guide |
| NFRs with metrics (latency, rate limits, PII handling, availability target) | Both | Spec Part 7 |
| Test plan + acceptance-criteria → test-case mapping | Both | Spec Part 4 ACs |
| Definition of Done (6-point) | Both | MY_PLAN §2 |
| The 4 security tests written as named stubs | Backend | MY_PLAN §8 |

**Gate 4:** Every FR has ≥1 test case; threat model exists; security tests named and wired into CI as failing placeholders.

---

### Phase 5 — Foundation build (Weeks 6–7)

**Goal:** plumbing runs before any feature. This is where 80% of beginner confusion happens, so build the smallest thing that crosses every layer once.

**What you are learning this phase (just-in-time):** Python typing & Pydantic, `async def` basics, FastAPI app factory + dependency injection, Docker Compose, Alembic migrations, Flutter (Dart) + state mgmt + http/Dio, Figma handoff, git branching/PRs.

**Build sequence (each is its own commit/PR):**
1. **`uv` + `pyproject.toml`** with ruff/mypy/pytest configured → run `pytest` (0 tests, green).
2. **`app/main.py`** as a pure factory; `GET /health → {"status":"ok"}`. No logic in `main.py`.
3. **`compose.yaml`**: FastAPI + Postgres + healthcheck; `.env.example` committed, `.env` gitignored.
4. **Alembic init** + first migration for `users` (empty schema, just creates the table). Practice migrate **and roll back** once.
5. **Flutter scaffold**: app with `http`/Dio client, base URL from env/config, one GET `/health` call, error mapping (401/404/network), loading/empty/error widgets. Also export Figma tokens (colors/typography/spacing) into a shared `theme.dart`.
6. **CI (GitHub Actions):** lint → type check → tests → `pip-audit` / Bandit. All green on `main`. Use the workflow in `TEMPLATES/02` as the copy-paste source.

**Beginner checkpoint:** open the app in an emulator, see `{"status":"ok"}` on screen. If it doesn't, the bug is almost certainly a base URL, port, or missing health route — check those before "learning more architecture".

**Gate 5:** Clean clone runs both sides; CI green; one health check passes end-to-end from phone to DB and back.
---

### Phase 6 — MVP build, in 6 vertical slices (Weeks 8–13)

One feature per week, each one complete end-to-end (backend + client + tests), not half-implemented layers. Do not start week N+1's slice until week N's passes its own tests.

| Week | Slice | Key deliverables | Owner (vertical effort) |
|---|---|---|---|
| 8 | Auth & identity | Register, login, refresh rotation, password recovery, age gate | Backend leads; Client wires screens |
| 9 | Onboarding | Interest selection (min-3 hard block), custom suggestion queue | Backend + Android in parallel |
| 10 | For You feed | Keyset pagination, explainability field, block filtering | Backend leads; Client feed screen |
| 11 | Posting + links | Create post, link (BR-04 3-level), category required | Backend leads; Client composer |
| 12 | Engagement | Heart (toggle), bookmark, follow/unfollow | Backend + Android in parallel |
| 13 | Profile & settings | Own posts, display-name toggle, username/password/number change | Backend + Android in parallel |

**Coding steps per slice (what a beginner actually types each week):**

| Week | First (backend, day 1–2) | Then (backend, day 3) | Then (Android, day 4–5) | Last (tests, day 6) |
|---|---|---|---|---|
| 8 Auth | `users` + `sessions` migrations; Argon2id hash; JWT issue/verify | Refresh rotation + recovery + rate limits | Login/register screens + token store (DataStore) + interceptor | 4 security tests named; ACs from FR-002/004/005 |
| 9 Onboarding | Seed 60–80 categories; `user_categories`; picker requires ≥3 (FR-007); suggestion queue (FR-008) | Rate limit + email-unverified guard | Multi-select picker; "blocked until 3" UX | Interests-required → 428 test |
| 10 Feed ⭐ | Write the SQL in `psql` by hand **first**; port to SQLAlchemy repo; cursor encode/decode; matched category; blocks both ways | Keyset pagination (no OFFSET); batched heart/bookmark check | `LazyColumn`, pull-to-refresh, load-more; empty state | Cursor round-trip test; no-dups; no-own-posts |
| 11 Posting | `posts` migration; body 1–2000 + category required; depth-3 service check; `ON DELETE SET NULL` | Anonymous flag from setting; rate limit 10/hr | Composer with category picker + "replying to" preview | Self-link and depth-4 rejection tests |
| 12 Engagement | Hearts toggle w/ transactional counter; bookmarks; follows w/ self-check | Reconciliation test (count from rows == counter) | Optimistic heart toggle + rollback; follow button states | Concurrency: same heart twice → one row |
| 13 Profile | Own posts query; username 409 generic; password change revokes sessions | Number change; display-name toggle | Settings screens; own-posts screen | Ownership tests (can't read/edit others') |

**Per-slice rule (newbie-safe):** before writing ORM code for a slice, run its core query in `psql` by hand and make it return correct rows. That habit removes ~70% of feed/posting bugs.

**Per-slice DoD (must all be true before the slice is "done"):**
- Runs on a clean machine.
- ≥1 test for the core logic; 90%+ on domain layer.
- Four security tests present and passing where data is touched.
- Reviewed by the other person via PR.
- Documented (README updated if contract changed).
- Merged to `main` and deployed to staging.

**Weekly cadence (every week, non-negotiable):**
- Sunday 30-min sync: plan the week, pick the slice.
- Saturday: ship and write the "Week N" note (shipped / bugs / blocked / next).
- One "no new topics" day per week to consolidate.

**Gate 6 (end of Week 13):** All 6 MVP slices pass DoD; CI green; 20+ tests; end-to-end demo by a non-author passes.

---

### Phase 7 — Hardening (Week 14)

**Goal:** remove the risk of a public launch embarrassing the team.

| Task | Owner |
|---|---|
| Concurrency tests on feed (idempotency, cursor stability) | Backend |
| Load smoke test on the For You query at 10k posts | Backend |
| Accessibility pass on client (screen-readers, tap targets, contrast) | Android |
| Reconciliation job for denormalised counts (BR-08) | Backend |
| Pen-test checklist pass (OWASP Top 10 against API) | Backend |
| Privacy review: PII fields never returned, soft-delete / erasure path tested | Both |

**Gate 7:** No unresolved high-severity security finding; feed query p95 documented and acceptable; erasure path tested.

---

### Phase 8 — Release readiness (Week 15)

**Goal:** a stranger could deploy, run, and understand the project.

| Task | Owner |
|---|---|
| README with setup, architecture diagram, API overview | Both |
| Deployment runbook (deploy, migrate, rollback, on-call triage) | Backend |
| Threat model + ADRs reviewed for accuracy vs. final code | Both |
| Pre-flight checklist (MY_PLAN §11) — every box ticked or explicitly waived | Both |
| "Won't have" list re-verified still cut | Both |

**Gate 8:** Pre-flight checklist 100% or every miss logged with a reason.

---

### Phase 9 — Deploy & launch (Week 16)

**Goal:** real URL, real (invited) users, real feedback.

| Task | Owner |
|---|---|
| Deploy backend + client to chosen hosting | Backend |
| Tag release `v1.0.0`; write changelog | Both |
| Invite beta cohort (30–50 users, then +5/week) | Both |
| Seed content: 200–500 posts across top 20 categories from founding accounts | Both |
| Monitoring live: logs, metrics, error tracking, empty-feed-rate alert | Backend |

**Gate 9:** A real person (not either author) completes the loop — register → pick interests → see a post → write a linked post — unaided, with no developer in the room.

---

### Phase 10 — Stabilisation & feedback (Weeks 17–20)

**Goal:** fix what reality shows you, not what you predicted.

- Weekly bug triage from real-user reports; P0 fixes ship within 24h, others weekly.
- Iterate UX from observed behaviour (empty states, onboarding friction).
- Re-run the survival test on any proposed new feature — if it's not core, it stays cut.
- Monthly checkpoint: answer the four checkpoint questions honestly (roadmap). If #1 or #2 is "no", delay v1.1 and fix.

**Gate 10:** Empty-feed rate below target for new users; zero P0 (auth/data-loss) open bugs for 7 consecutive days.

---

### Phase 11 — v1.1 increment (Weeks 21–24)

**Goal:** ship the first deferred feature by demand, not by assumption.

Candidate pool (pick *one*, ranked by real-user evidence, not guesses):
- Thread view polish / deeper thread navigation.
- Report/block/admin review tooling (if abuse appears).
- Profile picture upload (only if §10 privacy review passes).
- Search (still often cut — measure before adding).

Apply the full Phase 3–8 pipeline again for the chosen feature. Never hot-patch a feature that skips design.

---

### Phase 12 — Maintenance & retirement decision (Ongoing)

| Cadence | Task |
|---|---|
| Daily | On-call triage of error tracker; SLO burn-rate watch |
| Weekly | Dependency updates; security scans in CI |
| Monthly | Dependency-audit review; DB migration dry-run on restore; checkpoint questions |
| Quarterly | Threat-model refresh; dependency-adoption review (any new infra must pass the "two people cannot operate distributed systems" test) |
| On event | **Maintenance → feature work freeze** if reliability degrades |

**Retirement:** A product is sunset when it no longer delivers the core loop for real users or when maintenance exceeds feature value. Decide explicitly: keep evolving, maintenance-only, or sunset + data export + read-only archive. No "we forgot about it" endpoints.

---

## 3.5 Beginner coding breakdown (milestone-by-milestone, "what to type")

This is the same vertical slices as Phase 6, written for someone who has **never shipped a production
endpoint**. Each milestone has a goal, what to learn, exact first steps, and how to know it works.

### Milestone A — Auth (Week 8): the first production endpoint you'll ever write
**Learn (just-in-time):** Argon2id hashing, JWT access vs refresh, why we store the *hash* of a refresh token, HTTP 401/403/409/422.
**Type, in this order:**
1. Migration for `users` (id, email citext unique, username citext unique, password_hash, birthday, phone_e164, is_anonymous_default, created_at, deleted_at).
2. `password_hash = hash(password)` with Argon2id. **Never store plaintext.**
3. `register`: validate with Pydantic `extra="forbid"`, age gate, breached-password check. Response must contain **no** password_hash/birthday/phone.
4. `login`: generic error identical for wrong password and unknown account (no enumeration).
5. Issue JWT (15 min) + refresh token; store refresh **hash** in `sessions`.
6. Refresh rotation; reuse → revoke session.
7. Recovery: code, 5-wrong invalidation, revoke *all* sessions on success.
**Tests before code exists:** write the 4 security tests as failing stubs, then make them pass.
**Done when:** `curl` register/login/refresh works; refresh twice with same token → 401 the second time.

### Milestone B — Onboarding (Week 9): the screen the whole product depends on
**Type:** `categories` table + seed of 60–80; `user_categories`; picker endpoint writing **all selected rows before 200**; custom-suggestion queue with `pending` status; hard block below 3 interests (FR-007).  
**Done when:** a user with 0 interests gets `428 interests_required` on the feed, not an empty list.

### Milestone C — For You feed (Week 10): the hardest query
**Type:** (1) run the SQL from spec §3.7 **in `psql` by hand first**; (2) port to SQLAlchemy repo method; (3) cursor encode/decode with tamper → 400; (4) `matched_category` in response; (5) block filtering both directions; (6) batched heart/bookmark check (one round trip for 20 ids, never N+1).
**Tests:** For You only returns interest categories; a post appears once; no own posts; soft-deleted excluded; cursor round-trips with no gaps/dups.
**If the feed is slow:** run `EXPLAIN` before optimising. It is almost always the missing composite index `(category_id, created_at DESC, id DESC)`.

### Milestone D — Posting + links (Week 11): the defining feature
**Type:** `posts` table; body CHECK 1–2000; category_id NOT NULL; parent_post_id nullable with `CHECK (parent_post_id <> id)`; depth ≤3 in service (walk the chain); `ON DELETE SET NULL` so a deleted parent leaves children as roots; anonymous flag copied from user setting at creation.
**Tests:** self-link → 422; depth 4 → 422; deleted parent → child becomes root.

### Milestone E — Engagement (Week 12): transactions
**Type:** hearts toggle (insert incl. `heart_count++`, delete incl. counter--) **in one transaction**; `UNIQUE`/PK prevents double; bookmarks; follows with self-check.
**Add one audit test:** recompute counters from base tables and assert equality. This catches the drift that silently rots feeds.

### Milestone F — Profile & settings (Week 13): polish + security
**Type:** own posts query; username change (409 collision, generic): password change (revokes other sessions); number change; display-name toggle.  
**Done when:** someone else's token can't read or modify your profile; mass-assignment of `id`/`owner_id` is ignored.

---

### If you're stuck (professional triage order)
1. Reproduce in 5 lines with no framework (`python -c`/pytest). Logic bug? Fix here.  
2. DB? Run SQL by hand; check indexes; `EXPLAIN`.  
3. Interface? Open API contract — 90% of "broken" is a shape mismatch.  
4. Config/env? Base URL, ports, `.env`. Typos before re-architectures.  
5. Read the matching guide section. Only that section.  
6. Stop after 45 min. Write what you tried; ask partner / open issue.

### Daily routine (newbie-sustainable)
| Block | Time | Do |
|---|---|---|
| 1 | 60–90 min | Build one unit of the current slice (endpoint / screen / test). |
| 2 | 45–90 min | Make it run end-to-end; hit it with curl / emulator. |
| 3 | 15 min | Tests for what you just built. |
| 4 | 15 min | Commit + 3-line write-up. |

### What "done learning to code" looks like here
Not mastery. Competence = taking one FR, implementing backend-to-client, writing its tests, deploying it — in a week, alone, with tutorials closed for parts you already know. That's Week 13's bar.

---

## 4. Gantt (compact)

```text
Week : 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17..20 21..24  Ongoing
Phase: 0--1--2--3-----4--5-----6----------------7--8--9----10-----11-----12-----
```

Numbers after a gate failure slip the whole row right. Do not parallelise by pretending a failed gate passed.

---

## 5. Governance & change control

1. **Scope change after coding starts:** update this doc + `docs/01` product spec + decision log entry. Both agree. No exceptions.
2. **Breaking API change:** version bump (`/api/v2`) or negotiation with Android. Never silently change a shipped shape.
3. **Risk register:** any new risk discovered in build gets Part-12 status (id, impact, trigger, mitigation, owner) and is reviewed at the next sync.
4. **Decision log:** every significant decision dated with reasoning and rejected alternatives (MY_PLAN §12).

---

## 6. Metrics (the few numbers each phase cares about)

| Phase | Metric | Target |
|---|---|---|
| 5 (foundation) | CI time on main | < 5 minutes |
| 6 (build) | Tests on domain layer | ≥ 90% |
| 6 (build) | Slices shipped on time | 6/6 |
| 7 (hardening) | High-severity security findings open | 0 |
| 9 (launch) | First-session completed loop by a real user | ≥ 1 |
| 10 (stabilise) | Empty-feed rate for new users | < defined threshold |
| 10 (stabilise) | Open P0 bugs for 7 consecutive days | 0 |
| Ongoing | Mean time to recovery (MTTR) from outage | Documented + improving |

---

## 7. Templates to use at each phase

| Phase | Template |
|---|---|
| 2 | `TEMPLATES/01` T01 (charter), T12 (README skeleton) |
| 3 | `TEMPLATES/01` T03 (use cases), T04 (data model), T06 (ADR), T07 (API spec) |
| 4 | `TEMPLATES/01` T08 (threat model), T09 (test plan) |
| 5 | `TEMPLATES/02` (code shape), T16 (deployment runbook) |
| 8 | `TEMPLATES/01` T12 (README), T11 (risk register) |
| 9 | `TEMPLATES/01` T14 (changelog), T15 (post-mortem) |

---

## 8. Tracking sheet (copy into MY_PLAN/03 weekly)

```markdown
## Week N — YYYY-MM-DD
- Phase: ___
- Slice: ___
- Shipped:
- Bugs found and fixed:
- Blocked by:
- Gate status (pass/fail + reason):
- Next week:
```

---

*Anchoring note: replace "Week 0" with your actual kickoff date. If the two of you can only realistically free 15–20 hours/week, the durations above already assume that. If exam season hits, freeze at the current phase boundary (never freeze mid-slice) and resume after — MY_PLAN/03 covers this.*
