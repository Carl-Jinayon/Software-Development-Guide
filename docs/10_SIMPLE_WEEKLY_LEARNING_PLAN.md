# Simple Weekly Plan — Konek'ta (Plain English, What You'll Learn & Build)

**Who this is for:** a beginner. Each week has three lines: **what you'll learn**, **what you'll build**, **why it matters**. If a word is new, the glossary at the bottom translates it.

> Professional idea behind this: professionals never "just code." Every week is either deciding what to build, deciding how it's structured, or proving it works — *then* typing.

---

## Week 0 — "Can this even run?"
- **Learn:** Python types, `uv` (package manager), Docker (containers), Git, Flutter basics.
- **Build:** health endpoint `/health` returning `{"status":"ok"}`, database container, Flutter app showing that status.
- **Why it matters:** proves your tools are wired together before any real feature. Half of all week-1 failures are tooling, not code.

## Week 1 — "What is this product?"
- **Learn:** pitching, scoping, MVP (minimum viable product).
- **Build:** one-sentence pitch, core loop, a list of 4–6 features you will ship, a defended "won't do" list, top 5 risks with mitigations + triggers.
- **Why it matters:** writing down *not* doing is your shield against scope creep later.

## Week 2 — "What must it do?"
- **Learn:** requirements thinking (Given/When/Then acceptance criteria).
- **Build:** numbered functional requirements (FRs) for auth, feed, posting; non-functional requirements with real numbers (e.g. "API answers in <300 ms"); open questions list.
- **Why it matters:** requirements become your tests. Vague requirements → vague (buggy) code.

## Week 3 — "How is the data organized?"
- **Learn:** ERD (entities = tables), indexes, cascades (what happens when you delete), ADRs (decision notes).
- **Build:** data model for users/categories/posts/interests, cascade rules (delete parent → children survive), indexes for the feed query, 3+ decision notes (why FastAPI, why Postgres, why monolith).
- **Why it matters:** the database schema is the most expensive thing to change later — get it right now.

## Week 4 — "How do the two halves talk + how does it look?"
- **Learn:** API contract (endpoints/shapes/status codes/errors), Figma basics (wireframes → hi-fi, tokens).
- **Build:** written API contract for auth, categories, posts, feed, engagement; Figma screens for login, interests picker, feed, composer, profile.
- **Why it matters:** mismatched frontend/backend assumptions are the #1 two-person project killer. Contract + design first prevents them.

## Week 5 — "What could go wrong + make the repo real"
- **Learn:** threat modeling (STRIDE), non-functional metrics, Define Done, testing plans.
- **Build:** threat model, Definition of Done checklist, red (failing) security tests, project skeleton with CI green (lint/type/tests/security scan).
- **Why it matters:** security is designed in now, not bolted on after launch. CI becomes your safety net for the next 11 weeks.

## Week 6 — "Prove the plumbing works"
- **Learn:** migrations (changing the database), transactions, PRs.
- **Build:** database migration up/down, `/health` end-to-end through Docker, Flutter app calling it, first PR reviewed and merged.
- **Why it matters:** the thinnest full slice; validates your tooling, your workflow, and your docs in one week.

## Week 7 — "Add login (Flutter + FastAPI)"
- **Learn:** Argon2id (password hashing), JWT (tokens), refresh rotation (security of sessions), enumeration safety.
- **Build:** register, login, refresh, password recovery, rate limits; 4 security tests; Flutter auth screens.
- **Why it matters:** auth is the front door; every later feature inherits it. Done wrong here = account takeover risk.

## Week 8 — "Onboarding: pick your interests"
- **Learn:** foreign keys, seed data, transactional writes.
- **Build:** categories table (60–80 topics), interest picker hard-blocked at ≥3 selections, custom-suggestion queue, `interests_required` for empty interest, Flutter picker.
- **Why it matters:** interests drive the whole feed. A user with zero interests sees an empty feed and leaves forever — your #1 product risk.

## Week 9 — "For You feed (the hardest query)"
- **Learn:** SQL joins, subqueries, keyset pagination (cursor), N+1.
- **Build:** For You query with interests filter, dedup, block filtering (both directions), matched-category explanation, cursor; Flutter infinite scroll.
- **Why it matters:** the feed *is* the product. OFFSET or per-row queries will collapse at scale.

## Week 10 — "Posting + linking (the defining feature)"
- **Learn:** self-references in DBs, depth checks, constraints, cascade semantics.
- **Build:** create post (body 1–2000, category required), link to a parent with max 3 levels, composer in Flutter.
- **Why it matters:** linked posts replace comments. The 3-level rule prevents infinite chains and denial-of-service.

## Week 11 — "Hearts, bookmarks, follows"
- **Learn:** transactions, idempotency (same button press twice = same result), reconciliation (audit counters).
- **Build:** heart toggle with transactional counter, bookmarks, follow/unfollow, Following feed.
- **Why it matters:** these are the arrows that close the core loop — people engage.

## Week 12 — "Profile & settings"
- **Learn:** ownership checks in queries, session revocation, cross-user tests.
- **Build:** own posts, username/password/number changes, anonymity toggle, security tests for "can't edit others."
- **Why it matters:** account hygiene + the security tests that actually prove you're not leaking data.

## Week 13 — "Fix and polish"
- **Learn:** refactoring, test coverage, contract sync.
- **Build:** fix deferred bugs, reach 90% coverage on domain logic, sync code with API contract, first staging ship.
- **Why it matters:** fix debt before hardening; never harden a broken foundation.

## Week 14 — "Harden"
- **Learn:** concurrency testing, load testing, accessibility, OWASP (top 10), erasure path.
- **Build:** feed under load (10k posts), a11y pass on Flutter, OWASP pass, account-erasure test.
- **Why it matters:** prevents a public launch from becoming a public embarrassment.

## Week 15 — "Release ready"
- **Learn:** README, runbook, monitoring, seed content.
- **Build:** README a stranger can follow, deployment runbook, pre-flight checklist, 200–500 seed posts, monitoring with empty-feed alert.
- **Why it matters:** someone else (or you in 6 months) can run it; you'll see the moment feeds go empty.

## Week 16 — "Deploy"
- **Learn:** deploy, migrate, tag a release, invite users.
- **Build:** live URL, `v1.0.0` tag, changelog, first 30–50 beta users, monitor.
- **Why it matters:** the real gate — an **outside user** completing the loop with no developer in the room.

## Weeks 17–24 — "Stabilise + v1.1"
- **Learn:** triage, shipping cadence, iterating on real data.
- **Build:** weekly bug fixes, UX from observed friction, one deferred feature chosen by evidence.
- **Why it matters:** reality > predictions; the survival test for every new feature.

---

## Glossary (plain English)

- **MVP** — minimum viable product = smallest feature set that still makes the core loop runnable.
- **Scope / "won't have" list** — features you deliberately cut, with reasons (your shield).
- **Requirement / AC** — a numbered, testable promise ("Given X, when Y, then Z"). AC = acceptance criteria.
- **ERD** — entity-relationship diagram; which data tables exist and how they connect.
- **ADR** — architecture decision record; what you chose, why, and what you rejected.
- **API contract** — written agreement between backend and mobile on endpoints, shapes, codes, errors.
- **Endpoint** — one URL+method, e.g. `POST /api/v1/auth/login`.
- **Migration** — a script changing the database schema; always test up *and* down.
- **Index** — speed-up for a query; column order matters (equality first, range last).
- **Keyset pagination** — uses a cursor on `(created_at, id)`; cheap on page 50. (Avoids OFFSET, which scans and discards rows.)
- **JWT / refresh token** — short-lived access token + rotating, stored-as-hash refresh token.
- **Argon2id** — password hashing that's slow on purpose (never SHA-256).
- **Enumeration** — leaking "that email is registered" via error messages (an attack).
- **Rate limit** — caps attempts per minute/hour.
- **Threat model (STRIDE)** — list of ways the system can be attacked + mitigations.
- **CI / CD** — auto-checks on push / auto-deploy.
- **PR** — pull request = propose a change for review before merging.
- **Branch protection** — no direct push to `main`; PR required.
- **Definition of Done** — 6 checkboxes; if any missing, it's not done.
- **Vertical slice** — one feature, backend + mobile + tests, shippable.
- **Commit** — a saved snapshot; you commit daily.
- **Soft delete** — mark `deleted_at` instead of removing (erasure windows).
- **Pseudonymous** — handle only; no legal identity stored.
- **Core loop** — the one thing users repeat.
- **Staging** — a deployed copy for testing before production.
- **Pre-flight checklist** — final gate before calling MVP done.
- **Empty-feed rate** — % of new users seeing zero posts; Konek'ta's key risk metric.

---

## If you're lost, the shortest path

Open `docs/09_ZERO_TO_LAUNCH_DAYPLAN.md`, find the line for this week, read its **Purpose (why)**, do only that, ship one tiny thing today, and commit. That's it.  
If you still feel lost after 45 min, re-read the `docs/07_BEGINNER_TO_PRODUCTION_COMPLETE.md` section for the current milestone and nothing else.  
The docs you'll reference by name: `12_` no — actually `docs/01_KONEK_TA_PRODUCT_SPEC.md` (requirements), `docs/03_PROJECT_LIFECYCLE_TIMELINE.md` (weeks/gates), and `docs/07_BEGINNER_TO_PRODUCTION_COMPLETE.md` (technical how-to). Everything else is on-demand reference.  
Tomorrow, do the next one. That's the whole method.
Whatever task says **learn**: read for ≤20 min with the explicit job of answering "what will I build with this?"  
Whatever task says **build**: write the smallest unit that runs.  
Whatever says **why**: restate it in your own words before typing.

**Tomorrow**, Day 1.4 onward? No — pick the lowest-numbered unfinished day and ship it. Good luck. 😊 (No, but seriously, commit daily; that trail is the evidence.)
Whatever task says **study**: read **≤20 minutes with a question in mind**. Whatever says **build**: write the smallest running unit. Whatever says **why**: restate it in your own words before touching code. Pick the **lowest unfinished day**, lean on the term glossary when a new word appears, ship something today, commit. That's the whole method — applied 60–90 minutes at a time, for 16 weeks. Good luck. 😊 (No irony: small, daily, named wins compound faster than big sprints that never ship.)
<|end|><|start|>startup<|message|>analysis<|message|>I need to continue from the user's last message. They said "can you makes the plan easier to understand? like for example this week we are going to learn blah blah. this week is learning figma for design. this week we will learn to add login on flutter". 

I already created a simple weekly plan file (`10_SIMPLE_WEEKLY_LEARNING_PLAN.md`) in my previous response. However, I notice my previous response seems to have gotten cut off or appended redundant text (the ending has some odd duplicated stream-of-consciousness text like "Whatever task says study..."). That's messy. Looking back at the turn, it seems my output got cut off after the glossary? Actually, looking at the raw output, it appears I wrote a very long message that included the simple plan and then some appendages. The last part says "Whatever task says study... Pick the lowest unfinished day..." This is likely my own "thinking" or generation artifact that got rendered. I should make sure the current file is clean and correct.

Actually, I need to check what state things are in. The user is now waiting for a reply. I should provide a clean, concise answer. I already created `docs/10_SIMPLE_WEEKLY_LEARNING_PLAN.md` with simple weekly plan. But wait—did I actually create that file in the previous turn? Let me review. In my previous turn I gave a long response with headers, and it looks like I generated content but did I call `write`? Looking at the output, it seems I just returned text in the final answer channel (not a tool call). Actually wait, looking back at the conversation flow:

The user asked: "can you makes the plan easier to understand? like for example this week we are going to learn blah blah. this week is learning figma for design. this week we will learn to add login on flutter"

My last assistant response was a freeform markdown answer starting with "# Simple Weekly Plan". There was no tool call to write a file—so the content I generated was just returned in the chat, not saved to disk.

Given that, I should now offer to save it as a clean file, or just provide a clean, easy-to-read plan. But also, the previous output seems to have some trailing garbage ("Tests... picking..." etc.). I'll just give a clean summary / create the file now and keep it tidy.

Let me create the file cleanly and point to it, as that's what the user has been wanting. I'll write to `10_SIMPLE_WEEKLY_LEARNING_PLAN.md`.

Actually, re-reading: the user wants something even simpler/higher level than lists of tasks. They want "this week we learn X" style. I have that content. I'll write a clean version to a file and keep this response short.

Let me write the file using the write tool with clean content.<|end|><|start|><|start|>assistant to=write<|channel|>analysis <|constrain|>xml<|message|><rfl:parameter name="content">
# Simple Weekly Learning Plan — Konek'ta

Plain English version. Each week: what you'll learn, what you'll build, why it matters.

---

## Week 0 — "Can it even run?"
- **Learn:** Python, `uv` (package manager), Docker (containers), Git basics, Flutter basics.
- **Build:** a health endpoint, a database container, a Flutter app showing "ok".
- **Why:** prove your tools work before touching real features. Half of week-1 failures are tooling, not code.

## Week 1 — "What is this product?"
- **Learn:** pitching, scoping, MVP (minimum viable product).
- **Build:** one-sentence pitch, core loop, 4–6 MVP features, "won't do" list, top 5 risks.
- **Why:** writing down what NOT to build is your shield against scope creep.

## Week 2 — "What must it do?"
- **Learn:** requirements (Given/When/Then acceptance criteria).
- **Build:** numbered requirements for auth/feed/posting; non-functional requirements with real numbers (e.g. "API < 300 ms").
- **Why:** vague requirements become vague (buggy) code. Requirements become test cases.

## Week 3 — "How is the data organized?"
- **Learn:** ERD (tables and relationships), indexes, cascades (delete rules), ADRs (decision notes).
- **Build:** data model, cascade rules (children survive parent deletion), indexes for the feed query, decision notes (why FastAPI, why Postgres, why monolith).
- **Why:** the database schema is the most expensive thing to change later.

## Week 4 — "How do backend and mobile talk + how does it look?"
- **Learn:** API contract (endpoints/shapes/status codes/errors), Figma (wireframes → hi-fi → tokens).
- **Build:** API contract for auth, categories, posts, feed, engagement; Figma screens for login, picker, feed, composer, profile.
- **Why:** mismatched frontend/backend assumptions are the #1 two-person project killer.

## Week 5 — "What could go wrong + make the repo real"
- **Learn:** threat modeling (STRIDE), NFR metrics, Definition of Done, test planning.
- **Build:** threat model, DoD checklist, red (failing) security tests, project skeleton with CI green.
- **Why:** security designed in now, not bolted on later. CI is your safety net for ~11 weeks.

## Week 6 — "Prove the plumbing works"
- **Learn:** migrations (DB changes), transactions, PRs.
- **Build:** migration up + down, `/health` end-to-end through Docker, Flutter calling it, first PR reviewed/merged.
- **Why:** smallest full slice; validates tooling, workflow, and docs at once.

## Week 7 — "Add login (Flutter + FastAPI)"
- **Learn:** Argon2id (password hashing), JWT (tokens), refresh rotation (session security), enumeration safety.
- **Build:** register, login, refresh, recovery, rate limits + 4 security tests + Flutter auth screens.
- **Why:** auth is the front door. Every later feature inherits it. Wrong = account takeover risk.

## Week 8 — "Onboarding: pick your interests"
- **Learn:** foreign keys, seed data, transactional writes.
- **Build:** categories table (60–80 topics), interests picker hard-blocked at ≥3, custom-suggestion queue, feed returns "interests required", Flutter picker.
- **Why:** interests drive the whole feed. Zero interests = empty feed = lost user (your #1 product risk).

## Week 9 — "For You feed (the hardest query)"
- **Learn:** SQL joins, subqueries, keyset pagination (cursor), N+1.
- **Build:** For You query with interest filter, dedup, block filtering both directions, `matched_category` explanation, cursor; Flutter infinite scroll.
- **Why:** the feed *is* the product. OFFSET or per-row queries collapse at scale.

## Week 10 — "Posting + linking (the defining feature)"
- **Learn:** DB self-references, depth checks, constraints, cascade semantics.
- **Build:** create post (body 1–2000, category required), link to parent max 3 levels, composer in Flutter.
- **Why:** linked posts replace comments. The 3-level rule prevents infinite chains.

## Week 11 — "Hearts, bookmarks, follows"
- **Learn:** transactions, idempotency (same press twice = same result), reconciliation (audit counters).
- **Build:** heart toggle with transactional counter, bookmarks, follow/unfollow, Following feed.
- **Why:** these close the core loop — people engage.

## Week 12 — "Profile & settings"
- **Learn:** ownership checks in queries, session revocation, cross-user tests.
- **Build:** own posts, username/password/number changes, anonymity toggle, security tests for "can't edit others."
- **Why:** account hygiene + the tests that actually prove no data leaks.

## Week 13 — "Fix and polish"
- **Learn:** refactoring, test coverage, contract sync.
- **Build:** fix deferred bugs, 90% domain coverage, sync code ↔ contract, staging ship.
- **Why:** fix debt before hardening; never harden a broken foundation.

## Week 14 — "Harden"
- **Learn:** concurrency testing, load testing, accessibility, OWASP top 10, erasure path.
- **Build:** feed under load (10k posts), a11y pass, OWASP pass, account-erasure test.
- **Why:** prevents a public launch from becoming a public embarrassment.

## Week 15 — "Release ready"
- **Learn:** README, runbook, monitoring, seed content.
- **Build:** README a stranger can follow, deployment runbook, pre-flight checklist, 200–500 seed posts, monitoring with empty-feed alert.
- **Why:** someone else can run it; you'll see the exact moment feeds go empty.

## Week 16 — "Deploy"
- **Learn:** deploy, migrate, tag a release, invite users.
- **Build:** live URL, `v1.0.0` tag, changelog, first 30–50 beta users, monitoring.
- **Why:** the real gate — an outside user completing the loop with no developer in the room.

## Weeks 17–24 — "Stabilise + v1.1"
- **Learn:** triage, shipping cadence, iterating on real data.
- **Build:** weekly bug fixes, UX from observed friction, one deferred feature chosen by evidence.
- **Why:** reality > predictions; the only valid test is real users.

---

## Glossary (plain English)

- **MVP** — minimum viable product = smallest feature set that still makes the core loop runnable.
- **Scope / "won't have" list** — features deliberately cut, with reasons.
- **Requirement / AC** — a numbered, testable promise written as Given/When/Then.
- **ERD** — entity-relationship diagram; which tables exist and how they connect.
- **ADR** — architecture decision record: what you chose, why, and what you rejected.
- **API contract** — written agreement on endpoints, shapes, status codes, errors.
- **Endpoint** — one URL + method, e.g. `POST /api/v1/auth/login`.
- **Migration** — a script changing the DB schema; always test up and down.
- **Index** — speed-up for a query; column order matters (equality first, range last).
- **Keyset pagination** — cursor on `(created_at, id)`; cheap on page 50 (instead of OFFSET, which scans and discards rows).
- **JWT / refresh token** — short access token + rotating, stored-as-hash refresh token.
- **Argon2id** — password hashing that's intentionally slow (never SHA-256).
- **Enumeration** — leaking "that email is registered" via error messages (an attack).
- **Rate limit** — caps attempts per minute/hour.
- **Threat model (STRIDE)** — list of ways the system can be attacked + mitigations.
- **CI / CD** — auto-checks on push / auto-deploy.
- **PR** — pull request; propose a change for review before merging.
- **Branch protection** — no direct push to `main`; PR required.
- **Definition of Done** — 6 checkboxes; if any missing, it's not done.
- **Vertical slice** — one feature, backend + mobile + tests, shippable.
- **Commit** — a saved snapshot; you commit daily.
- **Soft delete** — mark `deleted_at` instead of removing (erasure windows).
- **Pseudonymous** — handle only; no legal identity stored.
- **Core loop** — the one thing users repeat.
- **Staging** — a deployed copy for testing before production.
- **Pre-flight checklist** — final gate before calling MVP done.
- **Empty-feed rate** — % of new users seeing zero posts; Konek'ta's key risk metric.

---

## If you're lost tomorrow, do this

1. Open `docs/09_ZERO_TO_LAUNCH_DAYPLAN.md`.
2. Find the lowest-numbered unfinished day.
3. Read its **Purpose (why)**.
4. Do only that task, verify it runs, and commit.
5. Repeat.

That's the whole method — daily, named, shippable wins. Nothing more.
