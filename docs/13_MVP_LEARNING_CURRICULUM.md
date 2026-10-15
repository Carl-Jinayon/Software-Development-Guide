# MVP Learning Curriculum — Konek'ta (Full Topic List, Ordered)

The exact list of topics you need, what each one is, and how to know you've learned it.  
Rule: **you are not "done" a topic until you can demonstrate it in ≤20 lines of code or one command.**  
Use the **Needed for** column: 🔴 before the MVP starts · 🟡 during a specific MVP week · 🟢 after launch.

---

## Track A — Foundations (Week 0) 🔴

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| Terminal basics | `cd`, `ls`, `mkdir`, `cat`, environment variables | You can navigate folders and print a file's contents | 🔴 |
| Git fundamentals | commit, branch, merge, diff, PR | You can undo a bad commit and open a PR | 🔴 |
| How the internet works | request → server → response, DNS/TLS in one sentence | You can explain why the app breaks when offline | 🔴 |
| **HTTP** | methods (GET/POST/PATCH/DELETE), URLs, headers, body, query params | You can explain the difference between `/posts` and `/posts?category=Gaming` | 🔴 |
| **Status codes** | 200/201/204/400/401/403/404/409/422/429/500 | You can pick the right code for "wrong password" vs "no permission" vs "invalid field" | 🔴 |
| **JSON** | objects, arrays, strings, numbers, null, `fromJson`/`json.loads` | You can write a JSON payload by hand and read one | 🔴 |
| REST basics | resources as nouns, statelessness, versioning | You can name 6 endpoints for Konek'ta | 🔴 |
| Pydantic models (intro) | schema = data contract | You can declare a model with types and required fields | 🔴 |

---

## Track B — Python for the backend (Weeks 0–1) 🔴

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| Variables & types | `str`, `int`, `bool`, `float`, `None` | You can annotate a function's params and return | 🔴 |
| Collections | list, dict, set, tuple; slicing; comprehension | You can build a list of dicts from another list | 🔴 |
| Functions | def, args, defaults, return | You can write `def rank(posts, interests)` | 🔴 |
| Modules & imports | `from x import y`, package layout | You know why `domain/` must not import FastAPI | 🔴 |
| Exceptions | try/except/raise, custom errors | You can raise a 404 with a clear message | 🔴 |
| File & env handling | read `.env`, never hardcode secrets | You can load `DATABASE_URL` from env | 🔴 |
| Type hints | `str`, `int \| None`, `list[dict]` | mypy passes on your modules | 🟡 |
| Dataclasses / Pydantic models | structured data with validation | You can validate incoming JSON | 🔴 |
| Async basics | `async def`, `await`, why I/O is slow | You can explain why we don't block on DB calls | 🟡 |
| Decorators (light) | what `@app.get` does | You can add a route by copying a pattern | 🔴 |
| `pathlib` / JSON files | reading fixtures for tests | You can load a test JSON file | 🟡 |

---

## Track C — SQL, PostgreSQL, data modeling (Weeks 2–3) 🔴

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| Tables, rows, columns | the shape of data | You can create a `users` table | 🔴 |
| `PRIMARY KEY` | unique row identity | You can explain why `id` is the PK | 🔴 |
| `UNIQUE` + `NOT NULL` | rules the DB enforces | You can prevent duplicate emails in the DB | 🔴 |
| **Foreign keys** | links between tables | You can link `posts.category_id → categories.id` | 🔴 |
| `CHECK` constraints | e.g. body length 1–2000 | You can enforce a rule without app code | 🔴 |
| **Cascades** (CASCADE/RESTRICT/SET NULL) | what happens on delete | You can explain why `parent_post_id` is SET NULL | 🔴 |
| CRUD SQL | SELECT/INSERT/UPDATE/DELETE | You can write all four without copying | 🔴 |
| `JOIN` | combining tables | You can join posts+users+categories | 🔴 |
| Subqueries & `IN` | `WHERE category_id IN (SELECT ...)` | You can filter a feed by interests | 🔴 |
| `ORDER BY` / `LIMIT` | sorting and paging | You can return the newest 20 posts | 🔴 |
| **Indexes** | speed-ups; equality first, range last | You can add `(category_id, created_at DESC)` and `EXPLAIN` | 🔴 |
| Aggregation | `COUNT`, `GROUP BY` | You can count hearts per post | 🟡 |
| `INSERT ... ON CONFLICT` | upsert for idempotency | You can make a toggle safe under races | 🟡 |
| Transactions | all-or-nothing operations | You can make heart + counter atomic | 🟡 |
| **Data modeling** | entities, relationships, rules | You produced the ERD | 🔴 |
| Migrations (Alembic) | versioned schema changes | You can migrate up and roll back | 🟡 |

---

## Track D — FastAPI + API design (Weeks 4–7) 🔴

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| App factory | `create_app()` | Your `main.py` has no logic | 🔴 |
| Routes | `@app.get/post/patch/delete` | You can add an endpoint in 5 lines | 🔴 |
| Path/query/body params | three kinds of input | You can tell them apart in your own words | 🔴 |
| Response models | what the API promises to return | A schema guarantees no `password_hash` leaks | 🔴 |
| Dependency injection | `Depends(get_session)` | You can swap the DB in tests | 🟡 |
| Lifespan / startup | engine creation, DB connect | Server boots cleanly with no warnings | 🟡 |
| Exception handlers | one error format (RFC 9457) | All errors look the same to the client | 🔴 |
| Middleware (light) | request id, logging, CORS | Every log line has a trace id | 🟡 |
| **OpenAPI / Swagger** | auto-generated docs | `/docs` matches your contract exactly | 🔴 |
| API versioning | `/api/v1` | You never break a shipped shape | 🟢 |
| Idempotency | same request twice = same result | Double-tap on heart creates one row | 🟡 |

---

## Track E — Security essentials (Week 5, then throughout) 🔴

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| Password hashing (Argon2id) | one-way slow hash | You never store or log a password | 🔴 |
| Salt & why SHA-256 is wrong | rainbow tables, speed | You can explain it in 2 sentences | 🔴 |
| **JWT** access + refresh | signed tokens, rotation | Reusing a refresh token revokes the session | 🔴 |
| Account enumeration | leaking "email exists" | login and reset return identical responses | 🔴 |
| Rate limiting | 5/min IP, 10/hr account | 6th attempt returns 429 | 🔴 |
| Authorization (BOLA) | can this user touch this row? | you cannot read another user's data | 🔴 |
| Mass assignment | posting `role: admin` | `extra="forbid"` blocks it | 🔴 |
| SQL injection | unparameterised SQL | you never use f-strings in SQL | 🔴 |
| PII handling | what never leaves the server | `birthday`/`phone` never in a response | 🔴 |
| HTTPS + secrets management | TLS, env-only secrets | no secret in git, ever | 🔴 |
| OWASP Top 10 (awareness) | the standard list | you can walk your API against it | 🟡 |
| STRIDE threat modelling | structured threat thinking | your threat model has mitigations | 🟡 |
| Erasure / privacy | delete + purge + retention | you can delete an account and its data | 🟡 |

---

## Track F — Flutter / Dart client (Weeks 0, 7–13) 🔴

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| Dart basics | variables, lists, maps, async/await | You can fetch JSON and print it | 🔴 |
| Widgets | `Scaffold`, `AppBar`, `Text`, `Column` | You can build a static screen | 🔴 |
| Stateless vs Stateful | rebuild vs mutate | You know when `setState` is needed | 🔴 |
| Lists | `ListView.builder` | You can render 20 posts | 🔴 |
| Navigation | `Navigator.push`, routes, named routes | You can go login → picker → feed | 🔴 |
| Forms + validation | `TextFormField`, validators | You can validate a login form | 🔴 |
| HTTP calls | `http` / Dio, timeouts, errors | You can show an error state on failure | 🔴 |
| Token storage | secure storage + interceptor | The bearer token is attached automatically | 🟡 |
| State management (light) | Provider/Riverpod/BLoC (pick one) | Feed state survives navigation | 🟡 |
| Loading / empty / error states | the three states every screen needs | Each screen handles all three | 🔴 |
| Pagination in UI | scroll to load next page | Cursor continues without duplicates | 🟡 |
| Accessibility basics | screen readers, tap targets, contrast | Android "TalkBack" can read your feed | 🟡 |
| Flutter testing (light) | widget test of one screen | One test renders a screen | 🟢 |

---

## Track G — Testing (Weeks 5, 7–14) 🔴

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| Test pyramid | unit > integration > e2e | You know where each test belongs | 🔴 |
| pytest basics | test functions, asserts | You can write `test_user_cannot_login_twice` | 🔴 |
| Fixtures | reusable setup/teardown | One fixture creates a test user | 🟡 |
| Testing pure domain logic | no DB, no framework | domain tests run in <50 ms | 🔴 |
| Integration tests (repo) | against real Postgres | You can test a migration up/down | 🟡 |
| The 4 security tests | BOLA read/write, mass assign, secrets | All pass on every data endpoint | 🔴 |
| Testable design | pure domain + repos | you never need the API to test logic | 🔴 |
| Property-based testing (awareness) | many random inputs | you know it exists for later | 🟢 |

---

## Track H — DevOps & delivery (Weeks 0, 5, 6, 15–16) 🟡

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| Docker basics | image vs container | You can explain why we don't install Python on the host | 🟡 |
| Docker Compose | run multiple services | `docker compose up` starts API+DB | 🟡 |
| Environment/config | 12-factor config | Nothing secret in code | 🟡 |
| CI (GitHub Actions) | automated checks on push | Every PR runs lint/type/test/scan | 🟡 |
| Health checks & graceful shutdown | is the app alive? | `/health` is used by the load balancer | 🟢 |
| Deployment (single host/PaaS) | put it online with HTTPS | you have a live URL | 🟡 |
| Observability basics | logs, metrics, errors | you get alerted when 5xx spikes | 🟢 |
| Backup/restore | don't lose the DB | you restored a backup once | 🟢 |

---

## Track I — Product & design (Weeks 1–4) 🔴

| Topic | What it is | "I know it when…" | Needed for |
|---|---|---|---|
| Pitch & core loop | 1 sentence + the loop | you can state both from memory | 🔴 |
| MVP scoping | survival + 60-second tests | 4–6 features survived | 🔴 |
| Requirements & ACs | Given/When/Then | every FR has an AC | 🔴 |
| Risk register | trigger + mitigation + owner | top risks have measurable triggers | 🔴 |
| ERD reading | read a diagram like a schema | you can explain FKs in your own diagram | 🔴 |
| API contract writing | shapes, codes, errors | your partner needs no questions | 🔴 |
| Figma wireframes | boxes + flow first | a flow works before any code | 🔴 |
| Design tokens | colours/type/spacing exported once | no colour literals in Dart | 🟡 |
| Screen states | loading/empty/error per screen | no screen has an undefined state | 🔴 |

---

## Order to learn (the sequence that works for a beginner)

```
Weeks 0–1   Track A (foundations) → Track B (Python basics) → Track I (scope/risks)
Week 2      Track C part 1 (tables, keys, CRUD SQL) + Track I (requirements)
Week 3      Track C part 2 (joins, indexes, cascades) + data modeling → ERD
Week 4      Track D part 1 (routes, params, response models) + Figma
Week 5      Track E (security basics) + Track G (testing basics)
Week 6      Track C part 3 (transactions, migrations) + Track H (Docker/CI)
Weeks 7–13  Track B (async), D (DI, errors), E (auth specifics), F (Flutter), C (queries)
Week 14     Track G (concurrency/load) + E (OWASP pass)
Weeks 15–16 Track H (deploy, monitor, backup) + Track I (real-user feedback)
```

You learn **one slice of each track per week**, not an entire track before starting. That's the difference between a beginner plan and a curriculum that never gets used.

---

## Self-test before you call yourself "MVP-ready"

Tick all of these. If any is unticked, that's your study list for the week.

- [ ] I can explain request → response → status code without notes.
- [ ] I can write a `SELECT ... WHERE ... ORDER BY ... LIMIT` by hand.
- [ ] I can create a table with PK, FK, and UNIQUE in SQL.
- [ ] I can add an endpoint that validates input and returns 422 on bad data.
- [ ] I can write a pytest test that fails then passes.
- [ ] I can commit, branch, push, and open a PR without copying commands blindly.
- [ ] I can build a Flutter screen that calls the API and shows loading/error states.
- [ ] I can explain why `OFFSET` is slow and keyset isn't.
- [ ] I can say why the API contract exists and who it protects.
- [ ] I can list the 6 MVP features and 5 won't-have items from memory.

---

## The only rule that matters

> **Learn 20%. Build 80%. Then go back and learn the next 20%.**

A topic is "learned" when you can use it on the current slice — not when you finish reading about it. Everything in this list is ordered so that you never need to know something *before* you need it, except the 🔴 foundations, which take ~4 weeks and unlock everything else.