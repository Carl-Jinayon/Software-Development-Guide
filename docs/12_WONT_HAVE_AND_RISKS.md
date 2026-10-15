# Won't-Have List + Risk List — Konek'ta (One Page)

Companion to `docs/11_RISKS_MVP_AND_SCALING.md` (full detail). This is the short version you can re-read in 5 minutes.

---

## Part 1 — Won't Have (v1 cut list)

Each row: **item · why it's cut · what would un-cut it.**

| Item | Why cut | Re-entry criteria (evidence needed) |
|---|---|---|
| Images / video / audio | Media pipeline (upload, storage, transcode, moderation) is the #1 killer of 2-person projects. Pure text is fully testable. | Users repeatedly ask to attach a screenshot AND you're past 10k posts/category. |
| Comments (separate content type) | P2: every reply is a post. One content type = one ranking system = one moderation pipeline. | Never. This is a product identity decision, not a feature gap. |
| Direct messages | Abuse-prone; needs reporting/moderation you don't have; not needed for the loop. | A measurable % of users say they can't coordinate without it. |
| Trending / "discover more" / engagement boost | Violates P1 (relevance over reach). Would break FR-009 AC-2, the core promise. | Never without changing the product spec first. |
| Ads / sponsored content | Violates P1; destroys pseudonymous trust. | Never in v1. |
| Automatic reveal of real identity | De-anonymisation feature = doxxing engine. Cut permanently (DEC-05). | Only user-controlled display name, ever. Automatic reveal: no. |
| User-created categories | Taxonomy fragments into synonyms within a week. | After a merge/alias process exists + moderation tooling. |
| Post titles | With a 2000-char limit, a title is scroll-waste in a feed. | Never in v1. |
| Post editing | Breaks the meaning of every linked post pointing at the old text. | Only with version history + "edited" markers. |
| Search | The interest-filtered feed already serves the loop; keyword search can come later. | Users demonstrably can't find a post they saw (track the complaints). |
| AI / semantic search | Costs money per call at scale; unneeded while categories do the job. | Your own query logs prove keyword search misses real demand. |
| Push notifications | Infra + abuse surface; not needed to make the loop work. | D1 retention is proven and users ask for it. |
| Offline mode / caching layer | Adds sync-conflict complexity to 2 people. | Field usage shows real offline need. |
| Dark mode | Pure polish. | Post-launch, cheap. |
| Follower counts / profiles-as-bragging surface | Off-brand: relevance over reach. | Never in v1. |
| Hashtags | Would fragment the taxonomy — categories already do this job. | Never in v1. |
| Groups / communities | Separate moderation + discovery problem; not the loop. | After v1 is stable with real users. |
| Donations (per spec's "communities ask for donations") | Payments, tax, fraud, chargebacks — a company, not a feature. | Requires legal/financial setup. |
| OAuth / Google login | One auth path done well beats many fragile ones. | After v1 auth is solid and users ask. |
| i18n / multi-language | v1 is single-locale by intent. | After a second-language user base exists. |
| Microservices / Redis / Kafka / K8s | Two people cannot operate distributed systems. | Only if a measured bottleneck demands it. |
| GraphQL | REST is simpler, cacheable, sufficient. | Never without a measured need. |
| Sharding / multi-region | Premature by orders of magnitude. | Year 2+, only under real scale pressure. |

---

## Part 2 — Risks (the ones that can kill the project)

Format: **what · trigger (how we know it's happening) · mitigation · owner.**

| # | Risk | Trigger | Mitigation | Owner |
|---|---|---|---|---|
| R-1 | **Empty feed on day 1** (no posts in picked categories) | First-session empty-feed rate > 5% | Seed ≥10 posts/category from disclosed founding accounts + friends; ≥200 posts total; closed beta 30–50, +5/week; explicit empty state; pause invites on spike. | Both |
| R-2 | **Nobody writes** (read-only users → loop dies) | <1 post written per weekly active user | Seed 3-deep reply chains as examples; one-tap "reply with this category"; founder presence; prompt after 2–3 reads. | Both |
| R-3 | **Taxonomy fragments** (Gaming / Games / Videogames) | >1 duplicate suggestion per week; a category with >3 aliases | Admin-only creation; suggestion queue (never auto-publish); `category_aliases` table; weekly merge review. | Backend |
| R-4 | **Feed relevance becomes meaningless** (users pick everything) | Any user following >30% of categories | 60–80 categories; min-3 / max-8 interests; track distribution; merge/split taxonomy. | Both |
| R-5 | **Interface mismatch between the two devs** | Any "I assumed you'd return..." moment | Written API contract before code; PR reviews include contract diff; "code vs doc disagree → code is wrong." | Both |
| R-6 | **Feed dies at scale** (OFFSET / N+1) | Feed p95 above budget at 10k posts | Keyset pagination mandatory; index `(category_id, created_at DESC, id DESC)`; batched heart/bookmark lookups; EXPLAIN in review. | Backend |
| R-7 | **Counter drift** (hearts lie) | Reconciliation job finds mismatch | Counter write inside the same transaction as the heart row; nightly reconciliation; audit test in CI. | Backend |
| R-8 | **Account takeover / enumeration** | Spike in failed logins from an IP; any "that email is registered" message | Argon2id; refresh rotation + reuse detection; generic login/reset responses; rate limits (5/min IP, 10/hr account). | Backend |
| R-9 | **PII breach in a pseudonymous app** | Any endpoint returning `birthday`/`phone`; any secret in git | No `real_name` field ever; PII never returned; `.env` gitignored from commit one; pip-audit + Bandit in CI; erasure tested. | Backend |
| R-10 | **De-anon feature reintroduced** | Any PR/branch adding "reveal identity on interaction" | DEC-05: cut permanently; only user-controlled display name; no automatic reveal endpoint. | Both |
| R-11 | **Culture collapse** (one bad early post defines the platform) | Any hate/NSFW/spam in first 100 user posts | Moderate seed content pre-launch; 5 written rules; report/block in v1; manual review of first 100 posts during beta. | Both |
| R-12 | **Scope creep** ("just add X") in week 7+ | Any feature proposal without a survival-test result | Won't-have list with reasons; scope changes need doc + spec + decision-log update + both agree; survival test applied. | Both |
| R-13 | **Bus factor = 1** (exams, illness, dropout) | Any 2-week single-person gap | Freeze at phase boundaries only; split by layer so either half demos alone; pure domain logic testable without infra; GitHub Issues as task list. | Both |
| R-14 | **Infra overreach** (microservices/Redis/K8s) | Any PR adding a service without a measured bottleneck | Rule: "can two people run this at 2am?" If no → blocked in review. | Backend |
| R-15 | **Cost surprise** (hosting/DB/email bills) | Unexpected invoice; monthly infra cost rising faster than users | Billing alerts; pin dependency versions; cost-per-1k-users tracked monthly; infra decisions as ADRs. | Backend |
| R-16 | **Schema change after launch breaks prod** | Any migration that drops/renames a column | Expand-and-contract (nullable → backfill → enforce); always test up *and* down; no destructive migration in one release. | Backend |

---

## The three that matter most (if you only track three)
1. **R-1 Empty feed** — no users, no product.
2. **R-5 Interface mismatch** — the #1 schedule killer for 2 people.
3. **R-6 Feed perf** — the product is the feed; if it's slow at 10k posts, the product is broken.

Everything else is important, but those three decide whether you launch.

**How to use this file:** read it at every Sunday sync. If a new risk appears during the week, add a row (with trigger + owner) before you fix it. If a Won't-have item is being reconsidered, the **re-entry criteria** column decides it — not enthusiasm.