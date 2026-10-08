# Reading Order

**The sequence for working through this repository.**

**Total: 13 files, ~16,200 lines. You do not read them in filename order.**

---

## The one rule that overrides this document

> **If you are stuck on something, stop reading and open the matching guide.**

This list is for **sequencing**, not **completion**. You will never read all 16,000 lines, and that is fine — `GUIDES/` is reference material, not a syllabus.

**Never read a guide about something you are about to do.** Read it when you are already doing the thing and hit a wall. That is when it sticks.

---

# Phase 0 — Before you write any code

**Duration: ~2 hours**

**This is the only phase you read cover to cover.**

| # | File | Read | Why this position |
|---|---|---|---|
| 1 | [`README.md`](README.md) | 10 min | The map. Know what exists |
| 2 | [`GUIDES/01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md`](GUIDES/01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md) → **Part 0 only** | 20 min | The 25 vocabulary terms. Everything else assumes them |
| 3 | Same file → **Part 1** (The Lifecycle) | 20 min | Where you actually are right now |
| 4 | Same file → **Part 2** (Planning + MVP by subtraction) | 30 min | Turns a vague idea into 4–6 features |
| 5 | [`MY_PLAN/01_MY_6_MONTH_ROADMAP.md`](MY_PLAN/01_MY_6_MONTH_ROADMAP.md) | 20 min | Why the app comes first, and how it fits the six months |

## Gate for Phase 0

**Do not proceed until you can do both of these:**

- [ ] State the app in one sentence
- [ ] List its 4–6 MVP features

---

# Phase 1 — Scope the project

**Duration: ~2–3 hours · Do this sitting together with your partner**

| # | File | Read | Produce |
|---|---|---|---|
| 6 | [`MY_PLAN/02_MY_APP_PROJECT_SPEC.md`](MY_PLAN/02_MY_APP_PROJECT_SPEC.md) → §5–6 | 30 min | The ERD + MVP feature list |
| 7 | [`TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md`](TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md) → **T01** (Project Charter) | 10 min | One-page charter |
| 8 | Same file → **T04** (Data Model) | 20 min | Final ERD |
| 9 | Same file → **T03** (Use Case Spec) | 30 min | 3–5 use case specifications |
| 10 | Same file → **T06** (ADR) | 20 min | 3 decision records: framework, database, monolith-vs-microservices |

## Gate for Phase 1

- [ ] Both people agree on the same 4–6 features
- [ ] Both people agree on the same data model
- [ ] Both people can name the same core loop

**This phase is the artifact that prevents the interface argument in week 3.**

---

# Phase 2 — Design

**Duration: ~3–4 hours · Together**

| # | File | Read | Produce |
|---|---|---|---|
| 11 | [`MY_PLAN/02_MY_APP_PROJECT_SPEC.md`](MY_PLAN/02_MY_APP_PROJECT_SPEC.md) → §1–4, 7–9 | 60 min | **The API contract — the most important document in the repository** |
| 12 | [`GUIDES/01`](GUIDES/01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md) → **Part 3** (Requirements & Analysis) | 45 min | Understanding of *why* the documents are shaped that way |
| 13 | Same file → **Part 4** (Design) | 45 min | Same, for architecture |
| 14 | [`GUIDES/06_ARCHITECTURE_AND_SYSTEM_DESIGN.md`](GUIDES/06_ARCHITECTURE_AND_SYSTEM_DESIGN.md) → §1–4 only | 45 min | Layering, port-and-adapter, why a monolith and not microservices |
| 15 | [`TEMPLATES/01`](TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md) → **T07** (API Specification) | 20 min | Expands §4 into the full contract |

## Gate for Phase 2

- [ ] Your partner can describe **every endpoint** without opening the code
- [ ] Neither of you has an open question about request or response shapes
- [ ] You have agreed on the Definition of Done (`MY_PLAN/02` §2)

---

# Phase 3 — Build the backend

**Reference. Read on demand, not in order.**

| # | File | Read | When |
|---|---|---|---|
| 16 | [`TEMPLATES/02_PYTHON_CODE_PATTERNS.md`](TEMPLATES/02_PYTHON_CODE_PATTERNS.md) | 90 min | **Read fully before writing backend code.** Copy its structure |
| 17 | [`TEMPLATES/01`](TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md) → **T08** (Threat Model) | 30 min | **Before the first endpoint** |
| 18 | [`GUIDES/04_TESTING_AND_QA_GUIDE.md`](GUIDES/04_TESTING_AND_QA_GUIDE.md) | Reference | While writing the first test |
| 19 | [`GUIDES/05_DEVOPS_CI_CD_DEPLOYMENT.md`](GUIDES/05_DEVOPS_CI_CD_DEPLOYMENT.md) → §1–5 | Reference | When setting up CI and Docker |
| 20 | [`GUIDES/03_SECURITY_ENGINEERING_GUIDE.md`](GUIDES/03_SECURITY_ENGINEERING_GUIDE.md) | Reference | While implementing authentication |

---

# Phase 4 — Ship

**Reference.**

| # | File | Read | When |
|---|---|---|---|
| 21 | [`TEMPLATES/01`](TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md) → **T12** (README) | 20 min | Before anyone else sees the project |
| 22 | [`GUIDES/05_DEVOPS_CI_CD_DEPLOYMENT.md`](GUIDES/05_DEVOPS_CI_CD_DEPLOYMENT.md) → §6–13 | Reference | Deploying, migrations, monitoring |
| 23 | [`MY_PLAN/02_MY_APP_PROJECT_SPEC.md`](MY_PLAN/02_MY_APP_PROJECT_SPEC.md) → §11 (Pre-flight checklist) | 10 min | Before calling the MVP done |

---

# Ongoing — across the six months

| # | File | When |
|---|---|---|
| 24 | [`GUIDES/02_ALGORITHMS_AND_DATA_STRUCTURES.md`](GUIDES/02_ALGORITHMS_AND_DATA_STRUCTURES.md) | Month 2 — read the section for that week |
| 25 | [`GUIDES/03_SECURITY_ENGINEERING_GUIDE.md`](GUIDES/03_SECURITY_ENGINEERING_GUIDE.md) | Month 4 — read in full |
| 26 | [`GUIDES/01`](GUIDES/01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md) → **Parts 5–16** | Month 3+ — methodologies, testing, Git, CI/CD, maintenance |
| 27 | [`MY_PLAN/03_WEEKLY_TRACKER.md`](MY_PLAN/03_WEEKLY_TRACKER.md) | Every Sunday |
| 28 | [`MY_PLAN/04_INTERNSHIP_PLAYBOOK.md`](MY_PLAN/04_INTERNSHIP_PLAYBOOK.md) | Week 20 — apply |

---

# Quick reference card

```
PHASE 0  Read      →  one sentence + 4-6 features
PHASE 1  Together →  charter, ERD, use cases, ADRs
PHASE 2  Together →  API contract           ← most important artifact
PHASE 3  Build    →  patterns + threat model, reference as needed
PHASE 4  Ship     →  README, deploy, pre-flight checklist
ONGOING  6 months  →  roadmap, tracker, internship playbook
```

---

# If you only do one thing today

Give me **one sentence describing your app idea** — unpolished is fine.

MVP-by-subtraction takes 30 minutes and produces the ERD and API contract, which turns [`MY_PLAN/02_MY_APP_PROJECT_SPEC.md`](MY_PLAN/02_MY_APP_PROJECT_SPEC.md) from a template into a document you and your partner can both code against.

**That single step moves you from Phase 0 to Phase 2.**

---

# Files you do not need to open yet

To avoid the temptation to read everything:

| File | When |
|---|---|
| `GUIDES/02_ALGORITHMS_AND_DATA_STRUCTURES.md` | Month 2 |
| `GUIDES/04_TESTING_AND_QA_GUIDE.md` | When you write your first test |
| `GUIDES/05_DEVOPS_CI_CD_DEPLOYMENT.md` | When you set up CI |
| `MY_PLAN/03_WEEKLY_TRACKER.md` | After your first week |
| `MY_PLAN/04_INTERNSHIP_PLAYBOOK.md` | Week 20 |
