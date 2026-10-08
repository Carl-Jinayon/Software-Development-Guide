# The Software Engineering Field Guide

### A complete reference for the whole software lifecycle

**Scope:** Everything from "a person has an idea" to "software is retired." Language-neutral in principle, with Python used throughout for examples.

**Who this is for:** Students and junior developers who want to understand how software is *actually* built by professionals — not just how to make code compile.

**Companion documents:**

| Document | Contents |
|---|---|
| `02_ALGORITHMS_AND_DATA_STRUCTURES.md` | Deep dive — every core algorithm implemented in Python |
| `03_SECURITY_ENGINEERING_GUIDE.md` | Deep dive — threat modelling, OWASP in depth, secure implementation |
| `04_TESTING_AND_QA_GUIDE.md` | Deep dive — pytest, test strategy, coverage, property-based testing |
| `05_DEVOPS_CI_CD_DEPLOYMENT.md` | Deep dive — pipelines, containers, deployment, observability |
| `06_ARCHITECTURE_AND_SYSTEM_DESIGN.md` | Deep dive — patterns, scalability, reliability, design interviews |
| `../TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md` | Fill-in templates for every document in Part 14 |
| `../TEMPLATES/02_PYTHON_CODE_PATTERNS.md` | Working code patterns for a production-shaped Python service |

---

## Table of Contents

**Part 0 — How to use this document**
- [0.1 Read this first](#01-read-this-first)
- [0.2 The single most important idea in this document](#02-the-single-most-important-idea-in-this-document)
- [0.3 Vocabulary you must know cold](#03-vocabulary-you-must-know-cold)

**Part 1 — The Lifecycle: the big picture**
- [1.1 The real lifecycle](#11-the-real-lifecycle)
- [1.2 Where the effort actually goes](#12-where-the-effort-actually-goes)
- [1.3 The three permanent phases](#13-the-three-permanent-phases)
- [1.4 The structural truth nobody tells students](#14-the-structural-truth-nobody-tells-students)

**Part 2 — Phase 1: Planning**
- [2.1 Product vision](#21-product-vision)
- [2.2 MVP by subtraction](#22-mvp-by-subtraction)
- [2.3 Feasibility analysis](#23-feasibility-analysis)
- [2.4 The risk register](#24-the-risk-register)

**Part 3 — Phase 2: Requirements & Analysis**
- [3.1 Functional vs. non-functional requirements](#31-functional-vs-non-functional-requirements)
- [3.2 The non-functional requirement catalogue](#32-the-non-functional-requirement-catalogue)
- [3.3 MoSCoW prioritisation](#33-moscow-prioritisation)
- [3.4 Use cases](#34-use-cases)
- [3.5 Data Flow Diagrams](#35-data-flow-diagrams-dfd)
- [3.6 Entity-Relationship Diagrams](#36-entity-relationship-diagrams-erd)
- [3.7 Database design and normalisation](#37-database-design-and-normalisation)

**Part 4 — Phase 3: Design**
- [4.1 What is architecture?](#41-what-is-architecture)
- [4.2 The universal client–server shape](#42-the-universal-clientserver-shape)
- [4.3 Architectural patterns](#43-architectural-patterns)
- [4.4 Layered architecture in depth](#44-layered-architecture-in-depth)
- [4.5 Monolith vs. microservices](#45-monolith-vs-microservices)
- [4.6 API design](#46-api-design)
- [4.7 UI/UX: wireframes, mockups, prototypes](#47-uiux-wireframes-mockups-prototypes)
- [4.8 What "architecture and design" means to a hiring manager](#48-what-architecture-and-design-means-to-a-hiring-manager)

**Part 5 — Methodologies: how professionals decide to build**
- [5.1 The core tension](#51-the-core-tension)
- [5.2 Waterfall](#52-waterfall)
- [5.3 Agile](#53-agile)
- [5.4 Scrum](#54-scrum)
- [5.5 Kanban](#55-kanban)
- [5.6 Extreme Programming](#56-extreme-programming-xp)
- [5.7 TDD](#57-tdd-test-driven-development)
- [5.8 The V-Model](#58-the-v-model)
- [5.9 The Spiral Model](#59-the-spiral-model)
- [5.10 DevOps](#510-devops)
- [5.11 Choosing a methodology](#511-choosing-a-methodology)

**Part 6 — Security: designed in, not added on**
- [6.1 The mindset](#61-the-mindset)
- [6.2 Authentication vs. authorisation](#62-authentication-vs-authorisation)
- [6.3 The OWASP Top 10](#63-the-owasp-top-10)
- [6.4 Hashing vs. encryption](#64-hashing-vs-encryption)
- [6.5 STRIDE threat modelling](#65-stride-threat-modelling)
- [6.6 The pre-code security checklist](#66-the-pre-code-security-checklist)
- [6.7 Privacy and compliance](#67-privacy-and-compliance)
- [6.8 Security in Python](#68-security-in-python)

**Part 7 — Phase 4: Development: what actually happens**
- [7.1 A professional's actual day](#71-a-professionals-actual-day)
- [7.2 The ten activities inside development](#72-the-ten-activities-inside-development)
- [7.3 Reading code is the job](#73-reading-code-is-the-job)
- [7.4 Debugging properly](#74-debugging-properly)
- [7.5 Refactoring](#75-refactoring)
- [7.6 Technical debt](#76-technical-debt)
- [7.7 Code review](#77-code-review)

**Part 8 — Algorithms & data structures**
- [8.1 What an algorithm actually is](#81-what-an-algorithm-actually-is)
- [8.2 What an algorithm is not](#82-what-an-algorithm-is-not)
- [8.3 Why algorithms matter](#83-why-algorithms-matter)
- [8.4 Complexity (Big-O) in plain language](#84-complexity-big-o-in-plain-language)
- [8.5 Data structures](#85-data-structures)
- [8.6 The classic algorithm catalogue](#86-the-classic-algorithm-catalogue)
- [8.7 Algorithms inside real software](#87-algorithms-inside-real-software)
- [8.8 Why algorithms are assessed in interviews](#88-why-algorithms-are-assessed-in-interviews)

**Part 9 — Phase 5: Testing** → see `04_TESTING_AND_QA_GUIDE.md`

**Part 10 — Version control and team workflow**

**Part 11 — CI/CD** → see `05_DEVOPS_CI_CD_DEPLOYMENT.md`

**Part 12 — Phase 6: Deployment** → see `05_DEVOPS_CI_CD_DEPLOYMENT.md`

**Part 13 — Phase 7: Maintenance & Phase 8: Retirement**

**Part 14 — The complete document set** → templates in `../TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md`

**Part 15 — Python reference**

**Part 16 — Appendix: glossary and reference**

---

# Part 0 — How to use this document

## 0.1 Read this first

This document is a **reference manual**, not a curriculum. You will not absorb it by reading start to finish once.

**How to actually use it:**

1. **Read Parts 1–6 once, slowly, now.** That is the mental model of how software gets made. Nothing else makes sense until you have it. Expect 3–4 hours.
2. **Then read the deep-dive companions.** Each expands one area of this guide substantially:
   - `02_ALGORITHMS_AND_DATA_STRUCTURES.md` if you want to actually implement these properly
   - `03_SECURITY_ENGINEERING_GUIDE.md` if you are building anything that handles untrusted input
   - `04_TESTING_AND_QA_GUIDE.md` if you want to prove your code works rather than hope it does
   - `05_DEVOPS_CI_CD_DEPLOYMENT.md` if you want to get software in front of users
   - `06_ARCHITECTURE_AND_SYSTEM_DESIGN.md` if you are preparing for design questions
3. **Use the templates as you go.** `../TEMPLATES/01_PROJECT_DOCUMENT_TEMPLATES.md` contains a fill-in version of every document described in Part 14.
4. **Rebuild this in your own words as you go.** Write a one-paragraph summary of each section. If you cannot, you have not understood it.

**A warning:** knowing the vocabulary of software engineering does not mean you can do software engineering. The vocabulary gets you an interview. The practice gets you the job. Nothing in this document substitutes for building something and having it fail.

## 0.2 The single most important idea in this document

> **Almost every professional software skill is a knowledge or judgement problem, not a typing problem.**

A professional's day is roughly:

- Understanding an ambiguous requirement and asking a clarifying question
- Reading code they did not write
- Deciding where a piece of logic belongs
- Writing a small amount of code
- Proving the code works and did not break anything
- Getting a human being to review it
- Deploying it without breaking production
- Writing down why you did what you did

You do not need to be the fastest typist. You need to be the person who knows **what to build, where it goes, how to prove it works, and how to not break things.**

## 0.3 Vocabulary you must know cold

Before anything else, be able to define these without hesitation:

| Term | One-line definition |
|---|---|
| **Requirements** | What the system must do, and how well |
| **Use case** | A specific thing a user does to achieve a goal |
| **Architecture** | The overall structure of the system and why it's structured that way |
| **Design** | Turning architecture into a concrete, detailed specification |
| **API** | The agreed contract for how two pieces of software talk |
| **Backend** | Server-side code: logic, data, security |
| **Frontend** | Client-side code: what the user sees and interacts with |
| **Database** | Structured, persistent storage |
| **Version control** | History of your code (Git) |
| **Commit** | A saved snapshot of your code |
| **Branch** | A parallel line of work |
| **Merge / Pull Request** | Combining branches, usually after review |
| **Test** | Automated proof that code works |
| **Bug** | A defect where behaviour differs from what was intended |
| **Deployment** | Putting your code onto real infrastructure |
| **CI/CD** | Automated building, testing, and deploying |
| **Sprint** | A short fixed cycle of work in Agile |
| **Backlog** | The ordered list of everything that might be built |
| **Refactoring** | Improving code structure without changing behaviour |
| **Tech debt** | Future cost of shortcuts taken now |
| **Algorithm** | A finite, precise, unambiguous sequence of steps solving a problem |
| **Data structure** | A way of organising data for particular operations |
| **Complexity (Big-O)** | How work grows as input grows |
| **Threat model** | Enumerating what could go wrong and defending against it |
| **Environment** | An isolated copy of your system: dev, staging, production |
| **Observability** | The ability to see what your running system is doing |

If you cannot define these, everything in Parts 1–15 will feel like vocabulary memorisation instead of understanding. Learn this table first.

---

# Part 1 — The Lifecycle: the big picture

## 1.1 The real lifecycle

Software does not get built in a straight line. It gets built in a loop.

```
PLAN ──▶ ANALYZE ──▶ DESIGN ──▶ DEVELOP ──▶ TEST ──▶ DEPLOY ──▶ MAINTAIN ──▶ RETIRE
   ▲                                                                          │
   └────────────────────────── feedback loop ─────────────────────────────────┘
```

And in modern practice, the loop tightens dramatically:

```
   ┌──────────────────────────────────────────────────────┐
   │  PLAN → BUILD → TEST → DEPLOY → FEEDBACK → PLAN     │   repeat every 1–4 weeks
   └──────────────────────────────────────────────────────┘
```

**Nobody gets it right the first time.** The lifecycle is not a pipeline you walk through once and finish. It is a cycle you spin through dozens of times, producing something real-but-incomplete earlier and earlier, and correcting course with actual feedback instead of assumptions.

## 1.2 Where the effort actually goes

| Phase | Output | Rough share of total effort |
|---|---|---|
| 1. Planning | Charter, scope, risk register | 5–10% |
| 2. Analysis | Requirements, use cases, ERD, DFD | 10–15% |
| 3. Design | Architecture, API, UI, threat model | 15–20% |
| 4. Development | Working code in version control | **30–40%** |
| 5. Testing | Test suite, bug reports | 20–25% (overlaps with 4) |
| 6. Deployment | Live system | 5–10% |
| 7. Maintenance | Ongoing releases | 20–50% — often the rest of the product's life |
| 8. Retirement | Archive, handover | Rare |

The surprise: **development and testing together are about a third of the work, and maintenance is often the longest phase of a system's entire life.** Google Sheets has been maintained for over a decade. Your first app will be maintained for years.

## 1.3 The three permanent phases

Your instinct was *build → maintain*. The correction:

| Phase | What it actually is |
|---|---|
| **Develop** | Write code. Ship it. |
| **Test** | **Not a separate later phase.** Testing happens continuously during development, in the same day, by the same person. Then again independently. |
| **Maintain** | Fix bugs, adapt to OS/API changes, improve, refactor, observe in production. |

And **testing is the phase people most wrongly believe is later.** A bug found by the developer's own test at 3pm costs 5 minutes. The same bug found by a user three weeks later costs days, plus a rollback, plus a postmortem.

## 1.4 The structural truth nobody tells students

> **Development and maintenance are not two different jobs. They are the same job, forever.**

There is no "development phase" after which someone else takes over. Every professional spends the vast majority of their career maintaining software that someone — often themselves — wrote years earlier.

That is why these matter more than raw coding speed:

1. Understand a requirement precisely, and flag ambiguity **before** wasting effort
2. Design a structure that survives two years of change
3. Write code other people can safely modify
4. Test it so changes do not break things
5. Debug methodically when they do break
6. Deploy without drama
7. Explain decisions to non-engineers

This is what "architecture and design" means in practice. It is a habit, not a talent.

---

# Part 2 — Phase 1: Planning

## 2.1 Product vision

**What it is:** A one-to-two page document answering *what*, *why*, and *for whom*.

```
PRODUCT NAME:      CardLab
ONE-LINE PITCH:    A flashcard app that schedules reviews using proven
                   memory algorithms instead of fixed intervals.
PROBLEM:           Medical and law students cram, then forget, because
                   fixed review intervals waste effort on material they
                   already know and neglect material they almost know.
WHO:               University students in high-retention courses.
WHY NOW:           Spaced-repetition algorithms (FSRS) are now provably
                   better than the 1987 SM-2 algorithm they replaced,
                   and no mainstream student app uses them.
SUCCESS METRIC:    100 weekly active users who study 5+ days per week.
```

If you cannot fill in a pitch like that, you do not have a project. You have a vibe.

## 2.2 MVP by subtraction

**MVP = Minimum Viable Product.** The words matter separately:

- **Product** — the thing you are making
- **Viable** — it actually does something real
- **Minimum** — the smallest amount that still counts

An MVP is **not a cheap version of the real thing.** It is the smallest version that can honestly answer one question: *would anyone actually use this?*

**The procedure:**

**Step A — Brain dump the full vision.** No filtering. Write every feature.

**Step B — Find the core loop.** The one activity the user repeats 5–50 times per week.

**Step C — Run the survival test on each feature:**

> *"If I deleted this feature, could the user still get the core value? Would they still use this next week?"*
>
> Yes → **CUT**. No → **KEEP**.

| Feature | If deleted, does the core still work? | Verdict |
|---|---|---|
| Present a question, user recalls the answer | No — this *is* studying | **KEEP** |
| Mark right / wrong | No — no scheduling happens without it | **KEEP** |
| Compute next review time | No — you'd show all cards in order | **KEEP** |
| Account / login | Yes — the app can still know "this is *my* deck" | **CUT** for v1 |
| Streaks and charts | Yes | **CUT** |
| Share decks publicly | Yes | **CUT** |
| Push notifications | Borderline — cut for v1, add week 2 | **LATER** |
| Dark mode | Yes, obviously | **CUT** |

**Step D — Cut dependency traps.** A leaderboard is worthless with three users. Comments are worthless with nothing to comment on. These arrive free once you have users.

**Step E — Apply the 60-second test.** Can a new user get real value within 60 seconds of opening it? If they must sign up, read a tutorial, and import data first — you have cut too little.

### Three myths to unlearn

| Myth | Reality |
|---|---|
| "MVP means low quality" | MVP means **narrow**, not **sloppy**. Reliable, just small. |
| "I shouldn't show anyone an ugly app" | Ugly is fine. You have no users yet, so there is nobody to impress. |
| "I'll build the full version after" | Nobody does. Success adds 3–4 features to v1, not 12. |

## 2.3 Feasibility analysis

Before building, answer: **should we do this at all?**

| Dimension | Question |
|---|---|
| **Technical** | Can it actually be built? Do we have the skills, hardware, libraries? |
| **Economic** | Does the cost of building + running it exceed the value it returns? |
| **Operational** | Will it fit how people actually work? |
| **Schedule** | Can it be finished in the time available? |
| **Legal** | Does it violate any law, licence, patent, or regulation? |

## 2.4 The risk register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Nobody wants it | Medium | Very high | Test before building — the MVP exists for this |
| The algorithm does not actually beat the baseline | Medium | High | Benchmark against existing approaches early, before investing |
| Dataset unavailable / unusable | Medium | High | Verify data licensing and format in week 1, before designing around it |
| A key library becomes unmaintained | Low | High | Avoid depending on anything you do not control |
| Scope creep | **High** | High | Write the "Won't have" list and defend it |
| Personal burnout (solo project) | Medium | Medium | Ship something small and real rather than something large and unfinished |

---

# Part 3 — Phase 2: Requirements & Analysis

## 3.1 Functional vs. non-functional requirements

**The single most important distinction in this document. Your academic rubric almost certainly grades on it.**

| Type | Definition | Example (flashcard app) |
|---|---|---|
| **Functional** | *What the system must DO* — features and behaviour | "The user can create a deck." "The system schedules the next review using the FSRS algorithm." "The user can search decks by title." |
| **Non-functional** | *How WELL it must do it* — quality attributes | "Results render in under 2 seconds." "Works offline." "Only a deck's owner may edit it." "Supports 10,000 cards without slowdown." |

Non-functional requirements are **harder to specify, easier to forget, and far more expensive to fix late.** Nearly every failed project is a failure of a non-functional requirement nobody wrote down.

## 3.2 The non-functional requirement catalogue

| Category | The question it answers |
|---|---|
| **Performance** | How fast? (response time, latency, throughput) |
| **Scalability** | Does it survive 10× the load? |
| **Availability** | Is it up, and what % of the time? |
| **Security** | Who can do what? What if they are malicious? |
| **Usability** | Can a non-expert use it without training? |
| **Accessibility** | Does it work for users with disabilities? (WCAG: screen readers, contrast, keyboard-only) |
| **Maintainability** | Can a stranger fix this in a year? |
| **Portability** | Does it run across platforms? |
| **Reliability / recovery** | If it crashes or data is deleted — what happens? Backups? |
| **Compliance** | Does it follow the law? (GDPR, HIPAA, PCI-DSS) |
| **Compatibility** | Which OS versions, browsers, devices? |

**Make each one testable.** "The app should be fast" is not a requirement. "p95 response time under 300ms" is.

## 3.3 MoSCoW prioritisation

| | Meaning |
|---|---|
| **Must have** | No product without it. Non-negotiable. |
| **Should have** | Very important, but the product survives without it. |
| **Could have** | If time permits. |
| **Won't have** | Explicitly out of scope. **Write this down.** |

> Writing "Won't have" is as important as writing "Must have." It is the primary defence against scope creep, the number one cause of missed deadlines in the software industry.

## 3.4 Use cases

A **use case diagram** shows who (actors) does what (use cases), and how they relate.

```
        ┌──────────────┐
        │   Student    │──────────────┐
        └──────┬───────┘              │
     ┌─────────┼──────────┬──────────┴─────┐
     ▼         ▼          ▼                ▼
 (Create    (Add card)  (Study deck)   (View progress)
  deck)                            │
     │                             │
     └──────────────┬──────────────┘
                    │  "decks are private"
          ┌─────────▼────────┐
          │  Reminder Service│   ← system actor
          └──────────────────┘
```

Every oval is a function, and together they become your functional requirements list.

### Use case narrative (write one per important use case)

```
USE CASE UC-03: Study a deck
  Actor:            Student
  Trigger:          Student taps "Study" on a deck
  Preconditions:    A deck exists with ≥1 due card
  Main flow:
     1. System displays the next due card (front only)
     2. Student recalls the answer, then taps to reveal
     3. System displays the back of the card
     4. Student taps "Got it" or "Missed it"
     5. System records the result
     6. System computes next review date via the FSRS algorithm
     7. System presents the next card
     8. Steps 1–7 repeat until no due cards remain
     9. System displays a summary screen
  Alternative flows:
     - Deck has no due cards → display "Next review: <date>"
     - Student taps "Undo" → revert last result
  Postconditions:   Every reviewed card has an updated schedule
```

Writing that once makes the entire implementation obvious. This single artifact saves days.

## 3.5 Data Flow Diagrams (DFD)

Shows how **data moves**: inputs, processes, transformations, storage, outputs.

```
   ┌───────────┐
   │  Create   │
   │   Card    │───────────────┐
   └─────┬─────┘               │
         │ front, back, deck  │
         ▼                     ▼
   ┌─────────────┐      ┌───────────────┐
   │  VALIDATE   │─────▶│    DATABASE   │
   │ reject if   │ ok   │   (Card table)│
   │ empty/>200  │      └───────────────┘
   └──────┬──────┘
          │ invalid
          ▼
   ┌──────────────┐
   │ Show error   │
   └──────────────┘
```

Level-0 DFD is the whole system as one bubble. Each level below it decomposes one bubble in detail.

## 3.6 Entity-Relationship Diagrams (ERD)

Shows **what data exists** and how it relates. The most valuable document in most projects — "the app is broken" is usually really "we never agreed on the data model."

```
┌──────────┐              ┌──────────┐              ┌──────────┐
│   USER   │1──────────M │   DECK   │1──────────M │   CARD   │
└──────────┘              └──────────┘              └──────────┘
PK id                     PK id                    PK id
   name                      title                     front
   email                 FK   user_id              FK   deck_id
   created_at            FK   algorithm_id            back
                                                       due_at
┌──────────────────┐                                  ease
│  REVIEW_LOG      │◀───────────────────────────────M  interval
└──────────────────┘                                  reps
PK id
FK card_id                                              ┌──────────────────┐
   reviewed_at                                        │    ALGORITHM      │
   was_correct                                       └──────────────────┘
   time_taken_ms                                     PK id
                                                    FK name
                                                        version
```

**Notation:** rectangle = entity (a thing), diamond = relationship, oval = attribute, `PK` = primary key (unique identifier), `FK` = foreign key (reference to another table's PK).

## 3.7 Database design and normalisation

**Normalisation** — organising tables so each fact is stored exactly once. This prevents contradictions, which is the number one source of data bugs.

| Form | Rule | Plain meaning |
|---|---|---|
| **1NF** | One value per cell | Do not store "Alice, Bob, Carol" in one field |
| **2NF** | Non-key columns depend on the whole key | No partial dependencies |
| **3NF** | Non-key columns do not depend on other non-key columns | No transitive dependencies |

**Example of why:** if a user's email lives in both `users` and `orders`, changing the email in one place leaves the other stale. Normalisation eliminates that class of bug.

**Denormalisation** goes the other way — deliberately duplicating data to make reads faster. Valid, but a conscious trade, never an accident.

**SQL vs. NoSQL:**

| | Relational (PostgreSQL, MySQL, SQLite) | NoSQL (MongoDB, Redis, DynamoDB) |
|---|---|---|
| Data shape | Tables with fixed columns | Flexible JSON-like documents / key-value |
| Relationships | Foreign keys, joins | Embedded, denormalised |
| Transactions | Strong (ACID) | Weaker |
| Scaling | Vertical by default; hard horizontally | Horizontal by design |
| Schema | Defined in advance | Flexible |
| **Use when** | Data is relational and correctness matters | Schema is unknown, or you need extreme write throughput |

**For most student and early-career projects: PostgreSQL.** It is one system doing everything well. Learning SQL properly is a permanent skill; NoSQL syntax is trivially learnable later.

---

# Part 4 — Phase 3: Design

## 4.1 What is architecture?

**Architecture** = the overall structure of the system and the reasoning behind it.

The reasoning matters more than the structure. Anyone can copy a diagram; the value is in *why this structure for this problem.*

## 4.2 The universal client–server shape

Nearly every application you will ever build has this skeleton:

```
┌────────────────────┐          HTTPS           ┌──────────────────────┐
│      CLIENT        │◀────────────────────────▶│       SERVER         │
│                    │                          │                      │
│  • UI              │   Request / Response     │  • Business logic    │
│  • Input handling  │◀─────────────────────────│  • Authentication    │
│  • Local cache     │        (JSON over HTTP)  │  • Authorisation     │
│  • Rendering       │                          │  • Validation        │
│                    │                          │  • Scheduling        │
│  Lives on:         │                          │                      │
│  phone / browser   │                          │  Lives on:           │
└────────────────────┘                          │  a machine you rent  │
                                                 └───────────┬──────────┘
                                                             │
                                                             ▼
                                                 ┌──────────────────────┐
                                                 │      DATABASE        │
                                                 └──────────────────────┘
```

**The separation is the whole point.** The **client** is the presentation layer — what the user sees and touches. The **server** owns the truth: business rules, security, data. The **database** stores it.

> **Golden rule of security: trust the server, never trust the client.** The client runs on hardware you do not control and can be modified. Every security decision must be re-checked on the server. Hiding a button is not protecting anything.

## 4.3 Architectural patterns

| Pattern | Shape | Why it exists |
|---|---|---|
| **Monolith** | One codebase, one deployable | Simplest. **Default right answer until proven otherwise.** |
| **Layered / 3-tier** | Presentation → Business → Data | Each layer has one job. Most common for applications. |
| **MVC** | Model (data) – View (UI) – Controller (logic) | Separates UI from logic. Classic for web. |
| **MVVM** | Model – ViewModel – View | Dominant on Android (Jetpack). ViewModel survives configuration changes. |
| **Client–Server** | Split across machines | The default shape of everything user-facing. |
| **Event-driven** | Parts communicate via messages ("UserRegistered") | Great for notifications, payments, integrations. Scales well. |
| **Microservices** | Many small independent services | Powerful at scale. Enormous complexity cost. |
| **Serverless** | Run functions per-request, no server to manage | Cheapest for spiky, unpredictable traffic. Cold starts, vendor lock-in. |
| **Offline-first / sync** | Local database is the source of truth for the UI; syncs to server | Native mobile feel, works offline. Hardest to get right. |

## 4.4 Layered architecture in depth

This is the structure you should internalise.

```
┌───────────────────────────────────────────────────────────┐
│  PRESENTATION LAYER                                        │
│  HTTP routes / controllers, request parsing, response      │
│  formatting.                                              │
│  Rule: knows about HTTP. Does NOT contain business logic.  │
└───────────────────────────┬───────────────────────────────┘
                            │  calls
┌───────────────────────────▼───────────────────────────────┐
│  BUSINESS / DOMAIN LAYER                                   │
│  The actual rules of your problem. Pure logic.             │
│  "Given a card's history, what is its next review date?"   │
│  Rule: knows nothing about HTTP or SQL. Fully testable.    │
└───────────────────────────┬───────────────────────────────┘
                            │  uses
┌───────────────────────────▼───────────────────────────────┐
│  DATA ACCESS LAYER (Repository)                            │
│  Talks to the database. CRUD operations.                   │
│  Rule: no business logic here — only persistence.          │
└───────────────────────────┬───────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────┐
│  DATABASE                                                  │
└───────────────────────────────────────────────────────────┘
```

**Why this matters enormously.** Because the business layer knows nothing about HTTP or SQL, you can test it with no server and no database running:

```python
# domain/scheduling.py  — pure logic, no imports from web or db

def next_review_interval(ease: float, reps: int, quality: int) -> int:
    """Return the interval in days before this card should be seen again."""
    if quality < 3:                      # failed recall
        return 1                          # back to tomorrow
    if reps == 0:
        return 1
    if reps == 1:
        return 6
    return round(ease * 6)               # mature cards grow fast


def test_second_review_is_six_days():
    assert next_review_interval(ease=2.5, reps=1, quality=5) == 6


def test_failure_resets_to_one_day():
    assert next_review_interval(ease=2.5, reps=4, quality=1) == 1
```

That runs in microseconds with no infrastructure. If your business logic is welded into HTTP handlers and SQL queries, you cannot test it that way at all — you must spin up a server and a database for every assertion.

**This is the single biggest architectural lesson:** push logic out of the edges and into a pure middle, so it becomes testable in isolation. Professionals will ask you about this in an interview.

**A realistic FastAPI structure** (the Python web framework you will most likely use):

```
myproject/
├── app/
│   ├── main.py              # app creation, middleware wiring
│   ├── config.py            # settings, environment variables
│   ├── api/
│   │   ├── routes.py        # HTTP endpoints (presentation)
│   │   └── schemas.py       # request/response models (validation)
│   ├── domain/
│   │   ├── scheduling.py    # pure business logic  ← testable, no deps
│   │   └── entities.py      # dataclasses / models
│   ├── repositories/
│   │   ├── card_repo.py     # database access only
│   │   └── user_repo.py
│   └── security/
│       ├── auth.py          # password hashing, token issue/verify
│       └── deps.py          # "who is the caller?" dependency
├── tests/
│   ├── test_scheduling.py   # fast, no infrastructure
│   ├── test_api.py          # spins up the app
│   └── test_repos.py        # uses a real (test) database
├── pyproject.toml
└── README.md
```

## 4.5 Monolith vs. microservices

| | Monolith | Microservices |
|---|---|---|
| **Deployment** | One unit | Many independent units |
| **Debugging** | One stack trace | Distributed tracing across services |
| **Local dev** | `docker compose up` | Run everything, or stub services |
| **Network failure** | Not a concern | A first-class problem |
| **Data consistency** | One transaction | Distributed transactions, eventual consistency |
| **Cost** | Low | High (service mesh, tracing, CI per service) |
| **Team size needed** | 1+ | Usually 10+ |

**Rule of thumb:** you cannot debug what you cannot see, and microservices destroy visibility. Every company that went microservices too early regrets it and consolidates back into a modular monolith. Start monolithic, split only when you have measured evidence (a specific bottleneck) and a team large enough to own it.

## 4.6 API design

**What is an API?** The agreed set of doors into your program. A frontend dev and a backend dev can be different people who never meet — they agree only on the shape of requests.

```python
# REST — the default choice
GET    /api/cards          → list cards (supports ?deck_id=3&page=2&limit=20)
GET    /api/cards/42       → fetch card 42
POST   /api/cards          → create
PATCH  /api/cards/42       → partial update
DELETE /api/cards/42       → delete
```

| Design decision | Guidance |
|---|---|
| **REST vs. GraphQL** | REST is simpler and fits most needs. GraphQL lets clients request exact fields; more machinery, use it when clients genuinely differ. |
| **Versioning** | `/api/v1/cards` — so old clients keep working when you change the shape |
| **Pagination** | Never return unlimited rows. `?page=&limit=` or cursor-based for large tables |
| **Error format** | Consistent and machine-readable: `{"error": {"code": "CARD_NOT_FOUND", "message": "..."}}` |
| **Status codes** | `200` ok · `201` created · `400` bad input · `401` not authenticated · `403` authenticated but not allowed · `404` not found · `422` validation failed · `500` server error |
| **Auth** | Bearer token in the `Authorization` header, never in the URL (URLs get logged) |
| **Rate limiting** | Protects you from abuse and from being crashed by traffic |
| **Idempotency** | For payments: retrying must not charge twice. Accept an `Idempotency-Key` header |
| **Documentation** | OpenAPI/Swagger — FastAPI generates it for free from your code |

## 4.7 UI/UX: wireframes, mockups, prototypes

Three distinct artifacts. Students conflate them.

| Artifact | What it is | Contains | Fidelity |
|---|---|---|---|
| **Wireframe** | Where things go | Grey boxes, real labels, arrows between screens | Lowest |
| **Mockup** | What it looks like | Real colours, fonts, imagery. Still not clickable | Middle |
| **Prototype** | How it feels | Clickable, animated, testable with real users | Highest |

### What a wireframe actually contains

Five ingredients, nothing else:

1. **Screens** — one drawing per screen in the main path
2. **Placeholder boxes** — `[image]`, `[question text]`, `[button]`
3. **Layout and hierarchy** — what is big, what is small, what is at the top
4. **Real micro-copy** — the actual words, not "Lorem ipsum"
5. **Arrows between screens** — the user flow

**Deliberately excluded:** colour, fonts, logos, images, animation, real data.

### The user flow (the more important half)

```
[Home: your decks]
        |
        | tap "Study"
        v
[Study: question shown]
        |
        | tap to reveal
        v
[Study: answer shown]
        |
        | tap "Got it" / "Missed it"
        v
[Results: "17 right, 3 wrong"]
        |
        | tap "Review the 3" ─────┐
        v                         |
[Study again, 3 cards] ──────────┘
        |
        | tap "Done"
        v
[Home: "Great work. Next review tomorrow 09:00."]
```

Read as a sentence: *I open the app, tap Study, recall the answer, reveal, mark it, finish, and if I got some wrong it loops me back to study only those.*

If you cannot draw this without getting tangled, **your app has a design problem.** Finding that out costs an hour with a pen. Finding it out after building costs weeks.

### Where to make wireframes

**Paper and pen** — genuinely fine and fastest. Draw a tall rectangle per screen, fill with boxes and labels, lay them in order, draw arrows.

**Free digital options:**
- **Excalidraw** (excalidraw.com) — best for flow diagrams, deliberately rough-looking, free, no account needed
- **Figma** — the industry standard. `Shift+2` = phone frame, `R` = rectangle, `T` = text, `F` = new frame, `A` = arrow
- **Google Drawings / PowerPoint** — entirely adequate
- **Pencil on index cards** — one card per screen

### Why bother, when you cannot draw?

1. **It is free validation.** Show wireframes to five people. Confusion is visible on a wireframe in five minutes; in a real app a user leaves in twenty seconds.
2. **It is the build specification.** Later, whether you or a no-code tool builds it, those wireframes are the instructions. It removes "I imagined it differently" arguments.
3. **It stops you building the wrong thing.** Most wasted projects were a clear plan that turned out to be the wrong plan.

## 4.8 What "architecture and design" means to a hiring manager

When an interviewer asks "walk me through how you'd design this", they are scoring five things:

| What they are assessing | What a strong answer contains |
|---|---|
| **Requirements** | Names both functional and non-functional requirements. Asks clarifying questions before answering. |
| **Data** | Identifies the core entities and relationships. Says why you chose SQL or NoSQL. |
| **Structure** | Layers or services, with a stated reason for the boundary. |
| **Failure modes** | What happens when the database is down? When two users edit the same thing? When the request is malicious? |
| **Trade-offs** | "Monolith because there's one team." "Postgres because the data is relational and correctness matters." Names what you gave up. |

**Nobody expects you to have the perfect answer.** They expect you to **reason out loud, state your assumptions, and acknowledge trade-offs.** A candidate who says "I'd start with a modular monolith, Postgres for relational data, put the scheduling logic in a pure service so it's unit-testable, and I would worry about horizontal scaling later if load justified it" is already more convincing than one who says "I would use microservices and NoSQL for scalability" without explaining why.

---

# Part 5 — Methodologies: how professionals decide to build

## 5.1 The core tension

Software is unusual because **the requirements change.** The customer does not know exactly what they want until they see something running.

Every methodology exists to answer one question:

> **How do we manage change when we cannot predict it in advance?**

| Methodology | Core answer | Best for |
|---|---|---|
| Waterfall | Lock requirements; change is expensive | Fixed-spec, regulated, contract work |
| Agile | Iterate in short cycles; embrace change | Uncertain products, changing requirements |
| Scrum | Fixed-length sprints with roles and events | Team delivery with a deadline |
| Kanban | Continuous flow with WIP limits | Maintenance, support, continuous delivery |
| XP | Engineering discipline: TDD, pairing, review | Small teams wanting quality discipline |
| Spiral | Prototype early to kill risks | Large, risky, novel projects |
| DevOps | Shorten the loop from code to production | Teams shipping continuously |

## 5.2 Waterfall — Sequential

```
Requirements ──▶ Design ──▶ Implementation ──▶ Testing ──▶ Deployment ──▶ Maintenance
     │              │              │                │            │             │
     └── locked     └── locked     └── locked       └── locked    └── locked    └── 🔒
```

| | |
|---|---|
| **Origin** | 1970s, from construction and manufacturing |
| **Strengths** | Clear milestones, fixed budget, heavy documentation, easy to audit |
| **Weaknesses** | Requirements change invalidates the whole plan. First working product appears very late. |
| **Used for** | Government contracts, defence, medical devices, automotive, anything regulated |
| **Avoid for** | Anything where you are learning what to build |

## 5.3 Agile — Iterative

From the **Agile Manifesto (2001)**, four values:

1. **Individuals and interactions** over processes and tools
2. **Working software** over comprehensive documentation
3. **Customer collaboration** over contract negotiation
4. **Responding to change** over following a plan

```
Plan ──▶ Build (2 weeks) ──▶ Test ──▶ Feedback ──▶ Plan ──▶ Build (2 weeks) ──▶ ...
              ▲                                                              │
              └──────────────────────────────────────────────────────────────┘
```

| | |
|---|---|
| **Strengths** | Fast feedback, handles change, low risk of building the wrong thing |
| **Weaknesses** | Weak documentation (hard to hand over), scope drifts, "we're Agile" is often used as an excuse for no planning |
| **Used for** | Startups, most modern product teams |

## 5.4 Scrum — the most common Agile framework

A **Sprint** is a fixed time box, usually two weeks. Everything must fit inside it. This is the discipline that stops Agile becoming chaos.

### Three roles

| Role | Responsibility |
|---|---|
| **Product Owner** | Decides *what* gets built and in what order. Single point of priority decisions. Shields the team from random requests. |
| **Scrum Master** | Facilitates the process, removes blockers, coaches. *Not* a line manager. |
| **Developers** | Build the thing. Cross-functional — frontend, backend, QA together. |

### Five events

| Event | Duration | What happens |
|---|---|---|
| **Sprint** | 2–4 weeks | The container for all the work |
| **Sprint Planning** | 1–4 hrs | Team agrees *what* to build this sprint, breaks it into tasks |
| **Daily Scrum** | 15 min | Standing, same time: what did you finish, what will you do, what is blocked |
| **Sprint Review** | 1–2 hrs | **Demo working software to stakeholders.** Gather feedback. |
| **Sprint Retrospective** | 45–90 min | "What should *we* change about how we work?" — about the process, not the product |

### Three artefacts

| Artefact | What it is |
|---|---|
| **Product Backlog** | The master, value-ordered list of everything that might get built |
| **Sprint Backlog** | The subset chosen for this sprint, broken into tasks |
| **Increment** | The actually-working software produced this sprint. Must be usable. |

> **The essential rule:** if it is not in the Sprint, it is not being built this sprint. If it does not fit the Sprint, it goes back to the backlog.

## 5.5 Kanban — continuous flow

```
┌──────────┬──────────────┬───────────┐
│ To Do    │ In Progress  │ Done      │
│ card     │ card         │ card      │
│ card     │ card  card   │ card      │
│ card     │              │ card      │
└──────────┴──────────────┴───────────┘
     ▲          │
     └── WIP limit: 3 ──┘   (stop starting until one finishes)
```

**WIP limits** are the key idea: they deliberately slow down *starting* so that *finishing* speeds up. From Toyota's manufacturing system.

| | |
|---|---|
| **Used for** | Maintenance teams, bug-fix backlogs, teams that ship continuously |
| **Advantage** | No artificial ceremony; no artificial deadline |
| **Risk** | Without WIP limits, work piles up "in progress" and nothing ever finishes |

## 5.6 Extreme Programming (XP)

| Practice | Meaning |
|---|---|
| **TDD** | Write a failing automated test first, then the minimum code to pass, then refactor |
| **Pair programming** | Two people, one keyboard. Catches mistakes and spreads knowledge live |
| **Code review** | Nobody merges their own code alone |
| **Continuous integration** | Every change merged and tested automatically, many times a day |
| **Refactoring** | Improving structure with no behaviour change |
| **Small releases** | Ship often, in tiny increments |
| **Collective code ownership** | Anyone can change anything; no silos |

## 5.7 TDD — Test-Driven Development

The cycle, three phases:

```
  ┌──────────────────────────────────────────┐
  │                                          │
  │   RED ──▶ GREEN ──▶ REFACTOR ───────────┤
  │  write a   write the   clean up the     │
  │  failing   minimum     code without     │
  │  test      to pass     changing          │
  │                        behaviour         │
  └──────────────────────────────────────────┘
```

**The discipline that makes it work: you are not allowed to write production code until you have watched a test fail.** If the test passes before you write anything, the test is not testing anything.

```python
# RED — write the failing test first
def test_empty_deck_has_no_due_cards():
    deck = Deck(cards=[])
    assert deck.due_cards(now=datetime(2026, 1, 1)) == []


# GREEN — now write the minimum code to pass
@dataclass
class Deck:
    cards: list

    def due_cards(self, now):
        return [c for c in self.cards if c.due_at <= now]


# REFACTOR — tidy up without changing behaviour
@dataclass
class Deck:
    cards: list

    def due_cards(self, now: datetime) -> list:
        """Cards whose scheduled review time has arrived."""
        return [card for card in self.cards if card.due_at <= now]


# The test was already correct. Refactoring changed nothing observable.
# That is exactly the definition.
```

**The benefit nobody mentions: the test is a written specification that cannot go stale.** Code documentation rots. Tests break when behaviour changes.

**Why TDD is hard for beginners and what to do:** you do not know what to test until you know what to build. TDD at your current stage is premature. **Write tests for the pure-logic layer of your second or third project**, not your first.

## 5.8 The V-Model

Waterfall's engineering cousin. Each development stage has a matching verification stage at the same level of detail. Standard in systems engineering and embedded work.

```
Requirements          ◀▶  Acceptance test
   System design      ◀▶  System integration test
       Architecture   ◀▶  Software integration test
            Unit design ◀▶ Unit test
```

Each test level is planned at the same time as the stage it verifies.

## 5.9 The Spiral Model

Boehm's answer to "waterfall fails when requirements are unclear." Loops through four stages repeatedly, each loop deeper and more committed.

```
   ┌─────────────┐
   │   PLAN      │◀──────────────┐
   └──────┬──────┘               │
          ▼                      │
   ┌─────────────┐               │
   │ ASSESS RISK │──────────────▶│  (each loop kills more risk
   └──────┬──────┘               │   before committing more effort)
          ▼                      │
   ┌─────────────┐               │
   │  DEVELOP    │──────────────▶│
   └──────┬──────┘               │
          ▼                      │
   ┌─────────────┐               │
   │  EVALUATE   │──────────────▶│
   └─────────────┘               │
          └──────────────────────┘
```

**Prototyping is the primary risk-reduction tool.** Useful when you are unsure whether your idea is even feasible.

## 5.10 DevOps

Not really a build methodology — a **working culture** that removes the wall between developing and operating. The phrase is *"you build it, you run it."*

| Practice | Meaning |
|---|---|
| **CI — Continuous Integration** | Every change is automatically merged, built, and tested. Commit several times a day. The point: never let a branch rot for three weeks. |
| **CD — Continuous Delivery / Deployment** | Every version passing tests is automatically released. Deployment becomes a non-event. |
| **Infrastructure as Code** | Servers defined in files (Terraform), not hand-clicked. Reproducible and auditable. |
| **Containers (Docker)** | Package the app with everything it needs to run, so it behaves identically everywhere. Kills "works on my machine." |
| **Observability** | Error tracking (Sentry), logs, metrics, uptime checks — in production, continuously. |
| **Blue/green deployment** | Run old and new side by side. Switch traffic only if the new one is healthy. Instant rollback. |

## 5.11 Choosing a methodology

| Situation | Methodology |
|---|---|
| Learning, small solo project | **Kanban** or a light Scrum — write a backlog, work in weekly batches |
| Academic project with fixed chapters | **Scrum**, with sprints mapped to chapters |
| Research/experimental work needing repeatable protocol | **Waterfall or V-Model** for the experimental portion |
| Startup, uncertain product | **Scrum + CI/CD** |
| Maintenance / bug backlog | **Kanban** |
| Most real companies | **Scrumban** (Scrum + Kanban) — real, and not a contradiction |

> **For you:** Scrum, with two-week sprints, a real backlog you maintain in GitHub Issues, and a sprint review before each sprint ends. Even solo, this forces you to actually finish something every two weeks, which is the entire skill.

---

# Part 6 — Security: designed in, not added on

## 6.1 The mindset

> **You are not building a castle. You are assuming the attacker is already inside and every single input is hostile.**

Security bolted on after code exists is *remediation*, and it costs 5–10× more than security designed in from the start. This is why the pre-coding phase includes a threat model.

## 6.2 Authentication vs. authorisation

The most-confused pair in the industry, and the root of most real breaches.

| | Question | Mechanism |
|---|---|---|
| **Authentication** | *Who are you?* | Password, fingerprint, face, one-time code, SSO (Google/GitHub) |
| **Authorisation** | *What are you allowed to do?* | Roles and permissions |

```
Alice logs in        ──▶ AUTHENTICATION (password accepted → she is Alice)
                            │
                            ▼
Alice opens Deck #7   ──▶ AUTHORISATION (she owns it → ALLOWED)
Alice opens Deck #9   ──▶ AUTHORISATION (she does not own it → 403 Forbidden)
```

**The critical design question: *can the user reach this by typing a different URL?***

- If the answer is "the **server** checks ownership on every request" → correct
- If the answer is "the **button is hidden**" → you have done **nothing**

This is **Broken Access Control**, the number one vulnerability on the OWASP list.

```python
# ✗ BROKEN — trusts the client
@router.get("/decks/{deck_id}")
def get_deck(deck_id: int, user=Depends(current_user)):
    return db.get(Deck, deck_id)      # any logged-in user can read ANY deck
                                       # just by guessing / changing the number


# ✓ CORRECT — the check lives in the data access layer
@router.get("/decks/{deck_id}")
def get_deck(deck_id: int, user=Depends(current_user)):
    deck = repo.get_for_user(deck_id, user.id)     # ownership baked into the query
    if deck is None:
        raise HTTPException(403, "Not your deck")   # deliberately not 404:
    return deck                                     # 403 leaks less about existence
```

## 6.3 The OWASP Top 10

The industry's standing list of the worst application vulnerabilities.

| # | Vulnerability | Plain explanation | Defence designed in advance |
|---|---|---|---|
| 1 | **Broken Access Control** | Users reach things they should not | Check permission on the **server**, on **every** request |
| 2 | **Cryptographic Failures** | Sensitive data left unencrypted | HTTPS everywhere, encrypt storage, hash passwords properly |
| 3 | **Injection** | Attacker inserts code into your input and your app runs it | Whitelist validation + **parameterised queries** — never build queries by string concatenation |
| 4 | **Insecure Design** | The design itself lacks security thinking | Threat modelling; rate limits; safe defaults |
| 5 | **Security Misconfiguration** | Default passwords, debug mode on, directories listed | Documented, repeatable, automated deployment config |
| 6 | **Vulnerable & Outdated Components** | A dependency has a known CVE | Automated updates; dependency scanning in CI |
| 7 | **Auth Failures** | Weak passwords, no lockout, sessions not invalidated | Proper hashing, MFA, session expiry, brute-force lockout |
| 8 | **Data Integrity Failures** | Code or data tampered with in transit or in the pipeline | Verified sources, signed artifacts, CI integrity checks |
| 9 | **Logging Failures** | The breach happened and nobody noticed | Structured logs, alerting, defined retention |
| 10 | **SSRF** | Attacker makes your server fetch a URL they control | Allow-list outgoing destinations |

### Injection, concretely

This is the most important one to internalise, because SQL injection is trivially avoidable once you know the fix.

```python
# ✗ VULNERABLE — string concatenation
def find_user(username):
    query = f"SELECT * FROM users WHERE username = '{username}'"
    return db.execute(query)

# Attacker sends:  ' OR '1'='1
# Resulting query:  SELECT * FROM users WHERE username = '' OR '1'='1'
# → returns every user in the database


# ✓ SAFE — parameterised. The driver handles escaping.
def find_user(username):
    return db.execute(
        "SELECT * FROM users WHERE username = %s",
        (username,)          # the value is sent separately, never interpreted as SQL
    )
```

**The rule: never build a query, a shell command, or an HTML string by concatenating untrusted input.** Ever. Use the API's parameter-binding feature.

Similarly for shell commands:

```python
# ✗ VULNERABLE — shell=True with user input
os.system(f"ping {user_input}")        # user_input = "; rm -rf /"

# ✓ SAFE — no shell, argument passed directly
subprocess.run(["ping", "-c", "4", user_input], check=True)
```

## 6.4 Hashing vs. encryption

The distinction students get wrong most often.

| | Hashing | Encryption |
|---|---|---|
| **Purpose** | Verify integrity / verify a password | Recover the original data |
| **Direction** | **One-way. Irreversible.** | **Two-way**, using a key |
| **Use for** | Passwords, checksums, tamper detection | Anything you must read back: messages, personal data at rest |
| **Algorithms** | bcrypt, Argon2, SHA-256 | AES (symmetric), RSA / ECC (asymmetric) |
| **Needs a key** | No | Yes |

### Password storage — no exceptions

```
✗ WRONG   store "hunter2" as plain text          → a database dump leaks every password directly
✗ WRONG   SHA-256("hunter2")                     → too fast; attackers test billions per second
✓ RIGHT   argon2.hash("hunter2")                  → deliberately SLOW + per-user random salt
```

```python
import argon2

# Registration
hashed = argon2.PasswordHasher().hash("hunter2")
# "$argon2id$v=19$m=65536,t=3,p=4$c29tZXNhbHQ$..."   ← salt embedded

# Login
def verify(stored_hash: str, supplied: str) -> bool:
    try:
        argon2.PasswordHasher().verify(stored_hash, supplied)
        return True
    except argon2.exceptions.VerifyMismatchError:
        return False


def test_correct_password_verifies():
    h = argon2.PasswordHasher().hash("hunter2")
    assert verify(h, "hunter2") is True


def test_wrong_password_is_rejected():
    h = argon2.PasswordHasher().hash("hunter2")
    assert verify(h, "hunter3") is False
```

**Why slow is right:** you want login to take ~100ms. That is invisible to one user and catastrophic for an attacker trying 10 billion combinations. The **salt** (random, unique per user, embedded in the hash) means identical passwords produce different hashes, so no precomputed table cracks them all at once.

### Encryption at rest

```python
from cryptography.fernet import Fernet

key = Fernet.generate_key()                     # from your secret manager, never hardcoded
cipher = Fernet(key)

token = cipher.encrypt(b"carder's private notes")   # stored in the database
plain = cipher.decrypt(token)                       # read back when needed
```

## 6.5 STRIDE threat modelling

For each component of your design, ask six questions:

| Letter | Question | Example |
|---|---|---|
| **S**poofing | Can someone pretend to be another user? | No signature on JWTs; weak session IDs |
| **T**ampering | Can data be modified in transit or at rest? | Unsigned cookies, no integrity check on uploads |
| **R**epudiation | Can someone deny having done an action? | No audit log of changes |
| **I**nformation disclosure | Can data leak to someone it should not? | Stack traces in production; verbose errors |
| **D**enial of service | Can the system be made unavailable? | No rate limit on login; expensive query on every request |
| **E**levation of privilege | Can someone gain rights they should not have? | Mass assignment — user posts `{"role": "admin"}` and your code assigns it blindly |

**Mass assignment is worth calling out** — it is the most common bug in hand-rolled Python APIs:

```python
# ✗ VULNERABLE
class UserUpdate(BaseModel):
    name: str
    role: str = "user"

@router.patch("/users/me")
def update_me(data: UserUpdate, user=Depends(current_user)):
    for k, v in data.model_dump().items():
        setattr(user, k, v)      # attacker sends {"role":"admin"} → promoted
    ...


# ✓ SAFE — whitelist the fields you allow
class UserUpdate(BaseModel):
    name: str | None = None
    bio: str | None = None
    # note: no `role` field exists, so it cannot be set from input

@router.patch("/users/me")
def update_me(data: UserUpdate, user=Depends(current_user)):
    for field in data.model_dump(exclude_unset=True):
        setattr(user, field, v)
    ...
```

## 6.6 The pre-code security checklist

| Concern | Decision made before coding |
|---|---|
| **Encryption in transit** | HTTPS/TLS 1.2+ everywhere, including internal API calls |
| **Encryption at rest** | AES-256 / Fernet on sensitive columns and backups |
| **Secrets management** | **Never hardcode API keys.** Use environment variables; rotate anything ever committed to Git |
| **Input validation** | Allow-list on both client (UX) and server (security). Length limits, type checks |
| **Output encoding** | Escape data before inserting into HTML — stops XSS |
| **Least privilege** | The app's database user gets only the permissions it needs. Not root. |
| **Multi-factor auth** | For any sensitive or irreversible action |
| **Session management** | Expiry; `HttpOnly`, `Secure`, `SameSite` cookies; invalidate server-side on logout |
| **Rate limiting** | On login attempts and every expensive endpoint |
| **CSRF protection** | SameSite cookies + token, for cookie-authenticated apps |
| **Dependency risk** | Check maintenance status, known vulnerabilities, licence, download count |
| **File uploads** | Validate type by content, not by extension. Never serve uploads as executable code |
| **Error handling** | Never return stack traces or SQL errors to the client. Log internally, return a generic message |
| **Security testing** | Unit tests on authorisation logic; dependency scanning in CI; penetration test before launch |
| **Incident response** | Decide now, while calm: who is notified, how fast, what is the procedure |

```python
# Secrets: the correct pattern
import os
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    argon2_time_cost: int = 3

    class Config:
        env_file = ".env"          # never commit .env — add it to .gitignore on day one

settings = Settings()
```

## 6.7 Privacy and compliance

| Law | Scope | Core requirement |
|---|---|---|
| **GDPR** | EU / anyone with EU users | Lawful basis for processing; consent; data subject rights (access, deletion, portability); breach notification within 72 hours |
| **Philippines Data Privacy Act (RA 10173)** | Philippines | Consent, purpose limitation, retention limits, secure processing |
| **HIPAA** | US healthcare | Very strict controls on protected health information |
| **PCI-DSS** | Anything handling card numbers | Never store raw card data; use a payment provider |

**Design questions to answer before coding:**

1. What data do we collect — and do we *need* each piece?
2. Where is it stored, and for how long?
3. Who can see it?
4. How does a user request deletion?
5. What happens to backups?
6. What is our breach notification procedure?

**Principle of least collection:** the best way to avoid a data breach is never to have the data.

## 6.8 Security in Python

Python is memory-safe, which removes a whole class of C-level bugs. It gives you **no** automatic protection against the vulnerabilities that actually matter in web applications.

| Risk | What to do |
|---|---|
| **SQL injection** | Parameterised queries. Never string-format a query. Use an ORM or the driver's binding API |
| **Pickle deserialisation** | `pickle.loads` on untrusted data is remote code execution. Use JSON |
| **`eval` / `exec` on input** | Never. Ever. Use `ast.literal_eval` for literals, `shlex.split` for shell args |
| **Subprocess with `shell=True`** | Pass argument lists instead |
| **`yaml.load`** | Use `yaml.safe_load` |
| **Dependency vulnerabilities** | `pip-audit`, Dependabot, or Snyk in CI |
| **Verbose errors** | Custom exception handler returning generic messages; log the detail server-side |
| **Path traversal** | Never build file paths from user input. Validate against an allow-list |
| **Regex denial of service** | Avoid pathological patterns; use `re2` if patterns are attacker-controlled |

```python
# ✗ RCE
import pickle
pickle.loads(untrusted_bytes)

# ✓ Safe
import json
data = json.loads(untrusted_bytes)
```

---

# Part 7 — Phase 4: Development: what actually happens

## 7.1 A professional's actual day

A mid-level developer, day 6 of a two-week sprint:

```
09:00  Standup (15 min) — three questions: what did you finish,
                            what will you do, what is blocked?
09:15  Pull latest code. Read the branch you are about to work on.
09:40  Open the ticket. Re-read the requirements.
       Question: is this unambiguous? → Message the product owner
       for clarification BEFORE writing anything.
10:00  Write the failing test for the ticket's core logic.
10:30  Write the code. It fails. Fix. It passes. Refactor.
12:00  Commit. Push. Open a Pull Request.
12:15  Review someone ELSE's PR and leave comments.
       (Reviewing others is 20–30% of a senior's time.)
13:30  Review feedback arrives. Address it. Push again.
       CI auto-runs: build → lint → tests → security scan. You watch.
15:00  Deploy to staging. Click through the feature yourself.
16:00  Found a bug. Fix it. Add the test so it never returns.
17:00  Update the ticket, the README, the changelog.
17:30  Think about tomorrow's ticket.
```

**Only about two of those eight hours were writing new code.** The rest was understanding requirements, reviewing, testing, deploying, and reading other people's code.

That is precisely what "not all about how good you code" means.

## 7.2 The ten activities inside development

| # | Activity | What it really is |
|---|---|---|
| 1 | **Understand the ticket** | Read the requirement; decide whether it is clear enough to build. Ambiguity is the #1 cause of wasted work. |
| 2 | **Read existing code** | Most of your time goes to understanding code that already exists. You rarely write a module from zero. |
| 3 | **Write the code** | The part everyone pictures. Realistically 20–30% of the phase. |
| 4 | **Write tests** | Automated verification that your change works and broke nothing else. |
| 5 | **Debug** | Read errors, add logging, isolate the path, form a hypothesis, test it, fix the **cause** not the symptom. |
| 6 | **Code review** | You review others' code; others review yours. Non-negotiable at any professional level. |
| 7 | **Version control** | Branch, commit, merge, resolve conflicts, revert. Daily. Non-negotiable. |
| 8 | **Refactor** | Improve structure with no behaviour change. The difference between code that works and code that survives two years. |
| 9 | **CI/CD** | Commit → automated build, test, deploy. Failure known in 90 seconds, not tomorrow. |
| 10 | **Document** | README, comments explaining **why** (not *what*), changelog, API docs. |

## 7.3 Reading code is the job

Three skills that separate juniors from mid-levels:

**1. Reading the call stack.** When a request arrives, trace it end to end: route → handler → service → repository → SQL. Draw it on paper. Do this before changing anything.

**2. Reading the git history.** `git log -p <file>` shows not just what the code is now but *why it changed*. This is frequently more informative than the code itself.

```bash
git log --oneline                     # recent commits
git log -p src/domain/scheduling.py   # full history of one file
git blame src/domain/scheduling.py    # who wrote each line, and in which commit
```

**3. Reading the tests as documentation.** A well-named test tells you what the code is supposed to do more clearly than the implementation does.

## 7.4 Debugging properly

Do **not** change things randomly. This sequence works:

```
1. REPRODUCE      Make the failure happen reliably. If you cannot reproduce
                  it, you cannot confirm a fix. Gather exact steps.
2. ISOLATE       Narrow to the smallest failing case. Binary search the input.
3. READ THE      The error message and full traceback. Python's tracebacks
   TRACE         show the exact call chain — read them bottom-up.
4. HYPOTHESISE   "I think it's X because Y." Write it down.
5. TEST THE      Change one thing. Predict the result BEFORE running.
   HYPOTHESIS    If the prediction is wrong, your hypothesis was wrong —
                  not your typing. That is valuable information.
6. FIX THE CAUSE Not the symptom.
7. ADD A TEST    So this specific failure can never return silently.
8. REMOVE THE    Delete any print/console.log you added.
   DEBUGGING
```

```python
import logging
logger = logging.getLogger(__name__)

# Good: tells you what the value was, not just that something happened
logger.info("scheduling card=%s quality=%s reps=%s → interval=%s",
            card.id, quality, reps, interval)

# Bad: forces you to guess what the state was
print("here", x)
```

**Debugging is a scientific method, not a guessing game.** Form hypotheses, predict, observe, revise.

## 7.5 Refactoring

**Refactoring** = improving code structure with **no change in behaviour**. Verified by the test suite passing unchanged.

| Code smell | Refactor to |
|---|---|
| Function longer than ~30 lines | Extract named functions |
| Deeply nested `if`/`for` | Early returns (guard clauses) |
| Same logic copy-pasted | One shared function |
| Unclear names (`a`, `data2`, `doThing`) | Names that say what it is |
| 5 parameters, 12 arguments passed | Group into a dataclass or object |
| Magic numbers (`if days > 30`) | Named constants (`MAX_INTERVAL_DAYS = 30`) |

**Rule:** always refactor with tests in place, and never mix a refactor with a behaviour change in the same commit. Reviewers cannot review two changes at once.

## 7.6 Technical debt

Every shortcut you take now costs 2–5× later. Maintenance is where it gets paid — or compounds.

| | Deliberate | Accidental |
|---|---|---|
| **Example** | "We will build the real search in Q3. The hardcoded filter is a placeholder for a known requirement." | "We skipped validation because it was annoying." |
| **Handling** | Tracked in the backlog with a date | Must be paid down immediately |
| **Rule** | Only take deliberate debt with a **written plan** to repay it | Never |

## 7.7 Code review

When you open a PR, a human looks at:

- **Correctness** — does it do what the ticket said?
- **Design** — is it in the right layer? Right names? Any duplication?
- **Security** — injection risk? Secrets? Unvalidated input?
- **Tests** — is this actually covered?
- **Readability** — will someone understand this in six months?

**Reviewing others' code is how you learn.** You see patterns and mistakes before you make them. Budget time for it, and treat critical comments as free teaching.

---

# Part 8 — Algorithms & data structures

## 8.1 What an algorithm actually is

> **A finite, precise, unambiguous sequence of steps that transforms input into output.**

| Word | Meaning |
|---|---|
| **Finite** | It ends. A clear stopping condition. No "repeat until it feels right." |
| **Precise / unambiguous** | Every step exactly defined. "Sort it nicely" is not an algorithm. "Sort ascending by second character" is. |
| **Input** | What you start with |
| **Output** | What you end with |
| **Steps** | The procedure itself |

**The recipe analogy.** A recipe is an algorithm: input (ingredients), steps (mix, whisk, bake at 180°C for 20 min), output (one cake), finite (the timer ends it), unambiguous (exact temperature and time).

**A bad recipe:** "add some flour, cook until it smells nice." Vague, unbounded, unverifiable — and undebuggable.

```python
# A good algorithm: finite, precise, verifiable
def binary_search(items, target):
    """items must be sorted ascending."""
    lo, hi = 0, len(items) - 1
    while lo <= hi:                       # explicit termination condition
        mid = (lo + hi) // 2
        if items[mid] == target:
            return mid
        if items[mid] < target:
            lo = mid + 1                  # halve the remaining range
        else:
            hi = mid - 1
    return -1
```

## 8.2 What an algorithm is not

| | Why not |
|---|---|
| **A programming language** | Python, Java, Kotlin are languages for *writing down* algorithms. The algorithm exists before any language. |
| **The code** | Code is one *implementation*. Another programmer could write a different implementation of the same steps. |
| **A data structure** | Data structures hold data; algorithms operate on it. Paired but different things. |
| **A formula** | Formulas are a special case. Most algorithms have steps, not just symbols. |

## 8.3 Why algorithms matter

For any task a correct answer exists via brute force. Algorithms exist because brute force stops working.

| Problem | Brute force | Smart algorithm |
|---|---|---|
| Find a name in 1M records | 1,000,000 checks | ~20 checks |
| Route across a country | Not computable | Dijkstra / A\* → instant |
| Detect duplicate content | O(n²) pair comparisons | Hashing → O(n) |
| Detect duplicate content | O(n²) pair comparisons | Hashing → O(n) |

**Three reasons this decides real architecture:**

1. **Feasibility** — sometimes brute force is not "slow", it is mathematically impossible
2. **Cost** — 10× faster is 10× less cloud bill
3. **Experience** — response time *is* the product. Nobody waits eight seconds.

```python
# The brute-force version that stops working at scale
def find_duplicates_bruteforce(items):          # O(n²)
    seen, dups = set(), set()
    for i, a in enumerate(items):
        for b in items[i + 1:]:
            if a == b:
                dups.add(a)
    return dups

# The version that scales
def find_duplicates(items):                     # O(n)
    seen, dups = set(), set()
    for item in items:
        if item in seen:                        # hash-table lookup: O(1)
            dups.add(item)
        seen.add(item)
    return dups
```

Identical output. 10,000 items: 50 million comparisons vs. 10,000. **That choice is made before writing code.**

## 8.4 Complexity (Big-O) in plain language

How does the work **grow** as input grows?

| Complexity | Name | Plain meaning | Example |
|---|---|---|---|
| **O(1)** | Constant | Same time regardless of size | Dictionary lookup by key |
| **O(log n)** | Logarithmic | Each step halves what remains | Binary search; database index lookup |
| **O(n)** | Linear | Double the work, double the time | Reading a file once |
| **O(n log n)** | Log-linear | Nearly linear; fine at enormous scale | Sorting a list |
| **O(n²)** | Quadratic | Double the work, **quadruple** the time | Comparing every item to every other |
| **O(2ⁿ)** | Exponential | Double the work, **double** the time | Brute-force password guessing |
| **O(n!)** | Factorial | Explosive | Travelling Salesman brute force |

**Why this decides architecture:**

```
Adding a friend, detecting duplicates naively:

   100 users     → 100 × 100     =      10,000 checks   (instant)
  10,000 users   → 10k × 10k     = 100,000,000 checks   (~1 second)
 1,000,000 users → 1M × 1M       =      10^12 checks   (never finishes)
```

Swap in a hash set (O(1) lookup): 10,000 checks regardless of size. Same feature, completely different architecture.

```python
import time

def bench(fn, data):
    start = time.perf_counter()
    fn(data)
    return time.perf_counter() - start

import random
random.seed(42)
data = [random.random() for _ in range(50_000)]

print(f"brute force: {bench(find_duplicates_bruteforce, data):.3f}s")
print(f"hash set:    {bench(find_duplicates, data):.6f}s")
```

Running it teaches something no reading can: **you can see the complexity gap on your own machine, with your own code.** Do this once and the concept is permanent.

## 8.5 Data structures

| Structure | Plain meaning | Why it exists | Python |
|---|---|---|---|
| **Array / List** | Ordered row of items | Fast access by position | `list` |
| **Linked List** | Items each pointing to the next | Cheap insert/delete in the middle | `collections.deque` |
| **Stack** | Pile, last in first out | Undo, call stack, expression parsing | `list` (append/pop) |
| **Queue** | Line, first in first out | Task processing, message queues | `collections.deque` |
| **Hash Table / Dict** | Super-fast key → value | Indexing, sessions, caches, counting | `dict`, `set`, `defaultdict` |
| **Heap** | Largest/smallest always on top | Priority queues: Dijkstra, scheduling | `heapq` |
| **Tree** | Branching hierarchy | File systems, menus, DB indexes | — |
| **Graph** | Nodes connected by edges | Roads, friendships, dependencies | `networkx`, adjacency dict |
| **Trie** | Tree keyed by prefixes | Autocomplete, spell-check, IP routing | — |

> **The question hiding here:** "which algorithm *and* which data structure?" The combination is the real answer. Sorting is meaningless without a list. Routing is meaningless without a graph.

## 8.6 The classic algorithm catalogue

| Problem | Algorithms | Complexity |
|---|---|---|
| **Sorting** | Merge sort, Quick sort, Heap sort, Counting sort, Insertion sort | O(n log n) |
| **Searching** | Linear search, Binary search, Jump search | O(log n) sorted |
| **Graph traversal** | BFS, DFS | O(V + E) |
| **Shortest path** | Dijkstra, A\*, Bellman–Ford, Floyd–Warshall | O((V+E) log V) |
| **Greedy** | Activity selection, Huffman coding, interval scheduling | Varies |
| **Dynamic programming** | Fibonacci, knapsack, longest common subsequence, edit distance | Varies |
| **Divide & conquer** | Merge sort, Quick sort, binary search | O(n log n) |
| **Backtracking** | N-Queens, Sudoku, constraint scheduling | Exponential worst case |
| **Hashing** | SHA-256, bcrypt, Argon2, consistent hashing | O(1) average |
| **Scheduling** | Priority queue + greedy, topological sort | Varies |
| **Matching** | Hungarian algorithm, Hopcroft–Karp | O(V³) / O(E√V) |

### The two you must know properly

**Breadth-First Search** — explore level by level using a queue. Finds the shortest path in an unweighted graph.

```python
from collections import deque

def bfs(graph, start, goal):
    """Return the shortest path start→goal, or None."""
    queue = deque([[start]])
    seen = {start}

    while queue:
        path = queue.popleft()
        node = path[-1]

        if node == goal:
            return path

        for neighbour in graph[node]:
            if neighbour not in seen:
                seen.add(neighbour)
                queue.append(path + [neighbour])

    return None


def test_bfs_finds_shortest_route():
    graph = {
        "A": ["B", "C"],
        "B": ["A", "D"],
        "C": ["A", "E"],
        "D": ["B"],
        "E": ["C", "F"],
        "F": ["E"],
    }
    assert bfs(graph, "A", "F") == ["A", "C", "E", "F"]


def test_bfs_returns_none_when_unreachable():
    graph = {"A": ["B"], "B": ["A"], "C": []}
    assert bfs(graph, "A", "C") is None
```

**Dijkstra's algorithm** — extends BFS to weighted graphs. The backbone of routing and mapping software.

```python
import heapq


def dijkstra(graph, start):
    """graph = {node: {neighbour: weight}}. Returns {node: min_distance}."""
    distances = {node: float("inf") for node in graph}
    distances[start] = 0
    queue = [(0, start)]
    visited = set()

    while queue:
        distance, node = heapq.heappop(queue)

        if node in visited:                  # stale heap entry, skip it
            continue
        visited.add(node)

        for neighbour, weight in graph[node].items():
            candidate = distance + weight
            if candidate < distances[neighbour]:
                distances[neighbour] = candidate
                heapq.heappush(queue, (candidate, neighbour))

    return distances
```

Notice `heapq` — the priority queue from the data-structures table is what makes Dijkstra efficient. **Algorithm and structure are a pair.**

## 8.7 Algorithms inside real software

| Feature | Algorithms under the hood |
|---|---|
| Google search | PageRank, TF-IDF/BM25, inverted index |
| Google Maps routing | Dijkstra, A\*, contraction hierarchies |
| Autocomplete as you type | Trie + edit distance + frequency ranking |
| Netflix / YouTube / Spotify recommendations | Collaborative filtering, matrix factorisation, multi-armed bandits |
| Social media feed ordering | Ranking / engagement-prediction models |
| Password storage | bcrypt, Argon2 |
| The HTTPS lock | TLS handshake, RSA/ECDSA, AES |
| `sorted()` in Python | Timsort (merge + insertion hybrid) — O(n log n) |
| ZIP files, JPEG, streaming video | Huffman coding, LZ77/LZW |
| Spotify shuffle | Fisher–Yates shuffle |
| Ride-hailing driver assignment | Bipartite matching / Hungarian algorithm |
| Error correction on CDs, QR codes | Reed–Solomon |
| Data delivery over the internet | Routing protocols, TCP congestion control |
| Training any machine learning model | Gradient descent + backpropagation |

## 8.8 Why algorithms are assessed in interviews

Nearly every software engineering interview — including junior and mid-level roles — includes an algorithms and data structures section.

What is actually being assessed is **reasoning under constraints**, not memorisation:

1. Can you model the problem?
2. Can you produce a correct solution?
3. Can you evaluate its complexity?
4. Do you know when your solution will fail at scale?

**Nobody asks whether you memorised Dijkstra.** They ask whether, given a problem, you can reason toward an approach and defend it. That is why understanding beats memorising: it is the transferable part.

Working fluency means:
- Core structures: hash table, array, stack, queue, tree, graph
- Core algorithms: binary search, BFS, DFS, sorting, basic dynamic programming
- The reflex of asking *"what is the complexity, and will this survive real data?"*

---

# Part 9 — Phase 5: Testing

**The premise to internalise: testing is not a phase after development. It happens continuously during development, in the same day, by the same person. Then again, independently.**

| Level | What it tests | Speed | Who writes it |
|---|---|---|---|
| **Unit** | One function in isolation. `add(2,3) == 5` | Milliseconds | Developer |
| **Integration** | Do two parts work together? Does the API talk to the DB? | Seconds | Developer |
| **E2E (end-to-end)** | Full user journey through the real system | Minutes | Developer or QA |
| **Regression** | Did this fix break something that used to work? | Varies | Automated |
| **Manual / exploratory** | "Does this feel right?" What automation cannot judge | — | QA + developers |

## What to test

- **Happy path** — the normal case works
- **Edge cases** — empty list, one item, maximum size, zero, negative, duplicates
- **Boundary values** — exactly at the limit, one below, one above
- **Error paths** — bad input, missing data, network failure, permission denied
- **Concurrency** — two users editing the same record simultaneously

## The professional standard

> **Every bug fix ships with a test that reproduces the bug.** The second time the same bug appears is a code review failure.

```python
def test_duplicate_email_is_rejected():
    # This test was written AFTER the bug was found in production.
    # Its existence means the bug can never return silently.
    ...

def test_empty_deck_returns_no_due_cards():
    ...

def test_interval_never_exceeds_configured_maximum():
    ...
```

## Coverage

**Coverage** = the percentage of your code executed by tests. A rough guide to what to aim for:

| Layer | Target |
|---|---|
| Business / domain logic | **90–100%** — this is where the value is |
| API routes | 70–85% |
| Database layer | 70–85% |
| UI / presentation | Lower is fine; test behaviour, not markup |

Treat coverage as a **diagnostic**, not a goal. 100% coverage with no meaningful assertions is worse than 60% with good ones.

## Test doubles

| Double | Definition | Use |
|---|---|---|
| **Mock** | Fake object returning canned data | Asserting interactions |
| **Stub** | Fake returning a fixed response | Simplifying a dependency |
| **Fake** | Working simplified implementation | In-memory database in tests |
| **Spy** | Real object recording how it was used | Verifying a call happened |
| **Monkeypatch** | Temporarily replace something at runtime | Patching a slow or non-deterministic function |

```python
# tests/test_cards_api.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)


def test_create_then_read_card():
    response = client.post("/api/cards", json={"front": "What is 2+2?", "back": "4"})
    assert response.status_code == 201
    card_id = response.json()["id"]

    fetched = client.get(f"/api/cards/{card_id}")
    assert fetched.status_code == 200
    assert fetched.json()["front"] == "What is 2+2?"


def test_rejects_empty_card():
    response = client.post("/api/cards", json={"front": "", "back": ""})
    assert response.status_code == 422


def test_cannot_read_another_users_card():
    """The access-control test. Write these — they are the tests that matter."""
    ...
```

## Manual and exploratory testing

Automation cannot judge some things. Every release still needs a human to ask: does it feel right? is anything confusing? does the error message make sense?

---

# Part 10 — Version control and team workflow

**Non-negotiable. No exceptions, from your first line.**

## Git essentials

```bash
git init                        # create a repository
git status                      # what has changed?
git add .                       # stage changes
git commit -m "msg"             # save a snapshot
git log --oneline --graph       # history
git diff                        # what exactly changed?
git switch -c feature/login     # new branch
git switch main                 # go back
git merge feature/login         # combine
git revert <sha>                # undo safely (unlike reset)
```

## Branching strategies

| Strategy | Structure | Best for |
|---|---|---|
| **GitHub Flow** | `main` + short-lived feature branches | Continuous deployment. Most common. |
| **Git Flow** | `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` | Versioned releases. Heavier. |
| **Trunk-based** | Everyone commits to `main` frequently, feature flags hide incomplete work | Mature teams, highest velocity, needs strong tests |

## Commit messages that get you hired

```bash
# ✗ Bad — no information, unfalsifiable
git commit -m "fixed stuff"
git commit -m "update"

# ✓ Good — type, scope, what changed, why
git commit -m "fix(auth): reject refresh tokens already used

A refresh token could be redeemed twice within its window,
producing two valid sessions for one login. Added a jti record
and rejected reuse. Closes #412."
```

**Conventional Commits:** `type(scope): summary` where type ∈ `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.

## .gitignore

```gitignore
__pycache__/
*.py[cod]
.venv/
.env                     # ← CRITICAL: never commit secrets
*.db
.pytest_cache/
.mypy_cache/
.DS_Store
```

**Before your first commit: add `.env` to `.gitignore`.** Once a secret is in git history it is compromised, even if you delete it later.

## Pull requests

```bash
git switch -c feature/spaced-repetition
# ... work, commit often ...
git push -u origin feature/spaced-repetition
# Open a PR on GitHub. Describe:
#   - what this does
#   - why
#   - how you tested it
#   - any risks or things reviewers should look at closely
```

**Review checklist (yours, when reviewing others):**

- Does it do what the ticket said, and nothing more?
- Is logic in the right layer? Is anything on the wrong side of a boundary?
- Any injection risk, hardcoded secret, unvalidated input?
- Are there tests, and do they test behaviour rather than implementation?
- Will this be understandable in six months?
- What is *not* covered, and does that matter?

## GitHub as a portfolio

**This matters more than any certificate.** A recruiter will look at your GitHub before your resume.

Make each repository have:

| Element | Why it matters |
|---|---|
| A real README with a screenshot | Most people never scroll past it |
| Clear setup instructions that actually work | Proves you can be followed |
| Commit history that shows steady work | Reads as a work ethic |
| Tests that pass | A green badge |
| A CI pipeline | Proves you understand automation |
| Issues and a project board | Proves you have used a real workflow |
| A licence file | Shows professional awareness |

---

# Part 11 — CI/CD

## What it actually is

```
   You push code
         │
         ▼
┌────────────────────────────────────────────┐
│  AUTOMATICALLY:                            │
│    1. Install dependencies                 │
│    2. Lint / format check                  │
│    3. Run unit tests                       │
│    4. Run integration tests                │
│    5. Security scan (pip-audit)            │
│    6. Coverage report                      │
│    7. Build                                  │
│    8. Deploy to staging                    │
│    9. Run E2E tests against staging        │
   10. Deploy to production                  │
└────────────────────────────────────────────┘
         │
         ▼  if anything failed:
    You know in 90 seconds, not on Monday
```

## GitHub Actions (free, and enough)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Lint
        run: |
          pip install ruff
          ruff check .

      - name: Test with coverage
        run: |
          pip install pytest pytest-cov
          pytest --cov=. --cov-report=xml --cov-fail-under=80

      - name: Security audit
        run: |
          pip install pip-audit
          pip-audit

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage.xml
```

**This file alone signals to a hiring manager that you understand automation, quality gates, and continuous delivery.** It takes one afternoon to set up. It is the highest return-per-hour item in this entire document.

## What CI is really for

| Purpose | Explanation |
|---|---|
| **Fast feedback** | Know within minutes, not days |
| **Consistency** | Same checks on every machine, every PR, every contributor |
| **Confidence to deploy** | Green pipeline = safe release |
| **Documentation of the process** | The YAML *is* the build documentation |
| **Onboarding** | A new developer runs one command and gets the same environment |

---

# Part 12 — Phase 6: Deployment

| Concept | Meaning |
|---|---|
| **Environments** | **Dev** (laptop) → **Staging** (production clone for testing) → **Production** (real users). Never test on production. |
| **Configuration** | Same code, different settings per environment — connection strings, API keys, feature flags. Read from environment variables. |
| **Containers (Docker)** | Package app + all dependencies so it runs identically everywhere. Kills "works on my machine." |
| **Hosting: PaaS** | Render, Railway, Fly.io, Vercel. Easiest, cheapest, least control. **Start here.** |
| **Hosting: IaaS** | Your own VM (Hetzner, DigitalOcean, AWS EC2). Full control, more work. |
| **Hosting: Serverless** | AWS Lambda, Cloudflare Workers. Pay per request. Cold starts. |
| **Release: blue/green** | Run old and new side by side; switch traffic only if healthy. Instant rollback. |
| **Release: canary** | Release to 5% of users first, watch metrics, then widen. |
| **Release: rolling** | Replace instances gradually. |

## Docker, briefly

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
docker build -t myapp .
docker run -p 8000:8000 --env-file .env myapp
```

## A real Python deployment (the actual commands)

```bash
# 1. Package and pin dependencies
pip install poetry
poetry init
poetry add fastapi "uvicorn[standard]" sqlalchemy psycopg2-binary alembic
poetry lock                    # → poetry.lock pins exact versions

# 2. Database migrations
alembic init alembic
alembic revision --autogenerate -m "create cards table"

# 3. Process model
# gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker

# 4. Containerise, or deploy straight to a PaaS

# 5. After deploy
curl -s https://myapp.com/api/health     # does the live system answer?
```

**A health endpoint is the single most useful five lines you will write:**

```python
@app.get("/api/health")
def health():
    return {"status": "ok"}
```

It lets your platform, your CI, and your monitoring all verify the app is alive.

---

# Part 13 — Phase 7: Maintenance & Phase 8: Retirement

## 13.1 Maintenance is the longest phase

| Type | What it covers |
|---|---|
| **Corrective** | Fixing bugs users reported |
| **Adaptive** | Platform changes — "Android API 36 changed this; the app crashes" |
| **Perfective** | Performance tuning, UX improvements, small features |
| **Preventive** | Refactoring, dependency updates *before* they break, tech debt paydown |
| **Monitoring / observability** | Crash reports, error tracking, analytics, uptime — continuously, in production |

**The key concept: tech debt.** Every shortcut costs 2–5× later. Maintenance is where it is paid — or where it compounds into something unmaintainable.

## 13.2 Observability

> **You cannot maintain what you cannot see.**

| Signal | What it tells you | Tool |
|---|---|---|
| **Logs** | What happened, in detail | `structlog` / `logging` + a log service |
| **Metrics** | Trends over time: latency, error rate, throughput | Prometheus, Grafana |
| **Traces** | Where in the request chain the time went | OpenTelemetry |
| **Error tracking** | Every crash, grouped, with the stack trace and the user | Sentry |
| **Crash reporting** | Same, for mobile | Firebase Crashlytics |
| **Uptime checks** | Is it reachable from the outside? | Platform health checks, UptimeRobot |
| **Analytics** | Are users actually using the feature? | PostHog, Plausible |

```python
import logging
import time
from functools import wraps

logging.basicConfig(level=logging.INFO)
log = logging.getLogger(__name__)


def timed(name):
    """Decorator: log duration of every call to the decorated function."""
    def decorator(fn):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            try:
                return fn(*args, **kwargs)
            except Exception:
                log.exception("%s failed", name)      # full traceback, still no secrets
                raise
            finally:
                log.info("%s took %.1fms", name, (time.perf_counter() - start) * 1000)
        return wrapper
    return decorator
```

## 13.3 Retirement

When a product is replaced or shut down — a real phase that professionals handle deliberately:

- Export and archive all user data in a portable format
- Give users notice and a migration path
- Shut down infrastructure; revoke credentials and certificates
- Preserve code and documentation for anyone who needs it
- Meet legal data-retention obligations
- Hand over to the successor system or close cleanly

---

# Part 14 — The complete document set

This is what professionals actually write before and during development.

| # | Document | The question it answers | Read by |
|---|---|---|---|
| 1 | **Product vision / pitch** | What is it, why, for whom? | Everyone |
| 2 | **Requirements (SRS)** | What must it do? Functional + non-functional, each with an ID and priority, MoSCoW | Dev, QA, client, regulators |
| 3 | **Feasibility & risk report** | Should we do this? What could go wrong? | Management, investors |
| 4 | **Use case diagram + narratives** | Who does what, in what order, with what pre/post conditions? | Dev, UX, QA |
| 5 | **Data Flow Diagram** | How does data move and transform? | Architects, analysts |
| 6 | **ERD / data model** | What data exists and how is it related? | Backend, DBA |
| 7 | **Architecture diagram** | What are the components and how do they connect? | Whole team |
| 8 | **SDD (design document)** | How is each module designed? Class diagrams, sequence diagrams, API contracts | Developers |
| 9 | **API specification** | How do the pieces talk? Endpoints, shapes, error codes, status codes | Frontend, backend, partners |
| 10 | **Security / threat model** | What can go wrong, who would, and what defends it? | Security, dev, compliance |
| 11 | **Wireframes / mockups / prototypes** | What does it look like and how is it used? | UX, UI, dev |
| 12 | **Style guide / design system** | Exact colours, fonts, spacing, component rules | UI, dev |
| 13 | **Tech stack decision record (ADR)** | Why this technology and not the alternative? | Dev team, future maintainers |
| 14 | **Project plan (WBS / Gantt)** | Who does what, when? | PM, team, client |
| 15 | **Test plan & test cases** | How do we prove it works? | QA, dev, client |
| 16 | **Coding standards / linter config** | How must code be written? | Developers |
| 17 | **Privacy & compliance doc** | What data, why, how long, who sees it? | Legal, compliance |
| 18 | **README + docs** | What is this, how do I run it, how is it structured? | Developers, evaluators |
| 19 | **User manual / help docs** | How does a non-developer use it? | End users |
| 20 | **Deployment runbook** | How do we ship? What do we do at 2am when it breaks? | DevOps, on-call |
| 21 | **Changelog** | What changed in each release? | Users, team |

**The rule for every non-technical document:**

> Write it **for a specific reader with a specific question.** Not "the System Design Document" but *"Marc, who joins in month four and has never seen this code: here is where the scheduling logic lives."*

Documents fail because nobody had a reader in mind.

---

# Part 15 — Python reference

Python has the lowest barrier to writing correct software, and it is the primary language in backend engineering, data analysis, automation, cybersecurity, and machine learning. This part is the language and tooling reference for the whole guide.

## Where Python is the dominant choice

| Area | Why Python |
|---|---|
| **Cybersecurity** | The scripting language of the field. Exploit development, automation, tooling, and security engineering are largely Python. |
| **Machine learning / AI** | PyTorch, scikit-learn, TensorFlow, Hugging Face. Overwhelmingly Python-first. Also the language of research papers. |
| **Backend web** | FastAPI, Django, Flask. Clean, typed, testable, boring in the best way. |
| **Data analysis** | pandas, NumPy, matplotlib. The default in every data field. |
| **Automation** | The best general-purpose scripting language by a wide margin. |
| **Interviews / learning** | Closest to pseudocode. Readable. Fewer syntax puzzles between you and the concept. |

## Tier 1 — Language fundamentals

```python
# Variables and types
name: str = "Alice"
age: int = 20
scores: list[float] = [98.5, 87.0, 91.25]
profile: dict[str, str] = {"name": "Alice", "course": "CS"}

# Control flow
if age >= 18:
    status = "adult"
else:
    status = "minor"

for score in scores:
    print(score)

while True:
    if done:
        break

# Functions — the core unit of design
def calculate_average(numbers: list[float]) -> float:
    """Return the arithmetic mean of a non-empty list of numbers."""
    if not numbers:
        raise ValueError("cannot average an empty list")
    return sum(numbers) / len(numbers)

# Comprehensions — the Pythonic idiom
evens = [n for n in range(100) if n % 2 == 0]
lookup = {word: len(word) for word in ["a", "bb", "ccc"]}

# Classes and dataclasses
from dataclasses import dataclass

@dataclass(frozen=True)                 # frozen = immutable, hashable
class Card:
    id: int
    front: str
    back: str
    due_at: datetime
    ease: float = 2.5

    def is_due(self, now: datetime) -> bool:
        return self.due_at <= now


# Exceptions
class InsufficientFundsError(Exception):
    pass


def withdraw(balance: float, amount: float) -> float:
    if amount > balance:
        raise InsufficientFundsError(f"{balance} < {amount}")
    return balance - amount
```

**Essential topics:** variables, types, control flow, functions, lists, dicts, sets, tuples, comprehensions, file I/O, exceptions, modules, OOP, type hints, iterators/generators, decorators (basic), `dataclasses`, virtual environments, `pip`.

**Do not skip type hints.** They cost almost nothing and make your code self-documenting and IDE-navigable. Professionals use them on every function they write.

### Tier 2 — The professional Python stack

The tooling a professional Python developer reaches for, and why.

| Tool | Purpose | Learn when |
|---|---|---|
| **venv / uv** | Isolated environments | Month 1 |
| **git + GitHub** | Version control | Month 1 |
| **pytest** | Testing | Month 1 (basic), Month 3 (fluent) |
| **ruff** | Linting + formatting, very fast | Month 2 |
| **mypy / pyright** | Static type checking | Month 3 |
| **FastAPI** | Modern typed web APIs | Month 3 |
| **SQLAlchemy** | ORM / database access | Month 3 |
| **Alembic** | Database migrations | Month 3 |
| **PostgreSQL** | Relational database | Month 3 |
| **Docker** | Containerisation | Month 4 |
| **Pydantic** | Data validation | Month 3 |
| **requests / httpx** | HTTP client | Month 2 |
| **structlog** | Structured logging | Month 4 |

### Tier 3 — Domain libraries

A map of the ecosystem by domain. Learn the fundamentals above first; these are libraries you reach for once the language is comfortable.

**Cybersecurity** (Month 4):

| Area | Python tools |
|---|---|
| Networking | `socket`, `scapy`, `requests`, understanding TCP/IP |
| Web security | Understanding OWASP, `requests`, Burp Suite, JWT handling |
| Scripting/automation | `subprocess`, `os`, `pathlib`, `argparse` |
| Cryptography | `hashlib`, `hmac`, `cryptography`, `secrets` |
| Password auditing | `passlib`/`argon2-cffi` |
| Static analysis | Reading source for injection, secrets, unsafe deserialisation |
| CTF practice | picoCTF, TryHackMe, HackTheBox |

**Machine learning / AI** (Month 5):

| Area | Library |
|---|---|
| Numerics | NumPy |
| Data manipulation | pandas |
| Plotting | matplotlib, seaborn |
| Classical ML | scikit-learn |
| Deep learning | PyTorch |
| Large language models | Hugging Face `transformers` |
| Notebooks | Jupyter |
| Experiment tracking | Weights & Biases, MLflow |

**Backend engineering** (the through-line, Month 3):

FastAPI + SQLAlchemy + PostgreSQL + pytest + Docker. This skillset applies to all three domains and is what most Python job postings actually ask for.

## Setup

```bash
# Python version manager (install via uv — fastest option available)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Project
mkdir myproject && cd myproject
uv init
uv add fastapi "uvicorn[standard]" sqlalchemy alembic psycopg2-binary pydantic-settings
uv add --dev pytest pytest-cov ruff mypy httpx

# Run
uv run pytest
uv run ruff check .
uv run uvicorn app.main:app --reload
```

**Use `uv`, not bare `pip`, once you are past week one.** It is dramatically faster and resolves dependency conflicts that would otherwise waste hours of your life.

---

---

# Part 16 — Appendix: glossary and reference

## Glossary

| Term | Definition |
|---|---|
| **Abstract** | Hiding complexity behind a simple interface |
| **API** | The contract for how two software components communicate |
| **Assertion** | A statement in code that must be true; raises if it is not |
| **Backend** | Server-side: business logic, data, security |
| **Cache** | A fast copy of data stored to avoid recomputation |
| **CI** | Continuous Integration — automatic build and test on every change |
| **CD** | Continuous Delivery/Deployment — automatic release |
| **Commit** | A saved snapshot of your code in Git |
| **Commit message** | The written explanation of what changed and why |
| **Coupling** | How much one module depends on another. High coupling is bad |
| **Deployment** | Putting code onto real infrastructure |
| **Dependency** | A library your project uses |
| **Docker** | A tool for packaging an app with all its dependencies |
| **Endpoint** | A specific URL your API responds to |
| **Environment variable** | Configuration supplied from outside the code |
| **Framework** | Pre-built structure you write code into (Django, FastAPI) |
| **Git** | Distributed version control system |
| **Idempotent** | An operation that produces the same result if repeated |
| **Interface** | The set of operations a module exposes |
| **Layer** | A horizontal band of responsibility (presentation / logic / data) |
| **Library** | Code you call |
| **Linting** | Automated style and error checking |
| **Microservice** | A small independently deployed service |
| **Migrating** | Versioned, scripted changes to a database schema |
| **Module** | A Python file; a unit of code |
| **Mutability** | Whether a value can be changed after creation |
| **ORM** | Object-Relational Mapper — work with tables as Python objects |
| **Package** | A directory of modules, distributable as one unit |
| **Parameterised query** | A query where values are sent separately, never interpolated |
| **PEP 8** | The Python style guide |
| **Pull request** | A request to merge one branch into another, with review |
| **Refactoring** | Improving code structure without changing behaviour |
| **Repository** | A project directory tracked by Git |
| **Repository pattern** | An abstraction layer between business logic and data access |
| **Runtime** | The period when code is actually executing |
| **Salt** | Random unique data added before hashing a password |
| **Serialization** | Converting objects to bytes (JSON, pickle) |
| **Staging** | A production-like environment for testing |
| **Static typing** | Declaring types for the checker rather than relying on inference |
| **Test double** | A stand-in for a real dependency in tests |
| **Token** | A string representing authorisation, e.g. a JWT |
| **Type hint** | `def f(x: int) -> str` — documentation and checking |
| **Virtual environment** | An isolated Python installation per project |
| **YAML** | A human-readable configuration format used by CI |

## The twenty questions to ask about any architecture

Before you commit to a design, be able to answer these. If you cannot, your design is not finished.

1. What are the system's components and what does each own?
2. Where does the business logic live, and can it be tested with no infrastructure?
3. What is the source of truth, and how does anything else stay in sync with it?
4. Which decisions does the server make, and which does the client? *(Correct answer: security decisions belong to the server.)*
5. How does data get validated, and where?
6. What happens if the database is unavailable?
7. What happens if two users modify the same record simultaneously?
8. How do we authenticate a user?
9. How do we authorise them per action?
10. What sensitive data do we hold, and how is it protected at rest and in transit?
11. What is the input we distrust, and what do we do with it?
12. What are the performance targets, and how do we measure them?
13. How do we scale if traffic grows 10×?
14. How do we deploy, and how do we roll back?
15. How do we know it is broken — logs, metrics, alerts?
16. What is our recovery plan if the database is corrupted?
17. How does a new developer get this running locally?
18. What is deliberately out of scope?
19. What is the biggest technical risk, and how are we reducing it?
20. What does this design make hard, and is that an acceptable trade?

## Closing note

The gap between a junior and a mid-level developer is not intelligence or talent. It is **repetition** — the same knowledge applied five or six times, until the process becomes automatic and the decisions become obvious.

Knowing how software is *supposed* to be built produces no ability at all. Build something, watch it fail, read the error, fix the cause, add the test, deploy it, get a user report, and do it again. That loop is the entire difference.

This document tells you what to build. Now go build it.

---

*Document version 1.0 · Companion to a working conversation · All code examples are illustrative and intended to be reimplemented by hand.*

