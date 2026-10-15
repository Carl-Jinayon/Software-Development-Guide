# My 6-Month Internship Roadmap

**This document is personal. It is not a reference.** It is my specific plan, and it is the only thing here I should be executing.

---

## My situation

| | |
|---|---|
| **Programme** | Computer Science undergraduate |
| **Current** | 3rd year, 1st semester |
| **Internship starts** | In 6 months |
| **Hours available** | 2–4 per day, 6 days per week (~15–20 hrs/week, ~450 hours total) |
| **Primary target** | **Full-stack development** |
| **Secondary target** | **Cybersecurity** |
| **Tertiary** | Machine learning / AI (harder to break into, so lower priority) |
| **Main language** | Python |
| **Side project** | Flutter client app, 2 people, may become the thesis |
| **Hardware** | No Mac → **iOS is off the table** |
| **Current state** | Has written small school projects. No production experience. Counts self as a beginner |

## My strategic position

**The AppSec niche is my angle.** A full-stack developer who can also secure what they build is more hireable than either a generic full-stack developer or a generic security person. That combination is a real, in-demand niche, and my target ranking puts me in it naturally rather than as a compromise.

## Hour allocation — deliberately unequal

| Area | Share | Why |
|---|---|---|
| **Python + backend engineering** | **~55%** | This is what gets me hired. Full-stack is the target |
| **Architecture, testing, DevOps** | **~20%** | The differentiator between junior and mid-level |
| **Cybersecurity** | **~15%** | Second target, and my differentiator |
| **ML / AI** | **~10%** | Enough vocabulary and one project. Not a priority |

**The reason this is unequal:** full-stack development has a much larger hiring surface than ML/AI engineering, which needs a graduate degree or a strong research record to compete for. Spreading myself across three equally would leave me employable in none.

## The learning structure — and why

**Not** a different topic every day. **Not** one topic per month with nothing else. The structure that works:

| Block | Duration | What |
|---|---|---|
| **Spine deep block** | 60–90 min | The month's focus topic. Learn AND write code in it |
| **Build block** | 45–90 min | Work on the month's project. This is where learning becomes real |
| **Maintenance slot** | 15–30 min | Rotate my other subjects. Keeps them alive without stealing depth |
| **Write-up** | 15 min | What did I learn, what broke, commit |

**The maintenance slot is the key trick.** It is how I keep three subjects warm while deepening one. Capped at 30 minutes — if it grows, I have accidentally started a second spine.

**Why daily topic rotation fails:** no depth, constant context switching, nothing compounds.
**Why monthly-only fails:** I forget Month 1 by Month 4, and I never build anything that fuses the topics.

## The fusion rule — my differentiator

**Never study SWE, cyber, and ML separately. Fuse them into single projects.**

| Weak (three shallow projects) | Strong (one project, three disciplines) |
|---|---|
| A plain CRUD app | **Auth system with anomaly detection** — sessions, DB, rate limiting (SWE) + threat model, MFA, injection defence (cyber) + model flags impossible login patterns (ML) |
| A security tool | **Log anomaly detector** — data pipeline, storage, API (SWE) + attack signatures, log integrity (cyber) + anomaly model vs. threshold baseline (ML) |
| An ML notebook | **Phishing-email classifier** — FastAPI serving the model, Docker, CI (SWE) + adversarial inputs, evasion analysis (cyber) + NLP model, precision/recall trade-offs (ML) |

**One of these done properly beats three of anything else.** In an interview it reads as: *this person can apply engineering judgement, not follow a tutorial.*

## Non-negotiable daily rules

1. **Never start a day by reading about something.** Start by writing something.
2. **Every day ends with a commit.** Even five lines. The trail is the evidence.
3. **One day a week is a "no new topics" day.** Consolidate, refactor, fix.
4. **Never skip two days in a row.** Momentum beats intensity.
5. **The maintenance slot never exceeds 30 minutes.**

---

# Month 1 — Python depth + how computers actually work

**Goal: write Python confidently, use Git properly, understand how the internet works.**

| Week | Focus | Deliverable |
|---|---|---|
| 1 | Types, control flow, functions, lists, dicts, comprehensions. 30+ exercises | GitHub repo, 30+ commits of Python exercises |
| 2 | OOP, dataclasses, exceptions, modules, file I/O, type hints. Build a CLI tool | A small terminal program with a README |
| 3 | Networking (DNS, TCP/IP, HTTP, TLS), how an OS works, how a database works | Written notes in my own words + a diagram of the request lifecycle |
| 4 | Git: branching, merging, conflict resolution, rebase, revert. GitHub profile. First PR to an open-source project | Clean history; a profile README |

**Reference:** `../GUIDES/01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md` Part 15 · `../TEMPLATES/02_PYTHON_CODE_PATTERNS.md`

### Setup (do this first)

```bash
# Install uv — dramatically faster than pip
curl -LsSf https://astral.sh/uv/install.sh | sh

# GitHub account + SSH key + profile
git config --global user.name "My Name"
git config --global user.email "me@example.com"
git config --global init.defaultBranch main
```

### Month 1 milestone

> I can write a 200-line Python program, explain what an HTTP request is, and use Git without panicking.

---

# Month 2 — Data structures, algorithms, Linux

**Goal: implement the core algorithms myself; be comfortable on a Linux terminal.**

| Week | Focus | Deliverable |
|---|---|---|
| 5 | Arrays, hash tables, stacks, queues. Implement and test my own | `algorithms/` package with tests |
| 6 | Binary search (including `lower_bound`), merge sort, quick sort. **Benchmark them against each other** | A benchmark script with a written comparison |
| 7 | BFS, DFS, Dijkstra, greedy basics. Apply to a real graph problem | A pathfinding project (e.g. routing a campus map) |
| 8 | Linux seriously: filesystem, permissions, processes, SSH, cron, `grep`/`find`/`awk`, systemd, Docker basics | A script automating a real task I actually do |

**Reference:** `../GUIDES/02_ALGORITHMS_AND_DATA_STRUCTURES.md`

**Critical instruction: implement from memory, then verify against the standard library.**

```python
def test_my_binary_search_matches_bisect():
    import bisect
    data = list(range(1000))
    for target in [0, 500, 999, 1000]:
        mine = binary_search(data, target)
        theirs = bisect.bisect_left(data, target)
        assert (mine == -1) == (theirs == 1000)
        if mine != -1:
            assert mine == theirs
```

### Month 2 milestone

> I can implement and explain binary search, BFS, and Dijkstra from memory, and navigate a Linux server without a tutorial.

---

# Month 3 — Full-stack backend. **The important month.**

**Goal: build a complete backend project the professional way. This is the project that gets me hired.**

| Week | Focus | Deliverable |
|---|---|---|
| 9 | FastAPI: routes, Pydantic validation, dependency injection, error handling | A working API with 8+ endpoints |
| 10 | PostgreSQL + SQLAlchemy + Alembic. **Write the ERD before the schema.** | Database with proper schema and migrations |
| 11 | pytest: unit, integration, E2E. Target 90%+ on the domain layer | 20+ meaningful tests |
| 12 | Docker, GitHub Actions CI, deploy, health endpoint, real README | **A live, public URL with a green CI badge** |

**The project: my app's backend.** Whatever the app idea is, build it as a real FastAPI service this month.

### The quality bar — no shortcuts

- [ ] Layered architecture (routes / domain / repositories) with pure, testable business logic
- [ ] Authentication with Argon2id password hashing
- [ ] Ownership-based authorisation on every endpoint
- [ ] Input validation with Pydantic, `extra="forbid"`
- [ ] CI pipeline green (lint → types → tests → security scan)
- [ ] 20+ meaningful tests, including the four core security tests
- [ ] Dockerised
- [ ] **Deployed with a live public URL**
- [ ] README a stranger could follow
- [ ] Threat model written (`../TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md` T08)

**Reference:** `../TEMPLATES/02_PYTHON_CODE_PATTERNS.md` · `../GUIDES/06_ARCHITECTURE_AND_SYSTEM_DESIGN.md`

**This single project, done properly, is worth more than any certificate.** It demonstrates architecture, security, testing, and DevOps simultaneously.

### Month 3 milestone

> I have a deployed, tested, secure, CI-verified API that a stranger can read about and understand.

---

# Month 4 — Cybersecurity, applied to Month 3

**Goal: understand how software breaks, and harden what I already built.**

| Week | Focus | Deliverable |
|---|---|---|
| 13 | Threat modelling (STRIDE), OWASP in practice, TLS, hashing vs. encryption | A threat model of my Month 3 project |
| 14 | Auth patterns: JWT, refresh rotation, RBAC, OAuth 2.0 + PKCE flows | Implemented properly |
| 15 | Python security tooling: `requests`, `socket`, `hashlib`, `argon2`, `secrets`, `pip-audit`. Read code for vulnerabilities | A script that audits a codebase for hardcoded secrets |
| 16 | CTF practice: picoCTF or TryHackMe. Write up one challenge I solved | A CTF write-up |

### Deliverables for this month

- [ ] Rate limiting on login and expensive endpoints
- [ ] MFA or token rotation implemented
- [ ] Threat model document written
- [ ] `pip-audit` in CI, failing the build on high severity
- [ ] Bandit running in CI
- [ ] Mass-assignment protection verified by test
- [ ] One CTF write-up published

**Reference:** `../GUIDES/03_SECURITY_ENGINEERING_GUIDE.md`

**Do not skip this month — it is my differentiator.** This is where I become "full-stack who secures what they build."

---

# Month 5 — Android client + a light ML slice

**Goal: finish the app, and get enough ML to have the vocabulary and one artifact.**

| Week | Focus | Deliverable |
|---|---|---|
| 17 | Flutter: Dart syntax, widgets, state mgmt, connecting to my API with http/Dio + Figma handoff | A working screen reading from my backend |
| 18 | Finish the client: screens, navigation, error states, loading states, offline handling | A usable app |
| 19 | ML: NumPy, pandas, scikit-learn. Train a classifier on a public dataset | A baseline model with proper evaluation |
| 20 | Serve the model through my FastAPI backend. Deploy. **START INTERNSHIP APPLICATIONS.** | A deployed ML endpoint |

**Division of labour with my partner:**
- My half: the backend (Python — directly serves my internship goals) + the ML endpoint
- Their half: the Android client

**Note on iOS:** I have no Mac. iOS is not feasible and not worth pursuing for my target roles anyway. If I want both platforms later, Flutter is the cheap path. Park that decision for 2027.

---

# Month 6 — Capstone, portfolio, interviews

**Goal: one substantial project tying everything together, plus a presentable portfolio.**

| Week | Focus | Deliverable |
|---|---|---|
| 21 | Decide: thesis or portfolio? Write the one-page brief | A project brief |
| 22–23 | Build. Every practice: Git, tests, CI, layers, threat model, Docker | A working deployed system |
| 24 | Documentation, README, architecture diagram, demo video, portfolio, **interviews** | A portfolio a recruiter can evaluate in 3 minutes |

**Apply for the internship in Week 20, not Week 24.** Applications are uncertain and take time.

### The thesis decision — resolve it early

| | Side project | Thesis |
|---|---|---|
| Who decides "done" | Me | The panel |
| Needs a novel contribution | No | **Yes** — that is Chapter 2 |
| Needs formal evaluation | No | **Yes** — Chapter 4 |
| Documentation | A good README | Chapters 1–5 with citations |
| Can I quit | Yes, gracefully | No |

**My strategy: build the side project first with thesis-level discipline, then evaluate in Month 2 whether there is a measurable angle worth writing up.**

```
Month 1–4:  Build it. Working backend, tests, CI, security, ML endpoint.
Week 8:     Honestly assess — is there something here worth measuring?
                │
       ┌────────┴────────┐
       ▼                 ▼
  YES, there is a    NO — but it is a strong
  measurable angle    portfolio project.
       │                 │
       ▼                 ▼
  Convert to thesis.   Keep it. Move on. It is
  Write Ch.2–5 from    stronger than most
  what I built.        student projects.
```

**This finds out in Month 2 whether I have a thesis, not Month 8.** That is the entire value of this order, and it costs nothing — the side project is the work I would do anyway.

**Ask my adviser early** whether a previously built system is acceptable as a thesis basis. That determines whether I keep the option open.

---

# My five portfolio pieces

| # | Project | What it proves | Built in |
|---|---|---|---|
| 1 | **The backend** | Architecture, testing, CI/CD, deployment | Month 3 |
| 2 | **The secured version** | Threat modelling, AppSec, OWASP | Month 4 |
| 3 | **The Android client** | Mobile, API integration, real-world completeness | Month 5 |
| 4 | **The ML endpoint** | Data work, model training, serving, evaluation | Month 5 |
| 5 | **The capstone or thesis system** | Depth in one domain, end-to-end ownership | Month 6 |

### Portfolio hygiene

- [ ] Every repo: README with a screenshot, real setup instructions, licence, CI badge
- [ ] A GitHub profile README that introduces me
- [ ] A `projects.md` or personal site listing all five
- [ ] Consistent commit history across all of it
- [ ] Test suites that pass

---

# Weekly rhythm at 2–4 hrs/day

| Day | Time | Activity |
|---|---|---|
| Mon | 2–4 hrs | **Spine topic** — learn it, then write code using it |
| Tue | 2–4 hrs | Continue spine; start the build block |
| Wed | 2–4 hrs | **Build block** — the month's project |
| Thu | 2–4 hrs | **Build block** |
| Fri | 2 hrs | **Maintenance slot** — rotate cyber / ML / CS reading or small exercises |
| Sat | 4–6 hrs | The week's main deliverable. Ship something |
| Sun | 1–2 hrs | Write-up, refactor, commit, plan the week. **No new topics** |

## The two rules that decide whether this works

1. **Never skip two days in a row.** Momentum matters more than intensity.
2. **Every day ends with a commit.** Even five lines. The trail is my evidence.

## Exams and holidays

- In finals week: drop to 30 minutes a day. **Just commit.** Do not break the streak.
- Reading-only is fine as maintenance, but every day needs at least one written line.

---

# Monthly self-check

At the end of every month, honestly answer these four:

1. **Can I do everything on this month's topic without looking anything up?**
2. **Have I shipped something that runs?**
3. **Is there GitHub evidence of it?**
4. **Could I explain what I built to a non-technical person in 2 minutes?**

**If the answer to #1 or #2 is no, delay the next month and fix it.** Rushing forward accumulates a hollow foundation, and hollow foundations collapse at exactly the moment I need them — during an interview.

---

# What to do the week before the internship

- [ ] Re-read my own README for each project. Be able to explain every line I wrote.
- [ ] Prepare two stories: one about a bug I found and fixed, one about a design decision I made and why.
- [ ] Know my CI pipeline, test suite, and deployment process **cold** — those are what interns get asked about.
- [ ] Write a list of what I do not know yet, and be ready to ask about it. **Knowing what you don't know is a professional skill.**

---

# References — what to read, and when

| Month | Read | From |
|---|---|---|
| 1 | Python official tutorial · *Automate the Boring Stuff* (free) | — |
| 2 | *Grokking Algorithms* · LeetCode Easy tier | `../GUIDES/02_ALGORITHMS_AND_DATA_STRUCTURES.md` |
| 3 | FastAPI docs · *The Pragmatic Programmer* | `../TEMPLATES/02_PYTHON_CODE_PATTERNS.md` |
| 4 | PortSwigger Academy (free) · OWASP Testing Guide | `../GUIDES/03_SECURITY_ENGINEERING_GUIDE.md` |
| 5 | Fast.ai (free) · scikit-learn docs | — |
| 6 | *Designing Data-Intensive Applications* (skim) | `../GUIDES/06_ARCHITECTURE_AND_SYSTEM_DESIGN.md` |

**General:** Clean Code · *Site Reliability Engineering* (free) · Real Python · Dan Luu · The Primeagen (short videos)

---

# The one-sentence version

> **Do not study three subjects. Master one — software engineering with Python — and use it as the instrument for the other two, shipping one real project every week and applying for the internship at the halfway point.**
