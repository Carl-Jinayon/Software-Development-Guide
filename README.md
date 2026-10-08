# Software Engineering Field Guide

A complete, layered reference for learning software engineering properly — plus a personal execution plan.

---

## Start here

**If you know nothing yet, read in this order:**

1. **[`READING_ORDER.md`](READING_ORDER.md)** — the exact sequence, with what to produce at each step. **Start here.**
2. **[`GUIDES/01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md`](GUIDES/01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md)** — Parts 1 through 6. This is the mental model of how software actually gets made. Expect 3–4 hours. Nothing else makes sense until you have it.
3. **[`MY_PLAN/01_MY_6_MONTH_ROADMAP.md`](MY_PLAN/01_MY_6_MONTH_ROADMAP.md)** — your actual to-do list. Week by week for six months.
4. **Everything else** — reference. Open it when you hit the topic.

---

## Structure

Three deliberately separate layers:

```
├── GUIDES/      Universal knowledge. Nothing personal. Reference for anyone.
├── TEMPLATES/   Universal artifacts. Copy, fill in, use.
└── MY_PLAN/     Personal. Only about me. Delete this and nothing breaks.
```

**The separation is the point.** `GUIDES/` is knowledge that does not expire. `MY_PLAN/` is a plan that gets replaced every six months.

---

## GUIDES — universal reference

| Document | Contents |
|---|---|
| **[01 — Software Engineering Field Guide](GUIDES/01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md)** | The master reference. 16 parts covering the entire lifecycle: planning, MVP scoping, requirements, use cases, ERDs, architecture, methodologies, security, development practice, algorithms, testing, Git, CI/CD, deployment, maintenance, retirement. Plus a Python reference and glossary. |
| **[02 — Algorithms and Data Structures](GUIDES/02_ALGORITHMS_AND_DATA_STRUCTURES.md)** | Deep dive. Complexity analysis, Python's real data structure costs, and every core algorithm implemented with tests: searching, sorting, stacks, hashing, recursion, backtracking, linked lists, trees, heaps, graphs (BFS/DFS/Dijkstra/A*/topological sort/MST), greedy, dynamic programming, tries, union-find, randomised algorithms. Plus benchmarking methodology and interview patterns. |
| **[03 — Security Engineering Guide](GUIDES/03_SECURITY_ENGINEERING_GUIDE.md)** | Deep dive. Security mindset, STRIDE threat modelling with a worked example, authentication, session and token management, OAuth 2.0 with PKCE, authorization and BOLA, all OWASP Top 10 with vulnerable and fixed code, hashing vs. encryption, input validation, API security, secrets management, dependency supply chain, file uploads, SSRF, deserialization, security headers, rate limiting, database security, logging and incident response, security testing, privacy compliance. |
| **[04 — Testing and QA Guide](GUIDES/04_TESTING_AND_QA_GUIDE.md)** | Deep dive. The test pyramid, what to test and why, pytest in depth (fixtures, parametrisation, markers), test doubles and when to use fakes over mocks, coverage and why the metric is misleading, integration and E2E, property-based testing with Hypothesis, concurrency testing, load testing, the N+1 detection test, manual and exploratory testing, anti-patterns, CI integration, testing legacy code. |
| **[05 — DevOps, CI/CD, and Deployment](GUIDES/05_DEVOPS_CI_CD_DEPLOYMENT.md)** | Deep dive. DevOps and DORA metrics, CI concepts, GitHub Actions in depth with real workflows, Docker and multi-stage builds, Compose, configuration and the twelve-factor pattern, deployment strategies (blue/green, canary, feature flags), hosting options compared, infrastructure as code, database migrations in production with the expand-and-contract pattern, observability (logs/metrics/traces/error tracking), rollbacks and release notes, secrets in CI, performance and cost. |
| **[06 — Architecture and System Design](GUIDES/06_ARCHITECTURE_AND_SYSTEM_DESIGN.md)** | Deep dive. Core principles, coupling and cohesion, layers and boundaries, clean architecture with the port-and-adapter pattern in Python, monolith vs. microservices and what to build instead, monorepo vs. polyrepo, event-driven architecture, caching and the three hard problems, database patterns (N+1, pagination, indexing), API architecture, scalability, reliability patterns (timeouts, retries with jitter, circuit breakers, idempotency), distributed system problems, the system design interview framework, three worked designs, anti-patterns. |

## TEMPLATES — fill-in artifacts

| Document | Contents |
|---|---|
| **[01 — Project Document Templates](TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md)** | 17 fill-in templates: project charter, requirements specification (with functional, non-functional, and traceability sections), use case specification, ERD spec, data flow diagram spec, architecture decision record, API specification, threat model, test plan, risk register, sprint planning, README, code review checklist, changelog, post-mortem, deployment runbook, ADR log. |
| **[02 — Python Code Patterns](TEMPLATES/02_PYTHON_CODE_PATTERNS.md)** | A production-shaped Python service. Project structure and the dependency rule, pyproject.toml with ruff/mypy/pytest configured, typed settings with validation, application factory with middleware, error handling with a stable contract, structured logging, SQLAlchemy models with constraints and indexes, repositories with ownership in the query, Pydantic schemas, routing, dependency injection, Argon2id and JWT with refresh rotation, Alembic migrations, test fixtures with transaction rollback, Hypothesis property tests, the four core security tests, multi-stage Dockerfile, Compose, type hints, async patterns, decorators, performance patterns, CI workflow. |

## MY_PLAN — personal

| Document | Contents |
|---|---|
| **[01 — My 6-Month Roadmap](MY_PLAN/01_MY_6_MONTH_ROADMAP.md)** | The tailored plan. Strategic position, unequal hour allocation, the spine + maintenance learning structure, the fusion rule, month-by-month weeks with deliverables and milestones, the AppSec thesis-decision strategy, five portfolio pieces, weekly rhythm, monthly checkpoints. |
| **[02 — My App Project Spec](MY_PLAN/02_MY_APP_PROJECT_SPEC.md)** | The two-person project. Division of labour by layer, Git workflow, what to avoid, **the full API contract both sides code against**, ERD, MVP scope by subtraction, architecture, security requirements, shipping cadence, pre-flight checklist, decision log, open questions. |
| **[03 — Weekly Tracker](MY_PLAN/03_WEEKLY_TRACKER.md)** | The file I check every week. Weekly and daily templates, the canonical schedule, the five non-negotiable rules, month checkpoints, how to handle exams and bad weeks, the progress log. |
| **[04 — Internship Playbook](MY_PLAN/04_INTERNSHIP_PLAYBOOK.md)** | Positioning, the application plan and timing, resume structure with example bullets, GitHub profile requirements, LinkedIn, interview preparation by stage, the two stories to prepare, questions to ask, common failure modes, follow-up template, application tracker. |

---

## How to use the whole thing

### Reading

```
GUIDES/01 → parts 1–6        the mental model
MY_PLAN/01 → week 1          what to actually do
GUIDES/*  → on demand        reference
TEMPLATES/* → when building   copy and fill in
```

### Doing

Every day:
1. Open `MY_PLAN/01` — what is this week's spine?
2. Learn the concept, then write code that uses it
3. Write up what you learned
4. Commit

Every week:
1. Open `MY_PLAN/03` — plan the week
2. Ship something that runs
3. Update the tracker

Every month:
1. Answer the four checkpoint questions honestly
2. If #1 or #2 is no, delay the next month and fix it

### Building

1. Copy the relevant template from `TEMPLATES/01` into `docs/`
2. Fill in every section with real values
3. Reference `TEMPLATES/02` for the code shape
4. Commit the documents alongside the code

---

## The three ideas that matter most

**1. Software engineering is not mostly coding.**
About 30–40% of the effort. The rest is deciding what to build, designing where things go, proving it works, and deploying without breaking things.

**2. Testing is not a phase at the end.**
It happens continuously, during development, by the developer. A bug found by your own test at 3pm costs 5 minutes; the same bug found by a user three weeks later costs days.

**3. Knowing the vocabulary is not the same as being able to do it.**
This repository can teach you what MVP scoping, threat modelling, and architectural layering mean. It cannot teach you to write them. The gap closes only by building something, watching it fail, and fixing it.

---

## Maintenance

| Layer | Update policy |
|---|---|
| `GUIDES/` | When something is wrong or a topic needs expanding. Never contains personal content |
| `TEMPLATES/` | When a pattern changes or a template is missing something |
| `MY_PLAN/` | Replaced every six months. It is a plan, not a reference |

**`MY_PLAN/` is disposable.** When the six months are up, delete it and write a new one. The guides and templates persist.
