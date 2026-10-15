# Zero-to-Launch Day Plan — Konek'ta (Study + Build, with Explanations)

**How to use:** each day is one focused session (60–120 min). Do tasks in order.  
**New rule added per your request:** every task now states **what it is for** ("Why") so you never do work whose purpose you can't name.  
**If you can't state the "Why" in one sentence before starting, re-read it — do not code.**

> Professional rule: a day is a "ship day" only if something runs on your machine at the end of it. A task without a named purpose is a task that will be redone.

---

## Part 0 — How to read this plan

Each day has five lines:

- **Learn (≤20 min):** one narrow topic (input).
- **Build (60–90 min):** one unit of code (output).
- **Purpose (why):** the professional reason this unit exists — link it to a product promise, a risk, or a Definition-of-Done clause.
- **Verify (5–10 min):** a concrete check.
- **Commit:** message + 3-line note.

**If Verify fails after 45 min:** stop, open the matching guide section for that topic only, then retry once. Do not add new features to escape a stuck bug.

---

## Part 0.5 — The three questions that drive every task

Before any code, the project must answer these. If they are missing, **stop**; that is why scope and risk are added back below.

1. **What is the core loop?** (the single action a user repeats that makes them return)
2. **What is the MVP?** (the smallest feature set that makes the loop runnable) + a **defended "Won't have" list**.
3. **What can sink this project?** (top risks with mitigation + measurable trigger).

Every "Why" in this plan references one of these three or a phase gate (G0–G9) from `docs/03`.

---

## WEEK 0 — Tooling only (no app code yet)

### Day 0.1 — Python + uv [Backend/DevOps]
- Learn: Python typing, venv, what a package manager does.
- Build: install `uv`; `uv init`; print `"hello"`; add one typed function.
- **Purpose (why):** Python + types are the spine of the whole backend; every later fix costs more without them. This is foundation work, not optional polish.
- Verify: `python main.py` runs clean.
- Commit: `chore(setup): python + uv`.

### Day 0.2 — Docker Compose basics [Backend/DevOps]
- Learn: service, image, volume, port (≤10 min video, then stop).
- Build: `compose.yaml` with postgres only; `docker compose up`; connect with `psql`.
- **Purpose (why):** production parity from day 1; "works on my machine" dies here. Also a rehearsal of disaster recovery (volumes can be destroyed and rebuilt).
- Verify: `\dt` runs (empty table list is fine).
- Commit: `chore(infra): postgres via compose`.

### Day 0.3 — FastAPI hello [Planning/Docs]
- Learn: path, function, decorator, async (≤10 min).
- Build: FastAPI app with `GET /health → {"status":"ok"}`; run `uvicorn`.
- **Purpose (why):** this is the thinnest possible end-to-end slice (HTTP in, JSON out). Everything later builds on this route shape and on the app factory pattern (logic never lives in `main.py`).
- Verify: `curl http://localhost:8000/health`.
- Commit: `feat(api): health endpoint`.

### Day 0.4 — Compose app + db together [Backend/DevOps]
- Learn: depends_on, env files.
- Build: add FastAPI service to `compose.yaml` using an env file; healthcheck from host.
- **Purpose (why):** reproduces the single most common week-1 failure (API up but DB unreachable) in a controlled way, and establishes the env pattern (`.env` gitignored) that protects secrets.
- Verify: `docker compose up` → `/health` reachable from host.
- Commit: `chore(infra): api + db compose`.

### Day 0.5 — Flutter scaffold [Flutter]
- Learn: widget tree, `StatelessWidget`, `runApp` (≤15 min).
- Build: `flutter create mobile --platforms=android`; one screen with a button showing "Hello".
- **Purpose (why):** the client must exist from day 1 so contracts are tested against a real consumer, not assumed. Prevents "backend looks fine, client can't parse it" discoveries late.
- Verify: emulator opens; button works.
- Commit: `feat(mobile): scaffold`.

### Day 0.6 — Flutter calls the backend [Flutter]
- Learn: `http.get`, `Uri.parse`, `FutureBuilder` (≤15 min).
- Build: one screen calls `/health`, shows status or mapped error.
- **Purpose (why):** the first proof that the two halves integrate over HTTP with a base URL and error mapping — the exact seam that later hosts auth, feeds, and posts.
- Verify: emulator displays `{"status":"ok"}`.
- Commit: `feat(mobile): health probe`.

### Day 0.7 — Git + CI skeleton [Backend/DevOps]
- Learn: PR, review, merge (≤10 min).
- Build: branch protection on `main`; GitHub Actions running lint/type/test/security (green with 0 tests — that is intentional).
- **Purpose (why):** the pipeline must exist before code depends on it; "CI is a gate, not a report." Also enforces the no-direct-push rule while you are still friends.
- Verify: open a dummy PR; all checks green.
- Commit: `ci: skeleton`.

---

## WEEK 1 — Scope and risk (what, why, what can kill it)

### Day 1.1 — One-sentence pitch + personas [Planning/Docs]
- Learn: why vague ideas fail (≤10 min).
- Build: pitch + 3 user classes + explicitly-not-target table.
- **Purpose (why):** the pitch is the filter for every future feature request; the not-target list gives you a documented "no" (e.g. "no ads, no brands") to cite in week 7.
- Verify: read pitch aloud in 15 s without jargon.
- Commit: `docs: vision`.

### Day 1.2 — Core loop [Planning/Docs]
- Learn: the single action users repeat that makes them return.
- Build: write the loop in 5 steps; map each step to a feature and a requirement ID.
- **Purpose (why):** if a feature doesn't advance an arrow in the loop, it is deferred by default. This is how you justify cuts.
- Verify: deleting a non-loop feature doesn't break the loop.

### Day 1.3 — Full vision (unfiltered) [Planning/Docs]
- Learn: why you list everything first (avoids premature optimisation of scope).
- Build: 15+ features, no filtering.
- **Purpose (why):** completeness now prevents "we forgot export!" arguments in month 4; deferred items still get tagged rather than lost.
- Verify: at least 15 items.

### Day 1.4 — Survival test (cut) [Testing]
- Learn: MVP by subtraction (≤15 min).
- Build: apply "if deleted, could the user still complete the loop and come back tomorrow?" to each item; keep only "no".
- **Purpose (why):** scope discipline is the #1 reason two-person projects survive. Cutting here is cheaper than cutting in code.
- Verify: 4–6 features remain; ≥10 cut.

### Day 1.5 — Won't-have list with reasons [Planning/Docs]
- Learn: how to defend a cut.
- Build: table of cut items + reason + evidence that would un-cut.
- **Purpose (why):** the shield you cite in week 3 when someone (possibly you) wants to sneak scope back in. "Won't have" is as important as "Will have."
- Verify: partner reads and agrees without new items.

### Day 1.6 — Risk scan (top risks + mitigations + triggers) [Planning/Docs]
- Learn: risk = impact × likelihood; a risk without a trigger is just worry.
- Build: ≥5 risks for Konek'ta with measurable triggers (e.g. empty-feed rate %, p95 query ms, failed-login threshold). Include mitigation and owner.
- **Purpose (why):** these are the project-killers you cannot code your way out of after they happen (content cold-start, taxonomy fragmentation, reveal abuse). Mitigations are designed now, cheaply.
- Verify: each risk has mitigation + trigger + owner.
- Commit: `docs: risks (mitigations + triggers)`.

### Day 1.7 — Project charter [Planning/Docs]
- Learn: scope + roles + timeline.
- Build: one page: goals, non-goals, owners, success criteria, timeline to MVP.
- **Purpose (why):** the charter is what you quote when arguments arise; "scope creep" loses when the agreed scope is one page.
- Verify: both signed.
- Commit: `docs: charter`.

---

## WEEK 2 — Requirements & user stories

### Day 2.1 — Functional requirements (ACs testable) [Testing]
- Learn: Given/When/Then.
- Build: 5 FRs for auth + onboarding, each AC numbered.
- **Purpose (why):** ACs become test cases later; vague requirements produce vague code. Each AC is a numbered, reviewable promise.
- Verify: each AC answers yes/no.

### Day 2.2 — Feed requirements [Planning/Docs]
- Build: FR-009…FR-014 with ACs.
- **Purpose (why):** the feed is the product; if it's underspecified you'll ship "a list" and call it a feed. Precision now saves rework.

### Day 2.3 — Posting & linking requirements [Planning/Docs]
- Build: FR-015…FR-019 including depth-3 and no-title decisions.
- **Purpose (why):** linking is the defining feature; underspecifying it makes the later BR-04 migration painful.

### Day 2.4 — Engagement/profile requirements [Planning/Docs]
- Build: MVP subset of FR-020…FR-038.
- **Purpose (why):** keeps NFR/PII rules (anonymity display, response shapes) explicit so accidental PII leaks are caught in review.

### Day 2.5 — Non-functional requirements (measurable) [Planning/Docs]
- Learn: NFR must be a number/formula, not an adjective.
- Build: latency (p95), rate limits, availability target, RTO, empty-feed threshold.
- **Purpose (why):** "fast" and "secure" are untestable; "p95 < 300 ms" is testable and reviewable.

### Day 2.6 — Open questions + resolution owners [Planning/Docs]
- Build: list with "resolve by" dates and owners.
- **Purpose (why):** an unanswered open question becomes a 2am disagreement; scheduling the resolution converts it into a task.

### Day 2.7 — Partner review of spec [Planning/Docs]
- Both read product spec parts 2, 4, 7, 8, 12, 13.
- **Purpose (why):** agreed spec = shared contract; disagreement now costs minutes, later costs days.
- Verify: all conflicts logged and resolved.

---

## WEEK 3 — Design (ERD + ADRs)

### Day 3.1 — ERD on paper [Planning/Docs]
- Learn: entities/relationships.
- Build: draw the five core entities.
- **Purpose (why):** the data model is the argument you cannot later win without a migration; settles it on paper first.

### Day 3.2 — Cascade rules [Planning/Docs]
- Learn: CASCADE/RESTRICT/SET NULL.
- Build: cascade table with rationale.
- **Purpose (why):** `SET NULL` on parent links is what makes deletion safe (one deletion can't wipe a thread).

### Day 3.3 — Business rules [Planning/Docs]
- Build: BR-01…BR-08 list with mechanism (DB or app).
- **Purpose (why):** rules enforced in the DB can't be forgotten in application code.

### Day 3.4 — Indexes with justification [Planning/Docs]
- Learn: equality-first/range-last.
- Build: index table + "the query it serves".
- **Purpose (why):** indexes without a query are write overhead; missing indexes become outages.

### Day 3.5 — ERD as code (mermaid) in repo [Planning/Docs]
- Build: commit mermaid ERD.
- **Purpose (why):** versioned design; PR-reviewable changes; renders on GitHub.

### Day 3.6 — 3 core ADRs [Planning/Docs]
- Framework, DB, monolith vs microservices — with rejected alternatives.
- **Purpose (why):** future you (and a thesis reviewer/interviewer) needs the *why*, not just the *what*.

### Day 3.7 — App-specific ADRs [Planning/Docs]
- Depth-3, admin-only categories, no title, no editing, SET NULL.

---

## WEEK 4 — API contract + Figma

### Day 4.1 — Conventions + status codes [Planning/Docs]
- Build: conventions table, status-code table.
- **Purpose (why):** contract consistency reduces client bugs (no ad-hoc shapes).

### Day 4.2 — Auth/account endpoints [Security]
- Shapes for register/login/refresh/recover.
- **Purpose (why):** auth is the front door; every later endpoint depends on its tokens.

### Day 4.3 — Category + interest endpoints [Planning/Docs]
- **Purpose (why):** onboarding is the most-abandoned screen; its endpoint must be reliable.

### Day 4.4 — Post + feed endpoints [Planning/Docs]
- Include `matched_category`, cursor, shapes.
- **Purpose (why):** the feed contract is the product contract; ambiguity here = rebuild.

### Day 4.5 — Engagement endpoints [Planning/Docs]
- Heart toggle, bookmark, follow.
- **Purpose (why):** these are the loop's closing arrows; their idempotency matters.

### Day 4.6 — Android-side error table [Planning/Docs]
- Map 401/403/404/409/422/429/network → UI.
- **Purpose (why):** keeps error UX consistent and reviewable.

### Day 4.7 — Figma low-fi [Figma]
- Learn: wireframes before hi-fi (≤15 min).
- Build: screens for login, picker, feed, composer, profile.
- **Purpose (why):** designing in pixels before code is cheaper than rewriting widgets after; empty/error states planned now.

---

## WEEK 5 — Security design + foundation

### Day 5.1 — Threat model (STRIDE) [Security]
- Build: assets, threats, mitigations + owners.
- **Purpose (why):** security designed in, not bolted on after the breach.

### Day 5.2 — NFRs with numbers [Planning/Docs]
- **Purpose (why):** gives a measurable bar for "good."

### Day 5.3 — Test plan (AC → test cases) [Testing]
- **Purpose (why):** tests prove features; without a plan, "done" is vibes.

### Day 5.4 — Definition of Done [Planning/Docs]
- Write 6-point checklist.
- **Purpose (why):** shared vocabulary for "done" eliminates "it works on my machine" disputes.

### Day 5.5 — 4 security tests as red stubs [Security]
- **Purpose (why):** TDD for security; stubs force the code to satisfy them.

### Day 5.6 — Project skeleton [Backend/DevOps]
- Backend layers + Flutter scaffold with Dio.
- **Purpose (why):** clean architecture is learned by copying the shape, not by inventing one.

### Day 5.7 — CI green on empty project [Backend/DevOps]
- **Purpose (why):** a green baseline means later failures are *yours*, not infra's.

---

## WEEK 6 — Milestone 0 daily (plumbing proof)

- 6.1 Config from env — *why: 12-factor + secrets hygiene.*
- 6.2 `/health` — *why: thinnest slice; pattern for all routes.*
- 6.3 Compose app+db — *why: deploy parity.*
- 6.4 Alembic baseline (up + down) — *why: rollback readiness.*
- 6.5 Flutter health screen — *why: client contract starts Day 1.*
- 6.6 First PR reviewed & merged — *why: workflow becomes habit.*
- 6.7 Outsider runs it — *why: validates docs, not assumptions.*

---

## WEEK 7 — Auth

- 7.1 `users` migration + rollback — *schema-first.*
- 7.2 Argon2id pure function + 3 tests — *no SHA-256; crypto at the edge.*
- 7.3 Register endpoint, no secrets in response — *PII contract.*
- 7.4 Login enumeration-safe + JWT issue — *security-critical path.*
- 7.5 Refresh rotation + reuse revoke — *session security.*
- 7.6 Recovery + rate limits — *account takeover defence.*
- 7.7 Flutter auth screens + end-to-end — *contract validated.*

---

## WEEK 8 — Onboarding

- 8.1 `categories` + seed 60–80 — *filter meaningful only if diverse.*
- 8.2 `user_categories` migration — *relation model.*
- 8.3 interest selection ≥3 hard — *product depends on this.*
- 8.4 custom suggestion queue (pending only) — *taxonomy defence.*
- 8.5 `428 interests_required` — *no silent empty feed.*
- 8.6 Flutter multi-select picker — *UX of the mandatory step.*
- 8.7 register → pick 2 (blocked) → pick 3 → feed — *end-to-end proof.*

---

## WEEK 9 — For You feed (hardest)

- 9.1 `posts` migration.
- 9.2 SQL by hand in psql (must succeed before ORM).
- 9.3 Port to repository; `EXPLAIN` for index use.
- 9.4 add index + re-verify.
- 9.5 cursor encode/decode + tamper rejection.
- 9.6 `matched_category` + dedup + blocks.
- 9.7 Flutter feed + M3 tests.

---

## WEEK 10 — Posting + links

- 10.1 post create (body + category).
- 10.2 depth-3 pure service check.
- 10.3 ON DELETE SET NULL verified.
- 10.4 anonymous flag + rate limit.
- 10.5 Flutter composer.
- 10.6 self-link + depth tests.
- 10.7 linked post visible in feed end-to-end.

---

## WEEK 11 — Engagement

- 11.1 heart toggle transactional.
- 11.2 double-heart + concurrency tests.
- 11.3 reconciliation audit test.
- 11.4 bookmarks.
- 11.5 follows + self-check.
- 11.6 Following feed.
- 11.7 Flutter optimistic states; full loop demo.

---

## WEEK 12 — Profile & settings

- 12.1 own posts feed.
- 12.2 username 409 (generic).
- 12.3 password change + session revoke.
- 12.4 number/bio/anonymity.
- 12.5 cross-user + mass-assignment tests.
- 12.6 Flutter settings.
- 12.7 partner demo (not you).

---

## WEEK 13 — Buffer + hardening start

- 13.1 fix deferred bugs.
- 13.2 raise domain coverage 90%.
- 13.3 sync code ↔ contract.
- 13.4 refactor one ugly module (no behaviour change).
- 13.5 re-run security tests.
- 13.6 first pre-flight pass.
- 13.7 ship to staging + smoke.

---

## WEEK 14 — Hardening

- 14.1 concurrency tests.
- 14.2 load smoke (10k; document p95).
- 14.3 a11y pass.
- 14.4 reconciliation nightly.
- 14.5 OWASP pass.
- 14.6 privacy/erasure test.
- 14.7 fix findings.

---

## WEEK 15 — Release readiness

- 15.1 README + diagram.
- 15.2 runbook.
- 15.3 threat model + ADR review vs code.
- 15.4 pre-flight 100% or waived with reasons.
- 15.5 seed content.
- 15.6 invite list.
- 15.7 monitoring.

---

## WEEK 16 — Deploy

- 16.1 deploy backend + client.
- 16.2 migrate prod (dry run first).
- 16.3 smoke on live.
- 16.4 tag v1.0.0 + changelog.
- 16.5 invite first cohort.
- 16.6 watch errors + empty-feed rate.
- 16.7 real-user gate (outside user unaided).

---

## WEEKS 17–24 — Stabilise + v1.1

Weekly pattern:
- Mon: triage; pick top 3.
- Tue–Thu: fix + ship.
- Fri: deploy + week note.
- Sat: user-observation hour.
- Sun: 30-min sync.

v1.1: one deferred feature, chosen by evidence, full cycle again.

---

## Appendix A — The 12 professional habits

1. Define Done in writing.  
2. Split by layer, not feature.  
3. Threat model before first endpoint.  
4. ERD before migration.  
5. API contract before either side.  
6. Pure domain logic.  
7. Ownership in the query.  
8. Migrations tested both ways.  
9. Tests before "done".  
10. PR review every change.  
11. Ship weekly, even broken.  
12. Decisions logged with reasons.

---

## Appendix B — Daily rhythm (unchanged)

Block 1 build · Block 2 run end-to-end · Block 3 tests · Block 4 commit + 3-line note.  
Tired → do block 4 only. Never skip two days.

---

## Appendix C — Triage when stuck (unchanged)

1. Reproduce in 5 lines.  
2. DB: SQL by hand + `EXPLAIN`.  
3. Interface: open contract.  
4. Config: base URL, ports, `.env`.  
5. Read only matching guide.  
6. Stop after 45 min; write what you tried; ask.

---

## Appendix D — Why this file exists (scope/risk restored)

- **Scope** is defined in Weeks 1.4–1.5 (why: scope drift is the #1 killer of 2-person projects; defended cuts are your shield).
- **Risk** is defined in Week 1.6 (why: project-sinking risks are cheap to mitigate before code and expensive after; triggers let you act before the crash).
- Every day's "Why" ties to one of: a product promise, a risk, a gate (G0–G9), or a Definition-of-Done clause.

**If you only do one thing today:** write the one-sentence pitch + list 4–6 MVP features. That single step is the first domino; everything else depends on it.
