# DevOps, CI/CD, and Deployment Guide — Deep Dive

Companion to Parts 11–13 of `01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md`.

---

## Table of Contents

- [1. What DevOps actually is](#1-what-devops-actually-is)
- [2. Continuous Integration](#2-continuous-integration)
- [3. GitHub Actions in depth](#3-github-actions-in-depth)
- [4. Docker](#4-docker)
- [5. Configuration and environments](#5-configuration-and-environments)
- [6. Deployment strategies](#6-deployment-strategies)
- [7. Hosting options compared](#7-hosting-options-compared)
- [8. Infrastructure as Code](#8-infrastructure-as-code)
- [9. Database migrations in production](#9-database-migrations-in-production)
- [10. Observability](#10-observability)
- [11. Rollbacks and releases](#11-rollbacks-and-releases)
- [12. Secrets in CI](#12-secrets-in-ci)
- [13. Performance and cost](#13-performance-and-cost)
- [14. Learning resources](#14-learning-resources)

---

# 1. What DevOps actually is

## 1.1 The definition that matters

DevOps is **not** a tool and **not** a department. It is a set of practices that remove the wall between writing software and running it.

The three core ideas:

| Idea | Meaning |
|---|---|
| **Continuous delivery** | Any commit that passes tests can be released to production automatically |
| **Automation** | Anything a human does repeatedly by hand should be a script |
| **Shared ownership** | The team that builds it also runs it. "You build it, you run it" |

## 1.2 The four metrics that matter

| Metric | What it measures | Good |
|---|---|---|
| **Deployment frequency** | How often you ship | Multiple per day |
| **Lead time for changes** | Commit to production | Minutes to hours |
| **Change failure rate** | Deployments causing incidents | Low, trending down |
| **Time to restore** | Detect to fixed | Minutes |

Together these are **DORA metrics** (DevOps Research and Assessment). They are the best available proxy for how healthy a delivery process is.

**The key insight:** you cannot improve lead time for changes by working faster on any individual change. You improve it by making changes smaller. High-performing teams deploy many small changes frequently, not few large ones.

## 1.3 The maturity ladder

```
Level 1  Manual       Deploy by hand, from one machine, at night
Level 2  Automated   CI runs tests; deploy still manual
Level 3  Continuous  CI/CD pipeline deploys automatically
Level 4  Measurable  DORA metrics tracked, alerting in place
Level 5  Optimised   Small batches, fast rollback, blameless post-mortems
```

Most teams are at Level 2. Any improvement here compounds.

---

# 2. Continuous Integration

## 2.1 Why CI exists

Without it, integration bugs accumulate for weeks. Classic failure mode:

```
Developer A: feature/login     ────────────────►
Developer B: feature/checkout  ───────────────►
                                        merge conflict hell
                                     "works on my machine"
                                     "works on my branch"
                                     3 days lost
```

**The rule that prevents this: integrate at least daily, and never let a branch live longer than a few days.**

CI enforces this mechanically. If `main` is green every few hours, the cost of integrating is measured in minutes rather than days.

## 2.2 The CI pipeline

```
commit pushed
      │
      ▼
┌─────────────────────────────────┐
│ 1. Install dependencies          │  ← cache this
│ 2. Format check (black/ruff)    │  ← seconds
│ 3. Lint (ruff)                  │  ← seconds
│ 4. Type check (mypy)            │  ← seconds
│ 5. Unit tests + coverage        │  ← seconds to minutes
│ 6. Security scan (pip-audit)    │  ← seconds
│ 7. Build Docker image           │  ← minutes
│ 8. Integration tests            │  ← minutes
│ 9. Deploy to staging            │
│10. Smoke tests against staging  │
│11. Deploy to production         │  ← only if everything passed
└─────────────────────────────────┘
      │
      ▼ failure → developer notified in ~2 minutes
```

**Fail fast.** Linting takes 2 seconds. Running a 10-minute test suite first wastes everyone's time.

## 2.3 What makes CI valuable

| Benefit | Why |
|---|---|
| **Fast feedback** | Know within minutes, not days |
| **Consistency** | Same checks on every machine and every PR |
| **Confidence to deploy** | Green pipeline = safe release |
| **Documentation** | The YAML *is* the build documentation |
| **Onboarding** | One command gets a new developer running |
| **Merged confidence** | `main` is always releasable |

---

# 3. GitHub Actions in depth

## 3.1 Workflow file structure

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Cancel superseded runs — saves CI minutes, nothing else needs the old commit
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read          # least privilege — the default should be minimal
  pull-requests: write

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python with caching
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
          cache-dependency-path: poetry.lock

      - name: Install
        run: |
          pip install -e ".[dev]"

      - name: Format check
        run: ruff format --check .

      - name: Lint
        run: ruff check . --output-format=github

      - name: Type check
        run: mypy app/

  test:
    runs-on: ubuntu-latest
    needs: quality          # don't waste time testing what won't lint
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.11", "3.12", "3.13"]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip

      - run: pip install -e ".[dev]"

      - name: Unit tests with coverage
        run: |
          pytest tests/unit -m "not slow" \
            --cov=app --cov-report=xml --cov-report=term -q

      - name: Enforce coverage threshold
        run: coverage report --fail-under=80

      - name: Security audit
        run: |
          pip install pip-audit
          pip-audit --strict || exit 1

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-${{ matrix.python-version }}
          path: coverage.xml

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
      - uses: docker/build-push-action@v5
        with:
          push: false
          tags: myapp:${{ github.sha }}
```

## 3.2 Caching

Caching is the single biggest CI speed optimisation. Installing dependencies twice is pure waste.

```yaml
- uses: actions/setup-python@v5
  with:
    cache: pip                    # built-in, requires a requirements file
    cache-dependency-path: poetry.lock
```

```yaml
# Manual caching for compiled artefacts
- name: Cache pip wheels
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

## 3.3 Matrix builds

Test every version that could be affected.

```yaml
strategy:
  fail-fast: false        # test all versions even if one fails — you want the full picture
  matrix:
    python-version: ["3.11", "3.12", "3.13"]
    os: [ubuntu-latest]
    include:
      - python-version: "3.12"
        os: macos-latest      # catch platform-specific bugs
```

## 3.4 Deployment workflow

```yaml
name: Deploy

on:
  push:
    tags: ["v*"]            # deploy on version tags — not every commit

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production          # gives you required-reviewer protection
    concurrency:
      group: production-deploy
      cancel-in-progress: false      # never cancel a production deploy

    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            registry.example.com/myapp:${{ github.ref_name }}
            registry.example.com/myapp:latest
          cache-from: type=registry
          cache-to: type=registry,mode=max

      - name: Run migrations
        run: |
          pip install -e ".[dev]"
          alembic upgrade head
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Deploy to production
        run: |
          # platform-specific deploy (Render, Fly, Railway, ECS, K8s…)
          curl -X POST "${{ secrets.DEPLOY_HOOK_URL }}" -H "Content-Type: application/json"

      - name: Smoke test
        run: |
          sleep 10
          curl --fail --retry 3 --retry-delay 5 https://myapp.com/api/health
```

**The post-deploy smoke test is what stops you shipping a broken release and finding out from users.**

## 3.5 Common GitHub Actions pitfalls

| Pitfall | Consequence | Fix |
|---|---|---|
| Using `pull_request_target` with checkout | **Remote code execution** in your repo | Never combine `pull_request_target` with untrusted checkout |
| Floating action tags (`@v4`) | Supply-chain risk | Pin to a SHA |
| No `permissions` block | Excessive default token scope | Set minimal permissions |
| Secrets in workflows triggered by forks | Leak to anyone | Use `pull_request`, not `pull_request_target` |
| `if: always()` on deployment | Deploys broken code | Only deploy from protected branches |
| No caching | 3× slower pipelines | Cache aggressively |

## 3.6 Pre-commit hooks

Catch problems locally, before they reach CI.

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: detect-private-key
      - id: check-merge-conflict

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.5.0
    hooks:
      - id: ruff
        args: [--fix]

  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

  - repo: local
    hooks:
      - id: pytest-fast
        name: Fast unit tests
        entry: pytest tests/unit -m "not slow" -x -q
        language: system
        pass_filenames: false
        stages: [pre-push]
```

```bash
pip install pre-commit
pre-commit install
pre-commit install-hooks
```

---

# 4. Docker

## 4.1 What a container is

A container is a process with its own filesystem view, sharing the host kernel. It is **not** a lightweight virtual machine — there is no separate kernel.

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Container  │  │   Container  │  │   Container  │
│   Python app │  │   Postgres   │  │     Redis    │
├──────────────┤  ├──────────────┤  ├──────────────┤
│  Own FS      │  │  Own FS      │  │  Own FS      │
├──────────────┴──┴──────────────┴──┴──────────────┴──┐
│                    Host kernel                       │
└─────────────────────────────────────────────────────┘
```

## 4.2 A production Dockerfile

```dockerfile
# ---- Stage 1: build ----
FROM python:3.12-slim AS builder

WORKDIR /build

# Compile wheels once — much faster than installing at runtime
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY pyproject.toml poetry.lock ./
RUN pip install poetry \
    && poetry config virtualenvs.create false \
    && poetry export -f requirements.txt --output requirements.txt

COPY . .
RUN pip wheel --wheel-dir /wheels -r requirements.txt


# ---- Stage 2: runtime ----
FROM python:3.12-slim AS runtime

# Run as a non-root user
RUN groupadd --system app && useradd --system --gid app app

WORKDIR /app

COPY --from=builder /wheels /wheels
COPY pyproject.toml ./
RUN pip install --no-cache-dir --no-index --find-links=/wheels -r <(pip freeze | grep -v "^-" | head -0) 2>/dev/null \
    || pip install --no-cache-dir --no-index --find-links=/wheels /wheels/*.whl \
    && rm -rf /wheels

COPY --chown=app:app . .

USER app                        # never run as root

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PORT=8000

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=20s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/api/health')"

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

## 4.3 Layer caching

Docker caches each instruction. **Copy the dependency manifest before the source code**, or every code change invalidates the dependency layer.

```dockerfile
# ✓ CORRECT — dependencies cached unless they change
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# ✗ WRONG — every code change re-installs everything
COPY . .
RUN pip install -r requirements.txt
```

## 4.4 Docker Compose

```yaml
# docker-compose.yml — full local stack in one command
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-localdev}
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports: ["5432:5432"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  api:
    build: .
    environment:
      DATABASE_URL: postgresql://app:${POSTGRES_PASSWORD:-localdev}@db:5432/appdb
      REDIS_URL: redis://redis:6379/0
    ports: ["8000:8000"]
    depends_on:
      db:
        condition: service_healthy      # wait for health, not just start
    volumes:
      - .:/app                        # live reload in development
    command: uvicorn app.main:app --host 0.0.0.0 --reload

volumes:
  pgdata:
```

```bash
docker compose up -d          # start everything
docker compose logs -f api    # follow logs
docker compose exec db psql -U app appdb
docker compose down -v        # stop and delete data
```

## 4.5 Anti-patterns

| Anti-pattern | Why |
|---|---|
| Running as root | Container escape becomes host compromise |
| `apt-get upgrade` in the image | Non-reproducible, bloats the image |
| Secrets in `ENV`/`ARG` | Visible in `docker history` and the image manifest |
| No healthcheck | The orchestrator cannot know when you are ready |
| Committing `.env` into the image | Leaks into every layer |
| `latest` tag in production | Non-reproducible; tag with the commit SHA |
| Bind-mounting source in production | Overwrites the image |

---

# 5. Configuration and environments

## 5.1 The same code, different configuration

```
                    ┌──────────────┐
   Development ────▶│              │
   Staging     ────▶│  ONE image   │────▶ Different env vars
   Production  ────▶│              │      Different secrets
                    └──────────────┘
```

**Build once, promote the same artefact through environments.** Building separately for each environment means you never test what you actually ship.

## 5.2 Twelve-factor configuration

| Factor | Rule |
|---|---|
| Config | Store config in **environment variables**, not files |
| Backing services | Treat databases and queues as attached resources |
| Disposability | Containers can be destroyed and recreated at will |
| Dev/prod parity | Keep gaps between environments minimal |
| Logs | Write to stdout; the platform collects them |
| Processes | One concern per process; run it in the background, never daemonise |

## 5.3 Typed settings with validation

```python
from functools import lru_cache
from pydantic import Field, field_validator
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    # Application
    app_name: str = "myapp"
    debug: bool = False
    environment: Literal["development", "staging", "production"] = "development"

    # Database
    database_url: str
    db_pool_size: int = Field(default=10, ge=1, le=100)
    db_max_overflow: int = Field(default=20, ge=0)

    # Security
    jwt_secret: str = Field(min_length=32)
    access_token_ttl_minutes: int = Field(default=15, ge=1, le=1440)
    allowed_origins: list[str] = []

    # Observability
    log_level: Literal["DEBUG", "INFO", "WARNING", "ERROR"] = "INFO"
    sentry_dsn: str | None = None

    @field_validator("jwt_secret")
    @classmethod
    def no_weak_secrets(cls, value: str, info) -> str:
        environment = (info.data or {}).get("environment", "development")
        if environment == "production" and "changeme" in value.lower():
            raise ValueError("Refusing to start with a default secret in production")
        return value

    @field_validator("database_url")
    @classmethod
    def require_ssl_in_production(cls, value: str, info) -> str:
        environment = (info.data or {}).get("environment", "development")
        if environment == "production" and not value.startswith("postgresql+ssl"):
            raise ValueError("Production database must use SSL")
        return value


@lru_cache
def get_settings() -> Settings:
    return Settings()


settings = get_settings()
```

**This pattern is worth the effort:** a misconfiguration that would otherwise surface at 3am in production fails at startup instead. The `@lru_cache` means environment variables are read once.

## 5.4 Environment comparison

| Variable | Development | Staging | Production |
|---|---|---|---|
| `DEBUG` | `True` | `False` | `False` |
| `ENVIRONMENT` | `development` | `staging` | `production` |
| `LOG_LEVEL` | `DEBUG` | `INFO` | `WARNING` |
| `DATABASE_URL` | localhost | staging DB | production DB |
| `ALLOWED_ORIGINS` | localhost | staging domain | real domain |
| `JWT_SECRET` | dev value | staging secret | production secret |
| Rate limits | generous | production values | production values |

---

# 6. Deployment strategies

| Strategy | How | Rollback | Use when |
|---|---|---|---|
| **Recreate** | Stop everything, start the new version | Redeploy the old version | Maintenance windows acceptable |
| **Rolling** | Replace instances gradually | Abort the rollout | Default for most applications |
| **Blue/green** | Run both versions; switch traffic | Instant traffic flip | Zero-downtime, database changes |
| **Canary** | New version to 5% of traffic, watch metrics | Shift traffic back | High-traffic, high-risk changes |
| **Feature flags** | Deploy code dark; enable gradually | Flip the flag off | Long-running experiments |

## 6.1 Blue/green

```
LOAD BALANCER
      │
   [ BLUE ]  ← current production, version 1.4
      │
   [ GREEN ]  ← new deploy, version 1.5
      │
   Switch traffic to GREEN
   If anything degrades → switch back to BLUE (instant)
```

**Zero downtime, instant rollback.** The cost is running two full environments.

## 6.2 Canary

```
1%  → watch error rate, latency, business metrics for 30 min
5%  → observe 1 hour
25% → observe 4 hours
100%
```

**Canary and feature flags are the two tools that let you decouple deployment from release.** Deploying is safe; enabling is a separate, reversible decision.

## 6.3 Rolling vs blue/green

| | Rolling | Blue/green |
|---|---|---|
| Downtime | None | None |
| Database migrations | Tricky | Manageable |
| Resource cost | 1× | 2× |
| Rollback speed | Slow (must wait for rollout) | Instant |
| Complexity | Low | Medium |

For most projects: **rolling updates.** Save blue/green for cases with schema changes or zero-downtime requirements.

---

# 7. Hosting options compared

## 7.1 The spectrum

```
Managed/PaaS                                     IaaS
─────────────────────��───────────────────────────────��──────────
Render  Heroku    Railway   Fly.io      AWS EC2    DigitalOcean
Vercel  Netlify   PythonAnywhere                  Hetzner
                    ↑
            Start here for learning.
            $0–$25/month.
```

## 7.2 Comparison

| Platform | Free tier | Ease | Cost at scale | Best for |
|---|---|---|---|---|
| **Render** | Web services spin down after 15 min idle | Very easy | $$ | Learning, small projects |
| **Railway** | Trial credits | Very easy | $ | Learning, quick deploys |
| **Fly.io** | Small allowance | Easy | $ | Apps needing moderate scale |
| **PythonAnywhere** | Yes | Easiest | $ | Pure Python, no Docker needed |
| **Vercel** | Generous | Very easy | $$ | Serverless, frontend |
| **AWS EC2** | 12-month free tier | Hard | $$$$ | Full control, learning IaaS |
| **Hetzner** | No | Medium | $ (cheapest) | Cost-sensitive production |

## 7.3 Managed database options

| Service | Notes |
|---|---|
| **Supabase** | Free tier, Postgres, includes auth and storage |
| **Neon** | Free serverless Postgres, scales to zero |
| **Railway Postgres** | Simple, bundled with the app deploy |
| **AWS RDS / Google Cloud SQL** | Production-grade, expensive |
| **PlanetScale** | MySQL, generous free tier |

**Use a managed database for your first real deployment.** Backups, patching, and failover are not your problem to solve while learning.

## 7.4 Container hosting

| Platform | Notes |
|---|---|
| **Fly.io** | Simple, global, reasonable pricing |
| **Render** | Web services with Dockerfiles |
| **AWS ECS / Fargate** | Production-grade, more setup |
| **Google Cloud Run** | Scales to zero, pay per request |
| **Fly / Railway** | Easiest starting point |

---

# 8. Infrastructure as Code

## 8.1 Why

Manual server configuration is undocumented, unreproducible, and drifts. IaC makes the environment a reviewable artefact.

## 8.2 Terraform, minimal example

```hcl
# main.tf
terraform {
  required_version = ">= 1.6"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" {
  region = var.region
}

resource "aws_s3_bucket" "uploads" {
  bucket = "myapp-uploads"
}

resource "aws_s3_bucket_public_access_block" "uploads" {
  bucket                  = aws_s3_bucket.uploads.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_versioning" "uploads" {
  bucket = aws_s3_bucket.uploads.id
  versioning_configuration { status = "Enabled" }
}
```

```bash
terraform init
terraform plan        # what WILL change — always read this
terraform apply
```

**Always run `terraform plan` before `apply`.** A careless `apply` can delete production resources, and the state file is the only record of what exists.

## 8.3 Alternatives

| Tool | Use for |
|---|---|
| Terraform | Cloud infrastructure, any provider |
| Pulumi | Same, with real programming languages instead of HCL |
| Docker Compose | Local development environments |
| Ansible | Configuring existing servers |
| Kubernetes manifests | Already on Kubernetes |

**For a first project, `docker-compose.yml` plus a managed platform is sufficient IaC.** Terraform is worth learning when you have infrastructure to manage.

---

# 9. Database migrations in production

## 9.1 The expand-and-contract pattern

Never write a migration that breaks the currently running code.

```
Release N:    ADD COLUMN new_field (nullable)      ← expand: old code ignores it
Release N+1:  Deploy code that writes new_field   ← now populated
              Backfill existing rows
Release N+2:  Deploy code that reads new_field    ← now required
Release N+3:  DROP COLUMN old_field               ← contract: nothing reads it
```

Each step is individually safe and reversible. The whole change takes several releases — that is the cost of zero downtime.

## 9.2 Migrations that will break a running app

| Migration | Breaks running code? |
|---|---|
| `ADD COLUMN nullable` | No |
| `ADD COLUMN NOT NULL` with default | No |
| `ADD COLUMN NOT NULL` without default | **Yes** — old INSERTs fail |
| `DROP COLUMN` | **Yes** if anything selects `*` |
| `RENAME COLUMN` | **Yes** |
| `ALTER COLUMN TYPE` | **Yes**, if incompatible |
| Adding a `UNIQUE` constraint | **Yes** if duplicates exist |
| Adding an index | No — but `CREATE INDEX` locks writes. Use `CONCURRENTLY` |

```python
# Alembic: add a nullable column first
def upgrade():
    op.add_column("users", sa.Column("avatar_url", sa.String(), nullable=True))


def downgrade():
    op.drop_column("users", "avatar_url")


# Build an index without locking writes (Postgres only)
def upgrade():
    op.execute("CREATE INDEX CONCURRENTLY idx_users_email ON users (email)")


def downgrade():
    op.execute("DROP INDEX CONCURRENTLY idx_users_email")
```

## 9.3 Safe rollback

**Data migrations are not reversible once data has changed.** A `downgrade()` function cannot un-delete rows.

| Change | Reversible? |
|---|---|
| Add column | Yes |
| Drop column | Yes, **if** you still have the data. Once dropped, it is gone |
| Delete rows | **No** |
| Transform data | **No** — needs a reverse transform, which may be lossy |

**Hence: back up before migrating, and test the restore.**

---

# 10. Observability

> **You cannot maintain what you cannot see.**

## 10.1 The three pillars

| Pillar | Answers | Tool |
|---|---|---|
| **Logs** | What happened? | `structlog` + Loki / CloudWatch |
| **Metrics** | How is it trending? | Prometheus + Grafana |
| **Traces** | Where did the time go? | OpenTelemetry + Tempo / Jaeger |

## 10.2 Structured logging

```python
import structlog
import logging
import sys

structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.JSONRenderer(),       # machines parse JSON, not strings
    ],
    wrapper_class=structlog.make_filtering_bound_logger(logging.INFO),
    logger_factory=structlog.PrintLoggerFactory(),
)

log = structlog.get_logger()
```

```python
@router.post("/api/v1/decks")
async def create_deck(payload: CreateDeck, request: Request,
                      user: User = Depends(get_current_user)):
    log.info("deck.create.start", user_id=user.id, title=payload.title)

    structlog.contextvars.bind_contextvars(request_id=request.headers.get("x-request-id"))
    log.info("deck.created", deck_id=deck.id, user_id=user.id)

    return deck
```

**JSON logs are queryable.** `grep` on formatted strings stops working the moment you need to answer "how many requests failed in the last hour for user 42." With JSON, it is one query.

## 10.3 Metrics

```python
from prometheus_client import Counter, Histogram, generate_latest

REQUEST_COUNT = Counter(
    "http_requests_total",
    "Total HTTP requests",
    ["method", "endpoint", "status"],       # never label by user_id — unbounded cardinality
)

REQUEST_LATENCY = Histogram(
    "http_request_duration_seconds",
    "HTTP request latency",
    ["method", "endpoint"],
    buckets=(0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0),
)


@app.middleware("http")
async def track_requests(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    duration = time.perf_counter() - start

    endpoint = route_template(request)        # "/decks/{id}", not "/decks/123"
    REQUEST_COUNT.labels(request.method, endpoint, response.status_code).inc()
    REQUEST_LATENCY.labels(request.method, endpoint).observe(duration)

    return response


@app.get("/metrics")
def metrics():
    return Response(generate_latest(), media_type="text/plain")
```

**Label cardinality is the trap.** Labelling by `user_id` creates millions of time series and will melt your metrics backend. Label by route template, method, status code — bounded sets only.

## 10.4 Error tracking

```python
import sentry_sdk

sentry_sdk.init(
    dsn=settings.sentry_dsn,
    environment=settings.environment,
    release=__version__,
    traces_sample_rate=0.1,          # 10% of traces; 100% is expensive
    profiles_sample_rate=0.1,
)
```

Sentry groups errors by stack trace and shows you exactly which user, which request, and which input caused each one.

## 10.5 Health and readiness

```python
@app.get("/api/health")
async def health():
    """Liveness — is the process running? Used by the orchestrator."""
    return {"status": "ok"}


@app.get("/api/health/ready")
async def readiness(session: Session = Depends(get_db)):
    """Readiness — can it serve traffic? Checks real dependencies."""
    try:
        session.execute(text("SELECT 1"))
        return {"status": "ready", "database": "ok"}
    except SQLAlchemyError as exc:
        raise HTTPException(503, f"Database unavailable: {type(exc).__name__}")
```

**Distinguish liveness from readiness.** Restarting a container because the database is briefly down turns a minor outage into a crash loop.

---

# 11. Rollbacks and releases

## 11.1 Rollback strategy

| Change type | Rollback method |
|---|---|
| Code only | Redeploy the previous image tag |
| Code + additive schema change | Redeploy the previous image — the extra column is harmless |
| Destructive schema change | **No fast rollback.** This is why expand-and-contract exists |
| Data deletion | **No rollback.** Backups only |
| Feature toggle | Flip the flag off — instant, no deploy |

## 11.2 Release checklist

```markdown
## Pre-release
- [ ] CI green on the exact commit being released
- [ ] Migrations tested on a production-like database
- [ ] Backup taken and restore verified
- [ ] Rollback command written down and tested
- [ ] Error tracking and alerting active
- [ ] Feature flags at their intended values

## Release
- [ ] Deploy
- [ ] Smoke test the critical path
- [ ] Watch error rate and p95 latency for 15 minutes
- [ ] Watch business metrics (signups, orders)

## Post-release
- [ ] Log what was released and when
- [ ] Monitor for 24 hours
- [ ] Schedule any deferred cleanup
```

## 11.3 Release notes

```markdown
## v1.5.0 — 2026-10-02

### Added
- Deck sharing between users (#412)
- Bulk card import from CSV (#418)

### Changed
- Review scheduling now uses FSRS instead of SM-2 (#390)
  - **Migration required:** run `alembic upgrade head` before deploying
- Search is now 40% faster (removed an N+1 query)

### Fixed
- Cards with empty backs no longer crash the study session (#421)
- Timezone handling in the review log was off by one day (#405)

### Security
- Rate limiting on the login endpoint
- Ownership checks added to three endpoints that were missing them (#419)

**Upgrade notes:** run migrations before deploying the new code.
Rollback: safe, but keep the old code until 2026-10-09.
```

**A good release note tells someone what to do, not just what changed.** Note the migration ordering and the rollback position.

---

# 12. Secrets in CI

## 12.1 The rules

| Rule | Reason |
|---|---|
| Never print a secret, even temporarily | CI logs are retained and searchable |
| Never pass secrets as command-line arguments | Visible in `ps` |
| Mask secrets in output | `add-mask` for dynamically created ones |
| Least-privilege tokens | Scope every credential to what it needs |
| Rotate on exposure | Assume anything in a log is compromised |
| Separate environments | Staging secrets must differ from production |

## 12.2 Patterns

```yaml
jobs:
  deploy:
    environment: production            # gives you required reviewers + scoped secrets

    steps:
      # Mask any dynamically derived secret
      - name: Configure registry
        run: |
          echo "::add-mask::$(cat /tmp/token)"
          echo "$REGISTRY_TOKEN" | docker login -u user --password-stdin registry.example.com
```

```python
# Mask values that originate outside the secrets store
import os

def mask(value: str) -> None:
    if os.environ.get("GITHUB_ACTIONS"):
        print(f"::add-mask::{value}")
```

## 12.3 Fork PRs and secrets

Workflows triggered by forks from outside contributors **do not receive secrets** — GitHub blocks this deliberately. That is a feature.

**The danger:** `pull_request_target` runs with full secrets and repository write access, even for fork PRs. If you combine it with checking out the PR's code, a contributor can execute arbitrary code with your secrets.

**Rule:** never use `pull_request_target` with a checkout of untrusted code.

---

# 13. Performance and cost

## 13.1 Common cost leaks

| Leak | Fix |
|---|---|
| A `1GB` database when `256MB` suffices | Right-size after measuring |
| No connection pooling — a new connection per request | Use a pool (SQLAlchemy's `pool_size`) |
| Cache miss on every identical request | Cache read-heavy data |
| `SELECT *` on large tables | Select only needed columns |
| Missing index → full table scan | Index the columns you filter and sort on |
| Log every debug message in production | Set the level to `WARNING` |
| `prettier=20` workers for a 1-CPU app | Match workers to cores |

## 13.2 Caching

```python
from functools import lru_cache


@lru_cache(maxsize=1000)
def expensive_computation(key: str) -> dict:
    """Only safe for pure functions of the arguments."""
    ...


def get_deck(deck_id: int, session: Session = Depends(get_db)) -> Deck:
    """
    HTTP caching. ETag lets the client send If-None-Match and get a 304
    with no body — often the cheapest optimisation available.
    """
    deck = session.get(Deck, deck_id)
    etag = compute_etag(deck)
    response = JSONResponse(content=serialise(deck))
    response.headers["ETag"] = etag
    return response
```

```python
# Redis cache with graceful degradation — a cache failure must not break the request
def get_deck_cached(deck_id: int, user_id: int) -> Deck | None:
    key = f"deck:{deck_id}"

    try:
        cached = redis.get(key)
        if cached:
            return Deck.model_validate_json(cached)
    except redis.RedisError:
        logger.warning("cache_unavailable", key=key)     # degrade, do not fail
        return repo.get_for_user(deck_id, user_id)

    deck = repo.get_for_user(deck_id, user_id)
    if deck:
        try:
            redis.setex(key, 300, deck.model_dump_json())   # 5-minute TTL
        except redis.RedisError:
            pass

    return deck
```

## 13.3 Connection pooling

```python
engine = create_engine(
    DATABASE_URL,
    pool_size=10,            # keep 10 connections open
    max_overflow=20,         # allow 20 more under burst
    pool_timeout=30,         # wait 30s for a connection, then error
    pool_recycle=1800,       # recycle after 30 min — avoids stale connections
    pool_pre_ping=True,      # test the connection before using it
    echo=False,
)
```

---

# 14. Learning resources

| Topic | Resource |
|---|---|
| **GitHub Actions** | Official docs; the starter-workflows gallery |
| **Docker** | *Docker Deep Dive* (free), Docker's own tutorials |
| **Deployment platforms** | Render, Fly.io, Railway — each has a good "deploy your first app" guide |
| **Observability** | *Observability Engineering* (Google); OpenTelemetry docs |
| **IaC** | Terraform's "Learn Terraform" (free, official) |
| **Reliability** | *Site Reliability Engineering* (Google, free) |
| **DORA** | dora.dev — the research behind the four metrics |
| **Incident response** | *The Site Reliability Workbook* (Google, free) |

## The habit that matters

> **Deploy early and often, even when it is not ready.**

The first deployment teaches you more in one hour than a week of reading. The certificate is expired, the DNS is wrong, the environment variable is missing, the migration did not run. Every one of those is a lesson you would rather have now than in front of a user.

Your first deploy will fail. That is the point of doing it while nobody depends on it.

---

## Closing

DevOps is not a job title. It is the practice of making deployment routine, failure visible, and recovery routine. Every pipeline you build makes the next one faster, and every incident you instrument properly becomes the last of its kind.

Automate the boring parts. Measure what you care about. Deploy when it is ready, not when it is perfect.
