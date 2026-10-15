# Risks, MVP Scope, and Scaling Plan — Konek'ta

Companion to `docs/01_KONEK_TA_PRODUCT_SPEC.md` (full detail) and `docs/03_PROJECT_LIFECYCLE_TIMELINE.md` (weeks/gates).  
This file is the **decision summary**: what we ship, what can kill us, and how we grow without breaking.

---

## 1. MVP — what ships in v1 (by subtraction)

The core loop a v1 user must complete: **read a post in a topic → write a linked post → the right people see it → they engage back.**

### Will ship (6 features)
1. **Register + login** (email/password, age gate, JWT, refresh rotation, recovery).
2. **Interest selection** — ≥3 from a curated list (hard block; custom suggestions → admin queue).
3. **For You feed** — interest-filtered, newest first, keyset pagination, explainable per row.
4. **Create post** — text 1–2000 chars, exactly one category, anonymity override.
5. **Link to another post** — at most one parent, max depth 3 (the defining feature).
6. **Heart / bookmark / follow** (engagement).

### Screen set (minimum)
- Landing (can be cut to two buttons if schedule slips)
- Register, Login, Recovery
- Interest picker (mandatory)
- For You / Explore / Following feeds
- Composer (new + linked)
- Post / thread view
- Profile (own posts) + Settings (username, password, number, anonymity toggle)

### Won't have — defended (write these down and hold them)

| Item | Reason (defend this in week 3) |
|---|---|
| Images/video/audio | Media pipelines are the #1 killer of 2-person projects; pure text tests in microseconds. |
| Comments (separate type) | P2 — every reply is a post. One content type, one ranking system, one moderation pipeline. |
| Direct messages | Abuse-prone; no need for the loop; needs moderation you don't have. |
| Trending/discover boost | Violates P1 (relevance > reach); engagement never overrides relevance. |
| Ads / sponsored content | Violates P1; also a trust risk for a pseudonymous app. |
| Show-real-identity reveal | De-anonymisation feature; cut from v1 (DEC-05); user-controlled display name only. |
| Microservices / Redis / Kafka / K8s | Two people cannot operate distributed systems; monolith + Postgres. |
| GraphQL | No need at this scale; REST is simpler and cacheable. |
| Public category creation | Taxonomy fragmentation within a week; admin-only until merge/alias (Q-5). |
| Post editing | Breaks meaning of every linked post pointing at the old text; delete + repost. |
| Dark mode, push notifications, offline | Nice-to-have; deferred to v1.1 if users ask. |
| Admin panel (full) | v1 has a moderation queue + admin merge for categories only. |
| Post titles | 2000-char limit + feed context make a title a scrolling waste. |
| OAuth login, SSO | One auth path done well > many fragile. |
| Multi-language, i18n | v1 is single locale by intent. |

### The 60-second test
A new user must be studying (reading, matching a topic) within 60 seconds of open, without configuring anything. If they must sign up + set preferences first, cut more. (Your registration flow already ends in mandatory interest selection, which is the one gate justified by empty-feed avoidance.)

---

## 2. Risks (the ones that can kill the project)

Each risk: **id · what · impact · trigger · mitigation · owner**. Only project-killing risks live here; line-item bugs live in issues.

### R-1 — Empty feed / content cold start (product-killer)
- **What:** on day 1, categories in the picker have no posts → reader sees nothing → leaves forever.
- **Trigger:** first-session empty-feed rate above threshold (e.g. 5%).
- **Mitigation:** seed 200–500 posts across top 20 categories before launch, from 2–3 founding (disclosed) accounts + real friends; ≥10 posts per category; closed beta 30–50 then +5/week; explicit empty state with "write first post" CTA; pause invites on spike.
- **Owner:** both (content is cross-functional).

### R-2 — Taxonomy fragmentation (relevance-killer)
- **What:** free-text categories fragment into synonyms within a week → "Gaming" vs "games" vs "vidya" → filter means nothing.
- **Mitigation:** admin-only creation; custom suggestions queue (never auto-publish); merge/alias process before opening to users.
- **Owner:** backend (taxonomy) + ops later.

### R-3 — Anonymity abuse / de-anonymisation surface (privacy-killer)
- **What:** automatic "reveal identity based on interactivity" = a doxxing feature engineered by users (follow 200 accounts → get 200 reveals).
- **Decision:** cut from v1 (DEC-05); only user-controlled display-name reveal (never automatic, never legal identity stored).
- **Mitigation:** server always knows author (moderation applies); report/block always available; no reveal API until v1.x with a legal basis.
- **Owner:** both.

### R-4 — Interface mismatch between the two sides (schedule-killer)
- **What:** backend returns shapes client didn't expect → discovered at 2am, costs days.
- **Mitigation:** API contract written and agreed **before** code; both sides code against it; if code and doc disagree, code is wrong; PR review includes contract diff.
- **Owner:** both.

### R-5 — Distributed-systems scope (ops-killer)
- **What:** two people trying to run microservices/Redis/Kafka/K8s lose a month to ops.
- **Mitigation:** modular monolith + Postgres + Compose; any new infra must pass "two people can operate this" (MY_PLAN §3). Block in PR review.
- **Owner:** backend.

### R-6 — Scheduling / bus factor (people-killer)
- **What:** exams, illness, or one person quitting stalls everything.
- **Mitigation:** weekly 30-min sync; freeze at phase boundaries only (never mid-slice); GitHub Issues as task list; division by layer means either half can demo independently; keep domain logic pure so it's testable without infrastructure.
- **Owner:** both.

### R-7 — De-anonymisation data breach
- **What:** a pseudonymous app storing identifying data is one breach from death.
- **Mitigation:** no `real_name` field ever; PII (birthday, phone) never returned; soft-delete + erasure path tested; `pip-audit` + Bandit in CI; threat model before first endpoint.
- **Owner:** backend.

### R-8 — Quiet failure of the feed query (perf-killer)
- **What:** `OFFSET` pagination or N+1 on 10k posts turns feeds unusable; users leave.
- **Mitigation:** keyset pagination mandatory; composite index `(category_id, created_at DESC, id DESC)`; reconciliation audit test; `EXPLAIN` in review for any feed PR.
- **Owner:** backend.

### R-9 — Scope creep during build
- **What:** "can't we just add X?" in week 7 rewrites week 3.
- **Mitigation:** "Won't have" list with reasons; scope changes require doc + spec + decision-log update + both agree; survival-test applied to any new proposal.
- **Owner:** both.

### R-10 — Dependency / hosting surprise
- **What:** a free tier disappears mid-project; migration costs weeks.
- **Mitigation:** pin versions; keep compose-based local as the canonical run path; infra decisions captured as ADRs; PaaS choices revisited quarterly.
- **Owner:** backend.

### Risk register rule
Every new risk discovered during build gets: id, impact (1–5), likelihood (1–5), trigger, mitigation, owner, review at next sync. If a risk has no mitigation **and** no detectable trigger, it's not "identified" — it's worrying. Fix the plan first.

---

## 3. How we scale (phased, professional)

Scaling is not "buy bigger servers." It is a sequence of **bottlenecks**, removed one at a time, with measurement before and after. Rule: **measure first, then scale the measured bottleneck.** Premature scaling (R-5, R-10 brother) is what kills teams.

### Phase S0 — v1 MVP (Weeks 0–16, ~2 people, 1 DB, 1 deploy)
- Stack: modular monolith (FastAPI) + Postgres + Flutter client + Compose/PaaS.
- User scale: hundreds → tens of thousands.
- Seed content ≥10 posts/category; closed beta 30–50 then +5/week.
- **What "scaling" means here:** it doesn't. The only SLA is "feed query p95 documented and acceptable on 10k posts." Don't buy more.

### Phase S1 — Density & reliability (Weeks 17–24, v1.1)
Triggered by evidence from real users, not by guesses.
- Move from PaaS to a single managed Postgres (managed backups, point-in-time restore).
- Add read replica **only if** read QPS justifies it; monitored, not assumed.
- Cache the category list + seed-level aggregation queries in-process (no Redis yet).
- CDN for any static assets (later: media).
- Backup/restore drill monthly; documenting RTO/RPO.
- Scale levers: connection pooling, index verification, N+1 elimination (already designed in).
- **Gate:** an actual metric (feed p95, error rate, cost/month) exceeds budget threshold.

### Phase S2 — Growth (months 6–12)
- Only add infrastructure when a measured bottleneck demands it:
  - **Read-heavy DB?** Add cache layer (Redis) — only the specific query classes that are hot, not "use Redis everywhere."
  - **Heavy write spikes?** Move background jobs (email, notifications) to an in-process queue or a small job runner; not Kafka.
  - **Media introduced (v2)?** Object storage (S3-compatible) + CDN + transcode pipeline — this is the biggest single cost; needs its own cost model.
  - **Search needed?** Add a search engine only when LIKE/FTS proves insufficient; defer until either.
- Introduce per-user rate limiting and abuse thresholds (you already have login/category/report).
- Begin contributor-funded moderation tooling (admin merge queue, ban flow).
- **Gate:** cost per 1k users + latency SLO both stable for 90 days before next infra change.

### Phase S3 — Scale-out (year 2+, only if product proves)
Triggers (all must be true): DAU > config threshold, monthly infra > budget line, single-instance latency breaching SLO under load.
- Split read replicas + (only if proven) separate read/write roles.
- Evaluate read→frontend caching at edge only for public content (Explore feed) — never For You (user-specific, violates P1 if cached wrong).
- Add monitoring/observability maturity: structured logs + metrics + traces + error tracking dashboard.
- Consider a dedicated media/infra service only once media launches and its cost is isolated.
- Legal/ops: privacy compliance (GDPR/CCPA-style erasure), ToS enforcement, appeal path, abuse SLA.

### Phase S4 — Maturity (year 3+)
- Multi-region only if data residency/latency demands it (rare for a social app at indie scale).
- Feature-flag system (server-side) for gradual rollouts.
- Schema-migration pipeline with expand-and-contract enforced (no downtime deploys).
- Public rate-limit SLA + status page for users.
- Documented incident-response playbook + on-call rotation (now viable because revenue exists).

### What we will deliberately NOT do as "scaling" (the avoid list)
- Microservices "for scale" with 2–3 people.
- Kubernetes for a monolith.
- GraphQL migration to "reduce over-fetching."
- Pre-emptive sharding.
- Separate ML serving before a model exists.

Each appears in MY_PLAN §3's "what to avoid" list for the same reason: operational overhead with no benefit at indie scale, and a 2-person team cannot run distributed systems.

### Scaling metrics (the few numbers that gate each phase)

| Phase | Metric | Decision rule |
|---|---|---|
| S0 | Feed p95 at 10k posts | Document; must be acceptable to proceed. |
| S0 | Error rate incl. 5xx | < 0.5% over 7 days. |
| S1 | DB CPU at peak | > 70% sustained → add replica or optimise. |
| S1 | Cost per 1k users | Plateaus or falls; if rising faster than DAU, optimise before growing. |
| S2 | Feed p95 under load | Threshold breach → cache/infra change. |
| S2 | Creator → regular ratio | If too few (content rot), revisit seeding/incentives before infra. |
| S3 | DAU + infra budget | Both must exceed thresholds before scale-out. |
| All | First-session empty-feed rate | If rises, add content — not servers. |

### The scaling principle in one line
**Scale the measured bottleneck; never pre-scale.** The cheapest scaling move at every phase is almost always "ship a cache" or "fix an N+1," not "add a machine" — and measurable bottlenecks are almost always content (density), not hardware, until v1.1+.

---

## Appendix — decision checkpoints

- **v1 ship gate (G9):** outside user completes the loop unaided with ≥10 posts/category visible.
- **v1.1 ship gate:** empty-feed rate stable under traffic; 7-day clean run; at least one deferred feature chosen by user evidence.
- **Scale-out gate:** two consecutive 90-day periods with cost/latency SLOs breached before action (avoids premature, expensive changes).

If a "scaling" request can't be tied to a metric in the table above, it's an idea — not a decision. Log it and watch for evidence.