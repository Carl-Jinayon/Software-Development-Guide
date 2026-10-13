# Konek'ta — App Flow (Visual Reference)

**Purpose:** a one-page visual of how the app actually works. Diagrams only.

**Full detail lives in [`01_KONEK_TA_PRODUCT_SPEC.md`](01_KONEK_TA_PRODUCT_SPEC.md).**
Every box below shows a requirement ID (`FR-xxx`) so you can jump to the exact
acceptance criteria when you build it.

---

# 1. The Core Loop

This is the whole product. Everything else supports it.

```
        ┌──────────────────────────────────────────────┐
        │                                              │
        ▼                                              │
    ┌────────┐                                        │
    │  READ  │     a post in a topic you follow        │
    └────┬───┘                                        │
         │                                            │
         ▼                                            │
    ┌────────┐                                        │
    │  THINK │     it makes you think of something     │
    └────┬───┘                                        │
         │                                            │
         ▼                                            │
    ┌────────┐                                        │
    │  WRITE │     you write a linked post             │
    └────┬───┘                                        │
         │                                            │
         ▼                                            │
    ┌────────┐                                        │
    │ REACH  │     people who follow that topic see it │
    └────┬───┘                                        │
         │                                            │
         ▼                                            │
    ┌────────┐                                        │
    │ ENGAGE │     they link back, heart, or follow   │
    └────┬───┘                                        │
         │                                            │
         └────────────────────────────────────────────┘
                       ↺  back to READ
```

**What each step needs to exist:**

| Step | Feature | FR |
|---|---|---|
| Read | For You feed, filtered to your interests | FR-009 |
| Think | no comments — you write a whole post instead | FR-024 |
| Write | composer with text + one category | FR-015 |
| Write | the link to the post you are answering | FR-016 |
| Reach | other people's interests match yours | FR-009 |
| Engage | heart · bookmark · follow | FR-020…023 |

> **If you delete any one of these, the app is not Konek'ta any more.**

---

# 2. First Run — the user's journey

The path for someone who has never opened the app.

```
    OPEN APP
        │
        ▼
  ┌────────────────────┐
  │  LANDING           │   "what is this?" — 3 to 5
  │  FR-001            │   swipeable screens
  └─────────┬──────────┘
            │
      ┌─────┴──────┐
      ▼            ▼
 ┌───────────┐  ┌───────────┐
 │ REGISTER  │  │  LOGIN    │  ← returning, but
 │ FR-002    │  │  FR-004   │    session is gone
 └─────┬─────┘  └─────┬─────┘
       │              │
       ▼              │
 ┌───────────────┐    │
 │ VERIFY EMAIL  │    │   6-digit code
 │ FR-003        │    │   ⚠ can still browse while pending
 └───────┬───────┘    │
         │            │
         ▼            ▼
   ┌────────────────────┐
   │  INTEREST PICKER   │  ◄── MANDATORY GATE
   │  FR-007  FR-008    │      minimum 3 topics
   └─────────┬──────────┘      none there? you suggest one
             │
             ▼
   ┌────────────────────┐
   │   FOR YOU FEED     │  ◄── first time here
   │   FR-009           │      only your topics, newest first
   └────────────────────┘
```

### The two rules on this screen

```
 ┌──────────────────────┐   ┌──────────────────────────────┐
 │ CANNOT SKIP THE      │   │ VERIFYING EMAIL IS NOT A     │
 │ INTEREST PICKER      │   │ WALL IN FRONT OF READING     │
 │                      │   │                              │
 │ No "skip for now".   │   │ Pick interests → read feed.  │
 │ Fewer than 3 →       │   │ Only POSTING is blocked.     │
 │ blocked.             │   │                              │
 │                      │   │ Otherwise the 60-second       │
 │ Why: an empty feed   │   │ test fails on the very first  │
 │ reads as a broken    │   │ session.                     │
 │ app.                 │   │                              │
 └──────────────────────┘   └──────────────────────────────┘
```

---

# 3. Daily Use — the returning user's journey

```
    OPEN APP
        │
        ▼
  ┌────────────────────┐
  │   FOR YOU FEED     │   your categories only, newest first
  │   FR-009           │   pull to refresh · scroll for more
  └─────────┬──────────┘
            │  tap a post
            ▼
  ┌────────────────────┐
  │   POST DETAIL      │   shows the post + anything
  │   FR-030  FR-012   │   linked to it
  └─────────┬──────────┘
            │
      ┌─────┴──────────────────────┐
      ▼                            ▼
 ┌──────────────┐          ┌────────────────────┐
 │  react       │          │  WRITE A LINKED    │
 │              │          │  POST              │
 │ heart    ♥   │          │                    │
 │ bookmark  ⭑ │          │  this is THE       │
 │ share    ↗   │          │  primary action.   │
 │              │          │  never "comment".  │
 │ FR-020..022 │          │  FR-024  FR-016    │
 └──────┬───────┘          └─────────┬──────────┘
        │                            │
        │                            ▼
        │                  ┌────────────────────┐
        │                  │    COMPOSER        │
        │                  │                    │
        │                  │  text              │
        │                  │  + one category    │
        │                  │                    │
        │                  │  FR-015  FR-017    │
        │                  └─────────┬──────────┘
        │                            │
        │                            ▼
        │                  ┌────────────────────┐
        │                  │    PUBLISHED       │
        │                  │                    │
        └──────────────────┤  it is now just    │
                           │  another post      │
                           └─────────┬──────────┘
                                     │
                                     ▼
                           (goes to section 5)
```

---

# 4. A Linked Post — how a reply works

There is no comment box anywhere in this app. To answer someone, you write your
own post and link theirs.

```
  Post A   @mara · Gaming
  ┌────────────────────────────────┐
  │ "Finally finished the 40-hour  │
  │  build I'd been avoiding."     │
  └───────────────┬────────────────┘
                  │
                  │  Post B links to Post A
                  ▼
  Post B   @dev · Gaming
  ┌────────────────────────────────┐
  │ "Same. What broke it for me    │
  │  was doing the UI last."       │
  └───────────────┬────────────────┘
                  │
                  │  Post C links to Post B
                  ▼
  Post C   @kai · Gaming
  ┌────────────────────────────────┐
  │ "Counterpoint — UI last is     │
  │  fine if you scope it to one  │
  │  screen."                      │
  └────────────────────────────────┘
```

**Why this matters — each post is ranked on its own:**

```
  ┌─────────────────────────────────────────────────────────┐
  │  With COMMENTS            │  With LINKED POSTS          │
  ├───────────────────────────┼─────────────────────────────┤
  │  one thread, one ranking   │  every post ranked          │
  │  reply dies with the post  │  separately                │
  │  reply reaches 1 audience  │  reply reaches everyone    │
  │  "amazing lol" costs the   │  picking that category     │
  │  same as a real answer     │                             │
  └───────────────────────────┴─────────────────────────────┘
```

### Hard limits on linking

```
  ┌──────────────────────────────────────────────────────┐
  │  one parent      a post links to ONE post, not many  │
  │  max depth 3     ▓ → ▒ → ░   ░ then REJECTED (422)   │
  │  no self-link    a post cannot link to itself        │
  │  text is never   the child stores a REFERENCE,       │
  │  copied          never a copy of the parent's text   │
  └──────────────────────────────────────────────────────┘
```

**Why depth 3:** a longer chain makes "show me this thread" an unbounded walk on
every thread view, and a cycle becomes a denial-of-service.

**Why the text is never copied:** the author can delete their post without
corrupting yours. It is credit, not reposting.

---

# 5. Where a Published Post Goes

This is the moment the app exists for.

```
                    ┌─────────────────────┐
                    │   @you posts in    │
                    │     "Gaming"       │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        ┌───────────┐    ┌───────────┐    ┌───────────┐
        │  @mara    │    │  @dev     │    │  @kai     │
        │           │    │           │    │           │
        │ follows   │    │ follows   │    │ follows   │
        │ Gaming    │    │ Gaming    │    │ Recipes   │
        └─────┬─────┘    └─────┬─────┘    └─────┬─────┘
              │                │                │
              ▼                ▼                ▼
        ┌───────────┐    ┌───────────┐    ┌───────────┐
        │  ✅ sees │    │  ✅ sees │    │  ❌ does   │
        │     it   │    │     it   │    │  not see  │
        │          │    │          │    │  it       │
        │ For You  │    │ For You  │    │  wrong    │
        └───────────┘    └───────────┘    │  topic    │
                                          └───────────┘
```

**Nobody else sees it. That is the entire product promise.**

---

# 6. The Three Feeds

```
  ┌────────────────┬────────────────┬────────────────┐
  │    FOR YOU     │    EXPLORE     │   FOLLOWING    │
  │    FR-009      │    FR-010      │    FR-011      │
  ├────────────────┼────────────────┼────────────────┤
  │ your           │ anything,      │ only people    │
  │ categories     │ you filter     │ you follow     │
  │ ONLY           │ it             │                │
  ├────────────────┼────────────────┼────────────────┤
  │ newest first   │ newest first   │ newest first   │
  ├────────────────┼────────────────┼────────────────┤
  │ not your own   │ not your own   │ not your own   │
  │ posts          │ posts          │ posts          │
  ├────────────────┼────────────────┼────────────────┤
  │ blocked out    │ blocked out    │ blocked out    │
  │ both ways      │ both ways      │ both ways      │
  ├────────────────┼────────────────┼────────────────┤
  │ shown ONCE     │ shown ONCE     │ no category    │
  │ if it matches  │ if it matches  │ filter — a     │
  │ 2 of your      │ 2 filters      │ follow is an   │
  │ topics         │                │ opt-in         │
  └────────────────┴────────────────┴────────────────┘
```

### Why you can always tell why you saw something

```
  ┌──────────────────────────────────────────────────────────┐
  │  every post in your feed carries a small chip:          │
  │                                                          │
  │      ♥ 12   "because you follow Gaming"     ⭑  ↗        │
  │                          ▲                               │
  │                          └── FR-012. the feed explains   │
  │                              itself, always.             │
  └──────────────────────────────────────────────────────────┘
```

---

# 7. What Gets Filtered Out

Every feed, every time.

```
   ❌  someone who blocked you
   ❌  someone you blocked            ← both directions, always
   ❌  a deleted account
   ❌  a deleted post
   ❌  your own post                  ← it lives on your profile
   ⚠️  a post matching 2 of your topics  → shown ONCE, not twice
```

---

# 8. The Empty Feed — the app's biggest risk

A new user registers, picks three topics, and those topics are empty. They leave,
and no amount of backend work fixes it.

```
   ❌ WITHOUT SEEDING

   @new user picks: Gaming · Cooking · Music
                    │       │       │
                    ▼       ▼       ▼
              ┌───────────────────────┐
              │                       │
              │                       │
              │      (nothing)        │   "this app is broken"
              │                       │
              │                       │
              └───────────────────────┘


   ✅ WITH SEEDING  (200+ posts before launch)

   @new user picks: Gaming · Cooking · Music
                    │       │       │
                    ▼       ▼       ▼
              ┌───────────────────────┐
              │ a post │ a post │ a post │
              └───────────────────────┘

   ┌────────────────────────────────────────────────────────┐
   │  3 questions to answer BEFORE launch:                  │
   │                                                        │
   │  1. Who writes the 200 seed posts?  (content work)     │
   │  2. Which 20 categories get them first?                │
   │  3. How many real people can we get into the beta?     │
   │                                                        │
   │  This is not a coding task. It cannot be done late.    │
   └────────────────────────────────────────────────────────┘
```

---

# 9. Where Everything Runs

```
   ┌─────────────────────────────┐
   │  ANDROID APP                │
   │  screens · navigation       │
   │  (your partner)             │
   └──────────────┬──────────────┘
                  │  HTTPS + JWT
                  ▼
   ┌─────────────────────────────┐
   │  FASTAPI                    │
   │                             │
   │  api/          ← HTTP only │
   │  domain/   ← the LOOP lives here ─┐
   │  repositories/ ← SQL ────────┐    │
   └─────────────────────────────┘  │    │
                                   │    │
                       ┌───────────┴────┴───┐
                       ▼                    ▼
              ┌─────────────┐      ┌─────────────┐
              │ PostgreSQL  │      │  the tests  │
              └─────────────┘      │  for domain/│
                                     │  run in ms  │
                                     │  no DB, no  │
                                     │  no server  │
                                     └─────────────┘
```

> **The rule:** the core loop lives in `domain/` and imports nothing from FastAPI
> or SQLAlchemy. That is what makes it testable in milliseconds instead of seconds.

---

# 10. The Whole Thing on One Page

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                 │
  │   LANDING → REGISTER → VERIFY → PICK TOPICS → FEED             │
  │                                                  │              │
  │                                                  ▼              │
  │                                    READ a post you follow       │
  │                                                  │              │
  │                                                  ▼              │
  │                                    THINK of something            │
  │                                                  │              │
  │                                                  ▼              │
  │                          WRITE a linked post (never a comment)   │
  │                                                  │              │
  │                                                  ▼              │
  │                            PEOPLE WHO FOLLOW THAT TOPIC SEE IT  │
  │                                                  │              │
  │                                                  ▼              │
  │                                  THEY LINK BACK, HEART, FOLLOW   │
  │                                                  │              │
  │                                                  └──────► loop  │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘

  CUT FROM v1:  images · video · ML categorisation · communities
                donations · reveal mechanic · DMs · comments
                notifications · search · trending · editing

  BUILT FOR v1: relevance feed · linked posts · heart · bookmark
                share · follow · interests · profile · settings
                + block · report · delete account
```

---

## Reference

| Need | Go to |
|---|---|
| Full requirements, acceptance criteria | [`01_KONEK_TA_PRODUCT_SPEC.md`](01_KONEK_TA_PRODUCT_SPEC.md) Part 4 |
| Why these features were cut | Same doc, Part 8 |
| Database tables, ERD, indexes | Same doc, Part 3 |
| API contract between you and your partner | [`../MY_PLAN/02_MY_APP_PROJECT_SPEC.md`](../MY_PLAN/02_MY_APP_PROJECT_SPEC.md) §4 |
