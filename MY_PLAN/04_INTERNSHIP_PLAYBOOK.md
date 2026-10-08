# Internship Hunt Playbook

**Personal. Starts Week 20 — apply during Month 5, not after Month 6.**

---

## Why this starts early

**Applying in Week 24 is the most common mistake.** Applications take time, involve waiting, and often require follow-up. Applying in Week 20 means I interview while I am still building, which is when I can talk about recent work with genuine enthusiasm.

---

## My positioning

**The one line that goes on my CV, in my cover letter, and in my intro:**

> **Full-stack Python developer who builds secure systems and can explain the architecture behind them.**

Not "a CS student who knows Python." Not "a beginner seeking an internship." A specific, defensible claim.

### Why this positioning

| Claim | Why it works |
|---|---|
| **Full-stack** | The largest junior hiring surface. Most common entry point |
| **Python** | Backend + data + security + ML in one language |
| **Secure systems** | The AppSec niche — more valuable than either skill alone |
| **Can explain the architecture** | The mid-level differentiator the user asked about |

### What I do NOT claim

- ❌ "AI/ML engineer" — I have one project, not a foundation. Overclaiming invites questions I cannot answer.
- ❌ "Full-stack Android/iOS developer" — one Kotlin project is not mobile experience.
- ❌ Any framework I have only read about.

**Underclaiming is recoverable. Overclaiming is fatal in an interview.**

---

## The application plan

| Phase | When | Action |
|---|---|---|
| **Prepare** | Weeks 18–19 | Portfolio polished, CV written, GitHub clean |
| **Apply** | Weeks 20–22 | 5–8 applications per week |
| **Follow up** | Ongoing | Follow up after 5–7 days with no response |
| **Interview** | Weeks 22–24 | Interview prep in parallel |

### Where to apply

| Source | Notes |
|---|---|
| **Direct applications** | Company careers pages — highest response rate, no algorithm filter |
| **Job boards** | LinkedIn, Indeed, JobStreet (PH), BuiltIn |
| **Referrals** | The highest conversion rate by a wide margin. Ask classmates, lecturers, alumni |
| **GitHub-driven** | Find companies whose repos interest me. Contribute, then apply |
| **Internship programmes** | Large companies have structured intern programmes |

**Reality check:** most large companies filter with an online assessment. Small and medium companies are more likely to read a CV and actually talk to me. **Apply broadly, but weight toward smaller teams** — where architecture and design actually get discussed.

---

## The resume

**One page. No more.** A second page means I have not prioritised.

### Structure

```markdown
# Name
City, Country · email · GitHub · LinkedIn

## Summary                                    ← 2 lines, no "aspiring" or "passionate"
Full-stack Python developer with a deployed, tested, and secured
API and an Android client. Focused on clean architecture and
application security.

## Projects                                   ← BEFORE education. This is my differentiator.
### Study Deck — full-stack spaced-repetition app
[link] · [live demo]
Python · FastAPI · PostgreSQL · Docker · GitHub Actions · Kotlin/Compose
- Built a REST API with layered architecture; business logic isolated from
  framework code, 94% test coverage on the domain layer
- Deployed with a green CI pipeline running lint, types, tests, and dependency
  security audits
- Wrote a STRIDE threat model; enforced Argon2id hashing, JWT rotation, and
  object-level authorisation on every endpoint
- Reduced p95 response time from 410ms to 90ms by fixing an N+1 query

### Login anomaly detector — AppSec + ML
Python · scikit-learn · FastAPI · Docker
- Trained a classifier to flag impossible login patterns, evaluated against a
  threshold baseline (F1, precision/recall, ROC-AUC)
- Exposed as a service behind rate limiting, with adversarial-input testing

## Experience
Any internships, freelance, class projects worth mentioning — with outcomes.

## Skills
**Languages:** Python, Kotlin, SQL, Bash
**Backend:** FastAPI, SQLAlchemy, PostgreSQL, Pydantic, Alembic
**Testing:** pytest, Hypothesis, coverage, integration/E2E
**DevOps:** Docker, GitHub Actions, GitHub Flow, Render/Neon
**Security:** OWASP Top 10, STRIDE, Argon2id, JWT rotation, Bandit, pip-audit
**Practices:** code review, ADRs, threat modelling, Scrum
```

### The rules

| Rule | Why |
|---|---|
| **Projects before education** | I have no internships. Projects are my experience |
| **Numbers, not adjectives** | "94% coverage" not "thorough testing" |
| **Every bullet shows an outcome** | Not "used Redis" but "cut p95 from 410ms to 90ms" |
| **Link everything** | Live URL, repo, demo video |
| **No "aspiring", "passionate", "hardworking"** | Everyone writes them. They mean nothing |
| **One page** | Anything more is unread |
| **Tailor the summary per application** | A security firm gets a different line than a startup |

---

## The GitHub profile

**Recruiters check GitHub before they read the CV.** This is not optional.

### Profile README

```markdown
# Hi, I'm [Name]

Third-year CS student. Building in public.

**Currently:** building a spaced-repetition study app (FastAPI + Kotlin)
**Learning:** security engineering, machine learning
**Seeking:** a full-stack / AppSec internship

## What I have built
| Project | Stack | What it demonstrates |
|---|---|---|
| [Study Deck](link) | Python, FastAPI, Postgres, Docker | Architecture, auth, testing, CI/CD |
| [Anomaly detector](link) | Python, scikit-learn | Security + ML, model evaluation |
| [Algorithms](link) | Python | Implementations from scratch, benchmarked |

## Currently learning
- Rust · Go · Anything my pull requests get rejected on

## Contact
- Email · LinkedIn
```

### Every repository needs

| Element | Why it matters |
|---|---|
| README with a screenshot | Most people never scroll past it |
| Setup instructions that **actually work** | Proves I can be followed |
| A green CI badge | Proves I understand automation |
| Tests that pass | Proves I verify my own work |
| Commit history showing steady work | Reads as work ethic |
| A licence file | Professional awareness |
| Issues and a project board | Proves I have used a real workflow |

### Portfolio hygiene checklist

- [ ] Profile README written and current
- [ ] Every project README has a screenshot at the top
- [ ] Every repo has a green CI badge
- [ ] Setup instructions tested on a clean machine
- [ ] No placeholder text anywhere (`<your name>`, `TODO`, `Lorem ipsum`)
- [ ] Licences on every repo
- [ ] Commit history shows consistent work, not one giant commit
- [ ] Contact details in the profile

---

## LinkedIn

**Optional but cheap and effective.**

| Section | What to write |
|---|---|
| **Headline** | "Full-stack Python Developer · Application Security · Building a study app" — not "CS Student Seeking Internship" |
| **About** | Two paragraphs. What I build, what I am learning, what I want |
| **Experience** | Projects presented as experience if legitimate |
| **Skills** | Add the specific tools. They are searchable |
| **Featured** | Pin the best repo and the live demo |
| **Open to work** | Turn it on with the "internship" signal — it is visible to recruiters |

**Connect with:** recruiters at target companies, engineers whose work I admire, alumni from my school. Send a personal note, not "I'd like to connect."

---

## Interview preparation

### What gets asked, by stage

| Stage | What they assess | Prepare |
|---|---|---|
| **HR / recruiter screen** | Why this company, why this role, communication | My pitch, my story, salary expectations |
| **Technical screen (call)** | Basic coding, debugging, project knowledge | Can I explain every line of my own project? |
| **Take-home / assessment** | Coding, algorithm, code review | DSA basics, idiomatic Python |
| **Onsite — coding** | Algorithms, data structures, complexity | `../GUIDES/02_ALGORITHMS_AND_DATA_STRUCTURES.md` |
| **Onsite — system design** | Architecture, trade-offs, failure modes | `../GUIDES/06_ARCHITECTURE_AND_SYSTEM_DESIGN.md` |
| **Onsite — behavioural** | Teamwork, conflict, ownership | Two stories prepared |
| **Onsite — project deep dive** | Do I actually understand my own work? | Everything I wrote |

### The two stories I must have ready

**Story 1 — A bug I found and fixed.**

```markdown
**Situation:** [Context]
**Problem:** [What was wrong, and how I noticed]
**Investigation:** [How I narrowed it down — the debugging process]
**Root cause:** [The actual cause, not the symptom]
**Fix:** [What I changed and why]
**Prevention:** [The test I added so it cannot return]
**Lesson:** [What I would do differently]
```

**Story 2 — A design decision I made and defended.**

```markdown
**Context:** [What problem I was solving]
**Options:** [At least two I considered]
**Decision:** [What I chose]
**Reasoning:** [Why — with trade-offs, not absolutes]
**Outcome:** [What happened, with a number if possible]
**What I'd change now:** [Honest self-assessment]
```

**These two stories demonstrate more than any technical answer.** They show investigation, judgement, ownership, and self-awareness.

### Questions to ask them

Asking good questions signals that I am evaluating them too.

- "What does a typical first month look like for this role?"
- "How does the team decide what to build?"
- "What's the biggest technical problem the team is facing right now?"
- "How does code review work here?"
- "What's something that surprised you in your first three months?"
- "Where do interns get to contribute in the first eight weeks?"

### Questions to expect

| Question | How to answer |
|---|---|
| "Why should we hire you?" | "I have shipped a deployed API with real auth, a threat model, and a green CI pipeline. Here is the live URL." |
| "What is your biggest weakness?" | A real one, with what I am doing about it. Not a disguised strength |
| "Do you have questions for us?" | Always. Never "no" |
| "Where do you see yourself in 3 years?" | Tied to their stack and their growth, not a generic ladder |
| "Tell me about a project" | The deep-dive story. This is where I win |

### Questions to ask myself before every interview

- [ ] Can I explain every line of code in my portfolio projects?
- [ ] Do I know why each architectural choice was made?
- [ ] Can I name a decision I would now make differently?
- [ ] Have I actually deployed, or only developed?
- [ ] Can I talk about a security vulnerability I found and fixed?
- [ ] Have I read anything about this company?

---

## Common failure modes

| Mistake | Consequence | Prevention |
|---|---|---|
| Applying in the final week | No time for follow-up | Apply from Week 20 |
| No live URL | "Does it actually run?" | Deploy everything |
| Resume longer than one page | Unread | Cut to one page |
| All projects, no depth | Nothing to discuss | Two or three projects I can defend in depth |
| Overclaiming a framework | Caught in the interview | Claim only what I have used |
| Same CV to every company | Obvious | Tailor the summary |
| No GitHub | Nothing to verify | Every project public |
| Cannot explain my own code | Fatal | Write it, read it, explain it out loud |
| No questions for them | Disinterest | Prepare five |
| Ghosting after an interview | Burns the bridge | Follow up within 24 hours |

---

## Follow-up template

```markdown
Subject: Following up — <Role>, <Company>

Hi <Name>,

Thank you for the conversation on <date> about the <Role> position. I
particularly enjoyed hearing about <specific thing they mentioned> — <one
sentence connecting it to something I have built>.

Since we last spoke I <specific thing: deployed X, added Y, finished Z>.
The live version is here: <link>.

I'd be glad to answer any further questions. Happy to share more detail on
<project most relevant to the role>.

Best regards,
Name
```

**Send within 3–5 business days of no response. After an interview, within 24 hours.**
Reference something specific from the conversation. Generic follow-ups are ignored.

---

## Application tracker

```markdown
| # | Company | Role | Applied | Source | Status | Follow-up | Notes |
|---|---|---|---|---|---|---|---|
| 1 | | | | Referral | Applied | | |
| 2 | | | | Direct | Interview scheduled | | |
```

**Track it.** Applications lost in an inbox are applications that will be forgotten, and following up is where a large share of offers actually come from.

---

## The week before an interview

- [ ] Re-read my own README for every project
- [ ] Deploy anything that is not deployed
- [ ] Read the job description again and map my experience to each requirement
- [ ] Prepare my two stories properly, out loud
- [ ] Review the company's product and tech stack
- [ ] Prepare questions for them
- [ ] Test the meeting setup if it is remote
- [ ] Sleep properly the night before

**The most common interview failure is being unable to explain your own project.** If I wrote it, I should be able to discuss every decision, trade-off, and bug in it without notes.
