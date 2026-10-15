# Master Execution Guide — Konek'ta (Complete, Single File)

**Project:** Konek'ta — a relevance-first social feed where you respond to someone's post by writing your own linked post.  
**Stack:** FastAPI + PostgreSQL backend · Flutter (Dart) client (Android first) · Figma for design · GitHub + GitHub Actions.  
**Team:** 2 people, ~15–20 hrs/week each, ~6 months.  
**Audience:** a beginner who has written small school projects and has never shipped production code.

> This is the single document. It merges the timeline, the coding roadmap, and the learning approach into one place. Open it at the start of each week; do what the current phase says; do not jump ahead.

---

## 1. How professionals actually build software (the mental model)

**Software engineering is ~30–40% coding.** The rest is deciding what to build, placing things, proving they work, and shipping without breaking things. That is why this guide spends the first 5 phases before any feature code exists.

Core flow in one line:

> Idea → one-sentence pitch → 4–6 MVP features → scope locked → ERD + ADRs + API contract → threat model + tests planned → foundation (CI green) → vertical slices shipped weekly → harden → release → stabilise on real users → iterate on evidence.

Three non-negotiables:
1. **Test continuously, by the developer.** A bug your own test finds at 3pm costs 5 minutes; the same bug from a user costs days.
2. **Ship weekly.** Even a broken deployed build teaches more than perfect code never released.
3. **Gate before transition.** A phase ends at a gate. If it fails, fix and re-gate; do not pretend and push forward.

---

## 2. How to learn while building (newbie protocol)

**Do not** finish a 40-hour tutorial before coding. Professionals learn by building vertical slices. The ratio: **read 10 minutes → build one running thing → commit → write 3 lines**.

Rules:
1. Read the guide section **only when stuck** on the problem in front of you. If you aren't stuck, close the file.
2. Every day ends with a commit and a 3-line write-up (what broke, what I learned).
3. Every concept earns one test and one commit. If you can't demo it in 5 lines, it isn't learned.
4. Never skip two days in a row. Momentum beats intensity.
5. One "no new topics" day per week to consolidate/refactor.
6. Freeze at a **phase boundary** when exams/life hits — never mid-slice.

**Mental model to hold throughout (memorise it):**
```
Flutter widgets --HTTP/JSON--> FastAPI --SQL--> PostgreSQL
  http/Dio                     routes.py     repositories.py
  state mgmt                   services.py   SQLAlchemy models
                               domain/ (pure Python, no framework)
```
**The single rule that makes testing and learning possible:** `domain/` imports **nothing** from FastAPI or SQLAlchemy. If you can't test it with pytest and no DB, it's in the wrong place. Authorisation belongs **in the query** (`WHERE owner_id = :user_id`), never in a check you "remember".

---

## 3. Roles, ownership, working agreement

**Split by layer, not by feature.** Splitting by feature produces half-implemented, mismatched code.

| My half (backend) | Partner's half (client) |
|---|---|
| FastAPI service | Flutter screens, navigation, state |
| PostgreSQL + migrations | Figma → widgets handoff |
| Business logic (pure, testable) | API integration (http/Dio) |
| Auth, authorisation, rate limiting | Local caching, offline states |
| Tests, CI, deployment, threat model | Error/loading states, Play Store build |

**Workflow rules (agree while you're still friends):** never push to `main`; `feature/x → PR → review → merge`; weekly 30-min sync; GitHub Issues as task list; agree "done" now.

**Definition of Done (all six, or it isn't done):** runs on a clean machine · ≥1 test for core logic · passes CI · reviewed by the other person · documented · merged to `main`.

---

## 4. The phase plan (week by week)

<|message|> Phase | Window | Core deliverable | Gate |
|---|---:|---|---|
| 0 Kickoff | W0 | Repo, tooling, shared vocab | Strangers can clone + run |
| 1 Discovery | W1 | One-sentence pitch, core loop, top risks | Pitch stated without reading |
| 2 Scope | W2 | MVP list (4–6) + defended Won't-have | Both lists identical |
| 3 Design | W3–4 | ERD, use cases, ADRs, **API contract**, Figma flows | Partner describes endpoints from memory |
| 4 Security design | W5 | Threat model, NFRs w/ metrics, test plan | FRs have test cases; security stubs in CI |
| 5 Foundation | W6–7 | CI green, Docker, first migration | Clean-clone runs; health end-to-end |
| 6 MVP build | W8–13 | 6 vertical slices, each shippable | All slices pass DoD; 20+ tests |
| 7 Hardening | W14 | Security, load, accessibility, erasure | No high-sev findings; erasure tested |
| 8 Release readiness | W15 | README, runbook, pre-flight | Checklist 100% or waived w/ reason |
| 9 Deploy & launch | W16 | Live URL, beta invites, seed content, monitoring | A real user completes the loop unaided |
| 10 Stabilise | W17–20 | Fix what reality shows | Empty-feed < target; 0 P0 for 7 days |
| 11 v1.1 | W21–24 | First deferred feature (by evidence) | Full cycle again for that feature |
| 12 Maintenance | Ongoing | Operate, patch, decide sunset | MTTR improving; SLOs met |

Professional principle: **freeze at a phase boundary, never mid-slice**, if exams/life intervene.

---

## 5. Phase details with "what to learn" and "what to type"

### Phase 0 — Kickoff (Week 0)
**Learning:** Git basics, how the repo is laid out, reading the spec.  
**Do:** read `GUIDES/01` Part 0–1; create repo + branch protection + issue labels; `docs/` folder; `.env` gitignored from **commit one**.  
**Gate 0:** a stranger clones and sees a running (empty) thing.

### Phase 1 — Discovery (Week 1)
**Learning:** how a vibe becomes a product.  
**Do:** one-sentence pitch; core loop (read → think → write linked post → right people see → engage back); name target users + explicitly-not-target; top 5 risks with **mitigation + measurable trigger**.  
**Gate 1:** pitch stated without reading; risks have triggers.

### Phase 2 — Scope (Week 2) — the most skipped, most fatal
**Learning:** MVP by subtraction.  
**Method:** write full vision → define core loop → survival test ("if I delete this, can the user still study/come back?") → 60-second test → cut.  
**Do:** MVP list (4–6); written "Won't have" with reasons; backlog tagged `later`; one-page charter.  
**Gate 2:** both answer "what are we building?" and "what are we not?" with identical lists.

### Phase 3 — Design (Weeks 3–4)
**What to learn:** ERD thinking (cascades, indexes), reading an API contract, Figma token handoff.  
**Do (in this order — order matters):**
1. ERD + business rules + indexes (formalise the domain model; cascade rules like `SET NULL` on `parent_post_id`).
2. Use cases (3–5 narratives) → become tests.
3. ADRs (min 3: framework, DB, monolith-vs-microservices, plus app-specific: keyset pagination, admin-only categories, no comment table).
4. **API contract** — the most important document. Endpoints, shapes, status codes, Android-side error handling table.
5. Figma: wireframes → hi-fi flows, screen map, empty/error states, tokens exported into `mobile/lib/theme.dart`.

**Gate 3:** partner describes every endpoint and ERD entity from memory.

### Phase 4 — Security design (Week 5)
**Learning:** STRIDE, NFR metrics, writing acceptance → tests.  
**Do:** threat model (STRIDE, assets, mitigations); NFRs with metrics (latency, rate limits, PII never returned); test plan mapping FR ACs → test cases; Definition of Done; the **4 security tests written as named stubs** already wired into CI.  
**Gate 4:** every FR has ≥1 test; threat model exists; security stubs fail in CI (placeholders).

### Phase 5 — Foundation build (Weeks 6–7)
**What to learn:** Python typing/Pydantic, async basics, FastAPI app factory + DI, Docker Compose, Alembic, Flutter HTTP + state, Figma handoff, Git PRs.

**Build (each its own PR):**
1. `pyproject.toml` (ruff/mypy/pytest) → `pytest` runs (0 tests, green).
2. `app/main.py` as pure factory; `GET /health`.
3. `compose.yaml` (FastAPI + Postgres + healthcheck); `.env.example`; `.env` ignored.
4. Alembic init + first (empty) migration; practise migrate + rollback.
5. Flutter scaffold: one screen calling `/health`, Dio client, base URL from config, loading/empty/error widgets; import design tokens.
6. CI: lint → types → tests → pip-audit/Bandit → `flutter analyze` + tests; green on `main`.

**Gate 5:** clean clone runs both sides; CI green; one health check end-to-end from phone to DB and back.

### Phase 6 — MVP build, 6 vertical slices (Weeks 8–13)
**Rule:** one feature per week, **end-to-end**, not half-layers. Don't start week N+1 until week N's tests pass.

| Week | Slice | Learn | Build order (backend first, then client) | Tests |
|---|---|---|---|---|
| 8 Auth | Argon2id, JWT, refresh rotation, enumeration safety | `users`+`sessions` migration; register (no secrets in response); login (generic error); refresh single-use + reuse revoke; recovery revokes all sessions; rate limits | 4 security tests; FR-002/004/005 ACs |
| 9 Onboarding | FK + seeds + transaction wrap | seed 60–80 categories; `user_categories`; picker requires ≥3; suggestion queue pending | interests-required → 428 |
| 10 For You feed ⭐ | SQL joins, keyset pagination, N+1 avoidance | **Write SQL in psql by hand first**; port to repo; cursor; matched category; block both ways; batched heart checks | only interest categories; once; no own posts; soft-delete excluded; cursor round-trip |
| 11 Posting+links | self-FK, depth check, constraints | body 1–2000 + category required; depth ≤3 in service; `ON DELETE SET NULL`; anonymous flag | self-link & depth 4 → 422; deleted parent → child root |
| 12 Engagement | transactions, upsert/toggle, idempotency | heart toggle w/ counter transactional; bookmarks; follows w/ self-check; reconciliation audit test | double heart → one row |
| 13 Profile/settings | ownership in queries | own posts; username 409 generic; password change revokes sessions; anon toggle | can't read/edit others; mass-assignment ignored |

**Daily at 10 min / 2–4 hrs:**
1. Backend one unit (endpoint/query/service).
2. Make it run end-to-end (curl/emulator).
3. Tests for what you built.
4. Commit + 3-line note.

**Per-slice DoD:** runs on clean machine · ≥1 core test · 90%+ domain · 4 security tests where data touched · reviewed · documented · merged to `main` and staging.

**Gate 6:** all 6 slices pass DoD; 20+ tests; a non-author completes the loop.

### Phase 7 — Hardening (Week 14)
**Learning:** concurrency, load, accessibility, erasure, OWASP.  
**Do:** concurrency/idempotency tests on feed; load smoke on For You at 10k posts; accessibility pass; reconciliation job for denorm counts; OWASP pass; privacy/erasure test.  
**Gate 7:** no unresolved high-sev finding; p95 documented; erasure tested.

### Phase 8 — Release readiness (Week 15)
**Do:** README + architecture diagram + API overview; deployment runbook (deploy/migrate/rollback/triage); threat model + ADRs reviewed vs final code; pre-flight checklist (all or waived w/ reason); re-verify Won't-have still cut.  
**Gate 8:** pre-flight 100% or logged misses.

### Phase 9 — Deploy & launch (Week 16)
**Do:** deploy (PaaS or Compose); tag `v1.0.0` + changelog; invite beta (30–50 then +5/week); seed 200–500 posts across top 20 categories from founding accounts; monitoring live (logs, metrics, error tracking, empty-feed-rate alert).  
**Gate 9 (real):** a person not on the team completes register → interests → read → write linked post **unaided, no developer present**.

### Phases 10–12 — Stabilise, v1.1, maintenance
- **10 (W17–20):** weekly bug triage (P0 within 24h), iterate UX from observed behaviour, survival-test every new feature proposal, monthly checkpoint (if #1/#2 is "no", delay v1.1).
- **11 (W21–24):** pick **one** deferred feature by real-user evidence; run the whole pipeline again for it (no skipping design because "it's small").
- **12 (ongoing):** daily on-call triage; weekly dependency/security scans; monthly audit + DB restore drill; quarterly threat-model refresh; explicit sunset decision (keep evolving / maintenance-only / sunset + export).

---

## 6. Figma design track (runs through Weeks 3–9)

Professionals **draw before they type**:

- Week 3: low-fi wireframes (boxes + labels) → validate flows with partner.
- Week 4: hi-fi only after flow is locked; empty/error/loading states designed **now**, not later.
- Tokens (colors, type scale, spacing) → single `theme.dart` imported by client. **No color literals in code.**
- Handoff: `docs/design/` with exported specs + "how to read this" note.
- Rule: token changes only via Figma + PR.

---

## 7. Quality bar (what "good" means here)

- **Tests:** pyramid. Unit tests on pure domain (microseconds, no DB). Repository integration tests against real Postgres. E2E smoke for the core loop.
- **Security:** Argon2id; JWT 15-min + refresh rotation w/ reuse detection; ownership **in the query**; parameterised SQL only; Pydantic `extra="forbid"`; rate limits; generic errors; HTTPS; pip-audit + Bandit in CI; `.env` gitignored; **threat model before the first endpoint**.
- **The 4 security tests** on every data endpoint: can't read another user's data; can't modify another's; mass assignment can't escalate; responses contain no secrets.
- **Performance:** keyset pagination (never OFFSET); composite index equality-first/range-last; no N+1 (batch per-page lookups); counts denormalised with a reconciliation audit.
- **Process:** CI < 5 min; domain coverage ≥ 90%; PR review every change; monthly restore drill.

---

## 8. Daily/weekly rhythms

**Daily (~2–4 hrs):** block 1 build one unit · block 2 run end-to-end · block 3 tests · block 4 commit + 3-line note.  
**Weekly (Sunday 30-min sync):** pick the slice; Monday ship plan; Saturday ship + "Week N" note (shipped/bugs/blocked/next).  
**Monthly:** answer the 4 checkpoint questions honestly; if #1 or #2 is "no", delay the next month and fix fundamentals.

---

## 9. If you get stuck — professional triage order

1. Reproduce in 5 lines with no framework (pytest / `python -c` / Flutter test). Logic bug? Fix here.
2. DB issue? Run SQL by hand; check indexes; `EXPLAIN`.
3. Interface mismatch? Open API contract — 90% of "broken" is a shape mismatch.
4. Config/env? Base URL, ports, `.env` typos.
5. Read only the matching guide section.
6. Stop after 45 min. Write what you tried; open an issue / ask partner. No spiralling.

---

## 10. Metrics that matter

| Phase | Metric | Target |
|---|---|---|
| 5 foundation | CI time | < 5 min |
| 6 build | domain coverage | ≥ 90% |
| 6 build | slices shipped | 6/6 |
| 7 harden | high-sev security findings open | 0 |
| 9 launch | real user completes the loop unaided | ≥ 1 |
| 10 stabilise | empty-feed rate (new users) | < threshold |
| 10 stabilise | open P0 bugs ≥ 7 days | 0 |
| ongoing | MTTR | documented + improving |

---

## 11. Artifact map (what to copy when)

- Charter / scope / timeline → `TEMPLATES/01` T01, T11.
- Requirements with IDs (FR/NFR) → T02.
- Use cases → T03; ERD → T04; DFD → T05; ADR → T06.
- API contract → T07; threat model → T08; test plan → T09.
- Sprint plan → T10; README → T12; review checklist → T13; changelog → T14; post-mortem → T15; runbook → T16; ADR log → T17.
- Backend code shape → `TEMPLATES/02`.
- Slicing/retirement/unlock → `MY_PLAN/03`.

---

## 12. Glossary (just enough)

- **MVP:** smallest thing delivering the core loop.
- **FR/NFR:** functional requirement / non-functional requirement (numbered, testable).
- **ADR:** architecture decision record (what/why/rejected).
- **Core loop:** the one action a user repeats that makes them return.
- **Vertical slice:** one feature, complete backend→client→tests, shippable.
- **Keyset pagination:** cursor on `(created_at, id)`, constant cost at every page.
- **N+1:** querying per row inside a loop; fix by batching.
- **Pseudonymous:** posting under a handle, no legal identity stored.
- **Gate:** measurable condition that must pass before the next phase starts.

---

## 13. Closing truth

The hardest part of this project is not coding. It is deciding what **not** to build, agreeing on it in writing with your partner before either of you types, and refusing to reopen the scope at 2am. Everything in this file is scaffolding for that one habit: write it down, agree, then code. If a stranger can read your repo and reproduce the product's promises and its known risks, you have already done professional work.
